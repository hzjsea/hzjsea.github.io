---
title: shell_shell中的妙用
date: 2021-05-17 11:07:36
categories:  [shell]
tags: [shell,awk,sed]
---


<!--more-->


# shell中的妙用

其实shell的时候用过中总是需要几个指令来做补充,这里汇总这些指令，并举例处理一些内容


## awk

## sed

## grep

## xargs 

这个命令真的很好用，主要是用来控制参数，为后面的命令做准备
https://segmentfault.com/a/1190000012566053

附上自己写的一段代码,目的是过滤掉当前目录中所有的软链接文件和软链接自身文件

```bash
find . -type f > oo && find . -type l | xargs readlink | sed -n 's#./##p' | xargs -I {} find .  -name {} | sort < oo | uniq -u
```

![](https://noback.upyun.com/2021-05-17-11-16-33.png!)


## sort  uniq

在我认为这两个基本上就是绑在一起了

```bash
去除重复行
sort file |uniq

查找非重复行
sort file |uniq -u

查找重复行
sort file |uniq -d

统计
sort file | uniq -c
```


## 输出重定向
```bash
find . -type f > oo && find . -type l | xargs readlink | sed -n 's#./##p' | xargs -I {} find .  -name {} | sort < oo | uniq -u 
```
重定向很方便

## shell 变量

> 这里举一个例子，这是一个shell脚本 获取内容如下，
一个是网卡的名字， 一个是需要导出的错误类型


```bash
#/bin/bash

### keywards
# $1 网卡
# $2 方向
# $3 type
### description
# 网卡  $1
# RX_error $4
# TX_error $12  


# RX_dropped $5     pass 先不监控
# TX_dropped $13    pass 先不监控
# RX_overrun $6     pass 先不监控
# TX_overrun $14    pass 先不监控

eth_name=$1
direction=0
type=""


get(){

    if [[ $3 = "error" ]];then
        if [[ $2 = "RX" ]] || [[ $2 = "rx"  ]] || [[ $2 = "Rx" ]] || [[ $2 = "rX" ]];then
            direction=4
        elif [[ $2 = "TX" ]] || [[ $2 = "Tx"  ]] || [[ $2 = "tx" ]] || [[ $2 = "tX" ]];then
            direction=12
        fi
    fi
}

get $1 $2 $3

cat /proc/net/dev | awk "{if ( \$1 ~ /${eth_name}/ ){print $""$direction""}}" ### 重点在这里 $""$direction"" == $0  $2 $1...
``` 

## 扩展


### 读取软连接地址 readlink