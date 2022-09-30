---
title: kob项目创建
date: 2022-09-26 19:45:53
tags: [kob,Java,Vue,项目]
excerpt: kob项目前端创建和后端创建
categories: kob项目
index_img: /img/index_img/3.png
banner_img: /img/banner_img/background6.jpg
---

<p class="note note-primary">
King of bot项目
</p>

## 项目简介
一个贪吃蛇项目，主要有人机对战、人人对战、对战回放的功能。项目前后端分离，前端使用Vue框架，后端使用Spring boot框架。

按照界面，项目分为以下几个部分：
- Pk：匹配界面（微服务），实况直播（websocket），真人Pk
- 对战列表：对战回放
- 排行榜：用户rank
- 用户中心：注册、登录、管理bot


## 创建项目

1. 创建vue项目，需要安装插件jQuery和bootstrap。在router.js中去掉Hash，可删除网址中的#
2. 创建springboot项目。


## 问题

1. vue中的<router-view></router-view>用于页面跳转
2. 如何设置背景

```css
<style>
body {
  background-image: url("./assets/background.png");
  background-size: cover;
}
</style>
```