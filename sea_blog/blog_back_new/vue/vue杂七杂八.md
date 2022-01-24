---
title: vue杂七杂八
date: 2021-03-25 11:03:57
categories:  [vue]
tags: [vue]
---


<!--more-->


# vue杂七杂八

记录一些vue中实用的方法


## v-for控制el-row以及el-col卡片的布局

案例如下
```js
<!DOCTYPE html>
<html xmlns:border-radius="http://www.w3.org/1999/xhtml" xmlns:margin="http://www.w3.org/1999/xhtml"

xmlns:color="http://www.w3.org/1999/xhtml">
<head>

<meta charset="UTF-8">
<link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/element-ui/lib/index.js"></script>
</head>
<body>
<div id="app" >
<div id="pro_form" style="position: relative;width: 100%;height: 1000px;">
    <div style="position: relative;top: 100px;left:130px;">
        <el-row>
            <!--就改这里一行-->
            <el-col :span="4" v-for="(project,index) in allprojects" :key="index" :offset="1" style="margin-bottom:40px">
                <el-card :body-style="{ padding: '0px', height:'360px'}" shadow="hover" style="width: 260px;height: 320px;">
                    <div style="padding: 6px;height: 310px;">
                        <div>
                            <div><font size="5">{{project.pcname}}</font></div>
                            <div style="position: relative;top: 15px;text-align: center;">{{project.pname}}</div>
                        </div>
                        <div style="position: relative;top: 30px;">
                            <img src="../images/project/prohead.jpg" class="image">
                            <div style="position: relative;top: 10px;left: 66px;"><i class="el-icon-time"></i>{{project.pdatesave}}</div>
                        </div>
                        <div style="position: relative;top: 45px;">
                            &nbsp;&nbsp;<i class="el-icon-view"></i><span>{{project.ppageview}}</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                            <el-button type="text"><font size="4">查看</font></el-button>
                        </div>
                    </div>
                </el-card>
            </el-col>
        </el-row>
    </div>
</div>
</div>
</body>
<style>

html,body{
    margin:0px;
    height:100%;
}
</style>
<script>

var vue=new Vue({
    el: '#app',
    data: {
        allprojects:[
            {
                pid:'123',
                pname:'pname',
                pcname:'pcname',
                pdatesave:'2018-4-9 11:11:11',
                ppageview:10
            },
            {
                pid:'123',
                pname:'pname',
                pcname:'pcname',
                pdatesave:'2018-4-9 11:11:11',
                ppageview:10
            },
            {
                pid:'123',
                pname:'pname',
                pcname:'pcname',
                pdatesave:'2018-4-9 11:11:11',
                ppageview:10
            },
            {
                pid:'123',
                pname:'pname',
                pcname:'pcname',
                pdatesave:'2018-4-9 11:11:11',
                ppageview:10
            },
            {
                pid:'123',
                pname:'pname',
                pcname:'pcname',
                pdatesave:'2018-4-9 11:11:11',
                ppageview:10
            },
            {
                pid:'123',
                pname:'pname',
                pcname:'pcname',
                pdatesave:'2018-4-9 11:11:11',
                ppageview:10
            }
        ]
    },
    methods: {//这里用于定义方法
    },
    computed: {//计算属性
    }
})
</script>
</html>
```
主要就是在el-row 下面的el-col实用v-for


## js中四种遍历数组的方法
https://blog.fundebug.com/2019/03/11/4-ways-to-loop-array-inj-javascript/
```js
arr.forEach((v, i) => console.log(v));
```
附上自己写的一串代码 
```js
 loading_NqaChart() {
                this.chartList.forEach((item) =>{
                    console.log(1)
                    var charts_name = item["id"]
                    this.myChart = this.$echarts.init(document.getElementById(charts_name));
                    this.myChart.setOption(this.option, true);
                    var option_keywards = item["option_keywards"]
                    var ns = this.getLocalTime(option_keywards.date_list)
                    var option = this.option
                    option.xAxis.data = ns // 时间信息
                    option.title.text = option_keywards.nqa_name // 图表名字
                    option.dataZoom[0].startValue = ns[0] // 初始时间
                    option.series[0].data = option_keywards.average_list // nqa average info
                    option.series[1].data = option_keywards.max_list_offset // nqa max info
                    option.series[2].data = option_keywards.min_list_offset // nqa min info
                    option.series[3].data = option_keywards.loss_list // nqa loss info
                    this.myChart.setOption(this.option);
                })
            },
```

## 在vue中动态循环多个echarts的图
这个的关键主要是
1. echarts的图需要在template中定义多个div-id 来定位图的位置
2. echarts所在的div-id必须有长度单位，在v-col中定义height就行
3. 必须初始化的时候设置option值
4. 加载后再次使用option覆盖

> 简化版

```html
<div class="chart-dis-area">
    <div v-for="(item,index) in chartList" class="chart-item-area">
        <div class="echarts" :id="item.id"></div>
    </div>
</div>

```
```js
// 默认 chartList 是一个空数组
init() {
    let arr = [];
    for(let i = 0; i < 6; i++) {
        let item = {
           barChart_option: '',       // chart 对象实例
           id: 'id' + i,       // 为了标示 id 不同
        }
        arr.push(item);
    }
    this.chartList = arr;


    
    this.$nextTick(() => {
        for(let i = 0; i < this.chartList.length; i++) {
            this.chartList[i].barChart = echarts.init(document.getElementById(this.chartList[i].id));
        }
        this.getListData();     // 请求网络获取数据
    })
}
```
```html
> 给一个我写的实例
<template>
        <el-container style="height: 100%">
            <el-main class="container">
                <el-row>
                    <el-col :span="7"  v-for="item in chartList" class="chart-item-area" v-bind:key="item[0]">  
                    <!-- 循环输出图 -->
                        <div class="grid-content bg-purple">
                            <div :id="item.id" class="charts" style="height: 300px;"></div>
                        </div>
                    </el-col>
                </el-row>
            </el-main>
        </el-container>
<!--        <div class="chart-dis-area">-->
<!--            <div v-for="item in chartList" class="chart-item-area" v-bind:key="item[0]">-->
<!--                <div class="echarts" :id="item.id" style="width: 500px;height: 500px"></div>-->
<!--            </div>-->
<!--        </div>-->
</template>
<script>
    export default {
        name: "",
        data(){
            return  {
                chartList:[],
                chartCount:0,
                option: {   // option值，用来定义echarts的属性值
                    title: {
                        text: '',
                        left: '1%'
                    },
                    tooltip: {
                        trigger: 'axis'
                    },
                    yAxis: [{
                        name:"Seconds",
                        nameTextStyle:{
                            fontWeight :"bolder",
                            fontSize: 20,
                        },
                        nameLocation:"end",
                        max:50,
                        min:0
                    },{
                        name:"Packet Loss Rate",
                        nameTextStyle:{
                            fontWeight :"bolder",
                            fontSize: 20,
                            align:"right"
                        },
                        nameLocation:"end",
                        max: 100,
                        min: 0,
                    }
                    ],
                    dataZoom: [{
                        startValue: '2021-03-16'
                    }, {
                        type: 'inside'
                    }],
                    xAxis: {
                        data:[],
                    },
                    series: [
                        {
                            name: 'average',
                            type: 'line',
                            data: [],
                            stack: 'offset-band',
                            symbol: 'none',
                            lineStyle:{
                                color:"#00FF00"
                            }
                        },
                        {
                            name: 'max',
                            type: 'line',
                            data: [],
                            lineStyle: {
                                opacity: 0
                            },
                            areaStyle: {
                                color: '#ccc'
                            },
                            stack: 'offset-band',
                            symbol: 'none',
                            step:"middle",
                        },
                        {
                            name: 'min',
                            type: 'line',
                            data: [],
                            lineStyle: {
                                opacity: 0
                            },
                            areaStyle: {
                                color: '#ccc'
                            },
                            stack: 'offset-band',
                            symbol: 'none',
                            step:"middle",
                        },{
                            name: 'loss',
                            type: 'line',
                            data: [],
                            lineStyle: {
                                color: '#FF0000'
                            },
                            symbol: 'none',
                        },
                    ]
                },
            }
        },
        methods:{
            async get_nqa_infos(group_name){
                await  this.$http.get(`/nqa/group/`,{params:{"group_name":group_name}}).then(res =>{
                    if (res.data.code != 200){
                        this.$message.error("数据获取失败")
                    }else {
                        this.$message.success("数据获取成功")
                        var nqa_infos = res.data.data.nqa_info
                        var charts_len = nqa_infos.length
                        this.chartCount = charts_len
                        let arr = []
                        for (let i=0;i<charts_len;i++){
                            let item = {
                                option_keywards: nqa_infos[i],
                                id: 'id'+i,
                            }
                            arr.push(item)
                        }
                        this.chartList = arr;
                    }
                }).catch(err =>{
                    console.log(err)
                })
                this.loading_NqaChart() // 加载
            },
            loading_NqaChart() {
                this.chartList.forEach((item) =>{
                    console.log(1)
                    var charts_name = item["id"]
                    this.myChart = this.$echarts.init(document.getElementById(charts_name));
                    this.myChart.setOption(this.option, true);
                    var option_keywards = item["option_keywards"]
                    var ns = this.getLocalTime(option_keywards.date_list)
                    var option = this.option
                    option.xAxis.data = ns // 时间信息
                    option.title.text = option_keywards.nqa_name // 图表名字
                    option.dataZoom[0].startValue = ns[0] // 初始时间
                    option.series[0].data = option_keywards.average_list // nqa average info
                    option.series[1].data = option_keywards.max_list_offset // nqa max info
                    option.series[2].data = option_keywards.min_list_offset // nqa min info
                    option.series[3].data = option_keywards.loss_list // nqa loss info
                    this.myChart.setOption(this.option);
                })
            },
            getLocalTime(ns) {
                for (let i = 0; i < ns.length; i++) {
                    ns[i] = new Date(parseInt(ns[i]) * 1000).toLocaleDateString().replace(/:\d{1,2}$/, ' ')
                }
                return ns
            }
        },
        created() {
            this.get_nqa_infos(this.$route.params.group_name)
            this.loading_NqaChart()
        }
    }
</script>

<style scoped>
    .container {
        /*display: inline-block;*/
        /*position: relative;*/
        /*padding-top: 61px ;*/
        /*margin-top: 61px;*/
        /*background-color: #E9EEF3;*/
        color: #333;
        text-align: center;
        line-height: 160px;
        height: 100%;
    }
</style>
```

## vue中生成左侧导航栏
https://www.jianshu.com/p/6b34abeb6db7
https://www.jianshu.com/p/7d998aae1fd7

## vue中axios传递参数
https://blog.csdn.net/Bule_daze/article/details/103820704