---
title: hexo+fluid搭建博客
date: 2022-08-17 18:23:53
excerpt: 搭建一个个人博客
tags: [博客]
categories: 日常
sticky: 
# index_img: /img/index_img/2.png
banner_img: /img/banner_img/background1.jpg
---
## hexo安装与配置
<p class="note note-primary">hexo有2种_config.yml文件，一个是根目录下的全局的_config.yml，一个是各个theme下的</p>

1. 下载并安装node.js
2. 命令行安装hexo 
3. 配置github
4. 配置SSH免密登录
5. 搭建博客


[参考文章](https://www.bilibili.com/read/cv12633102)

## 评论功能
在国际版leancloud创建应用

![](https://raw.githubusercontent.com/wangqianyv123/image-hosting/main/img/202208172110804.png)

创建Commnet数据库

![](https://raw.githubusercontent.com/wangqianyv123/image-hosting/main/img/202208172109839.png)

修改hexo的主题配置

```yml
valine:
  enable: true
  appid:  your app id
  appkey: your app key
  notify: false # mail notifier , https://github.com/xCss/Valine/wiki
  verify: false # Verification code
  placeholder: just so so
  guest_info: [nick,mail,link]
  pageSize: 10
```

## 图床

>使用又拍云+picgo构建图床。

picgo我们已经很熟悉了，可以快捷上传图片到图床。使用又拍云的云存储服务可以方便的搭建图床。

其中 **设定存储空间名** 为服务名，还需要设定关键的加速域名

<p class="note note-primary">
发现博客上线github后无法正常显示图片，因此又采用了GitHub图床。
</p>


![](https://raw.githubusercontent.com/wangqianyv123/image-hosting/main/img/202208172109946.png)


![](https://raw.githubusercontent.com/wangqianyv123/image-hosting/main/img/202208172108613.jpg)


