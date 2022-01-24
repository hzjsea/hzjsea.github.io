# filter过滤器
vue.js 允许你自定义过滤器，可被用于一些常见的文本格式化。过滤器可以用在两个地方：双花括号插值和 v-bind 表达式 (后者从 2.1.0+ 开始支持)。过滤器应该被添加在 JavaScript 表达式的尾部，由“管道”符号指示：

```js
{{ message | capitalize  }}

<div :id="message | capitalize"></div>

filters:{
  capitalize:function (value) {
   if (!value) return ''
   value = value.toSrting()
   retun value.charAt(0).toUpperCase() + value.slice(1) 
  }
}
```

其中capitalize会接受message的值 并在filters中使用对应的方法进行处理后再输出，与django中的filter类似

## 过滤器串联
```js
{{message | filterA | filterB}}

filters:{
  filterA:function(value){
    xxx
  } 
  filterB:function(value){
    yyy
  }
}

```

## 过滤器传值
```js
{{message | filterC('agr1','arg2')}}

filters:{
  filterC:function(value){
    xxx
  }
}
```



事例：
```js
<script>
export default {
  data() {}
  created() {},
  filters: {
    statusText(val) {
      if (val === undefined) return;
      if (val === 0) {
        return "已完成";
      } else if (val === 1) {
        return "待审核";
      } else if (val === 2) {
        return "配送中";
      } else {
        return "已取消";
      }
    },
    tagClass(val) {
      if (val === undefined) return;
      if (val === 0) {
        return "success";
      } else if (val === 1) {
        return "info";
      } else if (val === 2) {
        return "warning";
      } else {
        return "danger";
      }
    }
  },
  methods: {}
</script> 
```