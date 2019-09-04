---
layout: post
title: "springboot+dubbo+zk集群搭建"
date: 2019-02-09
tags: java
---

zookeeper的集群搭建在上一编已经说过，不会的可以查看。

下面开始搭建springboot+dubbo+zk注册中心的demo

**生产者**工程目录如图

![image](https://upload-images.jianshu.io/upload_images/14890912-21a28599d4ac740d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**一、创建dubbo-provider父工程**
一、创建dubbo-provider父工程

父pom.xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>
    <modules>
        <module>provider</module>
        <module>api</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>springboot</groupId>
    <artifactId>dubbo-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>dubbo-provider</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>



二、创建provider工程(Module)

pom.xml文件

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo-provider</artifactId>
        <groupId>springboot</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <packaging>jar</packaging>
    <artifactId>provider</artifactId>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.alibaba.boot/dubbo-spring-boot-autoconfigure -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.1.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
        <!-- exclusions去掉log4j的依赖，会冲突 -->
        <dependency>
            <artifactId>zookeeper</artifactId>
            <groupId>org.apache.zookeeper</groupId>
            <version>3.4.14</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>io.netty</groupId>
                    <artifactId>netty</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>springboot</groupId>
            <artifactId>api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>

**application.yml配置文件信息**
\#dubbo的name默认使用${spring.application.name}，所以不用配置dubbo.application.name也行
spring:
&ensp;application:
&ensp;&ensp;name: dubbo-provider
\#使用注解，要配置dubbo的扫描包路径
dubbo:
&ensp;scan:
&ensp;&ensp;base-packages: com.facade
\#dubbo启动的端口
&ensp;protocol:
&ensp;&ensp;port: 20880
&ensp;registry:
&ensp;&ensp;protocol: zookeeper
&ensp;&ensp;address: 192.168.79.135:2181,192.168.79.136:2181,192.168.79.137:2181

定义一个服务接口

![image](https://upload-images.jianshu.io/upload_images/14890912-795e90cb6b89d7a0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

服务接口的实现

![image](https://upload-images.jianshu.io/upload_images/14890912-391cc607f73d1a53?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

定义服务接口暴露出去的dubbo实现(因为实际项目中会包装一层facade，而不会将service当作dubbo接口)

![image](https://upload-images.jianshu.io/upload_images/14890912-15c108f2504decb4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**二、创建provider的api工程(Module)**

**api工程用于给消费者工程引用依赖得到api接口来调用**

**pom.xml文件，什么都不用依赖**

![image](https://upload-images.jianshu.io/upload_images/14890912-594a64f371128ec6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

暴露给其他服务的api接口

![image](https://upload-images.jianshu.io/upload_images/14890912-c775e518d7e68c9d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

****最后就是springboot启动服务的类****

![image](https://upload-images.jianshu.io/upload_images/14890912-cdfba6b4314a2cc5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**生产者工程搭建成功**

****消费者**工程目录如图**

![image](https://upload-images.jianshu.io/upload_images/14890912-d9a64e4c948d8ed8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**一、创建dubbo-consumer父工程**

**pom.xml文件**
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>
    <modules>
        <module>consumer</module>
    </modules>
    <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.1.4.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
   </parent>
   <groupId>springboot</groupId>
   <artifactId>dubbo-consumer</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <name>dubbo-consumer</name>
   <description>Demo project for Spring Boot</description>
   <properties>
      <java.version>1.8</java.version>
   </properties>
   <dependencies>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
      </dependency>
   </dependencies>
</project>


**application.yml配置文件**

\#dubbo的name默认使用${spring.application.name}，所以不用配置dubbo.application.name也行
spring:
&ensp;application:
&ensp;&ensp;name: dubbo-consumer
\#dubbo启动的端口
dubbo:
&ensp;protocol:
&ensp;&ensp;port: 20881
&ensp;registry:
&ensp;&ensp;protocol: zookeeper
&ensp;&ensp;address: 192.168.79.135:2181,192.168.79.136:2181,192.168.79.137:2181
这里用springboot的test来启动消费者来

![image](https://upload-images.jianshu.io/upload_images/14890912-d63c97be69100898?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**消费者工程搭建成功**

**现在启动生产者服务(直接启动springboot的main就可以了)**

![image](https://upload-images.jianshu.io/upload_images/14890912-d38244ee0f608c56?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

生产者启动成功

启动消费者(调用ConsumerTest测试)

![image](https://upload-images.jianshu.io/upload_images/14890912-db6a5c6b57672ac1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**调用成功，说明搭建的springboot+dubbo+zk集群成功了**

查询zk节点的变化，看看暴露的接口存到注册中心的是什么东西

发现多了dubbo节点，并者dubbo节点的子节点下有注册到zk上的服务(api接口)名称 有多少个服务，dubbo下就有多少个子节点

/dubbo/com.api.HelloFacade

![image](https://upload-images.jianshu.io/upload_images/14890912-341acf41942ffa58?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](https://upload-images.jianshu.io/upload_images/14890912-432d90d87235365e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接口节点上还有providers，configuartors等节点

![image](https://upload-images.jianshu.io/upload_images/14890912-9b8574eb28f0f41a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看providers节点下发现这一串东西(这是生产者注册到zk上的地址)

[dubbo%3A%2F%2F172.16.19.186%3A20880%2Fcom.api.HelloFacade%3Fanyhost%3Dtrue%26application%3Ddubboprovider%26bean.name%3DServiceBean%3Acom.api.HelloFacade%3A1.0.0%26dubbo%3D2.0.2%26generic%3Dfalse%26interface%3Dcom.api.HelloFacade%26methods%3DsayH%26pid%3D11836%26revision%3D1.0.0%26side%3Dprovider%26timestamp%3D1557481294861%26version%3D1.0.0]

百度一下url解码

[dubbo://172.16.19.186:20880/com.api.HelloFacade?anyhost=true&application=dubboprovider&bean.name=ServiceBean:com.api.HelloFacade:1.0.0&dubbo=2.0.2&generic=false&interface=com.api.HelloFacade&methods=sayH&pid=11836&revision=1.0.0&side=provider&timestamp=1557481294861&version=1.0.0]

**dubbo使用zookeeper作为注解中心，服务启动后，zk节点会创建/dubbo节点，然后把在dubbo节点上创建生产者注册到zk的服务(api接口)名称/dubbo/com.api.HelloFacade，该节点下会保存provider和consumer的节点，/dubbo/com.api.HelloFacade/provider节点(持久节点)下会保存生产者的地址信息(协议://ip+port/接口名称...)，provider节点下保存的 地址信息(临时节点)，生产者关闭服务地址节点就会被删除。**

看出这是dubbo协议的地址，我们的配置文件没配置dubbo的协议，说明dubbo默认使用的就是dubbo协议。

消费者服务启动时就会HelloFacade的代理对象

消费者调用dubbo接口的方法时会到zk下找到对应接口的生产者地址(172.16.19.186:20880)，消费者与生产者通过netty进行通信，生产者通过动态代理和反射机制把方法执行后的返回值通过netty通讯传给消费者

我们可以看看官网上的图

  0.服务容器负责启动，加载，运行服务提供者。

1.  服务提供者在启动时，向注册中心注册自己提供的服务。

2.  服务消费者在启动时，向注册中心订阅自己所需的服务。

3.  注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

4.  服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

5.  服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心

    --------------------------------------------

    第4步invoke是通过netty通信进行，消费者会生成动态代理类，调用服务的方法时会请求生产者，生产者会通过invoke调用方法，然后把返回值传给消费者 (相当于消费者proxy.newproxyinstance(...)时，代理对象会触发invocationHandler，invocationHandler接口的实现中处理与生产者进行传递消息，生产者把方法调用后的结果传递给消费者)

![image](https://upload-images.jianshu.io/upload_images/14890912-80cb84b4887ed1db?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下一编会对dubbo的生产者和消费者的一些参数配置和服务治理的参数进行测试说明。