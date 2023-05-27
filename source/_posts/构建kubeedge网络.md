---
title: 构建kubeedge网络
date: 2023-05-20 12:42:18
tags: [kubeedge]
excerpt: kubeedge目标一
categories: kubeedge
index_img: /img/index_img/14.png
banner_img: /img/banner_img/background10.jpg
---


使用桥接模式的虚拟机作为云节点，NAT模式的主机作为边缘节点，边缘节点可以通过IP地址访问云节点。

>桥接模式的虚拟机需要指定和宿主机相同的网段、网卡、网关

## Ubuntu20.04配置

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

## centos配置


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

8. 配置hostname

```bash
hostnamectl set-hostname master

vim /etc/hosts


10.0.8.10 master
10.0.8.11 node1
```


## 部署云节点

### 关闭防火墙

```bash
service iptables stop
systemctl stop firewalld.service
ufw disable
iptables -F
```

### 关闭swap

```bash
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

### 安装docker


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

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### 删除污点

```bash
kubectl describe nodes master | grep Taints
kubectl taint node master node-role.kubernetes.io/master-

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


## 部署边缘节点

### 关闭防火墙

```bash
service iptables stop
systemctl stop firewalld.service
ufw disable
iptables -F
```

### 关闭swap

```bash
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

### 安装docker


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



### 安装keadm

```bash
wget https://github.com/kubeedge/kubeedge/releases/download/v1.12.1/keadm-v1.12.1-linux-amd64.tar.gz

tar zxvf keadm-v1.12.1-linux-amd64.tar.gz
chmod +x keadm-v1.12.1-linux-amd64/keadm/keadm
mv keadm-v1.12.1-linux-amd64/keadm/keadm /usr/local/bin/
```

### 安装kubeedge

```bash
keadm join --token=$TOKEN --cloudcore-ipport=$SERVER --kubeedge-version=1.12.1
```


## 安装flannel插件

>Error from server: Get "https://10.0.8.21:10350/containerLogs/kube-flannel/kube-flannel-ds-tbqbr/kube-flannel": dial tcp 10.0.8.21:10350: connect: no route to host

```bash
kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### 边缘节点下载cni

```bash
wget "https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz"
mkdir -p /opt/cni/bin
tar xf cni-plugins-linux-amd64-v0.9.1.tgz -C /opt/cni/bin
```

### 修改flannel文件
[参考1](https://github.com/kubeedge/kubeedge/issues/2677)
[参考2](https://blog.csdn.net/weixin_43168190/article/details/127422527)

### 修改edgecore文件

[参考1](https://github.com/kubeedge/kubeedge/issues/2677)
[参考2](https://github.com/kubeedge/kubeedge/issues/4521)

使用containerd作为容器运行时
[参考3](https://cloud.tencent.com/developer/article/2163912)
[参考4](https://zhuanlan.zhihu.com/p/612051521)

修改完成后，为了让containerd正确的创建网络，安装cni-plugins-linux-amd64，编写cni配置文件
[参考5](https://github.com/kubeedge/kubeedge/issues/4589)

这样应该可以实现跨宿主机容器通信，如果不行的话，删除网桥，delete flannel和测试的pod，重新创建一次。

```bash
输入命令“ip link show”查看当前系统中的网桥信息，找到需要删除的网桥名称。
输入命令“ip link set cni0 down”将该网桥停止工作。
输入命令“brctl delbr cni0”删除该网桥。
```

检查flannel.1的neigh `ip neigh show dev flannel.1`，检查是否正确，如果不正确需要重新apply flannel.yaml

### 最终的结果

[部分配置文件]()
有三个节点，其中一个云节点桥接模式，两个边缘节点nat模式（在一个局域网），边缘节点可以ping通云节点，云节点无法ping通边缘节点。
![](https://raw.githubusercontent.com/univwang/img/master/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-05-23%20213844.png)

### 问题一：为什么flannel实现的只有边缘节点的跨宿主机容器通信，云节点无法通信？

![](https://raw.githubusercontent.com/univwang/img/master/202305232202718.png)
如图，使用`node1`节点的容器ping `node2`节点的容器，tcpdump发现会进行udp的封装然后进行发送，request和reply的目的地址为node1和node2的局域网ip地址


![](https://raw.githubusercontent.com/univwang/img/master/202305232200364.png)
使用使用`node1`节点的容器ping `master`节点的容器，发现无法得到reply，通过tcpdump捕获master节点的数据包，发现reply的目的地址为node1的局域网ip，这是不可达的，因此无法reply

### 构建kube-proxy，边缘节点和云节点都有kube-proxy，尝试解释为什么在云上局域网通过ClusterIP访问此服务无法访问通，然而在边缘局域网通过ClusterIP访问此服务可以访问通（越详细越好）

kube-proxy实现的DNAT的过程，将ClusterIP转化为指定pod的ip+port。pod访问clusterIP的流程，PREROUTING->KUBE-SERVICE->KUBE-SVC-??->KUBE-SEP-??，通对于进入 PREROUTING 链的都转到 KUBE-SERVICES 链进行处理；2、在 KUBE-SERVICES 链，对于访问 clusterIP 的转发到 KUBE-SVC-；
3、访问 KUBE-SVC- 的使用随机数负载均衡，并转发到 KUBE-SEP- 上；4、KUBE-SEP-，设置 mark 标记，进行 DNAT 并转发到具体的 pod 上，如果某个 service 的 endpoints 中没有 pod，那么针对此 service 的请求将会被 drop 掉。这样，通过DNAT，发送到ClusterIp的请求转发到了endpoint，也就是pod的Ip，结合问题一，因为中心云集群和边缘云集群不互通，中心云集群的pod无法访问边缘云集群的pod，因此也无法通过ClusterIp访问边缘节点上的service。



查看svc信息
```bash
root@master:~# kubectl get svc
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
hostname-svc   ClusterIP   10.107.32.57   <none>        12345/TCP   41h
kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP     3d4h
root@master:~# kubectl get pods -owide
NAME                             READY   STATUS    RESTARTS        AGE     IP            NODE     NOMINATED NODE   READINESS GATES
hostname-edge-84cb45ccf4-vhfj2   1/1     Running   0               41h     10.244.3.10   node1    <none>           <none>
nginx-1                          1/1     Running   0               2d18h   10.244.3.9    node1    <none>           <none>
nginx-2                          1/1     Running   0               2d18h   10.244.2.8    node2    <none>           <none>
nginx-3                          1/1     Running   3 (6h38m ago)   2d18h   10.244.0.15   master   <none>           <none>
```