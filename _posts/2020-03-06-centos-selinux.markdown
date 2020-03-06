---
layout: post
title:  "关闭SELINUX"
date:   2020-03-06 10:06:24 +0800
categories: jekyll update
---

安全增强型Linux（Security-Enhanced Linux）简称SELinux，它是一个 Linux 内核模块，也是Linux的一个安全子系统。

有如下两种关闭方法

1.临时关闭（不建议使用）
```shell
[root@hadoop102 ~]# setenforce 0
```
但是这种方式只对当次启动有效，重启机器后会失效。

2.永久关闭（建议使用）
修改配置文件/etc/selinux/config
```shell
[root@hadoop102 ~]# vim /etc/selinux/config
将SELINUX=enforcing 改为SELINUX=disabled
SELINUX=disabled
```

3.同步/etc/selinux/config配置文件
```shell
[root@hadoop102 ~]# xsync /etc/selinux/config
```

4.重启主机
```shell
[root@hadoop102 ~]# reboot
```