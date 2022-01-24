---
title: vuejs基础
categories:
  - 编程
  - vue
abbrlink: 7636890d
date: 2020-01-22 11:30:50
---


<!-- more -->
# vuejs模版语法
vuejs的可调式模式，在使用的过程中，如果我们想要改变一些值，可以在chrome中打开检查， 然后点击console，里面就可以调试vuejs了


## 插值 v-once
在vue的赋值中我们是这样使用的
```js
{{message}} 

var tt = new Vue({
    el:"#app",
    data:{
        message:"xxx"
    }
})
```
以上是简单的赋值，接下来我们介绍一下插值，仅一次性赋值的模版语法
![](http://img.noback.top/blog/imgvue.png) 
你会发现无论我们怎么修改他的内容，是无法修改的。

## 禁止渲染 v-html
\{\{\}\} 双打括号会把读到的东西渲染成文本，我们可以v-html可以禁止绚烂成文本，以html的形式输出,另外v-html所指定的data数据会覆盖当前便签内的文本
```html
<div id="app2">
        <p>{{message}}</p>
        <p v-html="message"></p>
        <p v-html="message">{{message}}</p>
    </div>
     
```
js
```js
 var tt = new Vue({
    el:"#app2",
    data:{
        message:"<h1>h1大小</h1>"
    }
})
```

## 属性绑定 v-bind  --> 缩写 :
```html
<div id="app3">
        <button v-bind:disabled="flag">{{message}}</button>
    </div>
     
```
```js
var dis = new Vue({
    el:"#app3",
    data:{
        flag:"null",
        message:"look me"
    }
})
```

flag 否定的标识有 null false undefined

## v-if 条件判断

```js
<p v-if="seen">现在你看到我了</p> 
```


## v-on 添加事件  --> 缩写 @
```bash
 <a v-on:click="doSomething">...</a>
```



