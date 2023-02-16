---
title: 使用Rally进行OpenStack测试
date: 2023-02-13 14:42:18
tags: [OpenStack]
excerpt: 使用Rally进行OpenStack测试
categories: OpenStack
index_img: /img/index_img/4.png
banner_img: /img/banner_img/background31.jpg
---

<!-- 19.png background31.png -->

## 入门

<a class="btn" target="_blank" rel="noopener" style="font-size:20px; color: green" href="https://rally.readthedocs.io/en/latest/index.html" title="github">Rally文档</a>

1. 下载Rally

```bash
pip install rally-openstack
```

2. 激活openstack

```bash
source admin-openrc.sh  
```

3. 创建数据库

```bash
rally db recreate
```

4. 创建deployment

```bash
rally deployment create --fromenv --name=existing

```
![](https://raw.githubusercontent.com/univwang/img/master/202302152156226.png)

5. 检查deployment

```bash
rally deployment check
```

![](https://raw.githubusercontent.com/univwang/img/master/202302152157296.png)

6. 进行测试

这个文件可作为测试样例 rally-openstack/samples/tasks/scenarios/nova/boot-and-delete.yaml 
```bash
git clone https://github.com/openstack/rally-openstack.git

rally task start boot-and-delete.yaml
```


## 进阶