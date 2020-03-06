---
layout: post
title:  "在阿里云上用ECS作为SNAT让VPC中其他主机访问外网"
date:   2020-02-28 18:01:34 +0800
categories: aliyun ecs nat
---

以下方案是用于测试，阿里云有提供NAT网关服务，不需要下面步骤，但需要单独付费

在阿里云上购买了几台ECS用于测试，使用其中一台ECS搭建NAT网关，其他机器通过它对外访问。
>
> 资源说明：
> 
>    VPC: CIRD    172.16.0.0/12
>    
> Switch: CIRD    172.31.3.0/24
> 
>  ECS01: 内网 ip  172.31.3.40     CentOS7.7 ， 绑定弹性公网IP（首先在阿里云上购买弹性IP服务EIP）
>  
>  ECS02: 内网 ip  172.31.3.39     CentOS7.7
>  
>  ......
>  
  
#### 在有公网的ECS（172.31.3.40）上操作：

### 设置SNAT规则
1.开启转发
```shell
$ echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf  
# 或者 vi /etc/sysctl.conf 增加 net.ipv4.ip_forward = 1
$ sysctl -p # 使配置生效
```

2.确认iptables服务
```shell
$ service iptables status             # 检查是否安装了iptables
$ yum install -y iptables-services    # 如已安装可跳过此步骤
$ systemctl enable iptables.service   # 设置防火墙开机启动
```

3.配置iptables做SNAT：
```shell
# iptables -t nat -I POSTROUTING -s VPC的IP段 -j SNAT --to-source 有公网IP的ECS内网IP
$ iptables -t nat -I POSTROUTING -s 172.16.0.0/12 -j SNAT --to-source 172.31.3.40
```

4.iptables规则重启会清空，永久生效，还需要保存在iptables配置文件中：
```shell
$ service iptables save
```

5.设置VPC路由条目

阿里云控制台 -> 专有网络VPC -> 专有网络 -> 管理 -> 路由表 -> 管理 -> 添加路由条目
![image](/images/2020/img00014.png)


#### 解释：
* 搭建NAT网关就是为了实现在相同VPC内，没有公网IP的ECS借助有公网的ECS访问外网，或者是外网通过端口映射访问到内网服务器。
* SNAT：实现没有公网IP的ECS实例借助有公网的ECS访问外网，但是外网无法访问到内网IP；
* DNAT：实现外网通过端口映射访问到内网服务器，但是不能实现内网ECS访问到外网。

#### 参考：
* https://yq.aliyun.com/articles/607330
* https://blog.csdn.net/u013600314/article/details/90237133