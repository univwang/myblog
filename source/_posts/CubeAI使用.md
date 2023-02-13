---
title: CubeAI使用
date: 2023-02-13 14:42:18
tags: [机器学习]
excerpt: CubeAI使用
categories: 机器学习
index_img: /img/index_img/2.png
banner_img: /img/banner_img/background27.jpg
---

[CubeAI](https://openi.pcl.ac.cn/cubeai/cubeai-model-zoo)


## 介绍

CubeAI是基于 ServiceBoot微服务引擎 开发的AI模型推理服务程序，可以进行微服务部署，自动化的提供接口。


## 样例

[手写数字识别](https://openi.pcl.ac.cn/cubeai/handwriting)


**app_core.py**文件是核心文件，其中init中定义AI模型，定义使用的函数，可以自动化的暴露接口。

```python

# -*- coding: utf-8 -*-
import random
from app.model_predict import ModelPredict
from cubetools import image_tools


class AppCore(object):

    def __init__(self):
        self.model_predict = ModelPredict()

    def predict(self, img):
        return self.model_predict.predict(img)

    def gen_demo_img(self):
        return image_tools.bin2url(image_tools.read_img('app/demo_data/{}.jpg'.format(random.randint(0, 9))))


if __name__ == '__main__':
    import os
    os.chdir('..')
    app_core = AppCore()

    for i in range(10):
        result = app_core.predict('app/demo_data/{}.jpg'.format(i))
        print(result)
```


## 使用方法

1. 首先在本地pycharm环境中进行环境搭建，参考Dockerfile下载安装必要的库。

2. 启动服务

```bash
serviceboot
```

3. 使用Dockerfile部署到Docker容器中