---
title: 电商后台管理系统(vue)-搭建基本环境
categories:
  - code
tags: 
  - vue
  - code
abbrlink: f538ba9d
date: 2020-01-19 12:53:37
---

elementUI电商项目1
<!-- more -->


## 创建基本环境
```bash
# 安装各种内容
yum install nodejs,vue,vue-cli
```

## 使用vue构建项目
```bash
# 创建项目的过程其实有很多种，一般建议用ui可视化创建和命令行创建，这里用可视化创建来举例子
vue ui 
```
![2020-01-19-12-59-20](http://img.noback.top/2020-01-19-12-59-20.png)
这里会给我们创建一个项目管理器，在里面我们可以创建项目和导入项目
![2020-01-19-13-02-22](http://img.noback.top/2020-01-19-13-02-22.png)
点击创建项目,输入完信息以后，选择模版，点击完成。


## 项目运行的几种方式
```bash
npm run 
# 会出现运行项目的几种方式，基础的配置中会出现下面三个
  serve
    vue-cli-service serve
  build
    vue-cli-service build
  lint
    vue-cli-service lint 
```
serve是热更新本地运行
build是打包项目
lint是编码规范检查



## 项目部署
1. github-page部署
因为我已经部署了博客，所以这里就先不用这种部署方式了
url:https://cli.vuejs.org/guide/deployment.html#github-pages
有用到的地方可以看一下
2. docker-nginx部署
这个之后马上就要用到了，docker+nginx的使用。



