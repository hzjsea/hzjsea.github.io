---
title: linux_command ss
categories:
  - linux
tags:
  - linux
abbrlink: d331dcdd
date: 2020-03-06 16:38:01
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux_command ss](#linux_command-ss)
  - [Use](#use)
    - [命令格式](#命令格式)
    - [参数](#参数)
    - [显示所有的TCP连接 或者 UDP连接](#显示所有的tcp连接-或者-udp连接)
    - [显示所有套接字（sockets）的大致情况](#显示所有套接字sockets的大致情况)
    - [列出所有打开的网络连接端口](#列出所有打开的网络连接端口)
    - [显示进行使用的socket](#显示进行使用的socket)
    - [找出打开套接字/端口应用程序](#找出打开套接字端口应用程序)
    - [匹配本地地址与远端地址](#匹配本地地址与远端地址)
    - [显示所有状态为etablished的SMTP连接](#显示所有状态为etablished的smtp连接)
    - [显示所有状态为etablished的HTTP连接](#显示所有状态为etablished的http连接)
    - [列举出处于 FIN-WAIT-1状态的源端口为 80或者 443，目标网络为 193.233.7/24所有 tcp套接字](#列举出处于-fin-wait-1状态的源端口为-80或者-443目标网络为-193233724所有-tcp套接字)
    - [ss 和 netstat 效率对比](#ss-和-netstat-效率对比)
    - [筛选端口](#筛选端口)
    - [端口过滤](#端口过滤)

<!-- /code_chunk_output -->
<!-- more -->


# linux_command ss

ss是Socket Statistics的缩写。顾名思义，ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。

查看网络连接你可以是使用netstat 或者 cat /proc/net/tcp
但是当你在连接数上万的服务器上使用netstat时候，会出现明显的卡顿


```bash
➜  ~ netstat
Active Internet connections
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4       0      0  10.0.28.62.63485       117.18.232.200.https   ESTABLISHED
tcp4       0      0  10.0.28.62.63467       210.22.248.181.https   ESTABLISHED
tcp4       0      0  10.0.28.62.63459       121.51.176.86.https    ESTABLISHED
tcp4       0      0  10.0.28.62.63458       121.51.176.86.https    FIN_WAIT_2
tcp4       0      0  10.0.28.62.63456       121.51.176.86.https    FIN_WAIT_2
tcp4       0      0  10.0.28.62.63451       121.51.176.86.https    FIN_WAIT_2
tcp4       0    517  10.0.28.62.63449       121.51.176.86.https    FIN_WAIT_1
tcp4       0      0  10.0.28.62.63443       121.51.176.86.https    FIN_WAIT_1

tcp4       0     31  10.0.28.62.63429       52.114.158.91.https    FIN_WAIT_1
tcp4       0      0  10.0.28.62.63424       10.0.28.58.ssh         ESTABLISHED



tcp4       0      0  10.0.28.62.63422       hkg12s13-in-f5.1.https ESTABLISHED

tcp4       0      0  10.0.28.62.63379       13.107.18.11.https     ESTABLISHED
tcp4       0      0  10.0.28.62.63378       13.107.18.11.https     ESTABLISHED


[root@hzj-machine ~]# cat /proc/net/tcp
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode
   0: 0100007F:0019 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 22560 1 ffff8840c72e0000 100 0 0 10 0
   1: 00000000:0016 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 21768 1 ffff8840c1b18000 100 0 0 10 0
   2: 3A1C000A:0016 3E1C000A:F7C0 01 00000000:00000000 02:000A38D8 00000000     0        0 119773 4 ffff8840c72e3640 20 4 27 10 -1
   3: 3A1C000A:0016 3E1C000A:F2F2 01 00000000:00000000 02:0002DF3E 00000000     0        0 117281 2 ffff8840c72e64c0 20 4 31 10 25
   4: 3A1C000A:0016 3E1C000A:F0A9 01 00000000:00000000 02:0008BF3E 00000000     0        0 70042 2 ffff8840c72e2e80 20 4 31 2 2
```


但是ss相对于这两种来说在速度上加快了很多,它利用到了TCP协议栈中tcp_diag。tcp_diag是一个用于分析统计的模块，可以获得Linux 内核中第一手的信息，这就确保了ss的快捷高效.当谈如果系统中没有tcp_diag,ss也可以正常运行。速度相对于有tcp_diag较慢



## Use


### 命令格式
```bash
ss [参数]
ss [参数] [过滤]
```

### 参数
```bash 
  -h, --help 帮助信息
-V, --version 程序版本信息
-n, --numeric 不解析服务名称
-r, --resolve        解析主机名
-a, --all 显示所有套接字（sockets）
-l, --listening 显示监听状态的套接字（sockets）
-o, --options        显示计时器信息
-e, --extended       显示详细的套接字（sockets）信息
-m, --memory         显示套接字（socket）的内存使用情况
-p, --processes 显示使用套接字（socket）的进程
-i, --info 显示 TCP内部信息
-s, --summary 显示套接字（socket）使用概况
-4, --ipv4           仅显示IPv4的套接字（sockets）
-6, --ipv6           仅显示IPv6的套接字（sockets）
-0, --packet         显示 PACKET 套接字（socket）
-t, --tcp 仅显示 TCP套接字（sockets）
-u, --udp 仅显示 UCP套接字（sockets）
-d, --dccp 仅显示 DCCP套接字（sockets）
-w, --raw 仅显示 RAW套接字（sockets）
-x, --unix 仅显示 Unix套接字（sockets）
-f, --family=FAMILY  显示 FAMILY类型的套接字（sockets），FAMILY可选，支持  unix, inet, inet6, link, netlink
-A, --query=QUERY, --socket=QUERY
      QUERY := {all|inet|tcp|udp|raw|unix|packet|netlink}[,QUERY]
-D, --diag=FILE     将原始TCP套接字（sockets）信息转储到文件
 -F, --filter=FILE   从文件中都去过滤器信息
       FILTER := [ state TCP-STATE ] [ EXPRESSION ]
```


### 显示所有的TCP连接 或者 UDP连接
```bash
## 显示tcp
ss -t -a
## 显示udp
ss -u -a
```

### 显示所有套接字（sockets）的大致情况
```bash
ss -s 
```

### 列出所有打开的网络连接端口
```bash
ss -l
```

### 显示进行使用的socket
```bash
ss -pl
```

### 找出打开套接字/端口应用程序
```bash
ss -pl | grep 3306
```

### 匹配本地地址与远端地址
之前匹配端口以及端口状态用的dport 和sport
这里使用匹配地址 src dst

```bash 
ss src 192.168.0.1
ss src 192.168.0.1:http
ss src 192.168.0.1:20

ss dst 192.168.0.1
ss dst 192.168.0.1:http
ss dst 192.168.0.1:20
```

### 显示所有状态为etablished的SMTP连接
```bash 
ss -o state established dport = :smtp or sport =:smtp
```

### 显示所有状态为etablished的HTTP连接
```bash 
ss -o state established dport = :http or sport =: http
```
### 列举出处于 FIN-WAIT-1状态的源端口为 80或者 443，目标网络为 193.233.7/24所有 tcp套接字
```bash 
ss -o state fin-wait-1 '( sport = :http or sport = :https )' dst 193.233.7/24
```

### ss 和 netstat 效率对比
```bash 
time netstat -at
time ss 

[root@localhost ~]# time ss   
real    0m0.903s
user    0m0.849s
sys     0m0.053s
[root@localhost ~]# time netstat -at
real    0m1.529s
user    0m0.001s
sys     0m0.001s
```

### 筛选端口
```bash 
# 过滤只显示所有ssh连接的开启状态的接口
 ss -o state established  dport = :ssh or sport = :ssh 
             # Display all established ssh connections.

# 过滤 80 443 8100端口的内容
state = established
ss2 -4itn state $state dport = :80 or sport = :80 or dport = :443 or sport = :433 or sport = :8100 or dport = :8100
```

### 端口过滤
将本地或者远程端口和一个数比较
```bash 
# 形式
# ss dport OP PORT 
# ss sport OP PORT
ss dport =: http  # 等于http
ss dport \> :1024 # 端口号大于1024
ss dport \< :3100 # 端口号小鱼3100
ss dport != :22 # 端口号不等于22
ss dport eq :22 # 端口号等于22

```
OP 可以是这些
```bash
<= or le : 小于或等于端口号
>= or ge : 大于或等于端口号
== or eq : 等于端口号
!= or ne : 不等于端口号
< or gt : 小于端口号
> or lt : 大于端口号
```