---
layout: post
title: "linux安装redis"
date: 2018-10-06
tags: java
---

1、在linux的/usr目录创建redis文件夹
2、上传redis-4.0.9.tar.gz到redis文件夹
3、进入到redis目录解压 tar.gz    命令 tar -zxvf redis-4.0.9.tar.gz
4、解决完后，有redis-4.0.9的目录 进入目录 命令 cd redis-4.0.9
5、编译。命令 make
（注意，编译需要C语言编译器gcc的支持，如果没有，需要先安装gcc。可以使用rpm -q gcc查看gcc是否安装）
（利用yum在线安装gcc的命令    yum -y install gcc )
（如果编译出错，请使用make clean清除临时文件。之后，找到出错的原因，解决问题后再来重新安装。 ）
6、编译完后，有src目录，进入到src目录 命令 cd src
7、执行安装  命令 make install

到此就安装完成。但是，由于安装redis的时候，我们没有选择安装路径，故是默认位置安装当前目录 (/usr/redis/redis-4.0.9)。在此，我们可以将可执行文件和配置文件移动到习惯的目录
在/usr/local目录创建redis/bin和redis/ect目录
命令
cd /usr/local
mkdir -p /usr/local/redis/bin    
mkdir -p /usr/local/redis/etc
再进入安装的redis目录。把执行文件和配置文件复制到刚才创建的目录
cd /usr/redis/redis-4.0.9
复制配置文件到/usr/local/redis/etc目录
cp ./redis.conf /usr/local/redis/etc
进入src目录，复制执行文件到/usr/local/redis/bin目录
cd src
cp mkreleasehdr.sh redis-benchmark redis-check-aof redis-cli redis-server redis-sentinel /usr/local/redis/bin

启动redis，默认是前台启动，启动不能作其他操作，需要设置为后台启动
进入到/usr/local/redis/etc目录 修改 redis.conf文件
命令
cd /usr/local/redis/etc
vi redis.conf
找到daemonize no，改成yes(开启后台启动)       /daemonize 回车定位  N向上 n向下
找到requirepass 123456  修改Redis的连接密码为123456
bind 127.0.0.1注释掉  改成    # bind 127.0.0.1     允许外网访问
:wq保存

 启动redis 命令
cd /usr/local/redis/bin
./redis-server /usr/local/redis/etc/redis.conf          加上redis.conf文件启动，作用：比如设置了密码

linux进行redis客户端 设置了密码  使用redis密码连接  不然操作会报(error) NOAUTH Authentication required
./redis-cli -h 127.0.0.1 -p 6379 -a "123456" 


远程连接需要进入到/usr/local/redis/etc目录 修改 redis.conf文件   
1、把配置文件的bind 127.0.0.1注释掉  改成    # bind 127.0.0.1
2、设置密码  修改密码 # requirepass foobared  修改为requirepass 123456
3、开防火墙 开一个6379端口权限  或者关闭防火墙
firewall-cmd --zone=public --add-port=6379/tcp --permanent

firewall-cmd --reload