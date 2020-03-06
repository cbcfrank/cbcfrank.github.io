---
layout: post
title:  "Mysql5.6在Centos7上的安装排错过程"
date:   2020-03-05 15:26:38 +0800
categories: mysql5.6 centos install
---

首先准备mysql5.6相关的安装包

注意：一定要用root用户操作如下步骤；先卸载MySQL再安装

1.准备安装包
```shell
[root@hadoop102 mysql-libs]# ll
总用量 76048
-rw-r--r--. 1 root root 18509960 3月  26 2015 MySQL-client-5.6.24-1.el6.x86_64.rpm
-rw-r--r--. 1 root root  3575135 12月  1 2013 mysql-connector-java-5.1.27.tar.gz
-rw-r--r--. 1 root root 55782196 3月  26 2015 MySQL-server-5.6.24-1.el6.x86_64.rpm
```

2.检查并卸载本地的自带默认msyql及依赖
```shell
#查看MySQL是否安装
[root@hadoop102 mysql-libs]# rpm -qa|grep mysql
mysql-libs-5.1.73-7.el6.x86_64
[root@hadoop102 mysql-libs]# rpm -qa| grep mariadb
mariadb-libs-5.5.64-1.el7.x86_64

#如果安装了就先卸载
[root@hadoop102 mysql-libs]# rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_64
[root@hadoop102 mysql-libs]# rpm -e --nodeps mariadb-libs-5.5.64-1.el7.x86_64
```

3.安装MySQL服务器
```shell
# 安装
[root@hadoop102 mysql-libs]# yum localinstall MySQL-server-5.6.24-1.el6.x86_64.rpm

# 查看MySQL状态
[root@hadoop102 mysql-libs]# service mysql status

# 启动MySQL
[root@hadoop102 mysql-libs]# service mysql start
Starting MySQL..The server quit without updating PID file ([FAILED]/mysql/hadoop102.pid).

# 启动不了，如遇上述状况，查看 /var/lib/mysql/hadoop102.err
200306 10:53:15 mysqld_safe Starting mysqld daemon with databases from /data/mysql
2020-03-06 10:53:15 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-03-06 10:53:15 0 [Note] /usr/sbin/mysqld (mysqld 5.6.24) starting as process 2919 ...
2020-03-06 10:53:15 2919 [Note] Plugin 'FEDERATED' is disabled.
/usr/sbin/mysqld: Table 'mysql.plugin' doesn't exist
2020-03-06 10:53:15 2919 [ERROR] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
2020-03-06 10:53:15 2919 [Note] InnoDB: Using atomics to ref count buffer pool pages

# 提示Table 'mysql.plugin' doesn't exist
# 执行以下操作，找到 mysql_install_db 所在目录做一次初始化
[root@hadoop102 mysql-libs]# find / -name mysql_install_db
/usr/bin/mysql_install_db

# 执行/usr/bin/mysql_install_db 初始化数据库
[root@hadoop102 mysql-libs]# /usr/bin/mysql_install_db --user=mysql
FATAL ERROR: please install the following perl modules before executing /usr/bin/mysql_install_db:
Data::Dumper

# 如出现上述错误执行下
[root@hadoop102 mysql-libs] yum -y install autoconf
# 再执行
[root@hadoop102 mysql-libs]# /usr/bin/mysql_install_db --user=mysql

# 再次启动
[root@hadoop102 mysql-libs]# service mysql start
Starting MySQL..                                           [  OK  ]
```

4、更换持久化数据目录，目的是以后服务器销毁还能保留，如无该需求此步可忽略
```shell
# 在NAS挂载盘中创建对应目录，并做好本机软链
[root@hadoop102 mysql-libs]# mkdir -p /mnt/nas/hadoop102/data/
[root@hadoop102 mysql-libs]# ln -s /mnt/nas/hadoop102/data/ /data
[root@hadoop102 mysql-libs]# mkdir -p /data/mysql

# 停止服务器
[root@hadoop102 mysql-libs]# service mysql stop

# 修改mysql服务的启动文件数据目录
[root@hadoop102 mysql-libs]# vi /etc/rc.d/init.d/mysql
# 找到开头 datadir= 的地方改为 datadir=/data/mysql

# 剪切 /var/lib/mysql 下的所有文件到 /data/mysql 下
[root@hadoop102 mysql-libs]# mv /var/lib/mysql/* /data/mysql/

# 再次启动服务器
[root@hadoop102 mysql-libs]# service mysql start

```

5.安装MySQL客户端并重置用户权限

1).安装MySQL客户端
```shell
[root@hadoop102 mysql-libs]# yum localinstall MySQL-client-5.6.24-1.el6.x86_64.rpm
```

2).链接MySQL并修改密码
```shell
[root@hadoop102 mysql-libs]# mysql
mysql>SET PASSWORD=PASSWORD('000000');
mysql>exit
```

3).修改MySQL中user表中主机配置，配置只要是root用户+密码，在任何主机上都能登录MySQL数据库。
```shell
# 进入MySQL
[root@hadoop102 mysql-libs]# mysql -uroot -p000000

# 显示数据库
mysql>show databases;

# 使用MySQL数据库
mysql>use mysql;

# 展示MySQL数据库中的所有表
mysql>show tables;

# 展示user表的结构
mysql>desc user;

# 查询user表
mysql>select User, Host, Password from user;

# 修改user表，把Host表内容修改为%
mysql>update user set host='%' where host='localhost';

# 删除root用户的其他host
mysql>delete from user where Host='hadoop102';
mysql>delete from user where Host='127.0.0.1';
mysql>delete from user where Host='::1';

# 刷新使权限生效
mysql>flush privileges;

# 退出
mysql>quit;
```