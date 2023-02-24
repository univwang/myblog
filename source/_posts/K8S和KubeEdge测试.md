---
title: K8S和KubeEdge测试
date: 2023-02-20 14:42:18
tags: [K8S, KubeEdge]
excerpt: 使用kind kubemark CL2等工具进行集群测试
categories: KubeEdge
index_img: /img/index_img/5.png
banner_img: /img/banner_img/background1.jpg
---

<!-- 19.png background31.png -->


## 介绍

### 测试背景

[k8s的测试框架-ClusterLoader2](https://www.bilibili.com/video/BV1ZY411t7ge/?spm_id_from=333.337.search-card.all.click&vd_source=79e5dcf7c720cad10d7ab9bc065cbe1a)


### 测试工具

#### 1.kubemark

Kubemark是K8s官方提供的一个对K8s集群进行性能测试的工具。它可以模拟出一个K8s cluster（Kubemark cluster），不受资源限制，从而能够测试的集群规模比真实集群大的多。这个cluster中master是真实的机器，所有的nodes是Hollow nodes。Hollow nodes执行的还是真实的K8s程序，只是不会调用Docker，因此测试会走一套K8s API调用的完整流程，但是不会真正创建pod。

Kubermark是在模拟的Kubemark cluster上跑E2E测试，从而获得集群的性能指标。Kubermark cluster的测试数据，虽然与真实集群的稍微有点误差，不过可以代表真实集群的数据，因此，可以借用Kubermark，直接在真实集群上跑E2E测试，从而对我们真实集群进行性能测试。

kubemark是阉割版的kubelet，除了不调用CRI接口之外(即不调用Docker，直接返回)，其它行为和kubelet基本一致。

#### 2.kind

k8s集群的组成比较复杂，如果纯手工部署的话易出错且时间成本高。而本文介绍的Kind工具，能够快速的建立起可用的k8s集群，降低初学者的学习门槛。
Kind是Kubernetes In Docker的缩写，顾名思义，看起来是把k8s放到docker的意思。没错，kind创建k8s集群的基本原理就是：提前准备好k8s节点的镜像，通过docker启动容器，来模拟k8s的节点，从而组成完整的k8s集群。需要注意，kind创建的集群仅可用于开发、学习、测试等，不能用于生产环境。

#### 3.ClusterLoader2
ClusterLoader2 (CL2) is a "bring your own yaml" Kubernetes load testing tool being an official K8s scalability and performance testing framework.

[文档](https://github.com/kubernetes/perf-tests/tree/master/clusterloader2)


#### 4. minikube

一个单机生成k8s集群的工具

[官网](https://minikube.sigs.k8s.io/docs/start/)

#### 5. kube-prometheus

此项目是简化prometheus安装的一个开源项目

[官网]()

## 测试入门

### 1.搭建环境

参考: 

[edgemark_setup_guide](https://github.com/kubeedge/kubeedge/blob/master/build/edgemark/edgemark_setup_guide.md)

[clusterloader2](https://github.com/kubernetes/perf-tests/tree/master/clusterloader2)

[clusterloader2-get_start](https://github.com/kubernetes/perf-tests/blob/release-1.22/clusterloader2/docs/GETTING_STARTED.md)

按照[从0构建虚拟机](https://blog-univwang.vercel.app/2023/02/12/%E4%BB%8E0%E6%9E%84%E5%BB%BA%E8%99%9A%E6%8B%9F%E6%9C%BA/)构建虚拟机后

1. 下载perf-tests repository

git 设置代理
```bash
git config --global http.proxy http://192.168.1.102:7890
git config --global https.proxy https://192.168.1.102:7890

```

bash设置代理

```
vim ~/.bashrc
export https_proxy=192.168.1.102:7890

```


```bash
git clone https://github.com/kubernetes/perf-tests.git
cd perf-tests
```

2. 下载GVM(也可以只下载go)

```bash
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
source /root/.gvm/scripts/gvm
```

3. 下载go

```bash
yum install -y bison gcc make

gvm install go1.16 -B
gvm use go1.16
gvm linkthis k8s.io/perf-tests
```

4. 下载kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

curl -LO https://dl.k8s.io/release/v1.21.1/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

```

5. 创建kind集群

```bash
kind create cluster --image=kindest/node:v1.21.1 --name=test-cluster --wait=5m
kubectl get nodes
```
### 2.进行测试


```bash
# /root/perf-tests/clusterloader2/
mkdir test
vim config.yaml
vim deployment.yaml
```

```bash
# /root/perf-tests/clusterloader2/
go run cmd/clusterloader.go --testconfig=test/config.yaml --provider=kind --kubeconfig=${HOME}/.kube/config --v=2
```
![](https://raw.githubusercontent.com/univwang/img/master/202302221206175.png)

如果运行失败，则需要更换镜像`image: opsdockerimage/e2e-test-images-agnhost:2.32`
![](https://raw.githubusercontent.com/univwang/img/master/202302221222035.png)

### 3.测试分析

config
```yaml
name: test

namespace:
  number: 1

tuningSets:
- name: Uniform1qps
  qpsLoad:
    qps: 1

steps:
- name: Start measurements
  measurements:
  - Identifier: PodStartupLatency
    Method: PodStartupLatency
    Params:
      action: start
      labelSelector: group = test-pod
      threshold: 20s
  - Identifier: WaitForControlledPodsRunning
    Method: WaitForControlledPodsRunning
    Params:
      action: start
      apiVersion: apps/v1
      kind: Deployment
      labelSelector: group = test-deployment
      operationTimeout: 120s
- name: Create deployment
  phases:
  - namespaceRange:
      min: 1
      max: 1
    replicasPerNamespace: 1
    tuningSet: Uniform1qps
    objectBundle:
    - basename: test-deployment
      objectTemplatePath: "deployment.yaml"
      templateFillMap:
        Replicas: 10
- name: Wait for pods to be running
  measurements:
  - Identifier: WaitForControlledPodsRunning
    Method: WaitForControlledPodsRunning
    Params:
      action: gather
- name: Measure pod startup latency
  measurements:
  - Identifier: PodStartupLatency
    Method: PodStartupLatency
    Params:
      action: gather
```

deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Name}}
  labels:
    group: test-deployment
spec:
  replicas: {{.Replicas}}
  selector:
    matchLabels:
      group: test-pod
  template:
    metadata:
      labels:
        group: test-pod
    spec:
      containers:
      - image: registry.k8s.io/pause:3.1
        name: {{.Name}}
```


## 测试进阶