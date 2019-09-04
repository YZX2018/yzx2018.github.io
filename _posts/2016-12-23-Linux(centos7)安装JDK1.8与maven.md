---
layout: post
title: "Linux(centos7)安装JDK1.8与maven"
date: 2016-12-23
tags: java
---

**安装JDK1.8**

https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html  官网下载jdk1.8上传到linux服务器

压解 压解后有jdk1.8.0_191目录

tar -zxvf jdk-8u191-linux-x64.tar.gz  

配置环境变量  用vim编辑器来编辑profile文件，在文件末尾添加一下内容（按“i”进入编辑）：

vim /etc/profile

在文件末尾添加一下内容（按“i”进入编辑）

export JAVA_HOME=/data/jdk1.8.0_191

export JRE_HOME=${JAVA_HOME}/jre

export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH

export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin

export PATH=$PATH:${JAVA_PATH}

通过命令source /etc/profile让profile文件立即生效

source /etc/profile

完成

java -version



------

安装maven

官网下载maven上传到linux服务器

tar -zxvf apache-maven-3.6.0-bin.tar.gz

vim /etc/profile

加上

export M2_HOME=/data/apache-maven-3.6.0

在PATH后面加上  :表示配置下一个环境变量

:${M2_HOME}/bin



![img](https://upload-images.jianshu.io/upload_images/14890912-2a077cef18b3af7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过命令source /etc/profile让profile文件立即生效

source /etc/profile

mvn -version

完成