---
title: KubeEdge原理和搭建
date: 2023-01-29 14:42:18
tags: [KubeEdge]
excerpt: KubeEdge原理和搭建
categories: KubeEdge
index_img: /img/index_img/19.png
banner_img: /img/banner_img/background27.jpg
---


## 1. 使用CentOS-7-x86_64-Minimal-1908创建虚拟机

## 2. 主节点和从节点公共配置

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

## 3. 主节点安装

k8s组件只需要在云侧安装部署

1. 安装所需的软件
```bash
# 添加 k8s 安装源
cat <<EOF > kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
mv kubernetes.repo /etc/yum.repos.d/

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install -y kubelet-1.22.4 kubectl-1.22.4 kubeadm-1.22.4 docker-ce



systemctl enable kubelet
systemctl start kubelet
systemctl enable docker
systemctl start docker


# kubernetes 官方推荐 docker 等使用 systemd 作为 cgroupdriver，否则 kubelet 启动不了
cat <<EOF > daemon.json
{
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "registry-mirrors": ["https://ud6340vz.mirror.aliyuncs.com"]
}
EOF
mv daemon.json /etc/docker/
# 重启生效
systemctl daemon-reload
systemctl restart docker
```
遇到一个问题是kubelet和docker的cgroup不匹配
[问题解决](https://blog.csdn.net/mkdir_/article/details/109189436)
2. k8s初始化

```bash
kubeadm init --image-repository=registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16

# 出现问题
# kubeadm reset 多试几遍

mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```


3. 网络配置

coredns 处于pending状态

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

flannel拉取失败
等一会就行了，会自己重试

![](https://raw.githubusercontent.com/univwang/img/master/20230214163243.png)


4. 安装keadm

```bash
wget https://github.com/kubeedge/kubeedge/releases/download/v1.12.1/keadm-v1.12.1-linux-amd64.tar.gz

tar zxvf keadm-v1.12.1-linux-amd64.tar.gz
chmod +x keadm-v1.12.1-linux-amd64/keadm/keadm
mv keadm-v1.12.1-linux-amd64/keadm/keadm /usr/local/bin/
```

5. 部署安装cloudcore

```bash
keadm init --advertise-address=10.0.8.10 --set iptablesManager.mode="external" --profile version=v1.12.1
# --set cloudCore.modules.dynamicController.enable=true
```
execute keadm command failed: timed out waiting for the condition

[删除污点](http://www.yaotu.net/biancheng/54088.html)

```bash
kubectl describe nodes master | grep Taints
kubectl taint node master node-role.kubernetes.io/master-

```

<!-- [增加容忍度](https://blog.csdn.net/weixin_45566487/article/details/127184033) (未做) -->
[防止proxy调度到边缘节点](https://www.cnblogs.com/ltaodream/p/15200259.html)
<!-- [防止proxy调度](https://segmentfault.com/a/1190000040225049) -->
[修改docker文件](https://blog.csdn.net/douniwanwcy/article/details/123986354)

![](https://raw.githubusercontent.com/univwang/img/master/20230214172244.png)

## 4. 从节点安装


1. 修改主机名
```bash
hostnamectl set-hostname node1
```
2. 安装docker

```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl enable docker
systemctl start docker


# kubernetes 官方推荐 docker 等使用 systemd 作为 cgroupdriver，否则 kubelet 启动不了
cat <<EOF > daemon.json
{
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "registry-mirrors": ["https://ud6340vz.mirror.aliyuncs.com"]
}
EOF
mv daemon.json /etc/docker/

# 重启生效
systemctl daemon-reload
systemctl restart docker
```

3. 云端获取token

```bash
keadm gettoken
```
4. 设置环境变量

```bash
TOKEN=
SERVER=
```

5. 加入集群

```bash
keadm join --token=$TOKEN --cloudcore-ipport=$SERVER --kubeedge-version=1.12.1
```

6. 查看edgecore.service

```bash
systemctl status edgecore.service 
```
出现问题可以重启master节点试试

![](https://raw.githubusercontent.com/univwang/img/master/20230214175020.png)

7. 开启logs和exec

[参考](https://www.bilibili.com/video/BV1z84y1r79h/?spm_id_from=333.337.search-card.all.click&vd_source=79e5dcf7c720cad10d7ab9bc065cbe1a)

云端开启

8. 修改容器运行时

[官方教程](https://kubeedge.io/zh/docs/advanced/cri/)