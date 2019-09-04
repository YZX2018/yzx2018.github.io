---
layout: post
title: "dubbo初识与dubbo无注册中心本地测试"
date: 2018-02-14
tags: java
---



传统项目是把mvc都写在一个项目中，每个项目部署一个tomcat，项目与项目之间使用httpclient或者webservice来调用

传统项目架构图  项目中包含视图(freemarker) 控制层 业务层 数据访问层 静态资源等

![image](http://upload-images.jianshu.io/upload_images/14890912-12d0ab9882eec7f0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

随着项目的发展，传统的项目迭代困难，已经跟不上互联网的发展。项目架构变成了分布式微服务的架构形式。

前后端分离，控制层与服务层分离、以服务来划分各个模块的功能，每个服务都有独立的数据库

![image](http://upload-images.jianshu.io/upload_images/14890912-34c507ad0136ddc1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

随着客户越来越多，访问量越来越大，每个服务集群的地址也越来越多，传统的调用(httpclient和webservice)已经管理不了那么多的地址，客户不断增加，项目不断迭代，出现了越来越多问题。

？地址的维护、服务端的负载均衡、限流、容错、降级 、监控等服务治理的问题？

分布式微服务架构最大的问题就是服务治理

什么是服务治理？

  服务和服务之间有很多个url(注册中心(zk节点)管理，通过类调用)、依赖关系、负载均衡、容错、自动注册服务等

  服务治理主要作用是改变运行时服务的行为和选址逻辑，达到限流，权重配置等目的。

基于服务治理的问题就出现了dubbo框架

  Dubbo是一个RPC远程调用框架、分布式服务治理框架 

  RPC:远程调用，服务与服务之间相互进地通讯。

Dubbo四大模块：

  生产者Provider(暴露服务的服务提供方)

  消费者Consumer(调用远程服务的服务消费方)

  注册中心Registry(服务注册与发现的注册中心）

  监控中心Monitor(统计服务的调用次数和调用时间的监控中心)

---------------------------------

  Container(服务运行容器)，dubbo内置容器，发布提供服务时都会有运行的容器，类似与war项目发布到tomcat容器上

dubbo官方的图

![image](http://upload-images.jianshu.io/upload_images/14890912-c1def0f7588b2f2e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先生产者将服务发布到注册中心（zk），使用zk持久节点进行存储，消费订阅zk节点，一但有节点变更，zk通过事件通知传递给消费者，消费者可以调用生产者服务。

服务与服务之间进行调用，都会在监控中心，存储一个记录(日志)

dubbo能做什么？

  1.透明化的远程方法调用，就像调用本地方法一样调用远程方法，只需简单配置，没有任何API侵入。

  2.软负载均衡及容错机制，可在内网替代F5等硬件负鞭均衡器，降低成本，减少单点。

  3.服务自动注册与发现，不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的IP地址，并且能够平滑添加或删除服务提供者。

  4.Dubbo采用全Spring 配置方式，功明化接入应用，对应用没有任何API侵

只需用Spring加载Dubbo的配置即可，Dubbo基于Spring的Schema扩展进行加载。 

dubbo无注册中心本地测试演示  (消费者直接声明url)



生产者代码，创建api模块和provider模块(实现类)



service-api模块代码

HelloService类
package service;
public interface HelloService {
    String hello(String msg);
}

provider模块代码

HelloServiceImpl 类
package springbootdubboservice.dubboprovider.service;
import service.HelloService;

// 测试了可以不写@Service也能调用
public class HelloServiceImpl implements HelloService {
    @Override
    public String hello(String msg) {
        return "hello dubbo" + msg;
    }
}

Provider类

package yxh;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class Provider {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("dubbo-server.xml");
        context.start();
        System.in.read(); // 按任意键退出
    }
}

dubbo-server.xml配置文件

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="yxh-dubbo-server" owner="yxh"/>
    <!-- 不使用registry -->
    <dubbo:registry address="N/A"/>
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880"/>
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="dubbo.yxh.api.HelloService"
                   class="yxh.HelloServiceImpl"/>
</beans>

pox.xml配置文件

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo-server</artifactId>
        <groupId>good</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbo-provider</artifactId>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.6</version>
        </dependency>
    
        <!-- 高版本dubbo需要引入这个依赖 -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.32.Final</version>
        </dependency>
    
        <dependency>
            <groupId>good</groupId>
            <artifactId>service-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>



消费者代码，创建dubbo-client模块






Client类

package yxh;
import dubbo.yxh.api.HelloService;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class Client {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("dubbo-client.xml");
        HelloService helloService = (HelloService) context.getBean("helloService");
        System.out.println(helloService.hello("666"));
    }
}

dubbo-client.xml配置文件   消息接口加上url

<!-- 消费方应用信息 -->
<dubbo:application name="yxh-dubbo-client" owner="yxh"/>
<!-- 不使用registry -->
<dubbo:registry address="N/A"/>
<!-- 用dubbo协议在20880端口暴露服务 -->
<dubbo:protocol name="dubbo" port="20880"/>
<!-- 声明需要消费的服务接口 -->
<dubbo:reference id="helloService" interface="dubbo.yxh.api.HelloService"
url="dubbo://192.168.149.1:20880/dubbo.yxh.api.HelloService"/>


pom.xml配置文件

<dependencies>
    <!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.6.6</version>
    </dependency>
    <!-- 高版本dubbo需要引入这个依赖 -->
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.32.Final</version>
    </dependency>
    <dependency>
        <groupId>good</groupId>
        <artifactId>service-api</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>



启动生产者

右键Debug  Provider类的main方法即可



日志可看到dubbo的url地址为

dubbo://192.168.149.1:20880/dubbo.yxh.api.HelloService

和消费者配置的url一样



启动消费者

Client类 启动main 



打印出生产者的实例方法的结果

测试成功