---
title: 详解iptables配置
date: 2023-05-15 12:42:18
tags: [网络]
excerpt: 网络匹配处理
categories: 网络
index_img: /img/index_img/13.png
banner_img: /img/banner_img/background9.jpg
---

>iptables是一个在Linux操作系统上使用的防火墙软件。

- 它可以在Linux内核中通过配置iptables规则来控制网络流量的进出。iptables规则可以允许或者拒绝特定的网络数据包，也可以对数据包进行修改，例如改变它们的源地址、目的地址、端口等。
- iptables规则是由一系列的规则链组成的，每个规则链包含一组规则，这些规则描述了如何处理不同类型的数据包。iptables默认包含了三个规则链：INPUT，OUTPUT和FORWARD。分别代表着接收进来的数据包、传送出去的数据包和转发的数据包。
- iptables支持非常灵活的规则匹配方式，可以根据数据包的来源IP地址、目标IP地址、协议类型、端口号等多种参数来匹配规则。同时，iptables也支持高级功能，如网络地址转换（NAT）、端口映射（port forwarding）等。


## 介绍表调用顺序

在Iptables防火墙中包含四种常见的表，分别是filter、nat、mangle、raw。

- filter：负责过滤数据包。
filter表可以管理INPUT、OUTPUT、FORWARD链。
- nat：用于网络地址转换。
nat表可以管理PREROUTING、INPUT、OUTPUT、POSTROUTING链。
- mangle：修改数据包中的内容，例如服务类型、TTL、QOS等等。
mangle表可以管理PREROUTING、INPUT、OUTPUT、POSTROUTING、FORWARD链。
- raw：决定数据包是否被状态跟踪机制处理。
raw表可以管理PREROUTING、OUTPUT链。


raw表（raw table）：首先会处理raw表，用于对数据包进行最早的处理。它主要用于配置连接跟踪（connection tracking）和设置数据包的标记（marking）。

mangle表（mangle table）：接下来是mangle表，用于对数据包进行更细粒度的修改。它可以修改数据包的头部信息、标记数据包或者调整数据包的TTL（Time-to-Live）值。

nat表（nat table）：然后是nat表，用于进行网络地址转换（Network Address Translation，NAT）。它负责修改数据包的源地址、目标地址和端口等，实现端口映射、地址伪装等功能。

filter表（filter table）：最后是filter表，它用于数据包的过滤和防火墙功能。它根据规则集中定义的过滤规则来决定是否接受、拒绝或转发数据包。


## 介绍调用链的顺序


请求报文流入本地要经过的链：

请求报文要进入本机的某个应用程序，首先会到达Iptables防火墙的PREROUTING链，然后又PREROUTING链转发到INPUT链，最后转发到所在的应用程序上。
>PREROUTING—>INPUT—>PROCESS


请求报文从本机流出要经过的链：

请求报文读取完应用程序要从本机流出，首先要经过Iptables的OUTPUT链，然后转发到POSTROUTING链，最后从本机成功流出。
>PROCESS—>OUTPUT—>POSTROUTING

请求报文经过本机向其他主机转发时要经过的链：

请求报文要经过本机向其他的主机进行换发时，首先进入A主机的PREROUTING链，此时不会被转发到INPUT链，因为不是发给本机的请求报文，此时会通过FORWARD链进行转发，然后从A主机的POSTROUTING链流出，最后到达B主机的PREROUTING链。
>PREROUTING—>FORWARD—>POSTROUTING



- PREROUTING链：在数据包进入网络协议栈后，但在路由选择（没有决定从那个端口出去）之前，会首先经过PREROUTING链。在该链中，可以进行一些预处理操作，例如DNAT（目标地址转换）等。

- INPUT链：接下来，经过路由选择后，如果目标地址是本机，数据包将进入INPUT链。在INPUT链中，可以根据规则进行进一步的过滤和处理。如果数据包被接受，则交由上层应用程序进行处理。

- FORWARD链：如果目标地址不是本机，而是其他主机，则数据包将进入FORWARD链。在FORWARD链中，可以根据规则进行转发决策，即将数据包转发到适当的目标地址。

- OUTPUT链：在数据包离开本机之前，会经过OUTPUT链。在该链中，可以对源地址为本机的数据包进行处理。例如，进行源地址伪装、修改端口等操作。

- POSTROUTING链：最后，数据包离开本机之后，会经过POSTROUTING链。在该链中，可以进行一些后处理操作，例如SNAT（源地址转换）等。

## NAT


解析一个示例
```bash
Chain PREROUTING (policy ACCEPT 100 packets, 22509 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  102 22998 KUBE-PORTALS-CONTAINER  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* handle ClusterIPs; NOTE: this must be before the NodePort rules */
    3   172 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL
    2   112 KUBE-NODEPORT-CONTAINER  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL /* handle service NodePorts; NOTE: this must be the last rule in the chain */
```


当数据包进入网卡前，匹配KUBE-PORTALS-CONTAINER规则链，如果匹配失败，匹配DOCKER规则链，如果匹配失败，最后匹配KUBE-NODEPORT-CONTAINER


如果是从docker0网卡进入的，return，如果不是从docker0进入的，匹配目的端口1883和9100（这是docker的端口映射），修改目的地址和端口号为172.17.0.2:1883，进入内核进行转发。

```bash
Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
    1    60 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:1883 to:172.17.0.2:1883
    4   240 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:9100 to:172.17.0.3:9100
```



## 示例

```bash
Chain PREROUTING (policy ACCEPT 100 packets, 22509 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  102 22998 KUBE-PORTALS-CONTAINER  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* handle ClusterIPs; NOTE: this must be before the NodePort rules */
    3   172 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL
    2   112 KUBE-NODEPORT-CONTAINER  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL /* handle service NodePorts; NOTE: this must be the last rule in the chain */

Chain INPUT (policy ACCEPT 100 packets, 22509 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 160 packets, 25260 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  194 28049 KUBE-PORTALS-HOST  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* handle ClusterIPs; NOTE: this must be before the NodePort rules */
    5   325 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL
   13   805 KUBE-NODEPORT-HOST  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL /* handle service NodePorts; NOTE: this must be the last rule in the chain */

Chain POSTROUTING (policy ACCEPT 164 packets, 25500 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  221 29689 KUBE-POSTROUTING  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes postrouting rules */
    0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.2           172.17.0.2           tcp dpt:1883
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.3           172.17.0.3           tcp dpt:9100

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
    1    60 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:1883 to:172.17.0.2:1883
    4   240 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:9100 to:172.17.0.3:9100

Chain KUBE-KUBELET-CANARY (0 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain KUBE-MARK-DROP (0 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK or 0x8000

Chain KUBE-MARK-MASQ (0 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK or 0x4000

Chain KUBE-NODEPORT-CONTAINER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-state-metrics:kube-state-metrics */ tcp dpt:30080 to:169.254.96.16:39777

Chain KUBE-NODEPORT-HOST (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-state-metrics:kube-state-metrics */ tcp dpt:30080 to:169.254.96.16:39777

Chain KUBE-PORTALS-CONTAINER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.104.107.149       /* default/hostname-lb-svc:http-0 */ tcp dpt:12345 to:169.254.96.16:36821
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.110.93.31         /* default/hostname-svc:http-0 */ tcp dpt:12344 to:169.254.96.16:45602
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.109.93.22         /* default/ws-svc:http-0 */ tcp dpt:12348 to:169.254.96.16:37345
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.101.162.238       /* kube-system/metrics-server:https */ tcp dpt:443 to:169.254.96.16:41951
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.63.33          /* default/nginx-service */ tcp dpt:80 to:169.254.96.16:41331
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.101.6.91          /* default/tcp-echo-service:tcp-0 */ tcp dpt:2701 to:169.254.96.16:39180
    0     0 DNAT       udp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns */ udp dpt:53 to:169.254.96.16:42241
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns-tcp */ tcp dpt:53 to:169.254.96.16:38960
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:metrics */ tcp dpt:9153 to:169.254.96.16:33295
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.105.146.234       /* kube-system/kube-state-metrics:kube-state-metrics */ tcp dpt:8080 to:169.254.96.16:39777
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:cloudhub */ tcp dpt:10000 to:169.254.96.16:36576
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:cloudhub-quic */ tcp dpt:10001 to:169.254.96.16:37313
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:cloudhub-https */ tcp dpt:10002 to:169.254.96.16:38559
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:cloudstream */ tcp dpt:10003 to:169.254.96.16:43565
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:tunnelport */ tcp dpt:10004 to:169.254.96.16:38010
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.106.201.46        /* default/nginx-https:http */ tcp dpt:80 to:169.254.96.16:41972
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.106.201.46        /* default/nginx-https:https */ tcp dpt:443 to:169.254.96.16:43127

Chain KUBE-PORTALS-HOST (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.104.107.149       /* default/hostname-lb-svc:http-0 */ tcp dpt:12345 to:169.254.96.16:36821
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.110.93.31         /* default/hostname-svc:http-0 */ tcp dpt:12344 to:169.254.96.16:45602
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.109.93.22         /* default/ws-svc:http-0 */ tcp dpt:12348 to:169.254.96.16:37345
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.101.162.238       /* kube-system/metrics-server:https */ tcp dpt:443 to:169.254.96.16:41951
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.63.33          /* default/nginx-service */ tcp dpt:80 to:169.254.96.16:41331
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.101.6.91          /* default/tcp-echo-service:tcp-0 */ tcp dpt:2701 to:169.254.96.16:39180
    0     0 DNAT       udp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns */ udp dpt:53 to:169.254.96.16:42241
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns-tcp */ tcp dpt:53 to:169.254.96.16:38960
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:metrics */ tcp dpt:9153 to:169.254.96.16:33295
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.105.146.234       /* kube-system/kube-state-metrics:kube-state-metrics */ tcp dpt:8080 to:169.254.96.16:39777
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:cloudhub */ tcp dpt:10000 to:169.254.96.16:36576
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:cloudhub-quic */ tcp dpt:10001 to:169.254.96.16:37313
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:cloudhub-https */ tcp dpt:10002 to:169.254.96.16:38559
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:cloudstream */ tcp dpt:10003 to:169.254.96.16:43565
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.99.208.150        /* kubeedge/cloudcore:tunnelport */ tcp dpt:10004 to:169.254.96.16:38010
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.106.201.46        /* default/nginx-https:http */ tcp dpt:80 to:169.254.96.16:41972
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            10.106.201.46        /* default/nginx-https:https */ tcp dpt:443 to:169.254.96.16:43127

Chain KUBE-POSTROUTING (1 references)
 pkts bytes target     prot opt in     out     source               destination         
  221 29689 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0            mark match ! 0x4000/0x4000
    0     0 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK xor 0x4000
    0     0 MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service traffic requiring SNAT */
```