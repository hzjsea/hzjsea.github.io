---
title: 一些额外的笔记
categories:
  - shell
tags:
  - shell
abbrlink: 10076eb4
date: 2020-04-12 20:36:52
---

一些额外的笔记

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [查询安装包的配置文件](#查询安装包的配置文件)
- [login 与nologin](#login-与nologin)
- [快捷键](#快捷键)
- [命令](#命令)
- [重定向](#重定向)
- [管道与tee](#管道与tee)
- [&& 与; ||](#-与-)
- [shell中的通配符](#shell中的通配符)
- [echo 笔记](#echo-笔记)
- [type命令](#type命令)

<!-- /code_chunk_output -->

<!-- more -->

# 查询安装包的配置文件

```bash
rpm -qc bash
/etc/skel/.bash_logout
/etc/skel/.bash_profile
/etc/skel/.bashrc
```


# login 与nologin
https://www.bilibili.com/video/BV1g4411Y75K?p=6


# 快捷键
```bash
ctrl+R 查询
ctrl+D logout
ctrl+A 光标移动到最前
ctrl+E 移动到最后
ctrl+L 清屏
ctrl+U 向后删除
ctrl+K 向前删除
ctrl+S 锁屏幕
ctrl+Q 退出锁屏幕
```

# 命令
```bash
& 开启后台进程 ，连接断掉后消失
nohup & 开启后台进程，一直开着 ，并且将输出放到nohup.out中去
```

# 重定向
```bash
0 标准输入
1 标准输出
2 标准错误
> 覆盖
>> 追加
2> 错误重定向
2>> 错误重定向追加
2>&1 输出错误信息
/dev/null 空设备，相当于一个黑洞，你丢进去自己会丢掉
cat < xxx   输入信息
cat < xx.file > yy.file 复制
cat >>EOF EOF 输出多行
cat > file <<EOF EOF  多行输入
```

# 管道与tee
管道就是重定向标准输出给下一个指令的标准输入
tee就是在交给下一个标准输入之前复制一份输出到屏幕上面
```bash
[root@CodeSheep ~]# ls | tee file1 | cat -n
     1  assets
     2  djangorestframework
     3  EOF
     4  file1
     5  hello
     6  hzj
     7  install.sh
     8  logs.tar
     9  print
    10  Project
    11  Python-3.7.3
    12  Python-3.7.3.tgz
    13  rust-pro
    14  switch_to_cli.py
    15  test.py
    16  test.txt
    17  transmission
    18  tt
    19  tt.c
    20  xx
    21  xx.sh
```
之前到file1这里就结束了，但是又标准输出了一下
另外直接tee是覆盖  tee -a 是追加
```bash
cat xx | tee file | cat -n
cat xx | tee -a | cat -n
```

# && 与; || 
和判断 前一个正确则执行后面的
;判断，不收局限 顺序执行
或判断 前一个错误则执行后面的

# shell中的通配符
```bash
* 匹配任意多个字符
? 匹配任意一个字符
[] 匹配括号中的任意一个字符 
[^] 不是括号中的任意一个字符
() 在子shell中执行
{} 集合 touch file{1..9}  
    mkdir file1{,9} # 创建file1 和file9


\ 转义字符，转移元字符，上面的都是元字符
```



# echo 笔记

这里要区分mac与centos中的使用场景,其他内容都相同
```bash
# mac
echo "\033[1;36mSister Lin Fall from the Sky\033[0m"
# centos 需要有空格 echo -e"" 就显示不出来
echo -e "\e[1;36mSister Lin Fall from the Sky\e[0m"
```


```bash
-n 输出内容后不另起一行
-e 解析内容  比如制表符 \t  颜色\e[1;31m 大红色 \e[0m 恢复颜色
-E 不解析内容 默认-E

# 解析的内容直接man echo 看一下就有了
```
颜色输出
```bash
echo -e "\033[字背景颜色;字体颜色m字符串\033[控制码"
```


常用形式
```bash
echo -e 'nameserver 192.168.147.20\nnameserver 192.168.21.20\nnameserver 114.114.114.114' > /etc/resolv.conf


# -------- 颜色输出
TAG="\n\n\033[0;32m   ### "
END=" ### \033[0m\n"


result="命令类型: \n
 alias 别名 \n
 Keyword 关键字，shell保留字 \n
 function 函数 shell函数 \n
 builtin 内建命令，shell内建命令 \n
 file 文件，磁盘文件 \n
 unfound 没有找到"


echo -e $TAG $result $END
```

常用颜色定义
```bash
## tip blue
function blue(){
    echo -e "\033[34m[ $1 ]\033[0m"
}


## success green
function green(){
    echo -e "\033[32m[ $1 ]\033[0m"
}

## Error to warning with blink
function bred(){
    echo -e "\033[31m\033[01m\033[05m[ $1 ]\033[0m"
}

## Error to warning with blink
function byellow(){
    echo -e "\033[33m\033[01m\033[05m[ $1 ]\033[0m"
}


## Error
function bred(){
    echo -e "\033[31m\033[01m[ $1 ]\033[0m"
}

## warning
function byellow(){
    echo -e "\033[33m\033[01m[ $1 ]\033[0m"
}
```


# type命令
type命令用来显示的指定命令的类型，判断给出的指令是内部指令还是外部指令

```bash
命令类型:
 alias 别名
 Keyword 关键字，shell保留字
 function 函数 shell函数
 builtin 内建命令，shell内建命令
 file 文件，磁盘文件
 unfound 没有找到

-t：输出“file”、“alias”或者“builtin”，分别表示给定的指令为“外部指令”、“命令别名”或者“内部指令”；
-p：如果给出的指令为外部指令，则显示其绝对路径；
-a：在环境变量“PATH”指定的路径中，显示给定指令的信息，包括命令别名。

```

使用，比如我们在shell语句中的使用的[] 他其实也是一个内建命令
```bash
[root@CodeSheep ~]# type else
else is a shell keyword
[root@CodeSheep ~]# type then
then is a shell keyword
[root@CodeSheep ~]# type [
[ is a shell builtin
```