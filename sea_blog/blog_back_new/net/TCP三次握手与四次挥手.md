---
title: TCP/IP的三次握手与四次挥手
categories:
  - 网络
tags:
  - 网络
abbrlink: d44b9151
date: 2020-07-12 10:20:27
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [TCP/IP的三次握手与四次挥手](#tcpip的三次握手与四次挥手)
  - [TCP的6种标志位](#tcp的6种标志位)
  - [TCP三次握手](#tcp三次握手)
  - [TCP四次挥手](#tcp四次挥手)
  - [TCP的生命周期](#tcp的生命周期)
    - [客户端TCP生命周期](#客户端tcp生命周期)
    - [服务端TCP生命周期](#服务端tcp生命周期)
  - [SYN洪泛](#syn洪泛)
    - [SYN COOKIE](#syn-cookie)
  - [RST标志位](#rst标志位)

<!-- /code_chunk_output -->
<!-- more -->

# TCP/IP的三次握手与四次挥手


客户端与服务端的连接

## TCP的6种标志位
这些标志位被写在报文段的首位，用于识别报文段的作用
- SYN(synchronous建立联机)
- ACK(acknowledgement 确认)
- PSH(push传送)
- FIN(finish结束)
- RST(reset重置)
- URG(urgent紧急)
- Sequence number(顺序号码)
- Acknowledge number(确认号码)

## TCP三次握手


1. 客户端TCP首先向服务端的TCP发送一段特殊的TCP报文段，该报文段不包含任何的应用层数据,但是在报文段的首部标识了一个标志位 也就是`SYN=1` 因此我们把这个特殊的报文段叫做`SYN`报文段, 另外客户端TCP还要自带一个标识自己身份的id号，也就是`seq=client_isn` ,并将此编号放置于该起始的TCP SYN报文段的序号中。该报文段会被封装在一个IP数据报中，发送给服务器
2. 服务器接收到带有TCP-SYN的报文段的IP数据报后，服务器会从该数据报中提取TCP-SYN段报文，为该TCP连接分配TCP缓存和变量，并向该客户TCP发送允许连接的报文段。这个允许连接的报文段也不包含应用层数据。但是和客户端首次发来的TCP报文段一样，在他的头部也带有重要的标识。 首先是返回确认信息`ACK=client_isn+1`，再接着是建立连接`SYN=1`,最后是自己服务端的初始序号`seq=service_id`
3. 客户端在接收到`SYNACK`报文段之后吗，也要给改连接分配`缓存和变量`.并且客户机还要想服务器发送另外一个报文段 。 该报文段的主要内容为(`SYN=0`,`ACK=service_id+1`,`seq=client_id+1`)因为连接已经建立，所以SYN是被置为0的  并且，在第三个阶段中，报文段可以在负载中携带客户到服务器的数据
   
连接之后该做的事:

连接完成之后，客户和服务器之间就可以正常接发报文段了，并且在之后的每一个报文段中 SYN的比特都将被置为0

```bash
# 总结一下
# client -> service 1
SYN=1 ,seq=client_id
# service  -> client 2
SYN=1 ,seq=service_id ask=client_id+1
# client -> service 3
SYN=0,seq=client+1 ask=service_id+1

# client -> service n or  service -> client n
SYN=0
```

注意到三次握手的每一段都需要分配一定的缓存和变量来处理连接,这样的设定则会导致`SYN泛洪`


## TCP四次挥手

TCP的四次挥手包含了四个阶段，这里假设客户端发起停止

1. 客户端会发起一段特殊的TCP报文段，头部标识信息为`FIN=1` 
2. 服务器收到报文段后，返回一个确认报文段 `ACK`
3. 然后服务器发送一个终止报文段 `FIN=1`
4. 客户端接收到以后 发送一个确认报文段`ACK`
5. 一定时间后，客户端关闭 两个主机上的所有资源都被释放

## TCP的生命周期
在一个TCP连接中 TP的状态不断的在变迁


### 客户端TCP生命周期
![2020-07-22-14-43-41](http://noback.upyun.com/2020-07-22-14-43-41.png)

### 服务端TCP生命周期
![2020-07-22-14-56-02](http://noback.upyun.com/2020-07-22-14-56-02.png)




## SYN洪泛
之前有说过使用TCP三次握手建立连接的时候，每个阶段都需要分配一定的缓存和变量来维持状态
1. 服务端发送SYN
2. 客户端响应SYN 发送ack给服务端，并等待客户端的ACK报文段
如果客户发送ACK来完成三次握手的第三步，服务端在一分钟之后会终止该半开连接并回收资源

产生SYN洪泛，攻击者发送大量的TCP-SYN报文段，假设在一分钟之内发送了不可想象数量的TCPSYN报文段，但是不完成第三次握手的步骤，则在服务端会不断的为这些半开连接分配资源。短时间内这些资源不会被释放，于是会导致服务器的链接资源被消耗殆尽

### SYN COOKIE

<font color='red'>SYN COOKIE是怎么体现的 是seq=service_id 这个来体现的嘛</font>

SYN COOKIE是用来防御SYN洪泛的一种手段
1. 当服务器接收到SYNseq报文段之后，并不会去生成一个半开连接，也就是不会去分配缓存,服务器会生成一个初始的TCP序列号(序列号是SYN报文段的源和目的的IP地址与 端口号以及仅有改服务器知道的机器码通过散列函数生成的序列号)，服务器则发送具有这种特殊的序列号。 服务器并不记忆该cookie或任何对于SYN的其他状态信息
2. 当合法的用户接收到SYNack报文段之后 会返回一个SYNack。当服务器收到该Ack之后，需要验证该ack与前面发送的SYN济宁对应
   
:::
之前说了服务器在使用SYN-COOKIE之后不维护有关SYN报文段的记忆的，那他是怎么来验证客户端的合法性的呢
:::

这里客户端发送了的SYNACK报文段中包含了一下几个内容
```bash
SYN=1
seq=service_id+1
ask=client_id+1
```
其中`seq=service_id+1`就是用来验证合法性的，当客户端接收到这一段报文之后，会和之前一样生成一段TCP序列号，再将这段序列号+1 与seq中的进行比较，如果相符合，则表示报文段合法，如果不符合则直接丢弃
合法之后，服务器会生成一个具有套接字的全开连接

这样在一方面可以验证第二阶段中SYNACK报文段的合法性  又可以在第一阶段防止出现额外的半开连接 而导致服务器内存崩溃


 
##  RST标志位
当客户端发起一个SYN标志位到服务端时，但是服务端不接受这一段报文段，比如端口不被允许，则会发送一段标志位`RST=1`的报文段