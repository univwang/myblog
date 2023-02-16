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

通过 `rally deployment check` 可以查询当前的服务有哪些

![](https://raw.githubusercontent.com/univwang/img/master/202302161122376.png)

通过 `rally plugin list --name nova` 查询可用于测试的接口

![](https://raw.githubusercontent.com/univwang/img/master/202302161126051.png)

通过 `rally plugin show NovaServers.boot_server_attach_volume_and_list_attachments` 可以查看接口测试的详细信息

![](https://raw.githubusercontent.com/univwang/img/master/202302161127242.png)

```json
{% set flavor_name = flavor_name or "m1.tiny" %}
{% set image_name = "^(cirros.*-disk|TestVM)$" %}
{
    "NovaServers.boot_server_attach_volume_and_list_attachments": [
        {
            "args": {
                "flavor": {
                    "name": "{{flavor_name}}" //使用的内存、cpu组合（VCPUs、RAM、Root Disk）
                },
                "image": {
                    "name": "{{image_name}}" //使用的镜像
                },
                "volume_size": 1, //存储卷的大小
                "volume_num": 2, //存储卷的数量
                "boot_server_kwargs": {},
                "create_volume_kwargs": {}
            },
            "runner": {
                "type": "constant",
                "times": 5,//次数
                "concurrency": 2//并发量
            },
            "context": {
                "users": {
                    "tenants": 2,//租户数
                    "users_per_tenant": 2//每个租户的用户数
                }
            },
            "sla": {
                "failure_rate": {
                    "max": 0//sla设置，失败0%才算成功
                }
            }
        }
    ]
}
```


