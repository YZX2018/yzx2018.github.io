---
layout: post
title: "Linux(centos7)安装Jenkins与使用maven和git搭建Jenkins的自动部署"
date: 2017-03-13
tags: java
---

##自动化部署
**“自动化”的具体体现：向版本库提交新的代码后，应用服务器上自动部署，用户或测试人员使用的马上就是最新的应用程序**

Jenkins获取源码完成打包和部署
![image.png](https://upload-images.jianshu.io/upload_images/14890912-ef3493fbcfbec554.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


Jenkins 自身采用 Java 开发，所以要必须安装 JDK； 集成的项目基于 Maven 构架，所以 Maven 也必须安装；
首先要安装JDK1.8和maven
[安装JDK1.8和Maven教程](https://www.jianshu.com/p/ea5c72b273d7)

使用java通用的war
到官网下载Jenkins  https://jenkins.io/download/
![image.png](https://upload-images.jianshu.io/upload_images/14890912-ca250667be355cd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)或者

把war上传到linux
启动jenkins 不指定端口默认为8080   $\color{red}{也可指定jenkins的端口java -jar jenkins.war --httpPort=9090}$
java -jar jenkins.war  (前台启动方式 ctrl+c就关闭)
nohup java -jar jenkins.war --httpPort=8080 > /data/log/jenkins.log &  (**推荐**后台启动方式)


关闭防火墙
systemctl stop firewalld

http:localhost:8080 访问Jenkins
![image.png](https://upload-images.jianshu.io/upload_images/14890912-dd6ad252fbd7ae0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
cat /root/.jenkins/secrets/initialAdminPassword查看管理员密码
输入密码进入到
![image.png](https://upload-images.jianshu.io/upload_images/14890912-47622d349f7c1108.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击安装推荐的插件
![image.png](https://upload-images.jianshu.io/upload_images/14890912-bee8e4db387df68d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
等待安装（需要linux能够上网）  安装完成
![image.png](https://upload-images.jianshu.io/upload_images/14890912-d27d2cb61a15f46c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
右下角使用admin继续登陆   下一步完成之后  如果需要登陆admin  密码是上面查看的管理员密码
![image.png](https://upload-images.jianshu.io/upload_images/14890912-3edc032955327d36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Jenkins安装成功

**使用maven和git搭建Jenkins的自动部署**
需要安装git  [git安装教程](https://www.jianshu.com/p/9947b909ffbf)


在安装Jenkins中安装所需要的插件
点击系统管理->插件管理 ,安装以下插件
**1. Maven Integration**    jenkins 利用maven编译，打包，所需插件
![image.png](https://upload-images.jianshu.io/upload_images/14890912-39f09aaf2cc8fa88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**2. Deploy to Container**
![image.png](https://upload-images.jianshu.io/upload_images/14890912-f8a5aee9e74ef998.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装完插件后，重启Jenkins

系统管理->全局工具配置  ,配置路径
**maven配置**
![image.png](https://upload-images.jianshu.io/upload_images/14890912-d9067f10a3962adc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-61d6a01cc64cd458.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**JDK配置**
![image.png](https://upload-images.jianshu.io/upload_images/14890912-35a02ad4a6061970.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**git配置**
which git查询git的安装路径
![image.png](https://upload-images.jianshu.io/upload_images/14890912-dafb6be23dedb79f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**全局工具配置配置完成**

新建任务
![image.png](https://upload-images.jianshu.io/upload_images/14890912-1675d5eec74d9b80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**这里使用自己搭建的gitlab做为仓库，因为github的自动构建需要Jenkins有外网才行
配置git**
![image.png](https://upload-images.jianshu.io/upload_images/14890912-be67d9456dd570bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样就能够从gitlab上获取到源码，点击![image.png](https://upload-images.jianshu.io/upload_images/14890912-f0a2d4a49c9ee0ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)或者立即构建，就会执行mvn install打包(例子会生成war包)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-e3f4930f0adce9b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-3efe5139a6c64236.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把打包好的项目放到需要部署的的服务器上
配置**构建后操作**  构建的项目在target目录下
![image.png](https://upload-images.jianshu.io/upload_images/14890912-34585588852cf1b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-ee66aa9b15713e70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置需要把项目部署到哪个服务器的tomcat上
![image.png](https://upload-images.jianshu.io/upload_images/14890912-ba5af9226f14ac05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里Jenkins的服务器是192.168.79.131。
#**需要部署的项目的服务器放在192.168.79.132中，需要设置tomcat的账号密码，jenkins才能把部署传上去**
到192.168.79.132的tomcat配置文件中设置账号密码
vim /data/apache-tomcat-8.5.35/conf/tomcat-users.xml
加上
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username=""tomcat_user" password="123456" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
![image.png](https://upload-images.jianshu.io/upload_images/14890912-fe24bd24dd7cc58a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
tomcat8.5进行管理后台还需要配置webapps/manager/META-INF/context.xml
修改前
![image.png](https://upload-images.jianshu.io/upload_images/14890912-508b064bde9d3ff6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
指谁能访问，注释掉context或者把127修改成\d+   这是正则表达式
![image.png](https://upload-images.jianshu.io/upload_images/14890912-e962edd012c7f8c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击，登陆上面配置的账号密码 tomcat_user  123456  能登录进去证明配置成功
![image.png](https://upload-images.jianshu.io/upload_images/14890912-8c0494dcdd21d220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在配置文件server.xml加上修改 URL 地址的编码解码字符集
![image.png](https://upload-images.jianshu.io/upload_images/14890912-2086fbe499dba968.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
完成需要部署的服务器的tomcat配置

接下来继续配置Jenkins
![image.png](https://upload-images.jianshu.io/upload_images/14890912-fc5dd9b3ccd563c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-877dc97affbbf90a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
添加
![image.png](https://upload-images.jianshu.io/upload_images/14890912-deb30ab35a3a8251.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
保存
点击立即构建
![image.png](https://upload-images.jianshu.io/upload_images/14890912-27225d844148969b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
正在构建(就是重新打包)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-1c93b7bff8dc942e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
构建完成后查看部署项目的服务器上tomcat的webapp，就能看到abc.war。$\color{red}{是不是很神奇呢}$
![image.png](https://upload-images.jianshu.io/upload_images/14890912-100b27ce5d544f94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#$\color{red}{部署项目的服务器与Jenkins可以不是同一台服务器，这里的例子就是不相同的两台linux服务器}$

访问http://192.168.79.132:8080/abc/就能访问到部署的项目。
![image.png](https://upload-images.jianshu.io/upload_images/14890912-66506645362c4d93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


$\color{red}{现在测试修改代码然后提交到gitlab上。再点立即构建。看页面会不会修改}$
更新代码提交到gitlab
![image.png](https://upload-images.jianshu.io/upload_images/14890912-8715d61aa2d12a27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击立即构建
![image.png](https://upload-images.jianshu.io/upload_images/14890912-624de2e20bf9fcf4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
说明构建成功

$\color{red}{但是每次都要手动去点击立即构建，能不能在git pull操作时自动触发构建}$

安装Gitlab Hook Plugin插件：
#系统管理-管理插件-可选插件-Gitlab Hook和Build Authorization Token Root
![image.png](https://upload-images.jianshu.io/upload_images/14890912-bdeb4cd22e849fde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-4d2bd4a8181b8e6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在linux上执行命令生成token(身份验证令牌)
openssl rand -hex 12
![身份验证令牌](https://upload-images.jianshu.io/upload_images/14890912-df963c11c41f7871.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-5f2ca779c861bbc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
保存
![image.png](https://upload-images.jianshu.io/upload_images/14890912-a450a21dadbb679e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-e1c25440b7d9131e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**配置gitlab钩子配置**
进到gitlab的Settings-Integrations
![gitlab钩子配置](https://upload-images.jianshu.io/upload_images/14890912-3c61081a34a0470c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-cbed0c92a0bab62e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
add webhook之后会报这么一个错
![image.png](https://upload-images.jianshu.io/upload_images/14890912-42a459c6b2e2d1a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**gitlab 10.6 版本以后为了安全，默认不允许向本地网络发送webhook请求，如果想向本地网络发送webhook请求，则需要使用管理员帐号登录，默认管理员帐号是admin@example.com，密码就是你gitlab搭建好之后第一次输入的密码**
![image.png](https://upload-images.jianshu.io/upload_images/14890912-44fe90c1742b0f02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-3fc0108cdf4c622b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-c134b914a480f1f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再配置钩子
![image.png](https://upload-images.jianshu.io/upload_images/14890912-e42b0a2de20a2838.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
测试一下配置是否成功
![image.png](https://upload-images.jianshu.io/upload_images/14890912-f7169a6141224a55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Hook executed successfully: HTTP 201**表示成功
![image.png](https://upload-images.jianshu.io/upload_images/14890912-a098225612ac7640.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样就完成自动构建的配置了，直接git pull 提交代码，就会触发立即构建
![image.png](https://upload-images.jianshu.io/upload_images/14890912-2caf28e34868299e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-c3077dc52d9e1f9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**大功告成**
这里是war的jenkins构建。不知道springboot的jar和dubbo配置是否一致，后续学习