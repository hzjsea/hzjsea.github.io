---
title: 前端_vue知识
date: 2021-05-02 15:07:26
categories:  [前端]
tags: [vue,前端]
---


<!--more-->


# vue知识


## vue组件知识
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://unpkg.com/vue@next"></script>
</head>
<body>
    <div id="root"></div>
</body>
<script>

    // createapp 创建一个Vue应用，其实就是创建了一个实例，实例中有各种
    // 各样的属性，这个实例叫做app, 最后在将app使用mount方法改在到根组件中
    // 在这里  vm就是这里的根组件
    

    // 比如我们要操作组件里面的data ，我们可以这样调用
    // vm.$data.message = "foobar"

    // 传入的参数表示 这个应用外层的组件该如何展示
    const app = Vue.createApp({
        data(){
            return {
                message:"helloworld"
            }
        },
        template: "<div>{{message}}</div>"
    });
    
    // 组件加载完成之后才能挂载mount
    const vm = app.mount("#root");

</script>
</html>

```
![](https://noback.upyun.com/2021-05-02-15-09-05.png!)

## vue生命周期

生命周期函数，指的是在某一刻会自动执行的函数。

![](https://noback.upyun.com/2021-05-04-15-01-21.png!)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://unpkg.com/vue@next"></script>
</head>
<body>
    <div id="root"></div>
</body>
<script>

    // 生命周期函数，会在某一时刻会自动执行函数

    const app = Vue.createApp({
        data(){
            return {
                message:"helloworld"
            }
        },
        // beforeCreate()
        // 在实例生成之前会自动执行的函数
        beforeCreate() {
            console.log("beforeCreate")
        },
        // created()
        // 在实例创建过程中会自动执行的函数
        created() {
            console.log("create")
        },


        // 中间还会做一步 判断是否存在template
        // 如果存在则会讲template转换成一个方法
        template: "<div>{{message}}</div>",

        
        // beforeMount()
        // 在挂载数据之前会自动执行的函数
        //  在组件内容被渲染到页面之前
        beforeMount() {
            // 在渲染之前获取下root 下面的内容
            console.log(document.getElementById('root').innerHTML, 'beforeMount')
        },

        // 将组件内容渲染后到页面，替换根组件

        // 在组件内容渲染之后，
        mounted() {
            console.log(document.getElementById('root').innerHTML, 'Mounted')
        },
    });
    const vm = app.mount("#root");

</script>
</html>

```
![](https://noback.upyun.com/2021-05-04-15-15-37.png!)

> all in lifetime

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://unpkg.com/vue@next"></script>
</head>
<body>
    <div id="root"></div>
</body>
<script>

    // 生命周期函数，会在某一时刻会自动执行函数

    const app = Vue.createApp({
        data(){
            return {
                message:"helloworld"
            }
        },
        // beforeCreate()
        // 在实例生成之前会自动执行的函数
        beforeCreate() {
            console.log("beforeCreate")
        },
        // created()
        // 在实例创建过程中会自动执行的函数
        created() {
            console.log("create")
        },


        // 中间还会做一步 判断是否存在template
        // 如果存在则会讲template转换成一个方法
        template: "<div>{{message}}</div>",

        
        // beforeMount()
        // 在挂载数据之前会自动执行的函数
        //  在组件内容被渲染到页面之前
        beforeMount() {
            // 在渲染之前获取下root 下面的内容
            console.log(document.getElementById('root').innerHTML, 'beforeMount')
        },

        // 将组件内容渲染后到页面，替换根组件

        // 在组件内容渲染之后，
        mounted() {
            console.log(document.getElementById('root').innerHTML, 'Mounted')
        },
        

        // 当data中的数据发生变化的时候，会自动执行的函数
        beforeUpdate() {
            console.log("beforeUpdate")
        },
        
        // 当data中的数据发生变化，同时页面完成更新后，会自动执行的函数
        updated() {
            console.log("updated")
        },

        // 当vue 应用失效时，自动执行的函数
        //  调用app.unmount 显式的卸载app
        beforeUnmount() {
            console.log(document.getElementById('root').innerHTML, 'beforemounted') // 拿到的root
        },
        // 当vue应用失效时，且dom 完全销毁之时，自动执行的函数
        unmounted() {
            console.log(document.getElementById('root').innerHTML, 'unmounted') // 这里是拿不到root的 
        },
    });
    const vm = app.mount("#root");

</script>
</html>

```


## 好用的方法特性

### 插值表达式 v-html
这个表达式主要的作用就是防止变量被渲染

![](https://noback.upyun.com/2021-05-04-15-40-45.png!)


没有使用插值表达式之前
![](https://noback.upyun.com/2021-05-04-15-41-33.png!)

使用插值表达式之后
![](https://noback.upyun.com/2021-05-04-15-41-07.png!)

### 绑定属性 v-bing or :
```html
template: `<div :title="hellxxxoworld">helloworld<div>`
```

### 动态绑定

```html
<body>

    <div id="root" ></div>
    
</body>

<script>
    const app = Vue.createApp({
        data(){
            return {
                message: "helloworld",
                name: "title",
                event: "click"
            }
        },
            methods: {
                handleClick(){
                    alert("click me")
                }
            },
        template: `
            <div
                @[event]="handleClick"   --> @click="handleClick"
                :[name]="message"  --> :title="message"
            >
                {{message}}
            </div>`
            
    })
    app.mount("#root")
</script> 
```

### 只渲染data中的数据一次 v-once
```html
template: `<div v-once>{{meessage}}<div>`
```

### 判断 v-if
```html
<body>
    <div id="root" ></div>
</body>
<script>
    const app = Vue.createApp({
        data(){
            return {
                message: "helloworld",
                show: true
            }
        },
        template: `<div v-if="show">{{message}}</div>`
    })
    app.mount("#root")
</script>
```

### 事件绑定
```html
<body>
    <div id="root" ></div>
</body>
<script>
    const app = Vue.createApp({
        data(){
            return {
                message: "helloworld",
            }
            methods: {
                handleClick(){
                    alert("click me")       
                }
            },
        },
        template: `<div v-on="handleClick">{{message}}</div>`
    })
    app.mount("#root")
</script>
```


### 阻止跳转默认行为

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>helloworld</title>
    <script src="https://unpkg.com/vue@next"></script>
</head>

<body>

    <div id="root" ></div>
    
</body>

<script>
    const app = Vue.createApp({
        data(){
            return {
                message: "helloworld",
                name: "title",
                event: "click"
            }
        },
            methods: {
                handleClick(e){
                    e.preventDefault();
                }
            },
        template: `<form action="https://www.baiud.com" @click="handleClick">
                <button type="submit">提交</button>
            </form>`
            
    })
    app.mount("#root")
</script>
</html>
```


### 箭头函数中的this问题

```html

<body>
    <div id="root"></div>
</body>
<script>

    // 生命周期函数，会在某一时刻会自动执行函数

    const app = Vue.createApp({
        data(){
            return {
                message:"helloworld"
            }
        },
        methods: {
            // 这样写 this 代表的是这个vm
            handleClick(){
                alert(this.message)
            },

            // 这样写 this代表的是windows
            handleClick2: () =>{
                alert(this.message) // undefined
            }
        },
        template: `<div @click="handleClick">{{message}}</div>`,
    });
    const vm = app.mount("#root");

</script>
```

### 在插值表达式中使用方法

![](https://noback.upyun.com/2021-05-04-17-03-09.png!)

### 计算属性 computed

![](https://noback.upyun.com/2021-05-04-17-05-51.png!)

total的值会自动计算

方法中的内容 只有在页面刷新的时候才会重新计算，但是conputed中的内容，当涉及到的变量数值上发生变化的时候，就会重新计算

### 监听属性

![](https://noback.upyun.com/2021-05-06-16-34-51.png!)

当能够使用computed来实现的时候，推荐使用coputemd;


### 动态样式绑定
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://unpkg.com/vue@next"></script>
    <style>

        .red{
            color: red;
        }
        
        .blue{
            color: royalblue;
        }
    </style>
</head>
<body>
    <div id="root">
    </div>
</body>

<script>
    const app = Vue.createApp({
        data(){
            return {
                font_color: 'red',  // 字符串的形式
                font_color_object: {red: true, green: true},  // 对象的形式
                font_color_arrary: ['red','green',{brown:false}], // 数组的形式
                styleObject:{
                    color:'orange'
                } // 对象的形式 赋值给style
            }
        },
        template:`
            <span  :class="font_color">helloworld</span>
            </br>
            <span  :class="font_color_object">helloworld</span>
            <span  :class="font_color_arrary">helloworld</span>
            <span  :style="styleObject">helloworld</span>
        `
    });
    const vm = app.mount('#root')
</script>
</html>
```

如上这样绑定样式之后，可以通过数组或者object的形式来动态赋值多个样式
![](https://noback.upyun.com/2021-05-06-17-23-13.png!)

### 组件的调用

```html
<body>
   <div id="root"></div>
</body>

<script>
    const app = Vue.createApp({
        data(){
            return {
            }
        },
        template:`
            <demo/>
            `
    });
    
    // 加载组件
    app.component('demo',{
        template: `<div>single</div>`
    })

    const vm = app.mount('#root')
</script>
</html>
```