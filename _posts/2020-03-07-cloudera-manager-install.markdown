---
layout: post
title:  "Cloudera Manager在CentOS上安装"
date:   2020-03-07 13:55:24 +0800
categories: cloudera manager centos install
---


1.CM下载地址

1).[CM下载地址](http://archive.cloudera.com/cm5/cm/5/)

2).[离线库下载地址](http://archive.cloudera.com/cdh5/parcels)

2.CM安装

注：以下所有操作均使用root用户

1).创建/opt/module/cm目录
```shell
[root@hadoop102 module]# mkdir –p /opt/module/cm
```

2).上传cloudera-manager-el6-cm5.12.1_x86_64.tar.gz到hadoop102的/opt/software目录，并解压到/opt/module/cm目录
```shell
[root@hadoop102 software]# tar -zxvf cloudera-manager-el6-cm5.12.1_x86_64.tar.gz -C /opt/module/cm
```

3).分别在hadoop102、hadoop103、hadoop104创建用户cloudera-scm
```shell
[root@hadoop102 module]# 
useradd \
--system \
--home=/opt/module/cm/cm-5.12.1/run/cloudera-scm-server \
--no-create-home \
--shell=/bin/false \
--comment "Cloudera SCM User" cloudera-scm

[root@hadoop103 module]# 
useradd \
--system \
--home=/opt/module/cm/cm-5.12.1/run/cloudera-scm-server \
--no-create-home \
--shell=/bin/false \
--comment "Cloudera SCM User" cloudera-scm

[root@hadoop104 module]# 
useradd \
--system \
--home=/opt/module/cm/cm-5.12.1/run/cloudera-scm-server \
--no-create-home \
--shell=/bin/false \
--comment "Cloudera SCM User" cloudera-scm

参数说明：
--system 创建一个系统账户
--home 指定用户登入时的主目录，替换系统默认值/home/<用户名>
--no-create-home 不要创建用户的主目录
--shell 用户的登录 shell 名
--comment 用户的描述信息
注意：Cloudera Manager默认去找用户cloudera-scm，创建完该用户后，将自动使用此用户。

```

4).修改CM Agent配置

修改文件/opt/module/cm/cm-5.12.1/etc/cloudera-scm-agent/ config.ini的主机名称

```shell
[root@hadoop102 cloudera-scm-agent]# vim /opt/module/cm/cm-5.12.1/etc/cloudera-scm-agent/config.ini
# 修改主机名称
server_host=hadoop102
```

5).配置CM的数据库

拷贝mysql-connector-java-5.1.27-bin.jar文件到目录 /usr/share/java/	
```shell
[root@hadoop102 cm]# mkdir –p /usr/share/java/
[root@hadoop102 mysql-libs]# tar -zxvf mysql-connector-java-5.1.27.tar.gz

[root@hadoop102 mysql-libs]# cp /opt/software/mysql-libs/mysql-connector-java-5.1.27/mysql-connector-java-5.1.27-bin.jar /usr/share/java/

[root@hadoop102 mysql-libs]# mv /usr/share/java/mysql-connector-java-5.1.27-bin.jar /usr/share/java/mysql-connector-java.jar
```
注意：jar包名称要修改为mysql-connector-java.jar

6).使用CM自带的脚本，在MySQL中创建CM库
```shell
[root@hadoop102 cm-5.12.1]# 
/opt/module/cm/cm-5.12.1/share/cmf/schema/scm_prepare_database.sh mysql cm -hhadoop102 -uroot -p000000 --scm-host hadoop102 scm scm scm

JAVA_HOME=/opt/module/jdk1.8.0_144
Verifying that we can write to /mnt/nas/hadoop102/module/cm/cm-5.12.1/etc/cloudera-scm-server
Creating SCM configuration file in /mnt/nas/hadoop102/module/cm/cm-5.12.1/etc/cloudera-scm-server
Executing:  /opt/module/jdk1.8.0_144/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/mnt/nas/hadoop102/module/cm/cm-5.12.1/share/cmf/schema/../lib/* com.cloudera.enterprise.dbutil.DbCommandExecutor /mnt/nas/hadoop102/module/cm/cm-5.12.1/etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
[                          main] DbCommandExecutor              INFO  Successfully connected to database.
All done, your SCM database is configured correctly!

参数说明
-h：Database host
-u：Database username
-p：Database Password
--scm-host：SCM server's hostname
```

7).修改服务端启动对应的数据地址、端口、用户、密码
```shell
[root@hadoop102 module]# vi /opt/module/cm/cm-5.12.1/etc/cloudera-scm-server/db.properties
...
com.cloudera.cmf.db.host=hadoop102
com.cloudera.cmf.db.name=cm
com.cloudera.cmf.db.user=root
com.cloudera.cmf.db.password=000000
...

```

8).分发cm
```shell
[root@hadoop102 module]# xsync /opt/module/cm
```

9).创建Parcel-repo
```shell
[root@hadoop102 module]# mkdir -p /opt/cloudera/parcel-repo
[root@hadoop102 module]# chown cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo
```

10).拷贝下载文件manifest.json 、CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha1 、CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel到hadoop102的/opt/cloudera/parcel-repo/目录下
```shell
[root@hadoop102 parcel-repo]# ls
CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel  CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha1  
manifest.json
```

11).将CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha1：需改名为 CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha
```shell
[root@hadoop102 parcel-repo]# mv CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha1 CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha
```

12).在hadoop102上创建目录/opt/cloudera/parcels，并修改该目录的所属用户及用户组为cloudera-scm
```shell
[root@hadoop102 module]# mkdir -p /opt/cloudera/parcels 
[root@hadoop102 module]# chown cloudera-scm:cloudera-scm /opt/cloudera/parcels
```

13).分发/opt/cloudera/
```shell
[root@hadoop102 opt]# xsync /opt/cloudera/
```

3.启动CM服务

1).启动服务节点：hadoop102
```shell
[root@hadoop102 cm]# /opt/module/cm/cm-5.12.1/etc/init.d/cloudera-scm-server start
Starting cloudera-scm-server:                              [确定]
```

2).启动工作节点：hadoop102、hadoop103、hadoop104
```shell
[root@hadoop102 cm]# /opt/module/cm/cm-5.12.1/etc/init.d/cloudera-scm-agent start

[root@hadoop103 cm]# /opt/module/cm/cm-5.12.1/etc/init.d/cloudera-scm-agent start

[root@hadoop104 cm]# /opt/module/cm/cm-5.12.1/etc/init.d/cloudera-scm-agent start
```
注意：启动过程非常慢，Manager启动成功需要等待5分钟左右，过程中会在数据库中创建对应的表需要耗费一些时间。

如果遇到，启动失败，日志文件/opt/module/cm/cm-5.12.1/log/cloudera-scm-agent/cloudera-scm-agent.out
提示 /usr/bin/env: python2.6: No such file or directory ，则需要单独设置python2.6环境，<[参考]()>

3).查看被占用则表示安装成功了！！！
```
[root@hadoop102 cm]# netstat -anp | grep 7180
tcp        0      0 0.0.0.0:7180                0.0.0.0:*                   LISTEN      5498/java 
```

4).访问http://hadoop102:7180，（用户名、密码：admin）

![img](/images/2020/img00015.png)
 
4.关闭CM服务

1).关闭工作节点：hadoop102、hadoop103、hadoop104
```shell
[root@hadoop102 cm]# /opt/module/cm/cm-5.12.1/etc/init.d/cloudera-scm-agent stop
Stopping cloudera-scm-agent:                               [确定]
[root@hadoop103 cm]# /opt/module/cm/cm-5.12.1/etc/init.d/cloudera-scm-agent stop
Stopping cloudera-scm-agent:                               [确定]
[root@hadoop104 cm]# /opt/module/cm/cm-5.12.1/etc/init.d/cloudera-scm-agent stop
Stopping cloudera-scm-agent:                               [确定]
```

2).关闭服务节点：hadoop102
```shell
[root@hadoop102 cm]# /opt/module/cm/cm-5.12.1/etc/init.d/cloudera-scm-server stop
停止 cloudera-scm-server：                                 [确定]
```