---
layout: post
title:  "在CentOS安装JDK"
date:   2020-03-05 11:26:38 +0800
categories: centos java jdk
---

在CentOS安装JDK

1.解压JDK到/opt/module目录下，并修改文件的所有者和所有者组为root
```shell
[root@hadoop102 software]# ls jdk*
jdk-8u144-linux-x64.tar.gz
[root@hadoop102 software]# tar -zxvf jdk-8u144-linux-x64.tar.gz -C /opt/module/
```

2.修改文件的所有者和所有者组为root
```shell
[root@hadoop102 software]# cd /opt/module
[root@hadoop102 software]# chown root:root -R /opt/module/jdk1.8.0_144/
```

3.配置JDK环境变量
```shell
# 打开/etc/profile文件，在文件末尾添加JDK路径
[root@hadoop102 software]# vi /etc/profile

#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin

# 保存退出
```

4.让修改后的文件生效
```shell
[root@hadoop102 software]# source /etc/profile
```

5.测试JDK是否安装成功
```shell
[root@hadoop102 software]# java -version
java version "1.8.0_144"
```

6.确认java版本无误后完成

