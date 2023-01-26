---
title: mysql和redis的远程连接和
date: 2023-01-24 14:42:18
tags: [Linux, mysql]
excerpt: 远程连接mysql和redis
categories: Linux
index_img: /img/index_img/17.png
banner_img: /img/banner_img/background24.jpg
---

# mysql

mysql 默认开放端口3306，只能够本地访问，为了远程连接数据库，需要进行一些设置。

## 1. 开放端口

查看3306端口
```powershell
netstat -apn | grep 3306
```

修改bind-address

修改mysql的配置文件/etc/mysql/my.conf，有些版本配置文件地址为/etc/mysql/mysql.conf.d/mysqld.cnf，将bind-address地址设置为无ip访问限制：bind-address=0.0.0.0

重启mysql：

```powershell
service mysql restart
```

## 2. 修改用户权限
修改mysql的mysql数据库user表的Host

```bash
MySQL [(none)]>use mysql;
#查看现有用户,密码及允许连接的主机
MySQL [mysql]> SELECT User, Password, Host FROM user;        
+------+-------------------------------------------+-----------+
| User | Password                                  | Host      |
+------+-------------------------------------------+-----------+
| root | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 | localhost |
| root | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 | 127.0.0.1 |
+------+-------------------------------------------+-----------+
2 rows in set (0.00 sec)

#设置为所有IP都可以访问，比较危险，不建议。
MySQL [mysql]> UPDATE user SET Host='%' where user='root' AND Host='localhost' LIMIT 1;       
MySQL [mysql]> flush privileges;
#再次查看现有用户,密码及允许连接的主机
MySQL [mysql]> SELECT User, Password, Host FROM user;       
+------+-------------------------------------------+-----------+
| User | Password                                  | Host      |
+------+-------------------------------------------+-----------+
| root | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |           |
| root | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 | 127.0.0.1 |
+------+-------------------------------------------+-----------+

```

## 3. 检查防火墙和安全组

检测主机的防火墙和安全组

# redis

远程链接redis首先需要修改redis的配置文件redis.conf,这个文件在redis的目录下,我们使用vim编辑该文件。

1. 注释bind属性
2. 如果你想在后台启动，并不关闭，那么将下面参数设置为yes（开启守护线程）搜索：daemonize
3. 远程连接保护，这里选择关闭搜索：protected-mode

有两种启动方式
1. 直接启动 
   ```bash
   redis-server
   ```
2. 配置文件启动
   ```bash
   redis-server /etc/redis.conf
   ```