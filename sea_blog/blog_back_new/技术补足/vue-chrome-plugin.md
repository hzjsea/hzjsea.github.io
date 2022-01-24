---
title: vue-chrome-plugin
date: 2020-12-22 16:11:34
categories: [vue,]  
tags: [vue,chrome,plugin]
---


<!-- more -->
#  vue-chrome-plugin
使用vue搭建chrome插件
```bash
vue create demo && cd demo
# 安装插件,把下面的复制到package.json当中
"dependencies": {
    "core-js": "^2.6.5",
    "echarts": "^4.9.0",
    "element-ui": "^2.14.1",
    "v-charts": "^1.19.0",
    "vue": "^2.6.10",
    "vue-router": "^3.4.9"
}
vue add chrome-ext 
yarn install
```

> 删除不必要的文件
> vue-cli-plugin-chrome-ext 创建不是 SPA 应用，chrome 插件有很多界面，所以是每一个界面分别编译。
> 比如，src/popup 是一个界面，src/options 也是一个界面。
> 这样，原先很多文件都没用了，我们可以把它们删掉，比如 src/main.ts、src/App.vue。
> 安装完成之后,可以使用`npm run build-watch`


## 笑话网站
https://icanhazdadjoke.com/j/JmjbxkGJBAd

## 参考资料
https://nonlinearthink.github.io/2020/06/24/build-chrome-extension-vuecli4/#%E5%88%9B%E5%BB%BA-vue-cli-4-%E9%A1%B9%E7%9B%AE