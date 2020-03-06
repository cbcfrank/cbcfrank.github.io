---
layout: post
title:  "搭建大数据实验基础环境"
date:   2020-02-29 11:54:38 +0800
categories: bigdata experiment enviroment
---

启动三台虚拟机，设置好基础网络环境。<[详见]({% post_url 2020-02-28-aliyun-ecs-nat %})>

本实验在阿里云ECS上完成。

>
> ECS：
> 
>   hadoop102: 172.31.3.41  4核16G
>   
>   hadoop103: 172.31.3.40  2核4G
>   
>   hadoop104: 172.31.3.39  2核4G
>   
> NAS :  共享挂载到ECS中的 /mnt/nas 目录
> 

1.通过主机名相互访问
```shell
# 在hadoop102、hadoop103、hadoop104操作
[root@hadoop102 ~]# vi /etc/hosts
172.31.3.41 hadoop103
172.31.3.40 hadoop103
172.31.3.39 hadoop104
...

```
  
2.设置机器免密钥相互访问
```shell
# 在hadoop102、hadoop103、hadoop104操作
[root@hadoop102 ~]# ssh-keygen -t rsa
[root@hadoop102 ~]# ssh-copy-id hadoop102
[root@hadoop102 ~]# ssh-copy-id hadoop103
[root@hadoop102 ~]# ssh-copy-id hadoop104
```

3.在nas创建具体文件夹
```shell
[root@hadoop102 ~]# mkdir -p /mnt/nas/{hadoop102/module,hadoop103/module,hadoop104/module,software}
```

4.挂载nas的ecs目录
```shell
# hadoop102,hadoop103,hadoop104 会通过软链挂载到具体的主机

# 在 hadoop102上操作
[root@hadoop102 ~]# ln -s /mnt/nas/hadoop102/module /opt/module
[root@hadoop102 ~]# ln -s /mnt/nas/software /opt/software

# 在 hadoop103上操作
[root@hadoop103 ~]# ln -s /mnt/nas/hadoop103/module /opt/module
[root@hadoop103 ~]# ln -s /mnt/nas/software /opt/software

# 在 hadoop104上操作
[root@hadoop104 ~]# ln -s /mnt/nas/hadoop104/module /opt/module
[root@hadoop104 ~]# ln -s /mnt/nas/software /opt/software
```

5.[集群文件同步脚本]({% post_url 2020-03-06-xsync-shell-script %})

6.[集群整体操作脚本]({% post_url 2020-03-06-xcall-shell-script %})

7.jdk安装

1).基本配置<[参考]({% post_url 2020-03-05-jdk-centos-install %})>

2).将hadoop102中的JDK和环境变量分发到hadoop103、hadoop104两台主机
```shell
[root@hadoop102 opt]# xsync /opt/module/
[root@hadoop102 opt]# xsync /etc/profile
```
3).分别在hadoop103、hadoop104上source一下
```shell
[root@hadoop103 ~]$ source /etc/profile
[root@hadoop104 ~]# source /etc/profile
```

8.安装MySQL

1).安装包准备
```shell
# 上传mysql-libs.zip到hadoop102的/opt/software目录，并解压文件到当前目录
[root@hadoop102 software]# unzip mysql-libs.zip
[root@hadoop102 software]# ls
mysql-libs.zip
mysql-libs
```

2).安装Mysql服务端和客户端<[参考]({% post_url 2020-03-05-mysql-centos-install %})>

9.创建CM用的数据库
在MySQL中依次创建监控数据库、Hive数据库、Oozie数据库、Hue数据库

```shell
# 启动数据库
[root@hadoop102 ~]# mysql -uroot -p000000

# 集群监控数据库
mysql> create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

# Hive数据库 
mysql> create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

# Oozie数据库
mysql> create database oozie DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

# Hue数据库
mysql> create database hue DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

# 关闭数据库
mysql> quit;
```

10.下载第三方依赖

依次在三台节点（所有Agent的节点）上执行下载第三方依赖（注意：需要联网）
```shell
[root@hadoop102 ~]# yum -y install chkconfig python bind-utils psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb

[root@hadoop103 ~]# yum -y install chkconfig python bind-utils psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb

[root@hadoop104 ~]# yum -y install chkconfig python bind-utils psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb
```

11.关闭SELINUX<[参考]({% post_url 2020-03-06-centos-selinux %})>

为了避免安装过程出现各种错误，建议关闭：

设置好后，三台机器都需要重启
```shell
# 重启hadoop102、hadoop103、hadoop104主机
[root@hadoop102 ~]# reboot
[root@hadoop103 ~]# reboot
[root@hadoop104 ~]# reboot
```