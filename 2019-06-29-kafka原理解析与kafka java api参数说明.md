---
layout: post
title: "kafka原理解析与kafka java api参数说明"
date: 2019-06-29
tags: java
---

kafka由producer consumer broker topic partitions(分区)组成

kafka cluster(集群)就是由多个broker topic partitions(分区)组成

开局一张图

![image](https://gitee.com/yezhaoxin/static/raw/master/images/1240.jpg)

![image.gif](https://upload-images.jianshu.io/upload_images/14890912-968dfeef183e2a73.gif?imageMogr2/auto-orient/strip) 

上面表示一个kafka集群交互的流程图，kafka cluster中有3个brock(表示有3台kafka服务的集群)，有一个topic名称为topic0的主题，topic0主题有3个partitions(分区)，每个partition有3个副本(包含本身，partition本身也是副本，是leader副本。一个leader分区，两个follwer分区)

Producers往Brokers里面的指定topic0中写消息(发消息时指定主题的名称，默认会创建主题)，Consumers从Brokers里面拉去指定topic0的消息。

topic只是大的概论，真正存放消息的是partitions(分区)

从图可以看出 topic0创建了3个分区 ([1 p2 p0)，每个brock分配一个分区，上面的图包含了两个副本，如果只创建分区本身是图如下

![image](https://gitee.com/yezhaoxin/static/raw/master/images/kafka2.jpg)

![image.gif](https://upload-images.jianshu.io/upload_images/14890912-495f09f98442bc34.gif?imageMogr2/auto-orient/strip) 

分区上的存储的![image](https://upload-images.jianshu.io/upload_images/14890912-a4f994eaacbbe632.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.gif](https://upload-images.jianshu.io/upload_images/14890912-29586ac7da643f12.gif?imageMogr2/auto-orient/strip) 就是每一条的消息(相当于rocketmq的队列)，消息是有顺序的，从0开始(offset)，消费者pull消息的时候，就是从分区上获取里面存放的消息，topic的每个分区存放的消息都不一样

因为topic的每个分区存放的消息都不一样。所有当brock0挂掉后，p1上未被消费的消费就消费不到了。只有当重启之后才能消息。这样就没有起到消息集群容错的效果。所以有了分区副本的概念

![image](https://gitee.com/yezhaoxin/static/raw/master/images/kafka3.jpg)

![image.gif](https://upload-images.jianshu.io/upload_images/14890912-750ed70609cd608a.gif?imageMogr2/auto-orient/strip) 

上图表示topic0主题有3个分区(p1 p2 p0)，每个分区有两个副本(replica)(也可以说topic0有3个不同的分区，每个分区有3个副本(包括自己本身，本身是leader分区副本，其他两个是follwer分区副本))

分区分为leader分区和follwer分区，我们可以在zookeeper上查看topic0的分区的leader分布在哪个brock上

在zk的客户端执行命令get /brokers/topics/topic0  (这个命令可以查到topic0主题下partitions分布在哪个brock)

![image](https://gitee.com/yezhaoxin/static/raw/master/images/kafka4.jpg)

![image.gif](https://upload-images.jianshu.io/upload_images/14890912-ed1657872940a4b3.gif?imageMogr2/auto-orient/strip) 

可以看到"partitions":{"2":[1,2,0],"1":[0,1,2],"0":[2,0,1]}}，表示p2分区分布在brock.id为1和2和0的服务器上(因为有3个分布，每个brock上都会有，所有1 2 0 brock都有p2分区)

每个brock上都有p2分区，但是leader分区只有一个，其他的都是follwer分区。

在zk的客户端执行命令 get /brokers/topics/topic0/partitions/0/state  (可以查询主题下的partitions的leader partition(分区)在哪个brock上)

![image](https://gitee.com/yezhaoxin/static/raw/master/images/kafka5.jpg)

![image.gif](https://upload-images.jianshu.io/upload_images/14890912-05dc8764d0888be9.gif?imageMogr2/auto-orient/strip) 

"leader":2  可以看出 p0分区在brock.id=2的服务器上。

其他的分区leader分布按上面的命令查询到(没有截图出来)。p1分区在brock.id=0的服务器上、p2分区在brock.id=1的服务器上

leader分区在一图中都标红色了，其他都是follwer。

分区副本的作用是当leader分区的brock挂了，会在fowller分区上重新选出一个分区作为leader分区，能实现集群容错效果

leader副本：处理所有的读写请求(只有一个leader，其他的都是follwer)

follwer副本：不接收任何请求处理，只从leader副本同步消息日志

# 副本分配算法

副本是如何分配到brock上的？

将所有N个Brock和待分配的i个partition排序，将第i个partition分配到第(i%n)个Brock上，将第i个partition的第j个副本分配到第((i+j)%n)个brock上

# kafka高性能原因

### 1.消息顺序写入到磁盘

将写磁盘的过程变为顺序写，可极大提高对磁盘的利用率。Consumer通过offset顺序消费这些数据

### 2.零拷贝(直接在内核空间将数据拷贝到网卡缓存。减少了用户空间的操作过程)

### (类似于NIO的直接缓冲区，减少jvm内存的操作过程)

消息从发送到落地保存，broker 维护的消息日志本身就是文件目录，每个文件都是二进制保存，生产者和消费者使 用相同的格式来处理。在消费者获取消息时，服务器先从 硬盘读取数据到内存，然后把内存中的数据原封不动的通 过 socket 发送给消费者。虽然这个操作描述起来很简单， 但实际上经历了很多步骤。

![image](https://gitee.com/yezhaoxin/static/raw/master/images/kafka6.jpg)

![image.gif](https://upload-images.jianshu.io/upload_images/14890912-acbf6128c9e58e2a.gif?imageMogr2/auto-orient/strip) 

▪ 操作系统将数据从磁盘读入到内核空间的页缓存(Linux内核一种重要的磁盘高速缓存)

▪ 应用程序将数据从内核空间读入到用户空间缓存中

▪ 应用程序将数据写回到内核空间到 socket 缓存中

▪ 操作系统将数据从 socket 缓冲区复制到网卡缓冲区，以便将数据经网络发出

这个过程涉及到 4 次上下文切换以及 4 次数据复制，并且有两次复制操作是由 CPU 完成。但是这个过程中，第二、三步操作数据完全没有进行变化，仅仅是从磁盘复制到网卡缓冲区。

### 通过“零拷贝”技术，可以去掉这些没必要的数据复制操作， 同时也会减少上下文切换次数。现代的 unix 操作系统提供 一个优化的代码路径，用于将数据从页缓存传输到 socket； 在 Linux 中，是通过 sendfile 系统调用来完成的。Java 提 供了访问这个系统调用的方法：FileChannel.transferTo API

![image](https://gitee.com/yezhaoxin/static/raw/master/images/kafka7.jpg)

![image.gif](https://upload-images.jianshu.io/upload_images/14890912-aecc6fe422f56795.gif?imageMogr2/auto-orient/strip) 

### 使用 sendfile，只需要一次拷贝就行，允许操作系统将数据 直接从页缓存发送到网络上。所以在这个优化的路径中， 只有最后一步将数据拷贝到网卡缓存中是需要的。

--------------------------------------------------------

# java api kafka配置信息分析

producer 配置可选参数

# acks

配置表示 producer 发送消息到 broker 上以后的确认值。有三个可选项

### 1\. acks=0表示 producer 不需要等待 broker 的消息确认，发出消息那么就认为消息已成功写入Kafka，时效率高，但同时风险最大，server 宕机时，数据将会丢失

### 2\. acks=1 表示 producer 只需要获得 kafka 集群中的 leader 节点确认即可，这个选择时延较小同时确保了 leader 节点确认接收成功

### 3\. acks=all leader 节点在返回确认或错误响应之前，会等待所有同步副本都收到消息。如果和min.insync.replicas参数结合起来，就可以决定在返回确认前至少有多个副本能够收到消息。比如min.insync.replicas=1就需要至少一个follwer确认收到消息。相对安全，但是效率较低。但是由于 ISR 可能会缩小到仅包含一个 Replica，所以设置参数为all并不能一定避免数据丢失

# batch.size

生产者发送多个消息到 broker 上的同一个分区时，为了减少网络请求带来的 性能开销，通过批量的方式来提交消息，可以通过这个参数来控制批量提交的 字节数大小，默认大小是 16384byte,也就是 16kb，意味着当一批消息大小达 到指定的 batch.size 的时候会统一发送

# linger.ms

Producer 默认会把两次发送时间间隔内收集到的所有 Requests 进行一次聚合 然后再发送，以此提高吞吐量，而 linger.ms 就是为每次发送到 broker 的请求 增加一些 delay，以此来聚合更多的 Message 请求。 这个有点想 TCP 里面的 Nagle 算法，在 TCP 协议的传输中，为了减少大量小数据包的发送，采用了 Nagle 算法，也就是基于小包的等-停协议。

### batch.size 和 linger.ms 这两个参数是 kafka 性能优化的关键参数当二者都配置的时候，只要满足其中一个要 求，就会发送请求到 broker 上

# max.request.size

设置请求的数据的最大字节数，为了防止发生较大的数据包影响到吞吐量，默认值为 1MB

consumer配置可选参数

# group.id

当producer发送一条消息，相同group.id的多个consumer只有其中一个consumer能消费到

(比如 topic=hehe的主题发送了一条消息，group.id=666的组，有三个消费者监听了这个主题，但这条消息只会被其中一个consumer消费到)，group.id=999的一个consumer也能消费这条消息。

# enable.auto.commit

消费者消费消息以后自动提交，只有当消息提交以后，该消息才不会被再次接 收到，还可以配合 auto.commit.interval.ms 控制自动提交的频率。 当然，我们也可以通过 consumer.commitSync()的方式实现手动提交

# auto.offset.reset

这个参数是针对新的 groupid 中的消费者而言的，当有新 groupid 的消费者来 消费指定的 topic 时，对于该参数的配置，会有不同的语义

auto.offset.reset=latest 情况下，新的消费者将会从其他消费者最后消费的 offset 处开始消费 Topic 下的消息

auto.offset.reset= earliest 情况下，新的消费者会从该 topic 最早的消息开始 消费

auto.offset.reset=none 情况下，新的消费者加入以后，由于之前不存在 offset，则会直接抛出异常。

# max.poll.records

此设置限制每次调用 poll 返回的消息数，这样可以更容易的预测每次 poll 间隔 要处理的最大值。通过调整此值，可以减少 poll 间隔