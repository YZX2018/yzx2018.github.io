---
layout: post
title: "Linux(centos7) git安装"
date: 2016-12-24
tags: java
---

先到github上下载git的tar包    https://github.com/git/git/releases

上传到linux并解压

tar -zxvf git-2.19.1.tar.gz

进入目录配置 

cd git-2.19.1

make prefix=/usr/local/git all     

make prefix=/usr/local/git install

echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc

source /etc/bashrc

安装成功

查看

git --version



![img](https://upload-images.jianshu.io/upload_images/14890912-415de1ab846db9c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果报错就执行 以下三个加粗字体的命令 

**yum install autoconf**    因为要用到make configure，不然会报错**

**yum -y install gcc automake autoconf libtool make**    先安装gcc 才能执行make命令

因为是没有文件openssl/ssl.h，安装需要的依赖，使用命令：

**yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker** 