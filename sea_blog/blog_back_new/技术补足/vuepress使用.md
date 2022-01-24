---
title: vue-press使用
categories:
  - js
tags:
  - js
abbrlink: acd095ae
date: 2020-07-11 15:08:11
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Vuepress使用记录](#vuepress使用记录)
  - [使用vuepress](#使用vuepress)
  - [目录结构](#目录结构)
  - [vuepress对makrdown的进一步支持](#vuepress对makrdown的进一步支持)
  - [vuepress配置](#vuepress配置)
  - [github部署](#github部署)
  - [配置reco主题](#配置reco主题)

<!-- /code_chunk_output -->
<!-- more -->



# Vuepress使用记录
hexo加载文章有点慢，布局也有点丑，想更新一个新的主题
这里打算使用vuepreess作为新的博客后端



## 使用vuepress
```python
# 创建一个文件夹并且初始化
mkdir blog_test && cd blog_test && yarn init
# 安装vuepress 本地安装
yarn add -D vuepress
# 创建第一篇文档
mkdir docs && echo '# Hello VuePress' > docs/README.md
# 初始化
yarn init
# 启动本地服务器,支持热更新
yarn docs:dev  
```

## 目录结构
```bash
.
├── docs
│   ├── .vuepress (可选的)  # 主要存储的是你的主题配置文件
│   │   ├── components (可选的)
│   │   ├── theme (可选的)
│   │   │   └── Layout.vue
│   │   ├── public (可选的)
│   │   ├── styles (可选的)
│   │   │   ├── index.styl
│   │   │   └── palette.styl
│   │   ├── templates (可选的, 谨慎配置)
│   │   │   ├── dev.html
│   │   │   └── ssr.html
│   │   ├── config.js (可选的)
│   │   └── enhanceApp.js (可选的)
│   │ 
│   ├── README.md
│   ├── guide
│   │   └── README.md
│   └── config.md
│ 
└── package.json

docs/.vuepress: 用于存放全局的配置、组件、静态资源等。
docs/.vuepress/components: 该目录中的 Vue 组件将会被自动注册为全局组件。
docs/.vuepress/theme: 用于存放本地主题。
docs/.vuepress/styles: 用于存放样式相关的文件。
docs/.vuepress/styles/index.styl: 将会被自动应用的全局样式文件，会生成在最终的 CSS 文件结尾，具有比默认样式更高的优先级。
docs/.vuepress/styles/palette.styl: 用于重写默认颜色常量，或者设置新的 stylus 颜色常量。
docs/.vuepress/public: 静态资源目录。
docs/.vuepress/templates: 存储 HTML 模板文件。
docs/.vuepress/templates/dev.html: 用于开发环境的 HTML 模板文件。
docs/.vuepress/templates/ssr.html: 构建时基于 Vue SSR 的 HTML 模板文件。
docs/.vuepress/config.js: 配置文件的入口文件，也可以是 YML 或 toml。
docs/.vuepress/enhanceApp.js: 客户端应用的增强。

```


## vuepress对makrdown的进一步支持

emoji
```bash
:tada: :100:
```
目录
```bash
[[toc]]
```
自定义容器
```bash
::: tip
这是一个提示
:::

::: warning
这是一个警告
:::

::: danger
这是一个危险警告
:::

::: details
这是一个详情块，在 IE / Edge 中不生效
:::
```

代码块中的行高亮
```py
#``` js {4}
export default {
  data () {
    return {
      msg: 'Highlighted!'
    }
  }
}
#```
```


## vuepress配置
配置主要在`docs/.vuepress/config.js`中修改
```bash
# 支持行号显示
module.exports = {
  markdown: {
    lineNumbers: true
  }
}

```


## github部署
部署脚本
```bash
#!/usr/bin/env sh

# 确保脚本抛出遇到的错误
set -e

# 生成静态文件
npm run docs:build

# 进入生成的文件夹
cd docs/.vuepress/dist

# 如果是发布到自定义域名
# echo 'www.example.com' > CNAME

git init
git add -A
git commit -m 'deploy'

# 如果发布到 https://<USERNAME>.github.io
# git push -f git@github.com:<USERNAME>/<USERNAME>.github.io.git master

# 如果发布到 https://<USERNAME>.github.io/<REPO>
# git push -f git@github.com:<USERNAME>/<REPO>.git master:gh-pages

cd -
```

## 配置reco主题

<font color='red'>注意，这里的一些配置是reco中有的 reco是默认主题的超集</font>

```js
// 下载主题 本地
yarn add vuepress-theme-reco

// 修改配置文件 docs/.vuepress/config.js
module.exports = {
  head: [  // 移动端焦点问题
    ['meta', { name: 'viewport', content: 'width=device-width,initial-scale=1,user-scalable=no' }]
  ],
  theme: 'vuepress-theme-reco',
  themeConfig: {
    // 备案
    record: 'ICP 备案文案',
    recordLink: 'ICP 备案指向链接',
    cyberSecurityRecord: '公安部备案文案',
    cyberSecurityLink: '公安部备案指向链接',
    // 项目开始时间，只填写年份
    startYear: '2017',
    // author
    author: 'reco_luan',
    // 导航栏左侧logo
    logo: '/head.png', 
    // 开启搜索
    search:true,
    nav: [  // 添加时间线
    { text: 'TimeLine', link: '/timeline/', icon: 'reco-date' }
    ]
    // 头像
    authorAvatar: '/avatar.png',
    // 博客配置
    blogConfig: {
      category: {
        location: 2,     // 在导航栏菜单中所占的位置，默认2
        text: 'Category' // 默认文案 “分类”
      },
      tag: {
        location: 3,     // 在导航栏菜单中所占的位置，默认3
        text: 'Tag'      // 默认文案 “标签”
      },
    // 密钥
    keyPage: {
      keys: ['32位的 md5 加密密文'], // 1.3.0 版本后需要设置为密文
      color: '#42b983', // 登录页动画球的颜色
      lineColor: '#42b983' // 登录页动画线的颜色
    }，
    //导航菜单中使用主题的内置图标
    { text: 'Tags', link: '/tags/', icon: 'reco-tag' },
    }
  }
}
```
md5密文获取方式
https://vuepress-theme-reco.recoluan.com/views/1.x/password.html#%E5%8A%A0%E5%AF%86%E4%BB%8B%E7%BB%8D

写文章时添加的分类和标签
```bash
---
title: 【vue】跨域解决方案之proxyTable
date: 2017-12-28
categories: [frontEnd,hello]
tags: [vue,world]
keys:  '12345' # 密码
sidebar : auto  # 是否开启侧边栏  auto/true/false
publish : false # 是否发布 false/true
sticky : 
    - type : number # 文章置顶
---
```

