---
layout: post
title:  "如何在windows 7上搭建docker运行环境"
date:   2017-08-27 21:06:24 +0800
categories: docker machine git-bash
---
### 工具准备

#### 安装git bash
可以直接使用 [https://git-scm.com/download/win]() 下载安装

#### 安装docker-machine
将 [https://github.com/docker/machine/releases/download/v0.12.2/docker-machine-Windows-x86_64.exe]() 下载到 `c:\windows\` 下，并重命名为 `docker-machine.exe`

![image](/images/2017/img00003.png)

打开git bash ，确认docker machine版本信息

![image](/images/2017/img00004.png)

### 过程步骤

#### 创建一个安装好docker engine 的虚拟机

```shell
$ docker-machine create -d virtualbox test
```

![image](/images/2017/img00005.png)

（提示：下载`boot2docker.iso`过程中未科学上网时在国内会比较慢，可以将该iso先下载到本地  `C:\Users\你的账号\.docker\machine\cache`，这步骤不是必要的）

![image](/images/2017/img00006.png)

提示成功之后，可以使用 `docker-machine ls` 看下是否成功

```shell
$ docker-machine.exe ls
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
test   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.1-ce
```

#### 连接该虚拟机

```shell
$ docker-machine.exe ssh test
```

![image](/images/2017/img00007.png)

#### 下载你的第一个镜像（当然可以是其他的，这里我们以jekyll为例）

```shell
$  docker pull jekyll/jekyll
```

![image](/images/2017/img00008.png)

#### 可以开始运行你的第一个容器了（如果你使用的其他的镜像，下面的命令会不一样）

```shell
docker run --rm --volume=$PWD:/srv/jekyll -it jekyll/jekyll jekyll new myblog
```

![image](/images/2017/img00009.png)

```shell
$ docker run --rm --volume=$PWD/myblog:/srv/jekyll -p 4000:4000 -it jekyll/jekyll jekyll server
```

![image](/images/2017/img00010.png)

#### 打开浏览器运行 `http://192.168.99.100:4000`

![image](/images/2017/img00011.png)

#### 恭喜你