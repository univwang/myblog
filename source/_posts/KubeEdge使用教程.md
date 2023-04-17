---
title: KubeEdge使用教程
date: 2023-04-16 12:42:18
tags: [KubeEdge]
excerpt: KubeEdge使用教程
categories: KubeEdge
index_img: /img/index_img/11.png
banner_img: /img/banner_img/background7.jpg
---

## KubeEdge设备管理

![](https://raw.githubusercontent.com/univwang/img/master/202304161024156.png)


## KubeEdge路由管理

![](https://raw.githubusercontent.com/univwang/img/master/202304161122357.png)

eventBus发送到mqtt，serviceBus发送到http


### cloud----->eventbus

>http ->  mqtt

1. 边缘端开启eventbus
2. 云端开启router
3. 编写路由规则
4. 边缘端开启mqtt监听
5. 云端发送http请求


```yaml
# create-ruleEndpoint-eventbus
apiVersion: rules.kubeedge.io/v1
kind: RuleEndpoint
metadata:
    name: my-eventbus
    labels:
        description: test
spec:
    ruleEndpointType: "eventbus"
    properties: {}
```

```yaml
# create-ruleEndpoint-rest
apiVersion: rules.kubeedge.io/v1
kind: RuleEndpoint
metadata:
    name: my-rest
    labels:
        description: test
spec:
    ruleEndpointType: "rest"
    properties: {}
```

```yaml
# create-rule-rest-eventbus
apiVersion: rules.kubeedge.io/v1
kind: Rule
metadata:
    name: my-rule
    labels:
        description: test
spec:
    source: "my-rest"
    sourceResource: {"path":"/test"}
    target: "my-eventbus"
    targetResource: {"topic": "test"}
```

边缘端订阅mqtt
```bash
docker exec -it 1729fe56f022 sh
mosquitto_sub -t 'test' -d
```

云端发送http请求
```bash
curl -X POST -H "Content-Type: application/json" -d '{"message":"test"}' http://10.0.8.10:9443/node1/default/test
```




### cloud----->servicebus

>http -> http
1. 边缘端监听http
2. 云端发送http请求

```yaml
# create-ruleEndpoint-rest
apiVersion: rules.kubeedge.io/v1
kind: RuleEndpoint
metadata:
    name: my-rest
    labels:
        description: test
spec:
    ruleEndpointType: "rest"
    properties: {}
```

```yaml
# create-ruleEndpoint-servicebus
apiVersion: rules.kubeedge.io/v1
kind: RuleEndpoint
metadata:
    name: my-rest
    labels:
        description: test
spec:
    ruleEndpointType: "rest"
    properties: {}
[root@master servicebus-cloud-to-edge]# cat create-ruleEndpoint-servicebus.yaml 
apiVersion: rules.kubeedge.io/v1
kind: RuleEndpoint
metadata:
    name: my-servicebus
    labels:
        description: test
spec:
    ruleEndpointType: "servicebus" 
    properties: {"service_port": "6666"} # 边缘侧监听端口
```

```yaml
# create-rule-rest-servicebus
apiVersion: rules.kubeedge.io/v1
kind: Rule
metadata:
    name: my-rule-rest-servicebus
    labels:
        description: test
spec:
    source: "my-rest"
    sourceResource: {"path": "/source"}
    target: "my-servicebus"
    targetResource: {"path": "/target"}
```


边缘端监听http
```bash
# 6666端口
python3 webserver.py
```

云端发送http请求
```bash
curl -X POST -H "Content-Type: application/json" -d '{"message":"test"}' http://10.0.8.10:9443/node1/default/source
```

### edge_eventbus----->cloud

> mqtt -> http

1. 云端监听http
2. 边缘端发布mqtt消息


```yaml
# create-rule-eventbus-rest
apiVersion: rules.kubeedge.io/v1
kind: Rule
metadata:
    name: my-rule-eventbus-rest
    labels:
        description: test
spec:
    source: "my-eventbus"
    sourceResource: {"topic": "test", "node_name": "node1"}
    target: "my-rest"
    targetResource: {"resource": "http://127.0.0.1:8080/hello"}
```

云端
```bash
python3 webserver.py
```

边缘端
```bash
mosquitto_pub -t 'default/test' -d -m '{"edge_msg": "hhhh"}'
```