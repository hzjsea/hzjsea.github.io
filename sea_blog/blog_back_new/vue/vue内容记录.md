---
title: vue内容记录
categories:
  - js
tags:
  - js
abbrlink: edf59823
date: 2020-05-22 00:20:49
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [vue内容记录](#vue内容记录)
  - [Vue 报错 Invalid prop: type check failed for prop "value" 问题处理](#vue-报错-invalid-prop-type-check-failed-for-prop-value-问题处理)
  - [vue中为一个元素绑定多个click事件](#vue中为一个元素绑定多个click事件)
  - [vue 首次token登录无法验证 && axios以x-www-form-urlencoded方式提交，后端未收到数据](#vue-首次token登录无法验证--axios以x-www-form-urlencoded方式提交后端未收到数据)
  - [vue mounted方法中使用data变量](#vue-mounted方法中使用data变量)
  - [vue中定时输出和请求接口](#vue中定时输出和请求接口)

<!-- /code_chunk_output -->

<!-- more -->

# vue内容记录
因为vue自己没有系统的学习，有些东西会很容易忘记，这里需要做个总结

<font color='red'>查看尽量用ctrl+F来查找</font>

## Vue 报错 Invalid prop: type check failed for prop "value" 问题处理
![2020-05-22-00-23-05](http://noback.upyun.com/2020-05-22-00-23-05.png)

<font color='red'>解决办法</font>
此处报错重要信息 Invalid prop: type check failed for prop “value”. Expected Array, got String with value “”. 其实已经说明了报错的原因，此处prop 的属性值value 校验失败，（组件）期望的value 类型是Array 数组数据，但目前获取的值是String 类型空字符串""。 prop 作为属性用于Vue 组件，所以确定是哪个组件报错即可，查看prop 的value 格式修改为Expected 类型，比如此处为Array。


## vue中为一个元素绑定多个click事件

`<button @click="func_one();func_two()"></button>`

## vue 首次token登录无法验证 && axios以x-www-form-urlencoded方式提交，后端未收到数据
这是因为提交的格式有问题，一般情况下提交的内容为json的形式，比如你输入账号密码
```bash
{
  "username":"root",
  "password":"password"
}
```
但是这样的token请求是错误的，正确的token请求应该是`x-www-form-urlencoded`这个形式
总而言之就是给 你发送的数据编码，你用 application/json 就不用编码，直接传json数据到后端就行，但是 application/x-www-form-urlencoded 就要编码。

<font color='red'>解决办法 使用qs来对数据进行编码</font>

```js
// # 1. 安装qs
npm install qs -g 

// # 全局引入qs
//在main.js引入qs  
import qs from  'qs'  

//配全局属性配置，在任意组件内可以使用this.$qs获取qs对象 
Vue.prototype.$qs = qs

// 使用qs 
let postData = qs.stringify({
    id: 84,
    name: hzj
}) 

this.axios.post('/api/news/singel.php', postData)
.then((res) = >{
    console.log(res);
}).
catch((err) = >{
    console.log(err, 'error')
})

// result: http://127.0.0.1/api/news/singel.php?id=84&name=hzj
```


## vue mounted方法中使用data变量
vue里面无法用this调用data中的内容，这里要做一些转换，调用methods中的内容同理
```js
//创建一个this的实例
  data() {
    return {
      snmp_data: [],
      links: [],
      data: [],
      timer: "",
    }
  },
  methods: {
    loading_GraphChart(),
    update_graph()
  },
  mounted() {
    this.loading_GraphChart();
    this.timer = setInterval({
      // var that = this
      // var _this = this
      _this.update_graph()
      console.log(_this.data)
    }, 10000);
  },

// 方法2 用箭头函数
data() {
    return {
      snmp_data: [],
      links: [],
      data: [],
      timer: "",
    }
  },
  methods: {
    loading_GraphChart(),
    update_graph()
  },
  mounted() {
    this.loading_GraphChart();
    this.timer = setInterval(()=>{
      this.update_graph()
      console.log(this.data)
    }, 10000);
  },
```

## vue中定时输出和请求接口
```js
// 加载的时候写入计时器
data(){
  return{
    timer:""
  }
}
mounted() {
   this.timer = setInterval(function(){
		//执行内容
	}, 60000);
},

// 销毁的时候清除计时器
beforeDestroy() {
    clearInterval(this.timer);
}
```

