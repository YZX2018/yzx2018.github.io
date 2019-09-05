---
layout: post
title: "java微信公众平台开发(使用sunny ngrok设置外网url)"
date: 2018-02-02
tags: java
---

使用Sunny-Ngrok赠送的免费域名进行http服务映射

1、首先在本站注册成为会员

2、开通隧道

![blob.png](https://www.sunnyos.com/upfiles/images/201702/2214876988332365.png)

![blob.png](https://www.sunnyos.com/upfiles/images/201702/2214876990875395.png)

在上图中表单信息解释：

​    隧道名称：可以随便填写，只是为了使用者在往后之后这个隧道是干嘛的

​    前置域名：服务器免费赠送的域名，请不要带上后缀，如果要sunny.ngrok.cc 只需要填写 sunny 即可 ，千万不要填写成sunny.ngrok.cc否则域名会是sunny.ngrok.cc.ngrok.cc

​    本地端口：可以为同一个局域网内任意一台机器进行映射，只需要填对ip和端口就行，例如：192.168.1.1:80

​    http验证用户名：非必填项，在需要的时候填写，否则可以不填

​    http验证密码：非必填项，在需要的时候填写，否则可以不填

3、在需要映射的机器建立web服务

​    如果没有启动web服务将会看到下图提示

​    ![blob.png](https://www.sunnyos.com/upfiles/images/201702/2214876993514081.png)

4、下载客户端

​    ![blob.png](https://www.sunnyos.com/upfiles/images/201702/2214876994228691.png)

​    根据自己的操作系统需求下载相对于版本

5、启动映射服务

​    Linux或者Mac系统下启动，需要进入到客户端的目录执行以下命令，通过隧道管理找到隧道id，然后执行

```

```

1. `./sunny clientid 隧道id`

``

``

```
![blob.png](https://www.sunnyos.com/upfiles/images/201702/2214876996566971.png)   ![blob.png](https://www.sunnyos.com/upfiles/images/201702/2214876995473640.png)    ![blob.png](https://www.sunnyos.com/upfiles/images/201702/2214876995891358.png)    看到这样则是成功启动一条隧道，启动多条隧道把多条隧道id使用英文逗号隔开就可以了。例如：
```

``

``

```

```

1.   ./sunny clientid c7fb2defb4081919,3e23d14f040b2b12

​    上面命令就可以启动多条隧道了

6、访问服务

​    然后通过启动成功界面上的链接访问网站如果显示是内网搭建的web服务里面的内容就说明成功了



windows打开cmd，到Ngrok的目录下



1. sunny.exe clientid 隧道id