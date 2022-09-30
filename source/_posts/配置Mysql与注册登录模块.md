---
title: 配置Mysql与注册登录模块
date: 2022-09-28 11:05:08
tags: [kob,项目,Java]
excerpt: springboot配置数据库和注册登录
categories: kob项目
index_img: /img/index_img/2.png
banner_img: /img/banner_img/background2.jpg
---


## springboot配置数据库

```python
server.port= 5001
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.url=jdbc:mysql://localhost:3306/kob?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

第1行命令指定springboot开启的端口。
第2、3行指定数据库的用户名和密码。
第4行为数据库的地址和时区、读取格式。
第5行指定数据库的类型为MySQL。

## 注册登录模块

