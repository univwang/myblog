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

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
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


### 修改docker文件

修改完成后，不知道什么原因，在边缘节点创建pod的时候无法创建cni网桥，然后使用设置的网段创建pod，而是还是使用docker，不过会创建flannel.1，所以需要修改docker的网段。

[参考](https://zhuanlan.zhihu.com/p/403145258)


### 问题：为啥
