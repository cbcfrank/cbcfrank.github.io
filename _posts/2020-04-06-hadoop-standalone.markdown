---
layout: post
title:  "安装hadoop并在standalone模式下简单测试"
date:   2020-04-06 23:46:38 +0800
categories: hadoop standalone
---

1.安装jdk，[参考]({% post_url 2020-03-05-jdk-centos-install %})

2.下载hadoop二进制包

```shell
[root@hadoop211]# wget http://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/hadoop-2.7.2.tar.gz
```

3.设置hadoop环境变量

```shell
[root@hadoop211]# tar -zxvf hadoop-2.7.2.tar.gz -C /opt/module

[root@hadoop211]# vi /etc/profile

# 在文件尾部增加
## HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

[root@hadoop211]# source /etc/profile

[root@hadoop211]# hadoop version
Hadoop 2.7.2
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r b165c4fe8a74265c792ce23f546c64604acf0e41
Compiled by jenkins on 2016-01-26T00:08Z
Compiled with protoc 2.5.0
From source with checksum d0fda26633fa762bff87ec759ebe689c
This command was run using /opt/module/hadoop-2.7.2/share/hadoop/common/hadoop-common-2.7.2.jar

```

4.跑两个官方示例

* 官方Grep案例

```shell
[root@hadoop211]# mkdir -p /opt/tmp/hadoop/input
[root@hadoop211]# cp /opt/module/hadoop-2.7.2/etc/hadoop/*.xml /opt/tmp/hadoop/input
[root@hadoop211]# hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar \
 grep /opt/tmp/hadoop/input /opt/tmp/hadoop/output 'dfs[a-z.]+'
[root@hadoop211]# cat /opt/tmp/hadoop/output/*

```

* 官方WordCount案例

```shell
[root@hadoop211]# mkdir -p /opt/tmp/hadoop/wcinput
[root@hadoop211]# cp /opt/module/hadoop-2.7.2/etc/hadoop/*.xml /opt/tmp/hadoop/wcinput
[root@hadoop211]# hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar \
 wordcount /opt/tmp/hadoop/wcinput /opt/tmp/hadoop/wcoutput
[root@hadoop211]# cat /opt/tmp/hadoop/wcoutput/*

```