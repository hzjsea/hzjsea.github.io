---
title: 前端_flex布局
date: 2021-05-12 16:16:28
categories:  [vue]
tags: [vue,css,flex]
---


<!--more-->


# css布局

纯写css布局是不大合适的，可以参考一些现有的框架来处理布局上的问题 比如flex布局和grid布局
## grid布局
https://juejin.cn/post/6854573220306255880


## flex布局

参考链接
https://segmentfault.com/a/1190000023801961
https://www.huaweicloud.com/articles/d4c823f087370aaa4b0e068a92c96e0f.html
https://juejin.cn/post/6844903474774147086#heading-2
https://www.huaweicloud.com/articles/7a61d6628e533da5d99d6726da7a0e07.html


发现一个在vue中使用flex布局的问题

```html
<template>
  <div id="station">
      <div id="container">
              <div id="left_top_container">hellowlr</div>
            <div id="left_bottom_container">1</div>
             <div id="right_container">xx</div>
      </div>

  </div>
</template>

<script>
export default {


}
</script>

<style>

/* 最外城box */
#station{ 
    display: flex;
    flex-direction: row;
    flex-wrap:nowrap;
    justify-content:flex-start;
    align-items: flex-start;
}


#left_top_container{
    height: 20%;
    width: 200px;
    background-color: red;
}

#left_bottom_container{
    width: 200px;
    height: 20%;
    background-color: blue;
}

#right_container{
    width: 200px;
    height: 20%;
    background-color: green;
}

</style>
```
最外层box中我去设置flex,按道理他应该是横向排布的,但是并有
![](https://noback.upyun.com/2021-05-12-16-17-57.png!)

我猜测可能是vue中最外层的问题，所以这里要在 station中再设置一个container盒子，对他使用flex布局

```html
<template>
  <div id="station">
      <div id="container">
            <div id="left_top_container">hellowlr</div>
            <div id="left_bottom_container">1</div>
            <div id="right_container">xx</div>
      </div>
  </div>
</template>

<script>
export default {


}
</script>

<style>
#container{ 
    display: flex;
    flex-direction: row;
    flex-wrap:nowrap;
    justify-content:flex-start;
    align-items: flex-start;
}

</style>
```
![](https://noback.upyun.com/2021-05-12-16-19-28.png!)