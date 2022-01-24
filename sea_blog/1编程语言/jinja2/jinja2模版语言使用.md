# 模版语法
为了统一多种文件的配置手段，统一了一些模版语言
- yaml模版语法
- jinja2模版语言

# jinja模版语法
目前官方文档还是用py2作为讲解的
http://docs.jinkan.org/docs/jinja2/intro.html

安装
```bash
sudo pip install Jinja2
```

## 使用
```bash
from jinja2 import Template

# 使用Template获取模版
template = Template('Hello {{ name }}!')
# 使用render 混合模版，可以使用json 也可以使用Yaml
template.render(name='John Doe') 

```
## 模版文档
```bash
使用.或[]来表示最后一级(也称之为\[ \])
{{foo.bar}}  # 打印变量1
{{foo['bar']}} # 打印变量2
```
<font color='red'>实现</font>
为方便起见，Jinja2 中 foo.bar 在 Python 层中做下面的事情:

检查 foo 上是否有一个名为 bar 的属性。
如果没有，检查 foo 中是否有一个 'bar' 项 。
如果没有，返回一个未定义对象。
foo['bar'] 的方式相反，只在顺序上有细小差异:

检查在 foo 中是否有一个 'bar' 项。
如果没有，检查 foo 上是否有一个名为 bar 的属性。
如果没有，返回一个未定义对象。
如果一个对象有同名的项和属性，这很重要。此外，有一个 attr() 过滤 器，它只查找属性。


## 过滤器
过滤器中 使用| 与变量分隔，用() 传递参数 ，多个过滤器可以用管道|进行连接， 前一个过滤器输出被当作后一个过滤器输入
```bash
{{name | stringtags|title}}
{{list | join(',')}} b
```
过滤器总结
http://docs.jinkan.org/docs/jinja2/templates.html#builtin-filters

## 测试


## 注释
注释语法: {#.... #}
```jinja
{#
 dsdsd
#} 
```
## 
http://docs.jinkan.org/docs/jinja2/templates.html#line-statements