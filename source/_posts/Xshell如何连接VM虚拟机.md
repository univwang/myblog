---
title: Xshell如何连接VM虚拟机
date: 2022-10-06 22:40:18
tags: [Linux]
excerpt: vm通过net与主机连接，使用xshell连接虚拟机
categories: Linux
index_img: /img/index_img/10.png
banner_img: /img/banner_img/background11.jpg
---


## 配置VM

配置VMware的网络适配器，包括网关、子网地址范围
选择VMware编辑中的虚拟网络编辑器
![](https://raw.githubusercontent.com/univwang/img/main/20221007193039.png)

记住网关和子网范围
![](https://raw.githubusercontent.com/univwang/img/main/20221007193245.png)
## 配置主机网络连接

与配置vm类似，修改网络适配器的ip、网关

![](https://raw.githubusercontent.com/univwang/img/main/20221007193640.png)

重启网络

## 设置地址

修改虚拟机的网络，需要保证ip地址在子网内且不与广播地址、网关地址冲突。

开启虚拟机，cd 到 etc/sysconfig/network-scripts/ 目录下，ls 查看配置文件名


编辑 ifcfg-ens33 文件， 命令 vi ifcfg-ens33，按照如下进行修改

```sh
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static			#改为静态

DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=6eab4686-ca67-4038-95d3-f8994a7e294a
DEVICE=ens33
ONBOOT=yes					#改为yes

IPADDR=192.168.195.100		#添加静态IP地址，网段需要和步骤一中的子网IP网段一致
NETMASK=255.255.255.0		#添加子网掩码
PREFIX=24
GATEWAY=192.168.195.2		#添加网关IP地址，和步骤一中的网关IP一致
DNS1=114.114.114.114

```
重启网络systemctl start network.service


## 连接Xshell

可通过用户名和密码连接，如果想要ssh连接需要在Linux中下载ssh服务。