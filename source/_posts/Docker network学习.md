---
title: Docker network学习
date: 2022-10-09 20:40:18
tags: [docker, Linux]
excerpt: 学习Docker network的配置
categories: docker
index_img: /img/index_img/2.png
banner_img: /img/banner_img/background3.jpg
---


## docker中的Network是什么
<p class="note note-primary">容器和主机的网络</p>

Docker是通过Docker Network实现容器之间互访的，它是一个虚拟网，可以是通过桥接（bridge）的方式组建，也可以通过覆盖网（overlay）来实现。通过docker network命令可以创建、查看或删除Docker Network，比如通过docker network ls可以列出当前宿主机上运行的docker网路。

## network的分类
<table><thead><tr><th>Docker网络模式</th><th>配置</th><th>说明</th></tr></thead><tbody><tr><td>host模式</td><td>–net=host</td><td>容器和宿主机共享<code>Network namespace</code>。 <br>容器将不会虚拟出自己的网卡，配置自己的IP 等，而是使用宿主机的IP和端口。</td></tr><tr><td>container模式</td><td>–net=container:NAME_or_ID</td><td>容器和另外一个容器共享<code>Network namespace</code>。<br> kubernetes中的pod就是多个容器共享一个Network namespace。<br> 创建的容器不会创建自己的网卡，配置自己的 IP， 而是和<code>一个指定的容器共享IP、端口范围</code>。</td></tr><tr><td>none模式</td><td>–net=none</td><td>容器有独立的Network namespace，并没有对其进行任何网络设置，如分配veth pair和网桥连接，配置IP等。<br> <code>该模式关闭了容器的网络功能。</code></td></tr><tr><td>bridge模式</td><td>–net=bridge</td><td>(默认模式)。此模式会为每一个容器分配、设置IP等，并将容器连接到一个<code>docker0虚拟网桥</code>，通过<code>docker0网桥</code>以及<code>Iptable nat</code>表配置与宿主机通信</td></tr><tr></tr></tbody></table>


## network的使用

为了解决容器IP地址变化的问题，常常新建bridge网络，可以自动维护主机名和IP地址，实现容器间通过主机名通信。

```sh
$docker network create
$docker network connect
$docker network 1s
$docker network rm
$docker network disconnect
$docker network inspect
```

```sh
$docker run -it --network wangnet --name alpine1 alpine /bin/sh
```