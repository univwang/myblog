---
title: docker学习
date: 2022-10-08 16:40:18
tags: [docker, Linux]
excerpt: 学习docker的使用操作
categories: docker
index_img: /img/index_img/9.png
banner_img: /img/banner_img/background1.jpg
---

## docker的安装

## docker基本操作

### 查看docker信息（version、info）

```sh
# 查看docker版本  
$docker version  
  
# 显示docker系统的信息  
$docker info  
```



### 镜像操作

```sh
# 检索image  
$docker search image_name  
# 下载image  
$docker pull image_name  
# 列出镜像列表; -a, --all=false Show all images; --no-trunc=false Don't truncate output; -q, --quiet=false Only show numeric IDs  
$docker images  
# 删除一个或者多个镜像; -f, --force=false Force; --no-prune=false Do not delete untagged parents  
$docker rmi image_name  
提示Error response from daemon: No such image: zookeeper:latest使用docker rmi ID
# 显示一个镜像的历史; --no-trunc=false Don't truncate output; -q, --quiet=false Only show numeric IDs  
$docker history image_name

# 保存镜像到一个tar包; -o, --output="" Write to an file  
$docker save image_name -o file_path  
# 加载一个tar包格式的镜像; -i, --input="" Read from a tar archive file  
$docker load -i file_path  
  
# 机器a  
$docker save image_name > /home/save.tar  
# 使用scp将save.tar拷到机器b上，然后：  
$docker load < /home/save.tar
```

### 容器操作

```sh
# 在容器中运行"echo"命令，输出"hello word"  
$docker run image_name echo "hello word"  
# 交互式进入容器中  
$docker run -i -t image_name /bin/bash  
# 在容器中安装新的程序  
$docker run image_name apt-get install -y app_name

# 列出当前所有正在运行的container  
$docker ps  
# 列出所有的container  
$docker ps -a  
# 列出最近一次启动的container  
$docker ps -l

# 保存对容器的修改; -a, --author="" Author; -m, --message="" Commit message  
$docker commit ID new_image_name  


# 删除所有容器  
$docker rm `docker ps -a -q`  
  
# 删除单个容器; -f, --force=false; -l, --link=false Remove the specified link and not the underlying container; -v, --volumes=false Remove the volumes associated to the container  
$docker rm Name/ID  
  
# 停止、启动、杀死一个容器  
$docker stop Name/ID  
$docker start Name/ID  
$docker kill Name/ID  
  
# 从一个容器中取日志; -f, --follow=false Follow log output; -t, --timestamps=false Show timestamps  
$docker logs Name/ID  
  
# 列出一个容器里面被改变的文件或者目录，list列表会显示出三种事件，A 增加的，D 删除的，C 被改变的  
$docker diff Name/ID  
  
# 显示一个运行的容器里面的进程信息  
$docker top Name/ID  
  
# 从容器里面拷贝文件/目录到本地一个路径  
$docker cp Name:/container_path to_path  
$docker cp ID:/container_path to_path  
  
# 重启一个正在运行的容器; -t, --time=10 Number of seconds to try to stop for before killing the container, Default=10  
$docker restart Name/ID  
  
# 附加到一个运行的容器上面; --no-stdin=false Do not attach stdin; --sig-proxy=true Proxify all received signal to the process  
$docker attach ID  
# 进入容器  
$docker exec -it ID /bin/bash
```

### 常用操作

```sh
#删除虚悬镜像
$docker image prune

#删除退出状态的镜像
$docker rm $(docker ps -a|grep Exited|awk '{print $1}')

$docker rm $(docker ps -qf status=exited)

```


### 镜像仓库

## docker容器卷

```sh
docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名
```
-v参数，将主机目录挂载到宿主机目录，实现主机和宿主机的共享、联调。
--privileged=true 获取root权限

1. 数据卷可以在容器之间共享。
2. 卷中的更改可以直接实时生效。
3. 数据卷中的更改不会包含在镜像的更新中。
4. 数据卷的生命周期一直持续到没有容器使用它为止。

限制容器只读
![](https://raw.githubusercontent.com/univwang/img/main/20221008170705.png)


容器卷继承，其中父类为容器

```sh
docker run -it  --privileged=true --volumes-from 父类  --name u2 ubuntu
```


## Docker技巧

### docker如何在arm架构上运行amd镜像

```bash
$ vim /etc/docker/daemon.json
{
  "experimental": true
}

#验证buildx版本
docker buildx version

#重启docker ***
systemctl restart docker

#检查是否启用
docker info|grep Experimental


docker run --privileged --rm tonistiigi/binfmt --install all
docker buildx build -t colorization:v1 --platform=linux/amd64 . --load
```
