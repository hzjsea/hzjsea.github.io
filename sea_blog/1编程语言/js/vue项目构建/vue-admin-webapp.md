# 项目目录树
```bash
tree -L 1 

├── LICENSE
├── README.md
├── babel.config.js
├── node_modules
├── package-lock.json
├── package.json
├── postcss.config.js
├── public                   # 存放文件 favicon.ico index.html 
├── src
└── vue.config.js 

tree -L 1 ./src
./src
├── App.vue                  #### 出口文件
├── api                      #### 1. api 存放各种api接口，也就是通过axios获取内容
├── assets                   # 存放静态资源的地方
├── components               #### 组件
├── layout                   #### 整体框架  
├── main.js                  #### 主要出口文件
├── mixins
├── plugins                  # element 引用
├── router                   #### 路由
├── store                    #### 状态管理
├── style                    # 自定义样式
├── utils
├── vendor
└── views
```

## qs
第一次接触 qs 这个库，是在使用axios时，用于给post方法编码

```js
//qs.parse 把一段格式化的字符串转换为对象格式
let url = 'http://item.taobao.com/item.htm?a=1&b=2&c=&d=xxx&e';
let data = qs.parse(url.split('?')[1]);

// data的结果是
{
    a: 1, 
    b: 2, 
    c: '', 
    d: xxx, 
    e: ''
}


//qs.stringify 将多个参数对象(字典)格式化为一个字符串
let params = { c: 'b', a: 'd' };
qs.stringify(params)

// 结果是
'c=b&a=d'
//也就是通过多个参数+url 组成post

//参数排序
qs.stringify(params, (a,b) => a.localeCompare(b))
// 结果是
'a=b&c=d'
```
https://www.cnblogs.com/small-coder/p/9115972.html


## axios
axios在项目中的两种呈现形式
http://caibaojian.com/fetching-data-with-vue-js.html

api/index.js编写  可复制
```js
import axios from 'axios'
import Qs from 'qs'
import store from '@/store'
import router from '@/router'
import Vue from 'vue'
import { Loading, Message } from 'element-ui' // 引用element-ui的加载和消息提示组件

const $axios = axios.create({
  // 设置超时时间
  timeout: 30000,
  // 基础url，会在请求url中自动添加前置链接
  baseURL: process.env.VUE_APP_BASE_API
})
Vue.prototype.$http = axios // 并发请求
// 在全局请求和响应拦截器中添加请求状态
let loading = null

// 请求拦截器
$axios.interceptors.request.use(
  config => {
    loading = Loading.service({ text: '拼命加载中' })
    const token = store.getters.token
    if (token) {
      config.headers.Authorization = token // 请求头部添加token
    }
    return config
  },
  error => {
    return Promise.reject(error)
  }
)
// 响应拦截器
$axios.interceptors.response.use(
  response => {
    if (loading) {
      loading.close()
    }
    const code = response.status
    if ((code >= 200 && code < 300) || code === 304) {
      return Promise.resolve(response.data)
    } else {
      return Promise.reject(response)
    }
  },
  error => {
    if (loading) {
      loading.close()
    }
    console.log(error)
    if (error.response) {
      switch (error.response.status) {
        case 401:
          // 返回401 清除token信息并跳转到登陆页面
          store.commit('DEL_TOKEN')
          router.replace({
            path: '/login',
            query: {
              redirect: router.currentRoute.fullPath
            }
          })
          break
        case 404:
          Message.error('网络请求不存在')
          break
        default:
          Message.error(error.response.data.message)
      }
    } else {
      // 请求超时或者网络有问题
      if (error.message.includes('timeout')) {
        Message.error('请求超时！请检查网络是否正常')
      } else {
        Message.error('请求失败，请检查网络是否已连接')
      }
    }
    return Promise.reject(error)
  }
)

// get，post请求方法
export default {
  post(url, data) {
    return $axios({
      method: 'post',
      url,
      data: Qs.stringify(data),
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
      }
    })
  },
  get(url, params) {
    return $axios({
      method: 'get',
      url,
      params
    })
  }
}

```

axios拦截器
https://blog.csdn.net/weixin_39378691/article/details/83750056
