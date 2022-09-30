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
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。
```
top
```
## 创建用户并配置sudo权限

```sh
adduser me  # 创建用户
usermod -aG sudo me  # 给用户分配sudo权限
```


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