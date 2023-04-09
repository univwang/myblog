---
title: K8s使用教程
date: 2023-04-09 12:42:18
tags: [K8s]
excerpt: K8s使用教程
categories: K8s
index_img: /img/index_img/10.png
banner_img: /img/banner_img/background6.jpg
---

主要介绍一下K8s的功能和使用

## 1. 查看基本信息

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
kubectl get pod -o wide
kubectl get svc -A
kubectl get deploy -A
kubectl get rs -A

kubectl logs pod_name
kubectl describe pod pod_name

kubectl edit pod pod_name
kubectl edit svc svc_name
```

## 2. yaml文件编写
- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用Tal键，只允许使用空格
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
- ”#” 表示注释，从这个字符一直到行尾，都会被解析器忽略
- 注：- - - 为可选的分隔符 ，当需要在一个文件中定义多个结构的时候需要使用

### 1. 创建一个nginx的pod
```yaml
apiVersion: v1 #版本号
kind: Pod #类型
metadata: #元数据
  name: nginx-pod #名称
spec:
    containers: #容器
    - name: nginx #容器名称
        image: nginx:1.7.9 #镜像
        ports: #端口
        - containerPort: 80 #容器端口
```

### 2. 创建一个deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  # 部署名字
  name: test-k8s
spec:
  replicas: 2
  # 用来查找关联的 Pod，所有标签都匹配才行
  selector:
    matchLabels:
      app: test-k8s
  # 定义 Pod 相关数据
  template:
    metadata:
      labels:
        app: test-k8s
    spec:
      # 定义容器，可以多个
      containers:
      - name: test-k8s # 容器名字
        image: ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v1 # 镜像

```

## K8s网络模型
