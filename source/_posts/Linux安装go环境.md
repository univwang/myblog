---
title: Linux安装go环境.md
date: 2023-05-29 12:42:18
tags: [Go]
excerpt: 安装go环境
categories: Go
index_img: /img/index_img/15.png
banner_img: /img/banner_img/background11.jpg
---
两种方法，yum 安装和 tar.gz包安装。

### yum 安装[#](#yum-安装)

```shell
yum install golang
```

不方便管理

### tar.gz 包安装[#](#targz-包安装)

#### 1.下载安装包[#](#1下载安装包)

```shell
wget https://golang.org/dl/go1.17.1.linux-amd64.tar.gz
# 上面可能需要FQ，使用下面备用的
wget https://golang.google.cn/dl/go1.17.1.linux-amd64.tar.gz
```

#### 2.将下载的包解压到 `/usr/local`目录下[#](#2将下载的包解压到-usrlocal目录下)

```shell
tar -zxf go1.17.1.linux-amd64.tar.gz -C /usr/local
```

#### 3.将 /usr/local/go/bin 添加到环境变量[#](#3将-usrlocalgobin-添加到环境变量)

```shell
export PATH=$PATH:/usr/local/go/bin
```

上述只是临时生效，永久生效需修改 `/etc/profile` 文件

```shell
vim /etc/profile
```

在最后面添加

```bash
export GO111MODULE=on
export GOROOT=/usr/local/go
export GOPATH=/home/gopath
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

如图

[![](https://img2020.cnblogs.com/blog/1540346/202110/1540346-20211002163256891-1293767282.png)](https://img2020.cnblogs.com/blog/1540346/202110/1540346-20211002163256891-1293767282.png)

使环境变量生效

```shell
source /etc/profile
```

#### 4.查看[#](#4查看)

```shell
go version
```

#### 5.设置代理，加速[#](#5设置代理加速)

```none
go env -w GOPROXY=https://goproxy.cn,direct
```

接下来就能正常使用了。