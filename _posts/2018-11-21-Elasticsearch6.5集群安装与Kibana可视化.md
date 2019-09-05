---
layout: post
title: "Linux(centos7) Elasticsearch6.5集群安装与Kibana可视化"
date: 2018-11-21
tags: java
---

**必须要先安全JDK1.8或以上版本，内存配置1G以上，最好2G**

https://www.elastic.co/cn/downloads/elasticsearch#ga-release  官网下载tar

上传到linux 解压

tar -zxvf elasticsearch-6.5.0.tar.gz

进行bin目录

cd elasticsearch-6.5.0/bin/

启动ES

./elasticsearch

![img](https://upload-images.jianshu.io/upload_images/14890912-3999f788ec7d80bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新建用户 myes

adduser myes

给用户操作elasticsearch文件夹的权限（若要修改es数据和日志的存储路径，也需要对对应文件夹授权）

 chown -R myes /data/elasticsearch-6.5.0

切换到myes用户

su myes

启动ES

./elasticsearch

但是无法访问，进入到配置文件

cd elasticsearch-6.5.0/config/



![img](https://upload-images.jianshu.io/upload_images/14890912-144b2a6ee48940ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重启ES  (查询进程ps -ef|grep elasticsearch   kill -9 21667)

又报错了



![img](https://upload-images.jianshu.io/upload_images/14890912-6821c2af27e7930c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[1]解决方案

切换到root用户，编辑limits.conf添加类似如下内容

vi /etc/security/limits.conf

添加如下内容:

\* soft nofile 65536

\* hard nofile 65536

\* soft nproc 4096

\* hard nproc 4096

保存后重启**系统**让修改生效

[2]解决方案

[2]错误是因为操作系统的vm.max_map_count参数设置太小导致的：

切换到root用户修改配置sysctl.conf

vi /etc/sysctl.conf 

添加下面配置：

vm.max_map_count=262144

并执行命令，让修改生效：

sysctl -p

执行sysctl -a | grep "vm.max_map_count"命令查看修改是否生效

切换到es用户，重启es，启动成功 -d表示后台启动

./elasticsearch -d

//临时关闭

systemctl stop firewalld



![img](https://upload-images.jianshu.io/upload_images/14890912-11c86e255fe68adf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



//禁止开机启动

systemctl disable firewalld



Kibana安装

https://www.elastic.co/downloads/kibana  下载并上传到linux

tar -zxvf kibana-6.5.1-linux-x86_64.tar.gz

修改kibana配置文件   **elasticsearch.url必须是主节点的url**

vi kibana-6.5.1-linux-x86_64/config/kibana.yml

![img](https://upload-images.jianshu.io/upload_images/14890912-e6f92e6c61b1d310.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 启动kibana服务

./kibana-6.5.1-linux-x86_64/bin/kibana

启动成功

![img](https://upload-images.jianshu.io/upload_images/14890912-edc0bc18bdea9638.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

后台启动

nohup ./kibana-6.5.1-linux-x86_64/bin/kibana &

访问地址 192.168.79.130:5601



![img](https://upload-images.jianshu.io/upload_images/14890912-f8f5fb5c1a0e0628.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

修改ES的配置文件

\#添加下面配置

\#network.host 绑定ip

network.host: 127.0.0.1

.让kibana插件能够连接访问（配置允许跨域访问）

http.cors.enabled: true

http.cors.allow-origin: "*"



![img](https://upload-images.jianshu.io/upload_images/14890912-a92f59e967ed2e2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动ES

成功

![img](https://upload-images.jianshu.io/upload_images/14890912-e10aa4b9a4b66b92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



Elasticsearch6.5集群配置  1主2从

分布式集群配置（设置主从关系）

**master主节点配置**

**#集群名称cluster.name: wali**

**#指定该服务为指挥官角色**

**node.name: master**

**node.master: true**

**#绑定的ip    network.host: 127.0.0.1**



![img](https://upload-images.jianshu.io/upload_images/14890912-bf53b0f1947720d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置从机配置：2个从

同主机安装一样，在新目录安装elasticsearch，并将目录名设置为es_slave1 ...

cp -r elasticsearch-6.5.0 es_slave1

cp -r elasticsearch-6.5.0 es_slave2

修改yml文件配置

\#**集群名称，需要和主机的保持一致**

**es_slave1配置**

cluster.name: wali

\#设置节点名称

node.name: slave1

\#设置ip

network.host: 127.0.0.1

\#指定服务启动端口  因为默认端口号(9200)被主节点占用了

http.port: 8200

\#找到集群中的master

discovery.zen.ping.unicast.hosts: ["127.0.0.1"]



![img](https://upload-images.jianshu.io/upload_images/14890912-939fea9609fdafd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**es_slave2配置**

cluster.name: wali

\#设置节点名称

node.name: slave2

\#设置ip

network.host: 127.0.0.1

\#指定服务启动端口  因为默认端口号(9200)被主节点占用了

http.port: 8300

\#找到集群中的master

discovery.zen.ping.unicast.hosts: ["127.0.0.1"]



![img](https://upload-images.jianshu.io/upload_images/14890912-c092ffc5fc55f91c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**给myes用户配置权限**

**chown -R myes /data/es_slave1**

**chown -R myes /data/es_slave2**

**直接启动 slave会报错**

![img](https://upload-images.jianshu.io/upload_images/14890912-93d42bffe9e3cf24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**是因为复制的elasticsearch文件夹下包含了data文件中主节点数据，需要把从节点data文件下的文件清空。**

启动master和slave1



![img](https://upload-images.jianshu.io/upload_images/14890912-b1a19d47de8fffcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动slave2时报错，内存不足failed; error='Cannot allocate memory' (errno=12)

![img](https://upload-images.jianshu.io/upload_images/14890912-20edc81e9f957b23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进行slave2的配置文件config 把内存修改小一点

vi jvm.options



![img](https://upload-images.jianshu.io/upload_images/14890912-c458877113657ac0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动 3台ES 大功告成



![img](https://upload-images.jianshu.io/upload_images/14890912-6ade0c600ed5a111.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)