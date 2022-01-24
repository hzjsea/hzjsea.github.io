---
title: 电商后台管理系统(vue)-login模块
categories:
  - code
tags: 
  - vue
  - code
abbrlink: 4dba38f8
date: 2020-01-19 14:46:12
---

login模块
<!-- more -->

## login模块


### 登陆验证功能
element-ui为我们提供了良好的验证功能模块

1. 在el-form中绑定rules
2. 编写规则
3. 对应输入框指定prop传出信息
```js
<template>
  <div id="app">
    <div class="login_box">

      <!--表单登陆页面-->
      <div class="login_form">

        <!-- rules绑定过表单验证规则-->

        <el-form :rules="rules">
          <!--用户名，prop传出信息-->
          <el-form-item label="用户名" prop="username">
            <el-input v-model="form.username"></el-input>
          </el-form-item>
          <!--密码-->
          <el-form-item label="密码" prop="password">
            <el-input type="password"
                      v-model="form.password"></el-input>
          </el-form-item>
          <!--按钮-->
          <el-form-item label=""
                        class="btns">
            <el-button type="primary">登陆</el-button>
            <el-button type="info">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        username: "admin",
        password: "admin"
      },
      // 定义规则
      rules: {
          username: [
            // 空验证和格式验证
            { required: true, message: '错误', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '错误', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在3到5个字符',trigger: 'blur'}
          ],
        }
    };
  }
};
</script>

<style scoped>
#app {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  right: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
.login_form{
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;

}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style> 
```

### 重置表单 resetFields 
使用ref声明对象名称
给标签添加ref
```js
 <el-form :rules="rules" ref="loginform" :model="form"> 
```
写成方法，传递refname到script中
```js
<el-button type="info" @click="resetForms('loginform')">重置</el-button> 
```
重置表单
```js
methods: {
    //重置form表格
    resetForms(loginform) {
      this.$refs[loginform].resetFields();
    } 
```
![2020-01-19-14-56-50](http://img.noback.top/2020-01-19-14-56-50.png)

### 表单预验证 vaildate
给标签添加ref
```js
 <el-form :rules="rules" ref="loginform" :model="form"> 
```
写成方法，传递refname到script中
```js
<el-button type="info" @click="login('loginform')">登陆</el-button> 
```
登陆表单
```js
methods: {
    //登陆form表格
    login(editForm){
      this.$refs[editForm].validate(vaild => {
        console.log(vaild)
      });
    }
```

### 消息提示message
局部引用message组件
```bash
import { Message } from 'element-ui'
Vue.prototype.$message = Message
```
标签绑定属性
```js
<el-button type="primary"
                       @click="login('editForm')"
                       :plain="true">登陆</el-button>
            <el-button type="info"
                       @click="resetForms('editForm')"
                       :plain="true">重置</el-button> 
```
script中编写逻辑
```js
this.$message.success("登陆成功"); 
```


### 前端获取token存放
```js
  methods: {
    //重置form表格
    resetForms(editForm) {
      //根据refs获取实例对象，得到对象以后调用element-ui方法重置字段
      this.$refs[editForm].resetFields();
      this.$message.success("重置成功");
    },
    login(editForm) {
      // 获取对象内容，根据状态码验证是否登陆成功，并返回消息弹窗
      this.$refs[editForm].validate(async vaild => {
        if (!vaild) return;
        const { data: data, status: status } = await this.$http.post(
          "login/",
          this.form
        );
        //这里有个bug 不应该是直接拿的status状态码来看，而是应该应该拿到result返回数据中的status来看，由于前面接口没有写好，所以这里只能根据浏览器返回的status来看
        if (status != 200) return this.$message.error("登陆失败");
        this.$message.success("登陆成功");

        window.sessionStorage.setItem("token", data.token);  //存放token
        this.$router.push("/home");
      });
    }
  }
}; 
```

### 路由导航权限
路由导航守卫控制访问权限
如果用户没有登陆，但是直接通过URL访问特定页面，需要重新导航到登陆页面

原来的路由导航
```js
const routes = [
  { path: '/' , redirect: '/login'},
  { path: '/login' ,component: Login },
  { path: '/home',component: Home}
]
```
修改成路由导航守卫后

```js
//router.js
const routes = [
  { path: '/' , redirect: '/login'},
  { path: '/login' ,component: Login },
  { path: '/home',component: Home}
]

const router = new VueRouter({
  mode: 'history',
  // base: process.env.BASE_URL,
  routes
})

// 挂载路由导航守卫
router.beforeEach((to,form,next) => {
  // to 将要访问的路径
  // from 代表从那个路径跳转过来
  // next 是一个函数,表示放行
  // next() 放行  next('/login') 强制跳转


  if (to.path === '/login') return next()
  //获取token
  const tokenStr = window.sessionStorage.getItem('token')
  if(!tokenStr) return next('/login')
  next()
})

export default router

```

### 登陆与注销
基于token的方式实现退出比较简单，只需要本地的token即可，这样，后续的请求就不会携带token，必须重新生成一个新的token之后才可以访问页面
```bash
<template>
  <div id="app">
    /home
    <el-button type="info"
               @click="Userexit"
               :plain="true">退出</el-button>
  </div>
</template>

<script>
export default {
    methods:{
        Userexit(){
            window.sessionStorage.clear();
            this.$router.push('/login');
            this.$message.success("退出成功");
        }
    }
};
</script>
<style>
</style>
```