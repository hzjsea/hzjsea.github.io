# python-json 模块
JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。它基于ECMAScript的一个子集。 JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯(包括C、C++、Java、JavaScript、Perl、Python等)。这些特性使JSON成为理想的数据交换语言。易于人阅读和编写，同时也易于机器解析和生成(一般用于提升网络传输速率)。

JSON在python中分别由list和dict组成。

## python中的两个序列化模块
- Json 用于字符串和python数据类型间的转换
- pickle 用于python特有的类型和python的数据类型间进行转换

Json模块提供了四个功能：dumps、dump、loads、load
pickle模块提供了四个功能：dumps、dump、loads、load


json 
dumps把数据类型转换成字符串 
dump把数据类型转换成字符串并存储在文件中  
loads把字符串转换成数据类型  
load把文件打开从字符串转换成数据类型

### dumps
将字典转换称字符串
```python
import os
import json
test_dict = {'bigberg': [7600, {1: [['iPhone', 6300], ['Bike', 800], ['shirt', 300]]}]} 
print(test_dict)
print(type(test_dict))
# 转成字符串
test_string = json.dumps(test_dict)
print(test_string)
print(type(test_string))
```

### loads 
将字符串转换为字典
```python
new_dict = json.loads(json) 
print(new_dict)
print(type(new_dict))
```


### dump
将数据写入json文件中
```python
with open("../config/record.json","w") as f:
        json.dump(new_dict,f) 
        print("加载入文件完成...") 
```

### load
将文件打开，并将字符串变为数据类型
```python
with open("../config/record.json",'r') as load_f:
    load_dict = json.load(load_f)
    print(load_dict)
load_dict['smallberg'] = [8200,{1:[['Python',81],['shirt',300]]}]
print(load_dict)
 
 with open("../config/record.json","w") as dump_f:
        json.dump(load_dict,dump_f)

```

## 随机生成json数据
https://www.cnblogs.com/lhb25/p/tool-for-generating-random-json-data.html