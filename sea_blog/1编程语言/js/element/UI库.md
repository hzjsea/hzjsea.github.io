## 基于vue的UI库


## 安装
npm i element-ui -S


## 往webpack中添加element的插件
vue add element

可能无法在css中引用  &  这是sass的语法。需要安装
npm install --save-dev sass-loader
npm install --save-dev node-loader
修改style
<style lang="sacs"></style>

## 安装vue-router
npm install vue-router



## element中的动画
```bash
<template>
  <div id="test">
      <h1>动画测试</h1>
        <el-button type="button" @click="show =!show">click</el-button>
        <div id="test_div1" style="display:flex">
         <transition name="el-fade-in-linear">
             <div class="transition-box" v-show="show">look me</div>
        </transition>
        <transition name="el-fade-in">
        <div class="transition-box" v-show="show">look me to </div>
        </transition>
        </div>
  </div>
</template>

<script>
export default {
    name:"test",
    data() {
        return {
            show:true
        }
    },
}
</script>

<style>
.transition-box {
    margin-bottom: 10px;
    width: 200px;
    height: 100px;
    border-radius: 4px;
    background-color: #409EFF;
    text-align: center;
    color: #fff;
    padding: 40px 20px;
    box-sizing: border-box;
    margin-right: 20px;
  }
</style> 
```

如上所示 
el-fade-in-linear 和 el-fade-in 提供淡入淡出的效果
el-zoom-in-center el-zoom-in-top el-zoom-in-bottom 提供缩放的效果