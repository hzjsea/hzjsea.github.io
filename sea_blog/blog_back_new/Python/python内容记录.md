---
title: python内容记录
categories:
  - python
tags:
  - python
abbrlink: 605dab9f
date: 2020-05-22 00:27:22
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [python内容记录](#python内容记录)
  - [去除list中空字符串的最快方法](#去除list中空字符串的最快方法)
  - [返回字典中的key和value](#返回字典中的key和value)
  - [json库中dumps,dump,loads,load的区别与使用场景](#json库中dumpsdumploadsload的区别与使用场景)
  - [python中删除文件与目录的方法](#python中删除文件与目录的方法)
  - [python中字符串大小写转换](#python中字符串大小写转换)

<!-- /code_chunk_output -->


<!-- more -->

# python内容记录
python的一些快捷方法记录


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

## 计时任务
两个计时任务的库
- apSheduler
- celery