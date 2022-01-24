---
title: echart相关
date: 2021-03-01 15:59:14
categories:  [echarts]
tags: [echarts]
---


<!--more-->


# echart相关

## echarts卡顿

### dataZoom 卡顿
卡顿主要表现在tooltip显示和拖动dataZoom时
解决办法，关闭小点  关闭动画
option中设置 animation: false, animationDurationUpdate: 0
series中设置 showSymbol: false, hoverAnimation: false

### vue父子组件消失与显示
https://blog.csdn.net/qq_34802416/article/details/89302496

父组件
```js
<template>
  <div class="contain">
    <p>这里是父组件页面</p>
    <button @click="showDialog(true)">显示</button>
    <button @click="showDialog(false)">隐藏</button>
    <!-- 这里使用v-show -->
    <subDialog v-show="dialog_visible"></subDialog>  <!--这里定义变量-->
  </div>
</template>

<script>
// 引入子组件
import subDialog from "@/components/Dialog/subDialog";

export default {
  components: { subDialog },
  data() {
    return {
      // 控制子组件显示与隐藏的标识，类型为Boolean
      dialog_visible: false
    }
  },
  methods: {
    showDialog(visible) {
      this.dialog_visible = visible;
    }
  }
}
</script>

<style lang="scss" scoped>
/* 加入css代码仅是为了使界面更加直观 */
.contain {
  height: 300px;
  width: 400px;
  text-align: center;
  padding: 20px;
  background-color: #9dd3fa;
}
</style>
```
子组件
```python
<template>
  <div class="sub-contain">
    <p>这里是子组件窗口</p>
  </div>
</template>

<script>
export default {
  
};
</script>

<style lang="scss" scoped>
/* 加入css代码仅是为了使界面更加直观 */
.sub-contain {
  height: 150px;
  width: 100%;
  padding: 20px;
  text-align: center;
  margin-top: 20px;
  background-color: #1f6fb5;
  color: #fff;
}
</style>
```

https://blog.csdn.net/qq_34802416/article/details/89302496

### Vue搜索框 动态提示
```js
<el-autocomplete
 v-model="state4"
 :fetch-suggestions="querySearchAsync"
 placeholder="请输入内容"
 @select="((item)=>{handleSelect(item, index)})">
</el-autocomplete>

methods: {
    data(){
        return {
                  hostnames:[{
                      "vaue":"xx",
                      "address":"xx",
                  }],
        }
    }
    querySearchAsync(queryString, cb) {
      let hostnames = this.hostnames
      let results = queryString ? hostnames.filter(this.createStateFilter(queryString)) : hostnames;
      // 调用 callback 返回建议列表的数据
      // clearTimeout(this.timeout);
      // this.timeout = setTimeout(() => {
      //   cb(results);
      // }, 3000 * Math.random());
      cb(results)
    },
    createStateFilter(queryString) {
      return (state) => {
        return (state.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0);
      };
    },
```
必须是state.value 不是value 会报错


## 输入框中不管怎么输入都没有内容显示 
这是因为
```
<el-menu-item>
      <el-row class="autocomplete">
        <el-col :span="24">
          <el-autocomplete
              class="inline-input"
              v-model="state"
              :fetch-suggestions="querySearchAsync"
              placeholder="请输入内容"
              :trigger-on-focus="false"
              clearable
          ></el-autocomplete>
        </el-col>
      </el-row>
    </el-menu-item>
```
这里 v-model 绑定了state 
但是在 
```
data(){
    return {
    state: ''
}
}
```

## Promise { <state>: "fulfilled", <value>: undefined } 
这个是因为变量继承了函数
```
<script>
func fu1(){
    const {data : data } = this.$http.get(url)
     console.log("xxxx")
}
mounted() {
    this. = this.loadAll();
 }
</script>
data = 
```


## vue-echarts中series.data加载的问题
框架如下，现在要把series[0].data 中自动加载数据， 
一开始我是这样写的
```
data(){
   return {
    data: []

    option:{
        series:[{
            data: this.data
}]    
}
}
}

<script>
    const {data : data } = this.$http.get(url)
    this.data = data
</script>


这样写最后会出现 undefine

正确的写法应该是下面这种


<script>
    const {data : data } = this.$http.get(url)
    this.option.series[0].data = data
</script>
```


## npm 安装vue 3.0
先卸载2.0的 npm uninstall vue-cli -g 或 yarn global remove vue-cli
再安装3.0的 npm install -g @vue/cli
https://www.jianshu.com/p/2e640f547925
npm 更新npm版本的时候报错
https://blog.csdn.net/qq_31945977/article/details/81532729

npm audit fix
https://blog.csdn.net/weixin_40817115/article/details/81007774
sudo npm install npm -g


## echart  vue 相关
```
yarn run v1.19.1
$ vue-cli-service serve
internal/modules/cjs/loader.js:800
    throw err;
    ^

Error: Cannot find module './_baseValues'
Require stack:
- /Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/lodash/values.js
- /Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/webpack-merge/lib/index.js
- /Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/@vue/cli-service/lib/Service.js
- /Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/@vue/cli-service/bin/vue-cli-service.js
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:797:15)
    at Function.Module._load (internal/modules/cjs/loader.js:690:27)
    at Module.require (internal/modules/cjs/loader.js:852:19)
    at require (internal/modules/cjs/helpers.js:74:18)
    at Object.<anonymous> (/Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/lodash/values.js:1:18)
    at Module._compile (internal/modules/cjs/loader.js:959:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:995:10)
    at Module.load (internal/modules/cjs/loader.js:815:32)
    at Function.Module._load (internal/modules/cjs/loader.js:727:14)
    at Module.require (internal/modules/cjs/loader.js:852:19) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    '/Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/lodash/values.js',
    '/Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/webpack-merge/lib/index.js',
    '/Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/@vue/cli-service/lib/Service.js',
    '/Users/alpaca/gitProject/Npcsang/flow_diagram/charts/node_modules/@vue/cli-service/bin/vue-cli-service.js'
  ]
}
```
解决方法 https://blog.csdn.net/qq_40994418/article/details/105292120
重新npm install 
原因是 package.json 里面启动命令不是完整路径配置，需要npm install一下