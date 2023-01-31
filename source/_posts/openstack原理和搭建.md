---
title: OpenStack原理和搭建
date: 2023-01-27 14:42:18
tags: [OpenStack]
excerpt: OpenStack原理和搭建
categories: OpenStack
index_img: /img/index_img/18.png
banner_img: /img/banner_img/background25.jpg
---

## all-in-one安装流程
### Ubuntu20.04配置

默认是不允许root远程登录的，可以再配置文件开启。

sudo vi /etc/ssh/sshd_config

找到PermitRootLogin without-password 修改为PermitRootLogin yes （本人遇到过）

配置静态ip

```bash
vim /etc/netplan/00-installer-config.yaml 
```

```bash 
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      addresses:
        - 10.0.8.12/24
      gateway4: 10.0.8.254
      nameservers:
          addresses: [8.8.8.8, 114.114.114.144]
  version: 2
```

```bash
sudo netplan apply
```

配置国内镜像源
```bash
# 设置镜像源
sudo vim /etc/apt/sources.list

deb https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

# deb https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

# 更新
sudo apt update
```

配置国内pip(可能不需要)
```bash
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
```

配置github代理

```bash
git config --global http.proxy http://192.168.1.6:7890
```

修改python为python3(可能不需要)
```bash
rm -rf /usr/bin/python 
ln -s /usr/bin/pip3 /usr/bin/pip
ln -s /usr/bin/python3 /usr/bin/python
```

下载devstack
```bash
git clone https://opendev.org/openstack/devstack
```
配置devstack
```bash
vim local.conf

[[local|localrc]]
HOST_IP=10.0.2.15
GIT_BASE=https://github.com.cnpmjs.org

ADMIN_PASSWORD=password
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
```
将两个文件放到devstack/files
```
cirros-0.5.2-x86_64-disk
etcd-v3.3.12-linux-amd64.tar
```

下载安装openstack
```bash
FORCE=yes ./stack.sh
```

![](https://raw.githubusercontent.com/univwang/img/master/20230201011935.png)

### Centos 配置
1. 配置虚拟机镜像、内存、cpu等信息
2. 配置网络
    ```bash
    cat /etc/sysconfig/network-scripts/ifcfg-eth0 
    TYPE=Ethernet
    BOOTPROTO=static
    NAME=eth0
    DEVICE=eth0
    ONBOOT=yes
    IPADDR=10.0.8.10
    NETMASK=255.255.248.0
    GATEWAY=10.0.0.254
    DNS1=223.6.6.6
    DNS2=114.114.114.114
    ```
    ```bash
    systemctl restart network
    ```
    ```bash
    sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config; setenforce 0; systemctl stop firewalld; systemctl disable firewalld
    ```
    ```bash
    systemctl stop NetworkManager; systemctl disable NetworkManager
    ```
    ```bash
    curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
    yum clean all; yum makecache
    ```
    ```bash
    yum install -y bash-completion vim telnet bridge-utils yum-utils
    yum -y update
    reboot
    ```

3. 


## 解决没有认证的问题
```bash
Missing value auth-url required for auth plugin password
```
[OpenStack命令行工具 - 用OpenStack RC文件设置环境变量 - 《Openstack用户指南（简体中文版）》](https://www.bookstack.cn/read/openstack-end-user-guide-simplified-chinese/openstack_command_line_clients-cli_set_environment_variables_using_openstack_rc.md)

[RC文件分析](https://blog.csdn.net/u013469753/article/details/106623962)

1. 下载RC文件
2. 运行获取权限