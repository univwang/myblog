---
title: KubeEdge使用教程
date: 2023-04-16 12:42:18
tags: [KubeEdge]
excerpt: KubeEdge使用教程
categories: KubeEdge
index_img: /img/index_img/11.png
banner_img: /img/banner_img/background7.jpg
---

## KubeEdge设备管理

![](https://raw.githubusercontent.com/univwang/img/master/202304161024156.png)


## KubeEdge路由管理

![](https://raw.githubusercontent.com/univwang/img/master/202304161122357.png)

eventBus发送到mqtt，serviceBus发送到http


### cloud----->eventbus

>http ->  mqtt

1. 边缘端开启eventbus
2. 云端开启router
3. 编写路由规则
4. 边缘端开启mqtt监听
5. 云端发送http请求


### cloud----->servicebus

>http -> http
1. 边缘端监听http
2. 云端发送http请求


### edge_eventbus----->cloud

> mqtt -> http

1. 云端监听http
2. 边缘端发布mqtt消息