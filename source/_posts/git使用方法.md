---
title: git使用方法
date: 2022-09-03 00:18:59
tags: [git]
excerpt: 总结一下git的使用方法
categories: 日常
banner_img: /img/banner_img/background3.jpg
index_img: /img/index_img/4.png
---

## 简单使用

## 使用git的ssh远程登录

可使用ssh远程登录主机，不过每次登录需要密码
```powershell
ssh root@IP
```

可配置免密登录
1. 配置.ssh/config
   ```
    Host 别名
        HostName 登录IP
        User 登录用户
    Host 别名
        HostName 登录IP
        User 登录IP
        Port 登录端口（默认是22端口）

   ```
2. ssh-copy-id命令可以把本地的ssh公钥文件安装到远程主机对应的账户下。
3. 通过别名登录。

## 使用git管理多个用户

[参考文章1](https://blog.csdn.net/sinat_30075299/article/details/117587637)

[参考文章2](https://blog.csdn.net/qq_33256692/article/details/126014636)

设置git config, 其中univ是新生成的私钥
![](https://raw.githubusercontent.com/univwang/img/main/202209231635255.png)

使用如下命令连接远程仓库。
```powershell
git remote add origin git@univwang:univwang/blog.git
```