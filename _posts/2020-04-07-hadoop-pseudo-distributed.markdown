---
layout: post
title:  "hadoop的Pseudo-Distributed模式测试"
date:   2020-04-07 12:15:38 +0800
categories: hadoop pseudo-distributed
---

#### 1.安装jdk，[参考]({% post_url 2020-03-05-jdk-centos-install %})

#### 2.安装hadoop，[参考]({% post_url 2020-04-06-hadoop-standalone %})

#### 3.配置文件修改

```shell
## hadoop-env.sh 
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/hadoop-env.sh 
export JAVA_HOME=/opt/module/jdk1.8.0_144


## core-site.xml
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml
<configuration>
	<!-- 指定HDFS中NameNode的地址 -->
	<property>
	<name>fs.defaultFS</name>
	    <value>hdfs://hadoop211:9000</value>
	</property>

	<!-- 指定Hadoop运行时产生文件的存储目录 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/module/hadoop-2.7.2/data/tmp</value>
	</property>
</configuration>


## hdfs-site.xml
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/hdfs-site.xml
<configuration>
	<!-- 指定HDFS副本的数量 -->
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
</configuration>

```

#### 4.启动集群

1).格式化NameNode（第一次启动时格式化，以后就不要总格式化）

```shell
[root@hadoop211]# hdfs namenode -format
```

2).启动NameNode

```shell
[root@hadoop211]# hadoop-daemon.sh start namenode
```

3).启动DataNode
```shell
[root@hadoop211]# hadoop-daemon.sh start datanode
```

4).确认是否启动成功

```shell
# 注意：jps是JDK中的命令，不是Linux命令。不安装JDK不能使用jps
[root@hadoop211]# jps
2176 NameNode
2259 DataNode
2333 Jps
```

web端查看 http://hadoop211:50070/

5).查看产生的Log日志
		  
说明：在企业中遇到Bug时，经常根据日志提示信息去分析问题、解决Bug。

```shell
[root@hadoop211]# ls /opt/module/hadoop-2.7.2/logs
total 64
-rw-r--r--. 1 root root 23979 Apr  7 16:49 hadoop-root-datanode-hadoop211.log
-rw-r--r--. 1 root root   714 Apr  7 16:49 hadoop-root-datanode-hadoop211.out
-rw-r--r--. 1 root root 27653 Apr  7 16:56 hadoop-root-namenode-hadoop211.log
-rw-r--r--. 1 root root  4960 Apr  7 16:56 hadoop-root-namenode-hadoop211.out

```

6).注意：为什么不能一直格式化NameNode，格式化NameNode，要注意什么？

```shell
[root@hadoop211]# cat /opt/module/hadoop-2.7.2/data/tmp/dfs/name/current/VERSION
#Tue Apr 07 16:46:56 CST 2020
namespaceID=1667612766
clusterID=CID-4b8e5f21-529e-40e5-8ff5-55fcf64f3cc0
cTime=0
storageType=NAME_NODE
blockpoolID=BP-1450470986-192.168.31.211-1586249216548
layoutVersion=-63


[root@hadoop211]# cat /opt/module/hadoop-2.7.2/data/tmp/dfs/data/current/VERSION
#Tue Apr 07 16:49:29 CST 2020
storageID=DS-79880931-ab57-4cee-8ce1-9c5fe21d0812
clusterID=CID-4b8e5f21-529e-40e5-8ff5-55fcf64f3cc0
cTime=0
datanodeUuid=15e0af92-e7f4-49ff-b47f-17d508e14fa8
storageType=DATA_NODE
layoutVersion=-56

```

注意：格式化NameNode，会产生新的集群id,导致NameNode和DataNode的集群id不一致，集群找不到已往数据。所以，格式NameNode时，一定要先删除data数据和log日志，然后再格式化NameNode。


#### 5.操作集群

1).在HDFS文件系统上创建一个input文件夹

```shell
[root@hadoop211]# hdfs dfs -mkdir -p /user/bigdata/input

```

2).将测试文件内容上传到文件系统上

```shell
[root@hadoop211]# hdfs dfs -put /opt/tmp/hadoop/wcinput/*  /user/bigdata/input/
```

3).查看上传的文件是否正确

```shell
[root@hadoop211]# hdfs dfs -ls  /user/bigdata/input/
[root@hadoop211]# hdfs dfs -cat  /user/bigdata/input/hadoop-policy.xml
```

4).运行MapReduce程序

```shell
[root@hadoop211]# hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar \
 wordcount /user/bigdata/input/ /user/bigdata/output
```

5).查看输出结果

```shell
[root@hadoop211]# hdfs dfs -cat /user/bigdata/output/*
```

6).将测试文件内容下载到本地

```shell
[root@hadoop211]# hdfs dfs -get /user/bigdata/output/part-r-00000 /root/
```

7).删除输出结果

```shell
[root@hadoop211]# hdfs dfs -rm -r /user/bigdata/output
```

#### 6.启动YARN并运行MapReduce程序

1).配置

```shell
## 配置yarn-env.sh
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/yarn-env.sh

export JAVA_HOME=/opt/module/jdk1.8.0_144


## 配置yarn-site.xml
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/yarn-site.xml

<!-- Reducer获取数据的方式 -->
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN的ResourceManager的地址 -->
<property>
	<name>yarn.resourcemanager.hostname</name>
	<value>hadoop211</value>
</property>


## 配置 mapred-env.sh
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/mapred-env.sh

export JAVA_HOME=/opt/module/jdk1.8.0_144


## 配置： (对mapred-site.xml.template重新命名为) mapred-site.xml
[root@hadoop211]# mv /opt/module/hadoop-2.7.2/etc/hadoop/mapred-site.xml.template \
	/opt/module/hadoop-2.7.2/etc/hadoop/mapred-site.xml 
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/mapred-site.xml 

<!-- 指定MR运行在YARN上 -->
<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
</property>
```

2).启动集群

 * 启动前必须保证NameNode和DataNode已经启动
 * 启动

```shell
## 启动ResourceManager
[root@hadoop211]# yarn-daemon.sh start resourcemanager

## 启动NodeManager
[root@hadoop211]# yarn-daemon.sh start nodemanager
```

3).集群操作
 * YARN的浏览器页面查看 http://hadoop211:8088/cluster
 
```shell
## 删除文件系统上的output文件
[root@hadoop211]# hdfs dfs -rm -R /user/bigdata/output

## 执行MapReduce程序
[root@hadoop211]# hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar \
  wordcount /user/bigdata/input  /user/bigdata/output

## 查看运行结果
[root@hadoop211]# hdfs dfs -cat /user/bigdata/output/*
```

4).配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下：

```shell
## 配置mapred-site.xml
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/mapred-site.xml

<!-- 历史服务器端地址 -->
<property>
	<name>mapreduce.jobhistory.address</name>
	<value>hadoop211:10020</value>
</property>
<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop211:19888</value>
</property>

```
* 启动历史服务器

```shell
[root@hadoop211]# mr-jobhistory-daemon.sh start historyserver

##	查看历史服务器是否启动
[root@hadoop211]# jps
```

* 查看JobHistory http://hadoop211:19888/jobhistory

5).配置日志的聚集

日志聚集概念：应用运行完成以后，将程序运行日志信息上传到HDFS系统上。
日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试。
注意：开启日志聚集功能，需要重新启动NodeManager 、ResourceManager和HistoryManager。
开启日志聚集功能具体步骤如下：

```shell
##	配置yarn-site.xml
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/yarn-site.xml
<!-- 日志聚集功能使能 -->
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>

<!-- 日志保留时间设置7天 -->
<property>
<name>yarn.log-aggregation.retain-seconds</name>
<value>604800</value>
</property>


##	关闭NodeManager 、ResourceManager和HistoryManager
[root@hadoop211]# yarn-daemon.sh stop resourcemanager
[root@hadoop211]# yarn-daemon.sh stop nodemanager
[root@hadoop211]# mr-jobhistory-daemon.sh stop historyserver


##	启动NodeManager 、ResourceManager和HistoryManager
[root@hadoop211]# yarn-daemon.sh start resourcemanager
[root@hadoop211]# yarn-daemon.sh start nodemanager
[root@hadoop211]# mr-jobhistory-daemon.sh start historyserver


## 删除HDFS上已经存在的输出文件
[root@hadoop211]# hdfs dfs -rm -R /user/atguigu/output


##	执行WordCount程序
[root@hadoop211]# hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar \
  wordcount /user/bigdata/input /user/bigdata/output
```

* 查看日志，http://hadoop101:19888/jobhistory


6). 配置日志的聚集

* 日志聚集概念：应用运行完成以后，将程序运行日志信息上传到HDFS系统上。

* 日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试。

* 注意：开启日志聚集功能，需要重新启动NodeManager 、ResourceManager和HistoryManager。

开启日志聚集功能具体步骤如下：

```shell
## 配置yarn-site.xml
[root@hadoop211]# vi /opt/module/hadoop-2.7.2/etc/hadoop/yarn-site.xml

<!-- 日志聚集功能使能 -->
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>

<!-- 日志保留时间设置7天 -->
<property>
<name>yarn.log-aggregation.retain-seconds</name>
<value>604800</value>
</property>


## 关闭NodeManager 、ResourceManager和HistoryManager
[root@hadoop211]# mr-jobhistory-daemon.sh stop historyserver
[root@hadoop211]# yarn-daemon.sh stop nodemanager
[root@hadoop211]# yarn-daemon.sh stop resourcemanager


## 启动NodeManager 、ResourceManager和HistoryManager
[root@hadoop211]# yarn-daemon.sh start resourcemanager
[root@hadoop211]# yarn-daemon.sh start nodemanager
[root@hadoop211]# mr-jobhistory-daemon.sh start historyserver


## 删除HDFS上已经存在的输出文件
[root@hadoop211]# hdfs dfs -rm -R /user/bigdata/output


## 执行WordCount程序
[root@hadoop211]#  hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar \
 wordcount /user/bigdata/input /user/bigdata/output
```

* 查看日志，http://hadoop101:19888/jobhistory