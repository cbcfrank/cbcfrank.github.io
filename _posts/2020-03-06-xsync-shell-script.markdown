---
layout: post
title:  "集群文件同步脚本"
date:   2020-03-06 17:39:24 +0800
categories: xsnyc shell script
---

1）在/root目录下创建bin目录，并在bin目录下创建文件xsync，文件内容如下：
```shell
$ cd ~
$ mkdir bin
$ cd bin/
$ vi xsync
```

2)在该文件中编写如下代码
```shell
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4 获取当前用户名称
user=`whoami`

#5 循环
for((host=103; host<105; host++)); do
        echo ------------------- hadoop$host --------------
        rsync -av $pdir/$fname $user@hadoop$host:$pdir
done

```

3）修改脚本 xsync 具有执行权限
```shell
$ chmod 777 xsync
```