## vue-route路由走通
建立route的第一，我们先把整个流通走通了，再在上面完成扩展的功能
1. 首先我们需要安装vue-route
```js
npm install vue-router

import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter) 
```
2. 建立组件components
vue中的所有内容都可以理解为组件的安装
```html
<template>
  <div id="xxx">
      <h1>hello app</h1>
  <p>
      <router-link to="/next"> go to next</router-link>
  </p>

  <router-view></router-view>
  </div>
</template>

<script>
export default {
  name:'xxx'
}
</script>

<style>

</style> 
```
3. 建立vue-route
推荐给route一个单独的js ，然后引用到main.js中这样就能方便后期的整理天际
创建一个route.js,地址为main.js同路径
```bash
touch route.js 
```
引用相关内容，以及componets中的组件
```js
import Vue from 'vue';
import Router from 'vue-router';//引入vue-router
//引入在路由中需要用到的组件
import nt from './components/nextpage' //这里省略了.vue
Vue.use(Router);//加载全局组件Router
export default new Router({
	//mode:'history', //路由模式:默认为hash,如果改为history,则需要后端进行配合
	//base:'/',//基路径:默认值为'/'.如果整个单页应用在/app/下,base就应该设为'/app/'.一般可以写成__dirname,在webpack中配置.
	routes:[{
		path: '/nt', 
		name: 'nt', //给路由命名,设置的name要唯一!
		component: nt//就是第一步import的组件
		}]
})

```
4.在main.js中全局引用router
```js
...
import router from './router';
...


new Vue({
  el:'#app',
  ...
  router,
  render: h => h(App),
}).$mount('#app')

```
5.在app.vue中注册组件
```html
<template>
  <div id="app">
    <xxx></xxx>
  </div>
</template>

<script>
import xxx from './components/nextpage'
export default {
  name: 'app',
  components: {
    xxx,
  }
}
</script>

<style>
</style>

```

## 路由扩展配置
#### 动态路由传值
```js
var routes = [
    {
      path: '/detail/:id',
      name: 'Detail',
      component: Detail
    }
] 

<router-link :to='{name:"Detail",params:{id:x.name}}'>
    ...
</router-link> 
```
当匹配到该路由的时候，参数值灰白设置到this.$route.params中
```html
<template lang='html'>
    <div>
        <h1>水果详情页</h1>
        <h1>{{$route.params.id}}</h1>
    </div>  
</template>
```

#### 标签渲染
route-link 默认被渲染称a标签，如果要修改成其他标签，可以做修改
```html
<route-link to="/nextpage" tag="div"></route-link> 
```
如上就会被渲染成div的形式了，


#### 嵌套路由




## 案例
多个id 动态路由切换，使用监控达到组件的复用