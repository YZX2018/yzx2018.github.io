---
layout: post
title: "centos7安装mysql8.0"
date: 2018-12-06
tags: java
---

https://dev.mysql.com/downloads/mysql/  下载rpm的tar包并解压
![image.png](https://upload-images.jianshu.io/upload_images/14890912-3eb5ea07d1fe84ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14890912-c65fb14242c7ec4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


压解后将这几个文件上传到linux
mysql-community-common-8.0.16-2.el7.x86_64.rpm
mysql-community-libs-8.0.16-2.el7.x86_64.rpm
mysql-community-client-8.0.16-2.el7.x86_64.rpm
mysql-community-server-8.0.16-2.el7.x86_64.rpm
新建 /usr/mysql目录 使用winscp上传这四个安装包到linux服务器 /usr/mysql

1、安装新版mysql之前，我们需要将系统自带的mariadb-lib卸载
命令：
查询 rpm -qa|grep mariadb
删除 rpm -e mariadb-libs-5.5.56-2.el7.x86_64 --nodeps
2、进入到/usr/mysql目录，安装rpm
cd /usr/mysql(之前创建上传mysql的目录)
rpm -ivh mysql-community-common-8.0.16-2.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.16-2.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.16-2.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.16-2.el7.x86_64.rpm
安装到server会出现perl错误
yum install perl
再执行 rpm -ivh mysql-community-server-8.0.16-2.el7.x86_64.rpm
安装成功

启动mysql    
systemctl start mysqld
设置开机启动
[root@localhost ~]# systemctl enable mysqld
[root@localhost ~]# systemctl daemon-reload

mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个临时的默认密码。
[root@localhost ~]# vi /var/log/mysqld.log
查找到root@localhost: q2jvten?biYR      (不要最前面的空格)
进入到mysql服务
mysql -u root -p
q2jvten?biYR

进入后修改mysql密码 (备注 mysql5.7之后默认密码策略要求密码必须是大小写字母数字特殊字母的组合，至少8位)
进入mysql命令行；
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'GZyn@5180';
修改密码为 GZyn@5180   

注意：下面设置允许远程登录的两个版本操作不同。
设置允许远程登录(mysql5.7的方法)
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'GZyn@5180' WITH GRANT OPTION;
-------------------------------------------------------------------------------------------
设置允许远程登录(mysql8的方法)
mysql> use mysql;
2.查询host 
mysql> select user,host from user;
3.创建host 
如果没有”%”这个host值,就执行下面这两句: 
mysql> update user set host='%' where user='root'; 
mysql> flush privileges;


开放3306端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload

配置默认编码为utf8
修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'

重启mysql服务
systemctl restart mysqld


​	