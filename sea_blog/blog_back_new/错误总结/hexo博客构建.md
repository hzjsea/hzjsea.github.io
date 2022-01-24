---
title: hexo博客构建
date: 2021-06-29 16:47:52
categories:  [machine]
tags: [hexo]
---


<!--more-->


# hexo博客构建

> hexo本身是一种静态的博客， 静态博客也就是在上传到github上的时候会将你的markdown文件转换成静态页面，也就是html。
类似的静态页面还有hugo(go写的) vuepress(vue.js写的) 另外还有一些动态的博客，数据是放在数据库里面的，一般构建需要一个服务器支撑，用宝塔面板构建，类似的有 wordpress，typecho

span 这里只描述静态博客的创建流程

hexo博客的构建需要几个前提的准备
1. github账号
2. npm环境

## 创建github账号 

> windows安装

下载地址: https://git-scm.com/download/win, 一直next, 最后一步需要添加git到环境中 `Use Git from the Windows Command Prompt`

查看是否安装成功
```bash
git --version
```

> mac or linux 安装
```bash
brew install git
```

> 注册github账号

pass

## 安装npm环境

> windows安装

下载地址: https://nodejs.org/zh-cn/download/ , 下载后一直next 下去就OK

检验是否下载完成,跟java类似
```bash
node -v # 版本号
npm -v  # 版本号
```


> mac or linux 安装
```bash
brew install nodejs
```



## 安装hexo环境

```bash
npm install -g hexo-cli
```

## hexo命令解读
常用的hexo命令有如下几个
```bash
hexo init # 初始化一个blog文件夹， 相当于blok-workspace
hexo new page # 初始化一个自定义的页面
hexo c # 清除缓存 缓存指得是markdown 转换成的 html 文件
hexo s # 本地浏览
hexo g # 生成缓存 markdown -> html
hexo d # 将编译好的内容上传到github上去
```

## 创建一个hexo的工作目录

```bash
hexo init
```

```bash
ls -al => 

drwxr-xr-x   27 alpaca  staff     864 Jun 17 18:35 .deploy_git  # 
-rw-r--r--    1 alpaca  staff      25 Feb 19 11:16 README.md    #
-rw-r--r--    1 alpaca  staff    2760 Jun 17 11:27 _config.yml  # 基本的配置文件
drwxr-xr-x    4 alpaca  staff     128 Jun 16 14:23 index
drwxr-xr-x  364 alpaca  staff   11648 Jun 17 11:18 node_modules # node 软件管理
-rw-r--r--    1 alpaca  staff  304902 Jun 17 11:18 package-lock.json # node 软件管理的配置文件
-rw-r--r--    1 alpaca  staff     957 Jun 17 11:18 package.json
drwxr-xr-x   26 alpaca  staff     832 Jun 17 18:35 public # md编译后html存放的文件
drwxr-xr-x    5 alpaca  staff     160 Apr 25 08:16 scaffolds 
drwxr-xr-x    7 alpaca  staff     224 Jun 17 11:37 source # 存放md文件的地方
drwxr-xr-x   11 alpaca  staff     352 Jun 21 12:02 themes # 存放主题文件的地方
-rw-r--r--    1 alpaca  staff  115294 Jun 17 11:18 yarn.lock # node软件管理的文件 维持版本稳定
```

```bash
ls -al theme => 

drwxr-xr-x  13 alpaca  staff   416 Jun 29 17:07 .git           # git文件
-rw-r--r--   1 alpaca  staff    24 Feb  5 10:50 .gitignore     # git上传时忽略的文件
-rw-r--r--   1 alpaca  staff  1083 Feb  5 10:50 LICENSE        # 
-rw-r--r--   1 alpaca  staff  8407 Apr 25 10:05 _config.yml    # 主题配置文件
drwxr-xr-x   4 alpaca  staff   128 Feb  5 10:50 languages  
drwxr-xr-x   6 alpaca  staff   192 Feb 20 11:55 layout         # 主题页面及css样式
drwxr-xr-x   6 alpaca  staff   192 Feb  5 10:50 source         # 主题静态文件
```




主要原理就是,hexo的发布步骤分为
- hexo s 本地查看
- hexo g 编译md文件 转换成html文件，将他们分类然后放到public文件下面
- hexo d 部署到github上面
我们做的内容就是自己编译一个html文件放在public下面


## 发布自己的项目

## 题外话, 讲一些自己在实际写博客过程中遇到的坑

> 内容是最重要的 篇数是其次

篇数的多少并不重要，内容才是最重要的，可能一开始只是学习的内容，最为笔记而已，但随着不断的深入和修改，后面一篇完整的文章还是很有成就感和值得回味的

> markdown的本质

markdown的本质就是html 你可以理解为markdown定义了一些标识符 比如用 `#` 来区分标题的级别， 用 `>` 来表示下面为引用的内容 , 用 "\`\`\`rust\`\`\`" 来表示代码块, 这些标识符在被转译成html文件后会被重新转换成html对应的模板。
但本质上他也是可以识别html语言的，比如我们要标识一部分内容 字体需要大一些，颜色需要红一点

```bash
<div style='font-size:20px;color:red'>展示</div>
```


<div style='font-size:20px;color:red'>展示</div>