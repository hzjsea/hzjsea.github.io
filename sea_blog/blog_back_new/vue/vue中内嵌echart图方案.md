---
title: vue中内嵌echart图方案
categories:
  - js
tags:
  - echart
  - vue
abbrlink: a230946b
date: 2020-06-03 10:25:42
---
<!-- more -->




# vue中内嵌echart图方案


## 错误解决

报错内容如下
```bash
v-charts使用，报错找不到 echarts/lib/visual/dataColor
```

https://www.jianshu.com/p/4f776d66fa55
解决方法 降级echarts
手动降级echarts到v4.9.0
https://v-charts.js.org/#/start

## 嵌入echarts 具体代码

前面的步骤就过滤了，主要来看内嵌echart图的解决方案
首先安装eharts

```js
// main.js

// 下载
npm install echarts 
// 全局引用
import echarts from 'echarts'
Vue.prototype.$echarts = echarts


// ------------------------------------------

// router/index.js
// 登陆路由
import echarts from "../components/charts/Charts"
import echarts2 from "../components/charts/Charts2"
const routes = [
  { path: '/' , redirect: '/login'},
  { path: '/login' ,component: Login },
  //访问/home目录 重定向到欢迎页面
  {
    path: '/home',
    component: Home,
    children:[
      {path: '/home/users/authen',component: users},
      {path: '/home/rights',component: rights},
      {path: '/home/switchs/action',component: showconfig },
      // {path: '/home/charts',component: echarts}
      {path: '/home/echarts',component: echarts},
      {path: '/home/echarts2',component: echarts2}
    ]
  },
  {
    path: '/windows/',
    component: windowsTerminal,
  },
]



// -------------------------------------------


<template>
  <div class="left_inner"
       style="width: 100%;height: 100%">
    <!-- <div class="line"></div> -->
    <div id="yhtj"
         style="width: 100%;height: 100%">
    </div>
  </div>
</template>

<script>
export default {
  props: ["msg"],
  data() {
    return {
      snmp_data: [],
      links: [],
      data: [],
      option: {
        // xAxis: {
        //   show: true,
        //   type: "value"
        // },
        // yAxis: {
        //   show: true,
        //   type: "value"
        // },
        title: {
          // 图表名字
          text: "数据来自于交换机",
          subtext: "城域内部交换机流量图",
          left: "left",
          show: true
        },
        series: [
          {
            // 定义charts在容器中的位置
            // left: 20.0,
            // top: 20.0,
            // right: 150.0,
            // bottom: 25.0,
            width:500,
            zoom: 1.5, // 定义地图大小 区域图大小 关系到节点与节点之间的紧凑程度
            type: "graph", // 定义charts图形为桑基图
            layout: "none",
            // layout: "force",
            edgeSymbol: ["circle", "arrow"], // 开启箭头  这里用两种形式的结合体 分别是圆和箭头
            edgeSymbolSize: [0, 5], //箭头大小  // 这里对上面两种形式都做了大小处理
            //     edgeLabel: {                     // 线上面的注释
            //     normal: {
            //         show: true,
            //         position: "middle",
            //         textStyle: {
            //             fontSize: 14
            //         },
            //         formatter: "{c}次"
            //     }
            // },
            // edgeLabel: {
            //   normal: {
            //     show: true,
            //     postion: "middle",
            //     formatter: "{c}"
            //   }
            // },
            focusNodeAdjacency: true, // 指定的节点以及其所有邻接节点高亮
            nodeAlign: "right", // 定义节点位置
            roam: true, // 是否可拖拽

            lineStyle: {
              // 定义连接线样式
              normal: {
                width: 2,
                shadowColor: "none",
                color: "#142F54",
                curveness: 0.02,
                type: "solid" // 线的样式
              }
            },

            // 定义节点样式
            symbolSize: 20,
            symbol: "rect",
            // 节点样式
            itemStyle: {
              normal: {
                color: "#D7DBDD",
                // borderColor: "red",
                borderWidth: 2,
                shadowColor: "rgba(225,225,225,0.4)",
                shadowBlur: 10,
                shadowOffsetX: 10,
                shadowOffsetY: 10
              }
            },
            data: [],
            links: []
          }
        ]
      }
    };
  },
  methods: {
    loading_GraphChart() {
      let myChart = this.$echarts.init(document.getElementById("yhtj"));
      //   this.loading_data()
      //   this.option.series[0].data  = this.data

      //   console.log(this.option)
      myChart.setOption(this.option, true);
      myChart.showLoading();

      this.$http
        .get("echart/get/uplink")
        .then(res => {
          setTimeout(() => {
            // 获取到数据后隐藏加载动画
            myChart.hideLoading();
            if (res.data.meta.code != 0) {
              this.$message.error("获取数据失败");
            }
            this.$message.success("获取数据成功");
            this.snmp_data = res.data.data;
            for (let i = 0; i < this.snmp_data.length; i++) {
              var dict = {};
              var dict_links = {};
              dict["name"] = this.snmp_data[i].name;
              dict["x"] = parseInt(this.snmp_data[i].coordinates[0]);
              dict["y"] = parseInt(this.snmp_data[i].coordinates[1]);
              this.data.push(dict);
              dict_links["source"] = this.snmp_data[i].name;
              dict_links["target"] = this.snmp_data[i].targeTo;
              dict_links["value"] = this.snmp_data[i].diff_result;
              this.links.push(dict_links);
            }
            for (let i = 0; i < this.data.length; i++) {
              for (let j = 0; j < this.data.length; j++) {
                if (this.data[i]["name"] == this.data[j]["name"]) {
                  this.data.splice(i, 1);
                }
              }
            }
            myChart.setOption({
              // center: [115.97, 29.71],
              series: [
                {
                  data: [
                    { name: "POP-ZJ-KCH-PE1", x: 800, y: 0 },
                    { name: "POP-ZJ-XIH-PE1", x: 750, y:800},
                    { name: "POP-ZJ-FUD-PE1", x: -500, y: 800 },
                    { name: "POP-ZJ-SAD-S01", x: -500, y: 0 },
                    { name: "POP-ZJ-HGH-PE1", x: 200, y: -400 }
                  ],
                  //  data2: [
                  //   { name: "POP-ZJ-KCH-PE1", x: 800, y: 0 },
                  //   { name: "POP-ZJ-XIH-PE1", x: 800, y:800},
                  //   { name: "POP-ZJ-FUD-PE1", x: 0, y: 800 },
                  //   { name: "POP-ZJ-SAD-S01", x: 0, y: 0 },
                  //   { name: "POP-ZJ-HGH-PE1", x: 200, y: -400 }
                  // ],
                  links: this.links
                }
              ]
            });
          }, 500); //加载动画时长为0.5秒
        })
        .catch(err => {
          console.log(err);
        });
    }
  },
  mounted() {
    this.loading_GraphChart();
  }
};
</script>

<style>
</style>
```

vue文件中的大致结构
```js
<template>
  <div class="left_inner"
       style="width: 100%;height: 100%">
    <!-- <div class="line"></div> -->
    <div id="yhtj"
         style="width: 100%;height: 100%">
    </div>
  </div>
</template>

<script>
export default {
  props: ["msg"],
  data() {
    return {
      snmp_data: [],
      links: [],
      data: [],
      option: {
        },
        series: [{
          {},
            data: [],
            links: []
          }
        ]
      }
    };
  },
  methods: {
    loading_GraphChart() {
      let myChart = this.$echarts.init(document.getElementById("yhtj"));
      //   this.loading_data()
      //   this.option.series[0].data  = this.data

      //   console.log(this.option)
      myChart.setOption(this.option, true);
      myChart.showLoading();

      this.$http
        .get("echart/get/uplink")
        .then(res => {
          setTimeout(() => {
            // 获取到数据后隐藏加载动画
            myChart.hideLoading();
            }
            myChart.setOption({
              // center: [115.97, 29.71],
              series: [
                {
                  data: [
                    { name: "POP-ZJ-KCH-PE1", x: 800, y: 0 },
                    { name: "POP-ZJ-XIH-PE1", x: 750, y:800},
                    { name: "POP-ZJ-FUD-PE1", x: -500, y: 800 },
                    { name: "POP-ZJ-SAD-S01", x: -500, y: 0 },
                    { name: "POP-ZJ-HGH-PE1", x: 200, y: -400 }
                  ],
                  //  data2: [
                  //   { name: "POP-ZJ-KCH-PE1", x: 800, y: 0 },
                  //   { name: "POP-ZJ-XIH-PE1", x: 800, y:800},
                  //   { name: "POP-ZJ-FUD-PE1", x: 0, y: 800 },
                  //   { name: "POP-ZJ-SAD-S01", x: 0, y: 0 },
                  //   { name: "POP-ZJ-HGH-PE1", x: 200, y: -400 }
                  // ],
                  links: this.links
                }
              ]
            });
          }, 500); //加载动画时长为0.5秒
        })
        .catch(err => {
          console.log(err);
        });
    }
  },
  mounted() {
    this.loading_GraphChart();
  }
};
</script>

<style>
</style>
```
