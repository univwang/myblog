---
title: 认识go-micro
date: 2023-03-19 16:42:18
tags: [Go]
excerpt: 认识微服务框架go-micro
categories: Go
index_img: /img/index_img/8.png
banner_img: /img/banner_img/background4.jpg
---
[项目地址](https://github.com/CocaineCong/micro-todoList)


有user和task两个服务，user服务提供用户的增删改查，task服务提供任务的增删改查。网关模块，负责转发请求。增加了rabbitmq，用于异步处理。