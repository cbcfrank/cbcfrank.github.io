---
layout: post
title:  "集群整体操作脚本"
date:   2020-03-06 17:39:24 +0800
categories: xcall shell script
---

1）在/root/bin目录下创建脚本xcall.sh
```shell
[root@hadoop102 bin]$ vim xcall.sh
```

2）在脚本中编写如下内容
```shell
#! /bin/bash

for i in hadoop102 hadoop103 hadoop104
do
        echo --------- $i ----------
        ssh $i "$*"
done

```
3）修改脚本执行权限
```shell
[root@hadoop102 bin]$ chmod 777 xcall.sh
```