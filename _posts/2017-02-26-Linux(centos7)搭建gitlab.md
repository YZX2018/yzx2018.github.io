---
layout: post
title: "Linux(centos7)搭建gitlab"
date: 2017-02-26
tags: java
---

https://about.gitlab.com/install/#centos-7  官方安装gitlab的方法    

这里安装社区版

先执行官方文档的第一步  

防火墙打开HTTP和SSH访问

sudo yum install -y curl policycoreutils-python openssh-server

sudo systemctl enable sshd

sudo systemctl start sshd

sudo firewall-cmd --permanent --add-service=http

sudo systemctl reload firewalld

sudo yum install postfix

sudo systemctl enable postfix

sudo systemctl start postfix

第二步  社区版gitlab安装 https://packages.gitlab.com/gitlab/gitlab-ce  也可以先把rpm下载好https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7

这里使用在线下载

wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-11.4.5-ce.0.el7.x86_64.rpm/download.rpm

下载完成

![img](https://upload-images.jianshu.io/upload_images/14890912-2d58e03544cff015.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装rpm

rpm -ivh  gitlab-ce-11.4.5-ce.0.el7.x86_64.rpm

安装成功

修改gitlab的访问地址。

vim /etc/gitlab/gitlab.rb

![img](https://upload-images.jianshu.io/upload_images/14890912-2d9d0b1e34e6e365.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

http://192.168.79.129  (默认端口80)

保存之后更新配置

gitlab-ctl reconfigure (这一步需要等个几分钟)

**gitlab-ctl restart**

**这里就完成了**

注意 linux的内存要配置成4G以上，不然linux会很卡，访问不了

直接访问http://192.168.79.129就能进到页面