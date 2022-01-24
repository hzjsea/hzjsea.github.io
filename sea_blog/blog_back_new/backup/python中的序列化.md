---
title: python中的序列化
categories:
  - python
tags:
  - python
abbrlink: 3d479224
date: 2020-03-20 14:03:58
---

learn python
<!-- more -->

# python中的序列化

有很多地方我们需要使用到Python的序列化，其中最明显的地方就是web 框架中，这些数据从数据库中拿出来之后，经过orm提取数据，最后序列化成完整的json数据，然后返回给前台


## json数据结构
序列化与反序列化两个动作作用对象都是对json内容的整理
JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。JSON采用完全独立于语言的文本格式，这些特性使JSON成为理想的数据交换语言。易于人阅读和编写，同时也易于机器解析和生成。

```json
 { "programmers": [ { "firstName": "Brett", "lastName":"McLaughlin", "email": "aaaa" },
{ "firstName": "Jason", "lastName":"Hunter", "email": "bbbb" },
{ "firstName": "Elliotte", "lastName":"Harold", "email": "cccc" }],
"authors": [
{ "firstName": "Isaac", "lastName": "Asimov", "genre": "science fiction" },
{ "firstName": "Tad", "lastName": "Williams", "genre": "fantasy" },
{ "firstName": "Frank", "lastName": "Peretti", "genre": "christian fiction" }],
"musicians": [
{ "firstName": "Eric", "lastName": "Clapton", "instrument": "guitar" },
{ "firstName": "Sergei", "lastName": "Rachmaninoff", "instrument": "piano" }
] }
```

## 序列化的几种方法
Python序列化有很多的方法
- json
用于字符串和python数据类型间进行转换
- pickle
用于python特有的类型和python的数据类型间进行转换
- marshal


Json模块提供了四个功能：dumps、dump、loads、load
pickle模块提供了四个功能：dumps、dump、loads、load


json是可以在不同语言之间交换数据的，而pickle只在python之间使用。json只能序列化最基本的数据类型，josn只能把常用的数据类型序列化（列表、字典、列表、字符串、数字、），比如日期格式、类对象！josn就不行了。而pickle可以序列化所有的数据类型，包括类，函数都可以序列化。

### 使用json 序列化
json 
- dumps把字典转换成字符串 
- loads把字符串转换成字典
- dump把数据类型转换成字符串并存储在文件中  
- load把文件打开从字符串转换成数据类型
在python中json与dict类似

https://www.cnblogs.com/yyds/p/6563608.html
```py
import json
test_dict = {'bigberg': [7600, {1: [['iPhone', 6300], ['Bike', 800], ['shirt', 300]]}]}
# 转换成字符串 dumps
test_string = json.dumps(test_dict)
print(type(test_string))

# 转换成字典
test_json = json.loads(test_string)
print(type(test_json))

# 将字典转换成字符串写入文件
file = open('1.json','w',encoding='utf-8')
json.dump(test_dict,file)

# 将文件中的字符串转换成字典输出
file = open('1.json','r',encoding='utf-8')
info = json.load(file)
print(info)
```
