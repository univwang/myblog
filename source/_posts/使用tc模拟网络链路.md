---
title: 使用tc模拟网络链路
date: 2023-05-14 12:42:18
tags: [网络]
excerpt: 网络流量控制
categories: 网络
index_img: /img/index_img/12.png
banner_img: /img/banner_img/background8.jpg
---

>在Linux中，Traffic Control（TC）是一种实现流量控制和网络优化的工具。TC包括三个主要组件：qdisc、class和filter。

## 查看规则

- `tc qdisc show`: 查看所有队列规则。
- `tc qdisc show dev <device>`: 查看指定网络设备上的队列规则。
- `tc class show`: 查看所有分类规则。
- `tc class show dev <device>`: 查看指定网络设备上的分类规则。
- `tc filter show`: 查看所有过滤规则。
- `tc filter show dev <device>`: 查看指定网络设备上的过滤规则。

```bash
tc qdisc show
```


## 主要的编写规则

qdisc、class、filter

qdisc（队列规则）是指规定包在网络中排队的算法和策略。在TC中，qdisc用于实现网络中的流量控制和排队机制。可以将qdisc看作是一个管道，它定义了包在管道中如何进入、如何排队等。TC中的qdisc分为两种类型：classful和classless。

class（分类器）是指将网络流量分配到不同的队列中的策略和规则。在TC中，class用于为不同类型的流量提供不同的服务质量。可以将class看作是一个虚拟容器，它包含了qdisc和filter。class可以用来分类流量，并为每一类流量分配带宽、限制延迟等。

filter（过滤器）是指在网络中过滤和转发数据包的策略和规则。在TC中，filter用于根据数据包的源地址、目的地址、协议类型、端口号等特征，将流量分配到正确的class或qdisc中。通过filter，可以实现一些高级的流量控制和路由策略，例如负载均衡、流量限制和分流等。


htb 是一种基于层次 token bucket (HTB) 的队列规则，可以对网络流量进行分级和限制。

## 示例

设置发送到指定ip的链路带宽、时延、丢包、抖动情况。
```bash

tc qdisc add dev <device> root handle 1: htb default 10
tc class add dev <device> parent 1: classid 1:10 htb rate 10mbit delay 10ms loss 10% jitter 5ms
tc filter add dev <device> protocol ip parent 1:0 prio 1 u32 match ip dst <ip_address> flowid 1:10

```

```bash
tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip protocol 6 0xff match ip dport 80 0xffff flowid 1:10
```

在这个命令中，偏移量0xff和0xffff是用来匹配IP头和TCP头中的字段的。IP头和TCP头中的字段位于不同的偏移量上，因此需要不同的偏移量来匹配它们。

0xff是用来匹配IP头中的协议字段。协议字段位于IP头的第9个字节，偏移量为8。由于协议字段是一个8位的字段，因此使用0xff掩码可以将IP头中的协议字段与任何一个协议进行匹配。

0xffff是用来匹配TCP头中的目的端口字段。目的端口字段位于TCP头的第3和第4个字节，偏移量为2。由于目的端口字段是一个16位的字段，因此使用0xffff掩码可以将TCP头中的目的端口字段与任何一个端口进行匹配。