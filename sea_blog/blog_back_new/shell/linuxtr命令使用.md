---
title: tr命令
categories:
  - linux
tags:
  - linux
  - command
abbrlink: '88149077'
date: 2020-03-26 11:20:05
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux tr命令使用](#linux-tr命令使用)
  - [tr基本语法](#tr基本语法)
  - [实例](#实例)
  - [字符范围](#字符范围)

<!-- /code_chunk_output -->
<!-- more -->

# linux tr命令使用
tr 命令主要用于删除文件中控制字符或进行字符转换

## tr基本语法
```bash
tr [-d] [-c] [-s] [字符串1] [字符串2] 文件名
# 字符串1用于查询，字符串2用于处理各种转换。
-d 删除字符串中所有输入的字符串
-c 查询替换，先取字符串1 再取反 再查询 再替换成字符串2(可以理解成保留字符串1 其他都替换成字符串2)
-s 删除所有重复出现的字符序列，只保留第一个，就是将重复出现的字符串替换成为字符串2
```
字符串1和字符串2内容只能使用单字符，字符串范围或列表 比如字符串范围`[a-z]`表示小写字母、`[0-9]`表示数字。


## 实例
```bash
# 直接替换
echo "dsadsadasdasdsad" | tr d s
# res: ssassasassasssas

# 保留替换
echo "dsadsadasdasdsad" | tr -c d 1
# res: d11d11d11d11d11d1

# 压缩
echo "dsadsadasdasdsad" | tr -s sa 1
# res: d1d1d1d1d1d

# 仅压缩 不替换
echo -e "1\n\n2\n\n\n3" | tr -s '\n' 
# res: 1 
# 2 
# 3

# 删除
echo "dsadsadasdasdsad" | tr -d sa
# res: dddddd


# 删除空格
echo "   Hello World  " | tr -d '[ \t]'

# 大小写替换
echo "Hello World" | tr '[a-z]' '[A-Z]' #小写转大写，输出HELLO WORLD
echo "Hello World" | tr '[A-Z]' '[a-z]' #大写转小写，输出hello world
echo "Hello World" | tr '[A-Za-z]' '[a-zA-Z]' #大小写互换，输出hELLO wORLD

#删除非数字字符 
echo "2018abcdefdf06zzz01" |tr -d -c '[0-9]'
20180601
echo "2018abcdefdf06zzz01" | tr -cd '[:digit:]'

# 将小写字母改写成大写字母
echo 'myfreax' | tr '[:lower:]' '[:upper:]'

# 替换所有字母
echo "hello world" | tr -s '[:alnum:]' 1
# 1 1

```


## 字符范围
```bash
[:lower:] 小写字母
[:upper:] 大写字母
[:digit:] 所有数字字符
[:alnum:] 所有单词
[az] az内的字符组成的字符串。
[AZ] AZ内的字符组成的字符串。
[0-9]数字串。
\octal一个三位的八进制数，对应有效的ASCII字符。
[O*n]表示字符O重复出现指定次数n。因此[O*2]匹配OO的字符串。
```