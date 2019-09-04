---
layout: post
title: "分布式任务调度之XXL-JOB"
date: 2019-01-16
tags: java
---

xxl-job的github地址
https://github.com/xuxueli/xxl-job
xxl-job的文档地址
http://www.xuxueli.com/xxl-job/#/

传统的job，在tomcat服务集群时，在每台jvm都会配置相同的job配置，并且配置都是写死的，任务时间到时，每台服务都会执行任务，自身无法保证幂等性，如果想保证幂等性需要用到分布式锁或者通过标识来判断这个服务的job是否执行(true和false标识)，并且无法实现负载策略

XXL-JOB是一个轻量级分布式任务调度平台
支持job群集容错,负载均衡策略，job补偿(实现自动重试，多次失败会发邮件通知)，job日志记录，动态配置job规则

XXL-JOB搭建
https://github.com/xuxueli/xxl-job上下载xxl-job的源码项目
![image.png](https://upload-images.jianshu.io/upload_images/14890912-a6c6300c3f52715d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下载后解压，使用idea打开
![image.png](https://upload-images.jianshu.io/upload_images/14890912-18c2b137139f6647.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
xxl-job-admin模块是xxl-job服务端的可视化管理平台
xxl-job-core模块是xxl-job的源码
xxl-job-executor-samples模块是job客户端的例子代码(我们自己写的业务job)

步骤1：首先执行xxl-job项目的sql创建表和初始化默认的记录(默认使用mysql)，因为xxl-job的数据是保存在数据库的(创建的执行器信息、job任务的服务地址、admin登陆的账号密码等信息)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-519acd27730ef2fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
步骤二：修改xxl-job-admin的application.properties的数据库账号和密码，修成自己数据库的账号密码，其他配置用不上的可以使用默认的
![image.png](https://upload-images.jianshu.io/upload_images/14890912-6552769120fa1b6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

步骤三：目前最新版本admin模块也是使用springboot启动的，所以直接运行启动类就行了
![image.png](https://upload-images.jianshu.io/upload_images/14890912-d06f407025157b8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
启动后打开进入admin可视化管理界面
[http://127.0.0.1:8080/xxl-job-admin](http://127.0.0.1:8080/xxl-job-admin)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-0e0411b3aad6b569.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上的xxl-job的admin平台服务已经搭建好了，下面我们先创建自己的定时任务，我使用xxl-job源码中的xxl-job-executor-samples项目作为例子
![image.png](https://upload-images.jianshu.io/upload_images/14890912-6c8c0143ef8727ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用xxl-job-executor-samples中的xxl-job-executor-sample-springboot工程(相当于我们进行开发的项目代码)
需要注意yml配置文件的xxl.job配置
![image.png](https://upload-images.jianshu.io/upload_images/14890912-26950da717b2715d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面配置的端口，等一下admin平台创建执行器时需要用到。
我们自己创建一个job任务
![image.png](https://upload-images.jianshu.io/upload_images/14890912-b709e157e2c91ce3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
创建的job任务类需要加上@JobHandler(value = "myJob")注册并承继IJobHandler类
``@JobHandler(value = "myJob")``这个注解的作用是等下通过admin平台创建任务时需要指定到哪个任务代码的名称
这个项目的例子也是springboot的，直接启动类启动就行了。

admin平台和我们写的job代码的服务都启动好之后，我们通过admin平台关联需要执行任务的服务器地址和任务
1.在执行器管理创建执行器(配置job任务的服务地址)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-fcc21707110c76c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
机器地址的端口就是这里配置的端口
![image.png](https://upload-images.jianshu.io/upload_images/14890912-99b75f7c35d8f182.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个执行器的所创建的任务都会从配置机器地址去找

2.在任务管理为刚刚创建的执行器创建任务(给对应的服务器关联对应的执行任务)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-97d9a8a952a56ce0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这的JobHandler就是与我们的定时任务代码的@JobHandler(value = "myJob")对应
![image.png](https://upload-images.jianshu.io/upload_images/14890912-9366aec677335036.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

保存好之后点击操作的启动就会启动任务，等Cron到时间就会通过执行器机器地址并通任务的JobHandler的名称反射找到找到对应服务的对应定时任务类的execute方法来执行
![](https://upload-images.jianshu.io/upload_images/14890912-e3890d3b8472690d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-c318e006c580b3b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
就会有执行效果

如果admin需要群集，官方推荐的方法是：
调度中心支持集群部署，提升调度系统容灾和可用性。
调度中心集群部署时，几点要求和建议：
DB配置保持一致；
登陆账号配置保持一致；
集群机器时钟保持一致（单机集群忽视）；
建议：推荐通过nginx为调度中心集群做负载均衡，分配域名。调度中心访问、执行器回调配置、调用API服务等操作均通过该域名进行。