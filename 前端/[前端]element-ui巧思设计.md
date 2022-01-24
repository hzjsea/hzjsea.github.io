---
title: 前端_element-ui巧思设计
date: 2021-05-13 10:03:16
categories:  [vue]
tags: [vue]
---


<!--more-->


# element-ui巧思设计


通过element-ui 中的menu， 跳转路由的过程中传递参数
https://blog.csdn.net/qq_41893334/article/details/114067211


## 关于路径传输的问题

首先element-ui这个框架里面有个path可以传递路由+参数

```html
<el-menu-item 
              :index="item_children.path" 
              :key="item_children.name"
                >{{ item_children.name }}
              </el-menu-item>
      menulist: [
        {
          title: "装配线监控",
          id: 1,
          children: [
            {
              name: "OP130",
              title: "OP130",
              path: "/home/station?work_number=OP130",
              id: "1-1",
            },
            {
              name: "OP140",
              title: "OP140",
              path: "/home/station",
              id: "1-2",
            },
            {
              name: "OP160",
              title: "OP160",
              path: "/home/station?work_number=OP160",
              id: "1-3",
            },
          ],
        },
```
如上通过path + index的方式获取参数并传递，但是有个关于渲染的问题，一般情况下我们会在created中写一个方法来获取路由中的内容，如下
```html
 <span>{{  this.work_number }}</span>
<script>
export default {
  data() {
    return {
        work_number:"",
    };
  },
  methods: {
    get_work_number() {
      this.work_number = this.$route.query.work_number
      alert(this.work_number)
    },
  },
    created() {
    this.get_work_number()
  },
};
</script>
```
但是这样做的问题是，这个worknumber只能渲染一次，最好的做法是在直接在渲染页面的时候让他读取路由中的内容，如下
```html
<span>{{  this.$route.query.work_number }}</span>
```

<div style='font-size:20px;color:red'>但这样做并不提倡，属于懒加载的一种，后面很多请求都要跟着改变</div>

https://blog.csdn.net/weixin_44468194/article/details/106804818


## 表格分页处理
https://segmentfault.com/a/1190000016049838
https://www.jianshu.com/p/70facd19ec55


![](https://noback.upyun.com/2021-05-17-10-56-03.png!)



## vue中动态渲染表头


https://blog.csdn.net/yw00yw/article/details/86609332