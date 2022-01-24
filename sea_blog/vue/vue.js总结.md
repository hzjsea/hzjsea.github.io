---
title: vuejs基础2
categories:
  - code
tags: 
  - vue
  - code
abbrlink: 6412f439
date: 2020-01-22 11:32:11
---

vuejs基础2

<!-- more -->
https://cn.vuejs.org
线上:https://jsfiddle.net/chrisvfritz/50wL7mdz/

## 添加环境
通过引入的方式引入vue

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script> 
<!--生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```

```html
id="app" --- > //el:#app
v-bind:title = "message"  --- > //绑定标签属性(指令)
{{message}} ---- > data:{  message:"hello vue!"}
var app2  = new Vue({
})  ---> //申明vue的对象
//v-if=true  ----> //条件判断
//v-if:"seen"   new Vue({ el:'#app',data:{ seen:true}   })
//条件循环
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>

var app4 = new Vue({
    el:"#app-4",
    data:{
        todo:[
            {text:'xxx'},
            {text:'xxx'},
            {text:'xxx'},
        ]
    }

})
app4.todo.push({text:'yyy'}) //添加内容


//  v-on:click = "funcname"  自定义点击方法
<div id="app-5">
    <p>{{message}}</p>
    <button v-on:click="reverseMessage">反转消息</button>
</div>

var app5 = new Vue({
    el:"#app-5",
    data:{
        message: 'hello,vue.js!'
    }
    methods:{
        reverseMessage:function () {
            this.message = this.message.split('').reveser().join('')
        }     
        }
    }
})
// 表单双向绑定
<div id="app6">
    <p>{{message}}</p>

    <input v-model="message">
</div>

new app6 = new Vue({
    el:"#app6",
    data:{
        message: "hello world"
    }
})
```

## vue的生命周期
每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM等，但为了让开发者能更好的控制vue的整个流程，以及在特定时候添加特定功能，这时候我们必须要了解的是vue的生命周期
![](https://cn.vuejs.org/images/lifecycle.png)

先了解一下dom的解析流程
浏览器渲染引擎工作流程都差不多，大致分为5步，创建DOM树——创建StyleRules——创建Render树——布局Layout——绘制Painting
1.用HTML分析器，分析分析HTML元素，构建一颗DOM树(标记化和树构建)
2.用CSS分析器，分析CSS文件和元素上的inline样式，生成页面的样式表。
3.将DOM树和样式表，关联起来，构建一颗Render树(这一过程又称为Attachment)。每个DOM节点都有attach方法，接受样式信息，返回一个render对象(又名renderer)。这些render对象最终会被构建成一颗Render树。
4.有了Render树，浏览器开始布局，为每个Render树上的节点确定一个在显示屏上出现的精确坐标。
5.Render树和节点显示坐标都有了，就调用每个节点paint方法，把它们绘制出来。 

DOM树的构建是文档加载完成开始的？构建DOM数是一个渐进过程，为达到更好用户体验，渲染引擎会尽快将内容显示在屏幕上。它不必等到整个HTML文档解析完毕之后才开始构建render数和布局。

Render树是DOM树和CSSOM树构建完毕才开始构建的吗？这三个过程在实际进行的时候又不是完全独立，而是会有交叉。会造成一边加载，一遍解析，一遍渲染的工作现象。


<font color='red'>JS操作真实DOM的代价！用我们传统的开发模式，原生JS或JQ操作DOM时，浏览器会从构建DOM树开始从头到尾执行一遍流程。在一次操作中，我需要更新10个DOM节点，浏览器收到第一个DOM请求后并不知道还有9次更新操作，因此会马上执行流程，最终执行10次。例如，第一次计算完，紧接着下一个DOM更新请求，这个节点的坐标值就变了，前一次计算为无用功。计算DOM节点坐标值等都是白白浪费的性能。即使计算机硬件一直在迭代更新，操作DOM的代价仍旧是昂贵的，频繁操作还是会出现页面卡顿，影响用户体验。为什么需要虚拟DOM，它有什么好处?Web界面由DOM树(树的意思是数据结构)来构建，当其中一部分发生变化时，其实就是对应某个DOM节点发生了变化，虚拟DOM就是为了解决浏览器性能问题而被设计出来的。如前，若一次操作中有10次更新DOM的动作，虚拟DOM不会立即操作DOM，而是将这10次更新的diff内容保存到本地一个JS对象中，最终将这个JS对象一次性attch到DOM树上，再进行后续操作，避免大量无谓的计算量。所以，用JS对象模拟DOM节点的好处是，页面的更新可以先全部反映在JS对象(虚拟DOM)上，操作内存中的JS对象的速度显然要更快，等更新完成后，再将最终的JS对象映射成真实的DOM，交由浏览器去绘制。
</font>

https://www.jianshu.com/p/af0b398602bc



首先我们先来了解一下生命周期的几个阶段
1.beforeCreate
2.created

3.beforeMount
4.mount

5.beforeUpdate
6.update

7.beforeDestroy
8.destroyed


这个生命周期的过程分为三个阶段，初始化   运行中 销毁。


```html
<!-- // index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script> 
</head>
<body>

    <div id="app1">
        <p>{{message}}</p>   
        <input v-model="message">
    </div>

    <script src="./vuetest.js"></script>
</body>
</html>

```

```js
//vuetest.js

new Vue({
    el:"#app1",
    data:{
        message:"helloworld"
    },
    beforeCreate() {
      console.log("----------------------------")
      console.log("xxx我是beforeCreate")
      console.log(this.message+"    ,beforeCreate阶段输出了这条信息")
      console.log("结束了，我输出了message的信息吗")  
    },

    created() {
        console.log("----------------------------")
        console.log("我是create")
        console.log(this.message+"    ,created阶段输出了这条信息")
        console.log("结束了，我输出了message的信息吗")  
    },
    //在created阶段已经能够拿到具体的数据，也可以更改数据，且不会触发update函数
    //但是如果有两个created函数，则后一个重叠前一个
    created(){
        console.log("----------------------------")
        console.log("我是created2代")
        this.message = this.message + "!!!!!!!"
        console.log(this.message)
        console.log("over")
    },

    //虚拟dom创建完成,已经存在数据以及识别到了el(即虚拟dom)
    //可以修改数据，但是已经是最后一次机会了，且不会触发update函数
    beforeMount() {
        console.log("----------------------------")
        console.log("我是beforeMount")
        this.message = this.message + "~~~"
        console.log(this.message)
        console.log("over")
    },
    

    // 这里再次修改数据的话，会触发update函数
    mounted() {
        console.log("--------------------")  
        console.log("Mounted")
        //console.log(this.message+"我触发了update函数，所有渲染的内容也变了"))
        console.log("over")
    },


    //循环阶段必须触发才能进入，触发条件如修改数据，但是在Created和beforeMount两个阶段
    //是不会触发beforeupdate这个阶段的
    //不可以更改数据，否则死循环
    beforeUpdate() {
        console.log("--------------------")  
        console.log('beforeUpdate:重新渲染之前触发') 
        console.log('然后vue的虚拟dom机制会重新构建虚拟dom与上一次的虚拟dom树利用diff算法进行对比之后重新渲染')    
    },

    //这里不能更改数据，否则会陷入死循环
    updated() {
        console.log("--------------------")  
        console.log('updated:数据已经更改完成，dom也重新render完成')
    },

    beforeDestroy() {
        console.log('beforeDestory:销毁前执行（$destroy方法被调用的时候就会执行）,一般在这里善后:清除计时器、清除非指令绑定的事件等等...')
    },

    destroyed() {
        console.log('destroyed:组件的数据绑定、监听...都去掉了,只剩下dom空壳，这里也可以善后')
    },
})
```
输出内容
![](http://img.noback.top/blog/imgvue生命周期.png)

这个生命周期的如图
1.首先创建一个vue实例  new Vue() ,这时候他只是一个空的壳子，无法访问到数据和真实的dom。----> beforeCreate。     创建完实例之后就会执行 beforeCreate,在 beforeCreate中一般不执行任何的操作

2.之后执行 created函数，这个时候已经加载了数据，但是没有创建虚拟dom

3.之后执行 beforeMount函数，这个时候会创建一个虚拟的dom ，同时也加载了数据，你也可以选择修改数据，而且在此之前修改的数据都不会触发update

4.之后执行mounted函数，这个时候如果你修改了数据则会触发update函数

5.update函数需要被触发

6.destroyed 销毁