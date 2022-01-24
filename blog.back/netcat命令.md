---
categories:
  - linux
tags:
  - linux
  - command
abbrlink: e5362e2e
title: netcat命令
date: 2020-06-05 11:45:46
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->

# netcat命令 nc

端口是与 Linux 操作系统上的应用或进程的通讯端点的逻辑实体 nc命令主要用来查看远程端口是否开启
netcat（或简称 nc）是一个功能强大且易于使用的程序，可用于 Linux 中与 TCP、UDP 或 UNIX 域套接字相关的任何事情。

## 安装
```bash
yum install nc
sudo apt-get install nc
```


## 检测端口

检查某个IP单个或多个端口是否开启
`-z` – 设置 nc 只是扫描侦听守护进程，实际上不向它们发送任何数据。
`-v` – 启用详细模式
`-n` - 使用纯数字 IP 地址，即不用DNS解析域名，即禁用域名
`-w` - 设置超时值设置为1
```bash
# 示例:
nc -zv 192.168.20.1 20
nc -zv 192.168.20.1 20 21 23  # 支持多个多个端口
nc -zv 192.168.20.1 20-80 # 多组端口扫描


# 使用
alpaca:~/ $ nc -z 10.0.43.102 8889                                  
Connection to 10.0.43.102 port 8889 [tcp/ddi-tcp-2] succeeded!

# 详细模式
alpaca:~/ $ nc -zv 10.0.43.102 8889                                                                                                                               
found 0 associations
found 1 connections:
     1: flags=82<CONNECTED,PREFERRED>
        outif en0
        src 10.0.41.46 port 59996
        dst 10.0.43.102 port 8889
        rank info not available
        TCP aux info available

Connection to 10.0.43.102 port 8889 [tcp/ddi-tcp-2] succeeded!

# 端口扫描
alpaca:~/ $ nc -z 10.0.43.102 8880-8889                                                            
Connection to 10.0.43.102 port 8881 [tcp/*] succeeded!
Connection to 10.0.43.102 port 8887 [tcp/*] succeeded!
Connection to 10.0.43.102 port 8888 [tcp/ddi-tcp-1] succeeded!
Connection to 10.0.43.102 port 8889 [tcp/ddi-tcp-2] succeeded!

# 支持域名
alpaca:~/ $ nc -zv nono.guozen.cn 80                  
found 0 associations
found 1 connections:
     1: flags=82<CONNECTED,PREFERRED>
        outif en0
        src 10.0.41.46 port 60980
        dst 1.81.5.176 port 80
        rank info not available
        TCP aux info available

Connection to nono.guozen.cn port 80 [tcp/http] succeeded!
```


## 传送文件
<font color='red'>nc另外一个功能就是在内网传送大文件,nc的传输是单向的</font>

如果考虑到安全的话，就用scp把，scp的本质是ssh  有了一层加密

`-l` - 监听端口，管控传入的资料。
从server1拷贝文件到server2上。需要先在server2上，用nc激活监听。
```bash
# server1 当前ip为192.168.200.27
nc -l 8889 > input.txt # 接收8889传过来的内容，输出到input.txt文件中 

# server2
nc -w 1 192.168.200.27 8889 < abc.txt # 接收abc.txt中的内容，发送到192.168.200.27:8889端口上 



# 目录压缩传输
tar -cvf – dir_name | nc -l 1567
nc 192.168.200.17 1567 | tar -xvf -

# 使用bzip压缩传输
tar -cvf - dir_name | bzip2 -z | nc -l 1567
nc 192.168.200.17 1567 | bzip2 -d |tar -xvf -


# 加密传输
nc localhost 1567 | mcrypt –flush –bare -F -q -d -m ecb > file.txt
mcrypt –flush –bare -F -q -m ecb < file.txt | nc -l 1567
```

## 克隆硬盘或分区
操作与上面的拷贝是雷同的，只需要由dd获得硬盘或分区的数据，然后传输即可。
克隆硬盘或分区的操作，不应在已经mount的的系统上进行

```bash
# server1 监听端口123 将收到的内容dd到/dev/sda上
nc -l -p 1234 | dd of=/dev/sda

# server2 将/dev/sda中的内容 dd出来到对应的端口上
dd if=/dev/sda | nc 192.168.200.27 1234
```

## 长连接聊天
```bash
# server1
nc -l 1234
hello!
hi!

# server2
nc 192.168.200.27 1234
hello!
hi!
```