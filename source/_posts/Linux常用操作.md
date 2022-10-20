---
title: Linux常用操作
date: 2022-09-20 22:15:18
tags: [Linux]
excerpt: 记录Linux的常用操作
categories: Linux
index_img: /img/index_img/6.png
banner_img: /img/banner_img/background4.jpg
---

## 日常使用

### 查看linux资源占用情况
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。
```sh
top
```

### 查看linux硬盘空间使用

```sh
df -h
```

### 安装vm-tools

```sh
#无图形化界面
sudo apt-get install open-vm-tools
#有图形化界面
sudo apt-get install open-vm-tools-desktop
```
### Linux目录中/和~的区别

/是指根目录：就是所有目录最顶层的目录
~是当前用户的主目录：如果是root用户就是/root/目录， 如果是其他用户就是/home/下用户名命名的用户，该主目录是我们在新建某个Linux账户的时候指定的一个目录。 
比如，root用户的主目录是/root,那么~对应/root,如果我们新建一个用户wsys对应主目录/home/wsys,那么wsys登陆后的目录就是/wsys.

### Linux.开头的文件
点开头的文件是隐藏文件，需要加-a参数才能被显示和列表。如.tumx.conf

## 创建用户并配置sudo权限

```sh
adduser me  # 创建用户
sudo passwd aaa # 设置用户密码
usermod -aG sudo me  # 给用户分配sudo权限

```

给普通用户添加docker命令的权限
```sh
usermod -aG docker yukw
newgrp docker
```

用户名在/etc/passwd这个文件中；

密码在/etc/shadow中

## linux查询端口

netstat 是一个命令行工具，可以提供有关网络连接的信息。

显示所有已开放端口，请使用以下命令：`netstat -anp`

要列出正在侦听的所有 TCP 或 UDP 端口，包括使用端口和套接字状态的服务，请使用以下命令：`netstat -tunlp`

此命令中使用的选项具有以下含义：

-t – 显示 TCP 端口。-u – 显示 UDP 端口。-n – 显示数字地址而不是主机名。-l – 仅显示侦听端口。-p – 显示进程的 PID 和名称。仅当您以 root 或 sudo 用户身份运行命令时，才会显示此信息。

查询指定端口通过grep过滤：`netstat -tnlp | grep :80`

## ssh的使用
```
service sshd start
```
## mysql的安装与启动
安装
```
sudo apt-get install mysql-server
```
启动
```
sudo service mysql start
```
进入数据库后设置用户和密码
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'yourpasswd';

```

## nginx的安装与启动
启动nginx服务
```
service nginx start
```
启动后加载配置后刷新配置
```powershell
nginx -s reload
```
停止nginx服务
```powershell
sudo nginx -s stop
```