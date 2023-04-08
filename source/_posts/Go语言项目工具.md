---
title: Go语言项目工具
date: 2023-03-25 16:42:18
tags: [Go]
excerpt: 项目中常用的工具
categories: Go
index_img: /img/index_img/9.png
banner_img: /img/banner_img/background5.jpg
---


## 1. go work

>用于处理go项目多模块开发

[教程](https://juejin.cn/post/7077359097032474631)


## 2. etcd服务注册中心

```bash
docker pull appcelerator/etcd
docker run -d -p 2379:2379 -p 2380:2380 appcelerator/etcd --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://0.0.0.0:2379

```

[etcdkeeper](https://blog.csdn.net/qq_43573718/article/details/119175432#:~:text=ETCD%20keeper%E5%AE%89%E8%A3%85%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B%20%28%E7%AE%80%E6%B4%81%E7%89%88%29%201,1.%E4%B8%8B%E8%BD%BDETCDkeeper%E5%AE%A2%E6%88%B7%E7%AB%AF%202%202.%E5%90%AF%E5%8A%A8ETCD%E6%9C%8D%E5%8A%A1%203%203.%E6%B5%8F%E8%A7%88%E5%99%A8%E8%BE%93%E5%85%A5ETCD%E6%9F%A5%E7%9C%8B%E7%BD%91%E5%9D%80)

## 3. Prometheus监控
