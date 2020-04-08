---
layout: post
title:  "使用git的submodule方式管理微服务架构代码"
date:   2020-04-02 15:40:24 +0800
categories: git submodule microservices code
---

创建整体项目repo，此处命名为parent
该repo不做具体代码提交用，仅做流水线发布拉取代码的主repo。
```shell
$ mkdir parent
$ cd parent
$ git init
$ git remote origin set-url http://localhost:3000/demo-multi-module/parent.git
$ git push
```


不同的微服务模块放在不同代码仓库 module 中，创建具体的 repo，此处命名module。
```shell
$ mkdir module
$ cd module
$ git init
$ git remote origin set-url http://localhost:3000/demo-multi-module/module.git
$ git push
```

将module添加为parent的子目录
```shell
$ git submodule add http://localhost:3000/demo-multi-module/module.git module
$ git commit -m "some comment"
$ git push
```

后面具体module提交代码就在自己repo中
```shell
$ echo 'first line' >> example.txt
$ git commit -m 'add example.txt'
$ git push 
```

由于当module具体需要更新时，在parent的repo要cd进入目录去 git pull比较繁琐，
以下为遍历更新多个module目录的脚本
```shell
#! /bin/bash

path=`pwd`

ls $path | while read line
do
	if [ -d $line ];then
		echo $line is a directory;
		cd $line;
		git pull;
		cd ..
	else
		echo $line is a file;
	fi
done
```

参考：
* https://blog.csdn.net/guotianqing/article/details/82391665