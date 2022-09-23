---
title: Anaconda和pytorch的安装
date: 2022-09-21 20:09:11
tags: [机器学习, 安装]
excerpt: 记录安装Anaconda和pytorch的过程
categories: 机器学习
index_img: /img/index_img/5.png
banner_img: /img/banner_img/background5.jpg
---

## Anaconda安装

<a class="btn" target="_blank" rel="noopener" style="font-size:20px; color: green" href="https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/" title="github">清华源Anaconda镜像</a>
安装说明
![](https://raw.githubusercontent.com/univwang/img/main/202209212021946.png)

配置环境变量
> E:\Anaconda（Python需要）
E:\Anaconda\Scripts（conda自带脚本）
E:\Anaconda\Library\mingw-w64\bin（使用C with python的时候） E:\Anaconda\Library\usr\bin
E:\Anaconda\Library\bin（jupyter notebook动态库）

这时可以在cmd窗口使用`conda`和`python`命令
添加清华源
```powershell

conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2

conda config --set show_channel_urls yes
```

## pytorch安装
在anaconda中安装虚拟环境
```powershell

conda create -n pytorch python=3.9.12

```

查看可以安装的CUDA版本
```powershell
C:\Users\11516>nvidia-smi
Wed Sep 21 20:29:40 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 516.94       Driver Version: 516.94       CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name            TCC/WDDM | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ... WDDM  | 00000000:01:00.0 Off |                  N/A |
| N/A   47C    P0    N/A /  N/A |      0MiB /  4096MiB |      1%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

安装CUDA 11.3版本参考[Pytorch官网](https://pytorch.org/)

```powershell
conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch
# 注意去掉 -c pytorch 才能使用清华源
```

测试
```python
import torch
 
torch.cuda.is_available()       # cuda是否可用
 
torch.cuda.current_device()     # 返回当前设备索引
 
torch.cuda.device_count()       # 返回GPU的数量
 
torch.cuda.get_device_name(0)   # 返回gpu名字，设备索引默认从0开始

torch.cuda.version              # 返回cuda的版本
```

## jupyter notebook使用

可使用vscode编写jupyter notebook

可能遇到的几个问题

1. [如何添加没有出现的kernel](https://blog.csdn.net/songyuc/article/details/107669134)
2. [解决jupyter notebook无法连接内核问题](https://blog.csdn.net/weixin_46262216/article/details/120657115)
3. [Jupyter 无法启动虚拟环境的问题](https://tyne.cc/1035.html)
4. [运行Pytorch时出现Original error](https://blog.csdn.net/wuwenhuanwu/article/details/115877215)


接下来开始Pytorch吧！