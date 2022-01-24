---
title: Python Book
categories:
  - python
tags: 
  - python
abbrlink: 79b81c10
date: 2019-12-27 17:36:31
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [python Book](#python-book)
  - [内置函数](#内置函数)
    - [enumerate用法](#enumerate用法)
    - [abs函数用法](#abs函数用法)
    - [三元表达式](#三元表达式)
  - [math-paper](#math-paper)
    - [保留小数](#保留小数)
    - [Python-math库](#python-math库)
  - [list-papaer](#list-papaer)
    - [列表合并](#列表合并)
  - [os模块](#os模块)
    - [Python获取指定文件夹下的文件名](#python获取指定文件夹下的文件名)
    - [判断文件类型](#判断文件类型)
  - [文件操作](#文件操作)
    - [文件头部写如文件](#文件头部写如文件)
  - [去除list中空字符串的最快方法](#去除list中空字符串的最快方法)
  - [返回字典中的key和value](#返回字典中的key和value)
  - [json库中dumps,dump,loads,load的区别与使用场景](#json库中dumpsdumploadsload的区别与使用场景)
  - [python中删除文件与目录的方法](#python中删除文件与目录的方法)
  - [python中字符串大小写转换](#python中字符串大小写转换)
  - [内置函数divmodc](#内置函数divmodc)
  - [浮点除、地板除、取余除](#浮点除地板除取余除)
  - [字符串头部检测字符](#字符串头部检测字符)
  - [字符串](#字符串)
    - [字符串判断是否全是字母或者数字](#字符串判断是否全是字母或者数字)
  - [计时任务库](#计时任务库)

<!-- /code_chunk_output -->
<!--more-->
# python Book


## 内置函数

### enumerate用法
enumerate()是python的内置函数，使用场景就是将数组转换成有序号的元祖，字符串也可以，因为字符串本身就是一种数组
```bash
Python 3.7.5 (default, Nov  1 2019, 02:16:32)
[Clang 11.0.0 (clang-1100.0.33.8)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>> list=['1',2,3,4,5,6,6]
>>> print(enumerate(list))
<enumerate object at 0x100ddeaf0>
>>> newList = enumerate(list)
>>> for i in newList:
...     print(i)
...
(0, '1')
(1, 2)
(2, 3)
(3, 4)
(4, 5)
(5, 6)
(6, 6)
>>> newListString = enumerate("123456")
>>> for i in newListString:
...     print(i)
...
(0, '1')
(1, '2')
(2, '3')
(3, '4')
(4, '5')
(5, '6')
>>>
```

### abs函数用法
主要用来取绝对值
```bash
Python 3.7.5 (default, Nov  1 2019, 02:16:32)
[Clang 11.0.0 (clang-1100.0.33.8)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> a=-1
>>> print(abs(a))
1
>>> a=1
>>> print(abs(a))
1
```




### 三元表达式
```bash
# 判断x是否为负数
flag = True  if abs(x) == x else  False
flag = True if x < 0 else False
```
## math-paper
### 保留小数
使用字符串格式化
```python
a = 11.234
print("%.2f"%a) 
```
使用round内置函数
```python
a = 11.234
a = round(a,2)
print(a) 
```
使用decimal模块
```python 
from decimal import Decimal
a = 12.345
Decimal(a).quantize(Decimal("0.00")) 
```
### Python-math库
向上取整 math.ceil 
向下取整 math.floor
四舍五入 round
```bash
import math

#向上取整
print "math.ceil---"
print "math.ceil(2.3) => ", math.ceil(2.3)
print "math.ceil(2.6) => ", math.ceil(2.6)
 
#向下取整
print "\nmath.floor---"
print "math.floor(2.3) => ", math.floor(2.3)
print "math.floor(2.6) => ", math.floor(2.6)
 
#四舍五入
print "\nround---"
print "round(2.3) => ", round(2.3)
print "round(2.6) => ", round(2.6)
 
#这三个的返回结果都是浮点型
print "\n\nNOTE:every result is type of float"
print "math.ceil(2) => ", math.ceil(2)
print "math.floor(2) => ", math.floor(2)
print "round(2) => ", round(2)

```

## list-papaer

### 列表合并
```python
# 使用+号操作
list1 = [1,2,3,4,5]
list2 = ["z","xx",] 
list3 = list1+list2
# 使用extend
list4.extend(list1)
# 使用切片
list4[len(11):len(11)] = list5
```


## os模块

### Python获取指定文件夹下的文件名 

<font color='red'>模块os.walk可以遍历文件夹下的所有文件</font>

```python
os.walk(top, topdown=Ture, οnerrοr=None, followlinks=False)
# return  (dirpath dirnames filenames)
```
- dirpath: string 代表目录的路径
- dirnames list 包含了当前dirpath路径下所有的子目录名字（不包含目录路径）
- filenames：list，包含了当前dirpath路径下所有的非目录子文件的名字（不包含目录路径）。
```python
import os
 
def file_name(file_dir): 
    for root, dirs, files in os.walk(file_dir):
        print(root) #当前目录路径
        print(dirs) #当前路径下所有子目录
        print(files) #当前路径下所有非目录子文件
        for file in files:
            # os.path.splitext # 对文件名进行切割成数组
            if os.path.splitext(file)[1] == ".jpeg":
              print(file)
            else:
              pass
```

<font color='red'>模块os.listdir()可以得到当前路径下的文件名，不包括子目录中的文件</font>

```python
import os
# 遍历获取所有文件
def listdir(path, list_name):
    for file in os.listdir(path):
        file_path = os.path.join(path, file)
        # os.path.isdir(file) #判断文件是否为文件夹
        if os.path.isdir(file_path):
            listdir(file_path, list_name)
        elif os.path.splitext(file_path)[1]=='.jpeg':
            list_name.append(file_path)
```

### 判断文件类型
```python
os.path.isdir(path) # 是否为文件夹
os.path.isfile(path) # 是否为文件
```


## 文件操作
mode选项
r 读模式，只能读 文件不存在报错，
w 写模式，只能写 文件不存在则创建，
以上存在存在则清空在打开

rb 二进制读模式 只能读  文件不存在报错
wb 二进制写模式 只能写  文件不存在则创建，
以上存在则清空再打开 
有二进制内容 写入则需要encode("utf-8")

rb+ 二进制读模式 可读可写 文件不存在报错
wb+ 二进制写模式 可读可写 文件不存在则创建，
以上存在则清空再打开

a 追加写模式，文件存在则追加写入
a+ 追加读写方式 
以上存在则追加写入

默认进入为文件末尾
f.seek(0) 设置读写位置为开头
f.truncate(0) 将文章字节阶段为0

### 文件头部写如文件
```bash
with open(path, "r+") as f:
     old = f.read()
     f.seek(0)
     f.write(data)
     f.write(old)
```
或者
```bash
with open(path,"a") as f:
      f.seek(0)
      old = f.read()
      # 全部删除
      f.truncate(0)
      f.write("---")
      f.write("tags: xxx")
      f.write("---")
      f.close()
```


## 去除list中空字符串的最快方法
```python
get_list=[ x for i in list if x != '']
```

## 返回字典中的key和value
```py
d={1:'one',2:'two',3:'three'}
for k,v in d.items():
    print(k,v)
```

## json库中dumps,dump,loads,load的区别与使用场景
https://blog.csdn.net/Mr_EvanChen/article/details/77879967

## python中删除文件与目录的方法
https://www.cnblogs.com/chengd/p/7533473.html


## python中字符串大小写转换
```py
str = "www.runoob.com"
print(str.upper())          # 把所有字符中的小写字母转换成大写字母
print(str.lower())          # 把所有字符中的大写字母转换成小写字母
print(str.capitalize())     # 把第一个字母转化为大写字母，其余小写
print(str.title())          # 把每个单词的第一个字母转化为大写，其余小写

# WWW.RUNOOB.COM
# www.runoob.com
# Www.runoob.com
# Www.Runoob.Com
```

## 内置函数divmodc
divmod() 函数把除数和余数运算结果结合起来，返回一个包含商和余数的元组(a // b, a % b)
```bash
>>>divmod(7, 2)
(3, 1)
>>> divmod(8, 2)
(4, 0)
>>> divmod(1+2j,1+0.5j)
((1+0j), 1.5j)
```

## 浮点除、地板除、取余除
浮点除，不管前面的是整形还是浮点型 结果都是浮点型
```py
# 浮点除
nums1 = 20
nums2 = 10
print(nums1/nums2) 
nums1 = 10
nums2 = 3
print(nums1/nums2)
# 地板除
nums1 = 10
nums2 = 3
print(nums1//nums2)
# 余数除
nums1 = 10
nums2 = 3
print(nums1%nums2)
# 余数地板除
nums1 = 10
nums2 = 3
print(divmod(nums1,nums2))

# 2.0
# 3.3333333333333335
# 3
# 1
# (3, 1)
```

## 字符串头部检测字符
startswith() 方法用于检查字符串是否是以指定子字符串开头，如果是则返回 True，否则返回 False。如果参数 beg 和 end 指定值，则在指定范围内检查。
```bash
str = "this is string example....wow!!!";
print str.startswith( 'this' );
print str.startswith( 'is', 2, 4 );
print str.startswith( 'this', 2, 4 );
```


## 字符串

### 字符串判断是否全是字母或者数字
```bash
str.isnumeric(): True if 只包含数字 注意：此函数只能用于unicode string
str.isdigit(): True if 只包含数字
str.isalpha()：True if 只包含字母
str.isalnum()：True if 只包含字母或者数字
# 返回True or false

# 使用
str = "12usingLog.xslx"
res = "is integer" if str[:2].isdigit() else "not integer"
print(res)
res = "is digit" if str[3:5].isalpha() else "not digit"
print(res)
res = "not alnum" if str.isalnum() else "not alnum"
print(res)
```

## 计时任务库


