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

## 配置主机网络连接

与配置vm类似，修改网络适配器的ip、网关

## 设置地址

修改虚拟机的网络，需要保证ip地址在子网内且不与广播地址、网关地址冲突。

