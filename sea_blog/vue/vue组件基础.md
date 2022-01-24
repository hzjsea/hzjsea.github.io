---
title: vue组件基础
categories:
  - code
tags: 
  - vue
  - code
abbrlink: 1fd63f28
date: 2020-01-22 11:31:26
---

vue组件基础

<!-- more -->
// 组件.js
vue.component('xxx',{
    data:function(){
        return {
            count: 0,
            message: "xxx"
        }
    },


    template: '<h1>{{message}}</h1>'
})

// html
调用组件
<div id="app">
    <xxx></xxx>
</div>

//js

new Vue({el: '#app'})


## 组件复用

<div id="app">
    <xxx></xxx>
    <xxx></xxx>
    <xxx></xxx>
    <xxx></xxx>
</div>




vue.component("标签名",{

})


data必须是一个方法，返回参数
这与之前不同
之前
new Vue({
    data:{

    },
})


## 传递数据
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})





## 组件中的父子级

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script> 
    <title>Document</title>
</head>
<body>
    

    <!--静态绑定-->
    <div id="app4">
        <my-c text="我是父级text"></my-c>
    </div>
    
    <!--动态-->
    
    <div id="app5">
        <p>{{text}}</p>
        <input type="text" v-model="text">

        <!--请把他看成标签属性 用v-bind -->
        <child :text="text"></child>
    </div>

    <script>
        
        //父级动态
        new Vue({
            el:"#app5",
            data:{
                text:"我是父级的"
            }
        })

        //子级模版动态跳动
        Vue.component('child',{
            props:["text"],
            template:"<h1>我是子级，这是我调用来的{{text}}<h2>"
        })    
    </script>


    <!--静态-->
    <script>
        Vue.component('my-c',{
            props:["text"],
            template:"<h1>{{text}}</h1>"
        });

        new Vue({
            el:"#app4"
        });
        
    </script>

</body>
</html> 
```


在组件中返回数据可以使用data 以及 props ，但是这两者是有区别的，data只能返回当前组件的值,而props能够返回父组件的值，以上都是props调用父组件的例子，其中组件中的模版所创建出来的内容被称为子组件，分为动态和静态两种

```html
<blog-post title="My journey with Vue"></blog-post>
<blog-post title="Blogging with Vue"></blog-post>
<blog-post title="Why Vue is so fun"></blog-post>

Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
}) 
```
如上也是一种静态调用的方法,blog-post为父组件  h3为子组件

```html
new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})

<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>

```
如上是data调用当前组件的内容




![](https://pic2.zhimg.com/v2-088564a8a2a197078bf97499084b213d_r.jpg)
![](https://pic1.zhimg.com/v2-a0626563a46468cc19a601026e05db54_r.jpg)



## 监听子组件事件




## 全局注册
```html
Vue.component('component-a', { /* ... */ })
Vue.component('component-b', { /* ... */ })
Vue.component('component-c', { /* ... */ })

new Vue({ el: '#app' })

<div id="app">
  <component-a></component-a>
  <component-b></component-b>
  <component-c></component-c>
</div>
```

## 局部注册
```html
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* ... */ } 


new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```
组件之间只能越一级访问，不能跨多级，比如你想要在 B这个子组件中访问A这个子组件，则需要
```html
var ComponentA = { /* ... */ }

var ComponentB = {
  components: {
    'component-a': ComponentA
  },
  // ...
} 
```


在模块系统中局部注册
如果你还在阅读，说明你使用了诸如 Babel 和 webpack 的模块系统。在这些情况下，我们推荐创建一个 components 目录，并将每个组件放置在其各自的文件中。

然后你需要在局部注册之前导入每个你想使用的组件。例如，在一个假设的 ComponentB.js 或 ComponentB.vue 文件中：

import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA,
    ComponentC
  },
  // ...
}
现在 ComponentA 和 ComponentC 都可以在 ComponentB 的模板中使用了。