---
layout: post
title:  "如何通过remi安装php7.1"
date:   2017-09-02 09:50:34 +0800
categories: php remi centos
---

*本操作是在`centos 6.8`系统上的实际操作*

```shell

# epel源安装及配置REMI仓库

$ yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
$ yum install -y http://rpms.remirepo.net/enterprise/remi-release-6.rpm

# yum方式 指定通过remi源安装 --enablerepo=remi --enablerepo=remi-php71
# 后面的参数，
# 如是php自带扩展则使用如php-mcrypt
# 如是pecl上的扩展则使用如php-pecl-xdebug

$ yum install -y --enablerepo=remi --enablerepo=remi-php71 php php-devel php-pdo php-pecl-xdebug php-mbstring php-mcrypt php-bcmath php-xml php-mysqlnd mod_ssl php-pecl-redis
$ yum clean all
$ php --version
 
```

#### 恭喜你，已完成了。

---

##### 延伸

什么是epel？

如果既想获得 RHEL 的高质量、高性能、高可靠性，又需要方便易用(关键是免费)的软件包更新功能，那么 Fedora Project 推出的 EPEL(Extra Packages for Enterprise Linux)正好适合你。[EPEL](http://fedoraproject.org/wiki/EPEL) 是由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS、Scientific Linux 等提供高质量软件包的项目。

什么是remi？

Remi repository 提供了CentOS和RHEL的核心包的更新版本，是包含最新版本 PHP 和 MySQL 包的 Linux 源，由 [Remi](http://rpms.famillecollet.com/) 提供维护。有个这个源之后，使用YUM安装或更新PHP、MySQL、phpMyAdmin 等服务器相关程序的时候就非常方便了。当你需要一个更新包，而 CentOS/RHEL 没有及时提供更新时， REMI 仓库可以帮助你

*安装REMI仓库*

现在按照下面的步骤安装REMI仓库。

在CentOS 7上:
```shell
$ sudo rpm --import http://rpms.famillecollet.com/RPM-GPG-KEY-remi
$ sudo rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

在CentOS 6上:

```shell
$ sudo rpm --import http://rpms.famillecollet.com/RPM-GPG-KEY-remi
$ sudo rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```
安装REMI仓库要记住的一件事是不要在启用了REMI仓库时运行yum update。因为REMI仓库的包名与RHEL/CentOS中的相同，运行yum update可能会触发意外的更新。一个好办法是禁用REMI仓库，在你需要安装RMEI仓库中独有的包时再启用。



默认地，REMI是禁用的。要检查REMI是否已经成功安装，使用这个命令。你会看到几个REMI仓库，比如remi、remi-php55和remi-php56。

```shell
$ yum repolist disabled | grep remi
```

![image](/images/2017/img00012.jpg)

从REMI仓库中安装一个包

如上所述，最好保持禁用REMI仓库，只有在需要的时候再启用。

要搜索或安装REMI仓库中的包，使用这些命令:

```shell
$ sudo yum --enablerepo=remi search <keyword>
$ sudo yum --enablerepo=remi install <package-name>
```