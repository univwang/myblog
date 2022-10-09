---
title: Dockerfile学习
date: 2022-10-09 20:40:18
tags: [docker, Linux]
excerpt: 学习DockerFile的编写
categories: docker
index_img: /img/index_img/1.png
banner_img: /img/banner_img/background2.jpg
---


## Dockerfile是什么

<p class="note note-primary">构建docker镜像的文件</p>


## Dockerfile命令

<div class="table-box"><table><thead><tr><th>指令</th><th>语法</th><th>说明</th></tr></thead><tbody><tr><td>FROM</td><td><code>FROM &lt;image&gt;:&lt;tag&gt;</code></td><td>指明构建的新镜像是来自于哪个基础镜像，如果没有选择<code>tag</code>，那么默认值为<code>latest</code></td></tr><tr><td>MAINTAINER</td><td><code>MAINTAINER &lt;name&gt;</code></td><td>指明镜像维护者及其联系方式（一般是邮箱地址）。官方说明已过时，推荐使用<code>LABEL</code></td></tr><tr><td>LABEL</td><td><code>LABEL &lt;key&gt;=&lt;value&gt; ...</code></td><td>功能是为镜像指定标签。也可以使用<code>LABEL</code>来指定镜像作者</td></tr><tr><td>RUN</td><td><code>RUN &lt;command&gt;</code></td><td>构建镜像时运行的<code>Shell</code>命令，比如构建的新镜像中我们想在<code>/usr/local</code>目录下创建一个<code>java</code>目录</td></tr><tr><td>ADD</td><td><code>ADD &lt;src&gt;... &lt;dest&gt;</code></td><td>拷贝文件或目录到镜像中。src 可以是一个本地文件，还可以是一个<code>url</code>。然后自动下载和解压</td></tr><tr><td>COPY</td><td><code>COPY &lt;src&gt;... &lt;dest&gt;</code></td><td>拷贝文件或目录到镜像中。用法同 ADD，只是不支持自动下载和解压</td></tr><tr><td>EXPOSE</td><td><code>EXPOSE &lt;port&gt; [&lt;port&gt;/&lt;protocol&gt;...]</code></td><td>暴露容器运行时的监听端口给外部，可以指定端口是监听 TCP 还是 UDP，如果未指定协议，则默认为 TCP</td></tr><tr><td>ENV</td><td><code>ENV &lt;key&gt;=&lt;value&gt; ...</code></td><td>设置容器内环境变量</td></tr><tr><td>CMD</td><td><code>CMD ["executable","param1","param2"]</code></td><td>启动容器时执行的<code>Shell</code>命令。在<code>Dockerfile</code>中只能有一条<code>CMD</code>指令。如果设置了多条<code>CMD</code>，只有最后一条会生效</td></tr><tr><td>ENTRYPOINT</td><td><code>ENTRYPOINT ["executable", "param1", "param2"]</code></td><td>启动容器时执行的 Shell 命令，同 CMD 类似，不会被 docker run 命令行指定的参数所覆盖，如果设置了多条<code>ENTRYPOINT</code>，只有最后一条会生效</td></tr><tr><td>WORKDIR</td><td><code>WORKDIR param</code></td><td>为 RUN、CMD、ENTRYPOINT 以及 COPY 和 AND 设置工作目录</td></tr><tr><td>VOLUME</td><td><code>VOLUME ["param"]</code></td><td>指定容器挂载点到宿主机自动生成的目录或其他容器。一般的使用场景为需要持久化存储数据时</td></tr></tbody></table></div>