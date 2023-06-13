---
title: 构建kubeedge网络（二)
date: 2023-06-01 12:42:18
tags: [kubeedge]
excerpt: 整理kubeedge的安装过程
categories: kubeedge
index_img: /img/index_img/16.png
banner_img: /img/banner_img/background12.jpg
---


使用桥接模式的虚拟机作为云节点，NAT模式的主机作为边缘节点，边缘节点可以通过IP地址访问云节点，但是云节点无法ping通边缘节点。

>桥接模式的虚拟机需要指定和宿主机相同的网段、网卡、网关

最终得到三个节点，一个云节点，可以被边缘节点访问，两个边缘节点在局域网中，无法被云节点访问。
![](https://raw.githubusercontent.com/univwang/img/master/202306041515401.png)

## 基本配置


### ubuntu
默认是不允许root远程登录的，可以再配置文件开启。

sudo vi /etc/ssh/sshd_config

找到PermitRootLogin without-password 修改为PermitRootLogin yes （本人遇到过）

配置静态ip

```bash
vim /etc/netplan/00-installer-config.yaml 
```

```bash 
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      addresses:
        - 10.0.8.12/24
      gateway4: 10.0.8.254
      nameservers:
          addresses: [8.8.8.8, 114.114.114.144]
  version: 2
```

```bash
sudo netplan apply
```

配置国内镜像源
```bash
# 设置镜像源
sudo vim /etc/apt/sources.list

deb https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

# deb https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

# 更新
sudo apt update
```

关闭防火墙

```bash
service iptables stop
systemctl stop firewalld.service
ufw disable
iptables -F
```

关闭swap

```bash
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

安装docker


```bash
apt-get install docker.io
systemctl enable docker
systemctl start docker

cat <<EOF > daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://63xzb4n0.mirror.aliyuncs.com"]
}
EOF
mv daemon.json /etc/docker/
systemctl daemon-reload
systemctl restart docker
```

修改时区
```bash
date
sudo timedatectl set-timezone Asia/Shanghai
date
sudo systemctl restart rsyslog
```

### centos


1. 配置网络

```bash
TYPE=Ethernet
BOOTPROTO=static
NAME=ens33
DEVICE=ens33
ONBOOT=yes
IPADDR=10.0.8.10
NETMASK=255.255.255.0
GATEWAY=10.0.8.254
DNS1=8.8.8.8
DNS2=114.114.114.114
```

2. 更换阿里源


```bash
yum install -y wget
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
yum -y update
```

3. 安装一些工具

```bash
yum install -y bash-completion vim telnet bridge-utils yum-utils git
```

4. 关闭SElinux和防火墙

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo systemctl stop firewalld
sudo systemctl disable --now firewalld
```

5. 关闭swap

```bash
sudo swapoff -a && sysctl -w vm.swappiness=0
sudo sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
```

6. 网络配置，开启转发机制

```bash
cat >> /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
EOF

modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```

7. 同步时钟


```
yum install -y ntpdate
ntpdate cn.pool.ntp.org
```



## 云节点配置

### 安装docker和k8s

```bash
apt-get install docker.io
systemctl enable docker
systemctl start docker

cat <<EOF > daemon.json
{
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "registry-mirrors": ["https://63xzb4n0.mirror.aliyuncs.com"]
}
EOF
mv daemon.json /etc/docker/
systemctl daemon-reload
systemctl restart docker
```

### 修改时区
```bash
date
sudo timedatectl set-timezone Asia/Shanghai
date
sudo systemctl restart rsyslog
```

### 设置源

```bash
sudo apt-get install -y ca-certificates curl software-properties-common apt-transport-https curl


curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

sudo apt-get update
apt-get install -y kubelet=1.22.13-00 kubeadm=1.22.13-00 kubectl=1.22.13-00
```

### k8s集群初始化

```bash
kubeadm init --image-repository=registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16

```

### 安装网络插件

到[我的配置文件](https://github.com/univwang/kubeedge_configure_file)中找到kube-flannel

**edgecore**需要设置dynamic

### 删除污点

```bash
kubectl describe nodes master | grep Taints
kubectl taint node master node-role.kubernetes.io/master-

```

### 防止proxy调度
因为使用了edgemesh，一般edge端不需要kube-proxy

```bash
kubectl edit daemonset -n kube-system kube-proxy

spec:
  ...
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/edge
                operator: DoesNotExist
    ...            

```
### 安装keadm

```bash
wget https://github.com/kubeedge/kubeedge/releases/download/v1.12.1/keadm-v1.12.1-linux-amd64.tar.gz

tar zxvf keadm-v1.12.1-linux-amd64.tar.gz
chmod +x keadm-v1.12.1-linux-amd64/keadm/keadm
mv keadm-v1.12.1-linux-amd64/keadm/keadm /usr/local/bin/
```

### 安装kubeedge

```bash
keadm init --advertise-address=10.128.156.99 --set iptablesManager.mode="external" --profile version=v1.12.1
```


### 修改cloudcore

```bash
kubectl edit cm cloudcore -n kubeedge

开启
```

## 边缘节点

### 安装kubeedge

1. 云端获取token

```bash
keadm gettoken
```
2. 设置环境变量

```bash
TOKEN=
SERVER=
```

3. 加入集群

```bash
keadm join --token=$TOKEN --cloudcore-ipport=$SERVER --kubeedge-version=1.12.1
```

4. 查看edgecore.service

```bash
systemctl status edgecore.service 
```

### 修改edgecore

参考[我的配置文件](https://github.com/univwang/kubeedge_configure_file)
主要修改容器运行时，cgroup，开启metaServer


### 修改容器运行时

修改edgecore后，配置contianerd

```bash
systemctl enable --now containerd
mkdir /etc/containerd
containerd config default > /etc/containerd/config.toml

mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v0.9.1.tgz



sed -i 's/config\_path\ =.\*/config\_path = \"\/etc\/containerd\/certs.d\"/g' /etc/containerd/config.toml

mkdir /etc/containerd/certs.d/docker.io -p



cat > /etc/containerd/certs.d/docker.io/hosts.toml << EOF

server = "https://docker.io"

[host."https://vh3bm52y.mirror.aliyuncs.com"]

  capabilities = ["pull", "resolve"]

EOF


sed -i 's/SystemdCgroup\ =\ false/SystemdCgroup\ =\ true/g' /etc/containerd/config.toml



并且将 sandbox_image = "registry.k8s.io/pause:3.6"
修改为 sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6"

```

为了contained正常运行，添加cni配置文件，注意需要网段配置和flannel.1保持一直
```bash
cat > /etc/cni/net.d/10-containerd-net.conflist <<EOF
{
     "cniVersion": "0.4.0",
     "name": "containerd-net",
     "plugins": [
       {
         "type": "bridge",
         "bridge": "cni0",
         "isGateway": true,
         "ipMasq": true,
         "promiscMode": true,
         "ipam": {
           "type": "host-local",
           "ranges": [
             [{
               "subnet": "10.244.2.0/24"
             }],
             [{
               "subnet": "2001:db8:4860::/64"
             }]
           ],
           "routes": [
             { "dst": "0.0.0.0/0" },
             { "dst": "::/0" }
           ]
         }
       },
       {
         "type": "portmap",
         "capabilities": {"portMappings": true}
       }
     ]
    }
EOF
```

```bash
systemctl daemon-reload 
systemctl restart containerd
systemctl status containerd
systemctl status edgecore
```

## 问题一：尝试解释为什么在云上局域网通过ClusterIP访问此服务无法访问通，然而在边缘局域网通过ClusterIP访问此服务可以访问通

安装上述步骤部署环境，得到了三个节点的集群，每个节点部署了kube-proxy和flannel，每个节点包含了cni0和flannel.1两个新增的网络设备。

想要通过ClusterIP访问服务，需要通过kube-proxy，kube-proxy实现了DNAT的过程，将ClusterIP转化为指定pod的ip+port。pod访问clusterIP的流程，PREROUTING->KUBE-SERVICE->KUBE-SVC-??->KUBE-SEP-??.

1.对于进入 PREROUTING 链的都转到 KUBE-SERVICES 链进行处理；
2、在 KUBE-SERVICES 链，对于访问 clusterIP 的转发到 KUBE-SVC-；
3、访问 KUBE-SVC- 的使用随机数负载均衡，并转发到 KUBE-SEP- 上；
4、KUBE-SEP-，设置 mark 标记，进行 DNAT 并转发到具体的 pod 上，如果某个 service 的 endpoints 中没有 pod，那么针对此 service 的请求将会被 drop 掉。这样，通过DNAT，发送到ClusterIp的请求转发到了endpoint，也就是pod的Ip。

当前主流的cni插件并不具备跨子网流量转发的能力，本身依赖网络三层可达。flannel基于vxlan技术，实现了pod IP的访问。
因为中心云集群和边缘云集群不互通，中心云集群的pod无法访问边缘云集群的pod，因此也无法通过ClusterIp访问边缘节点上的service。

![](https://raw.githubusercontent.com/univwang/img/master/202305232202718.png)
如图，使用`node1`节点的容器ping `node2`节点的容器，tcpdump发现会进行udp的封装然后进行发送，request和reply的目的地址为node1和node2的局域网ip地址


![](https://raw.githubusercontent.com/univwang/img/master/202305232200364.png)
使用使用`node1`节点的容器ping `master`节点的容器，发现无法得到reply，通过tcpdump捕获master节点的数据包，发现reply的目的地址为node1的局域网ip，这是不可达的，因此无法reply

因为中心云集群和边缘云集群不互通，中心云集群的pod无法访问边缘云集群的pod，因此也无法通过ClusterIp访问边缘节点上的service，边缘集群相互之间可达，可以通过上述流程访问到ClusterIP对应的pod（在在边缘局域网）


## 问题二：尝试解释为什么在云上局域网通过ClusterIP访问此服务可以通（如果仅回答因为装了edgemesh不得分，需要回答清楚原理），然而通过PodIP访问此应用不能通
[全网最全EdgeMesh Q&A手册](https://zhuanlan.zhihu.com/p/585749690)

[KubeEdge云原生边缘计算公开课19-EdgeMesh特性源码解析](https://www.bilibili.com/video/BV1bY4y1y75M/?spm_id_from=333.337.search-card.all.click&vd_source=79e5dcf7c720cad10d7ab9bc065cbe1a)


安装部署edgemesh以后，会在所有节点创建一个edge-agent 的pod，负责拦截Service服务请求到edge agent中处理。edgemesh通过kubeedge边缘侧list-watch的能力，监听service、endpoints等元数据的增删改，再根据service、endpoints的信息创建iptables规则

```bash
iptables -t nat -nvL
```
![](https://raw.githubusercontent.com/univwang/img/master/202306011033356.png)

因为edge-agent通过libp2p的技术实现了tunnel，通过Bootstrap、MDNS和DHT这些组件，实现了agent之间的相互连接，具有跨子网的能力，可将请求转发到指定节点，实现了跨子网通信。 edgemesh将边缘节点之间的通信分为局域网内和跨局域网，局域网内通信：直接访问，跨局域网通信：打孔成功时，代理之间建立直接隧道，否则通过中继转发流量

**然而通过PodIP访问此应用不能通？**

通过PodIP的访问并没有被edgemesh拦截，通常情况下，边缘侧不使用kube-proxy和flannel，pod的ip由每个节点的容器运行时负责分配，无法访问ip也无法访问svc，当使用flannel等网络插件后，可通过唯一的podIP进行应用访问，不过，这依赖于三层网络的互通，与问题一的实质类似，云节点无法跨子网访问边缘容器，因此无法通过podIP访问应用。


## 高阶任务（能提出思路也行，有代码实现可加分）：
>【目标：尝试提前解决问题，同时也快速学习edgemesh原理与CNI原理；通过借助已有开源CNI项目解决问题】通过设计并实现一些方案，让edgemesh具备如下能力：CNI基础功能——容器网段划分（允许直接引用已有的开源CNI插件代码）CNI基础功能——podIP的流量转发（允许直接引用已有的开源CNI插件代码）

学习了flannel cni网络插件的使用,实现了podIP流量的转发.

>【目标：挑战目标】通过设计并实现一些方案，让同一局域网内的podIP流量转发使用CNI插件自身的能力，仅有跨子网的podIP流量转发通过libp2p转发

项目规划中有较详细的思路,其中的关键点1. 使用cni网络插件分配容器网段,edgemesh获取和更新集群节点网络信息 2. 修改iptables规则,拦截不同局域网的podIP流量 3. 使用tunnel打通节点的流量,可以让不同的端口监听不同节点容器网段的流量,然后进行流量转发