---
title: 网络graphviz图
categories:
  - 网络
tags:
  - 网络
abbrlink: fc1589ad
date: 2020-07-20 08:56:53
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [网络graphviz图](#网络graphviz图)
  - [ISO七层模型](#iso七层模型)
  - [TCP/IP5层模型](#tcpip5层模型)
  - [TCP/IP5层模型互动](#tcpip5层模型互动)
  - [三次握手与四次握手模型](#三次握手与四次握手模型)
  - [机房互联对接流程](#机房互联对接流程)

<!-- /code_chunk_output -->
<!-- more -->


# 网络graphviz图

## ISO七层模型
```dot
digraph G{
        label="7层模型"
        node [shape = record];
        client [label="{应用层|运输层|网络层|数据链路层|物理层}"]
}
```
## TCP/IP5层模型

```dot
digraph G{
        label="5层模型"
        node [shape = record];
        client [label="{应用层|运输层|网络层|数据链路层|物理层}"]
}
```

## TCP/IP5层模型互动
```dot
digraph G{
        label="5层模型"
        node [shape = record];
        client [label="{应用层|运输层|网络层|数据链路层|物理层}"]
        
        client:应用层->client:运输层
        client:运输层->client:网络层
        client:网络层->client:运输层
        client:应用层->client:运输层
        client:应用层->client:运输层
}

```



## 三次握手与四次握手模型

```dot
digraph G {
    label="三次握手"
    client -> service [label="syn=1,client_id" color="red"]
    service -> client [label="syn=1,client_id+1 service_id" color="red"]
    client -> service [label="syn=0,client_id+1 service_id+1" color="red"]
}

```

```mermaid
graph LR
    三次握手
    A --syn=1,client_id--> B
    E -- this is text --- F
    A --syn=1,client_id+1,service_id--> B
    B --syn=0,client_id+1,service_id+1--> A
```

```dot
digraph G{
        lable="客户端-TCP生命周期"
        NOSTATUS -> SYN_SENT [label=" 发送一条TCP连接请求 发送SYN=1 q=client_id"]
        SYN_SENT -> ESTABLISHED [label="接收服务端的SYN=1 seq=service_id ask=client_id+1"]
        ESTABLISHED -> ESTABLISHED [label="三次握手的前两个阶段已经完成了连接 所以第三阶段没有状态变化 SYN=0 seq=client_id+1 ask=service_id+1"]
        ESTABLISHED ->  FIN_WAIT_1 [label="发送FIN"]
        FIN_WAIT_1 -> FIN_WAIT_2 [label="接收ACK 不发送"]
        FIN_WAIT_2 -> TIME_WAIT [label="接收FIN 发送ACK"]
        TIME_WAIT -> CLOSED [label="等待30秒"]
        CLOSED -> NOSTATUS1
}
```


```dot
digraph G{
        label="服务端-TCP生命周期"
        NOSTATUS->LISTEN [label="创建一个监听套接字"]
        LISTEN -> SYN_RCVD [label="接收SYN，发送SYN&ACK SYN=1;seq=service_id+1 ask=client_id+1"]
        SYN_RCVD -> ESTABLISHED [label="接收ACK"]
        ESTABLISHED -> CLOSED_WAIT [label="接收FIN 发送ACK"]
        CLOSED_WAIT -> CLOSED [label="等待30秒"]
        CLOSED -> NOSTATUS1 
}

```





## 机房互联对接流程

```mermaid
graph TD
        member(对接人员) --> select2{初始互联个数}
        select2 --单个互联单个汇聚口--> 走accress --> setting[single_port port access vlan 2010]

        select2 --> select3{多个互联形式}
        select3 --多个互联access--> 单口access --> setting2[single_port access vlan2010 single_port2 access vlan 2020]
        select3 --多个互联trunk--> 汇聚走trunk --> setting3[B300 port trunk vlan 2010 2020]
        select2 --单个互联多个汇聚口--> 汇聚走access --> setting4[B200 port access vlan 2010]


        linkStyle 0 stroke-width:2px,fill:none,stroke:red;
        linkStyle 1 stroke-width:2px,fill:none,stroke:red;
        linkStyle 3 stroke-width:2px,fill:none,stroke:red;
        linkStyle 4 stroke-width:2px,fill:none,stroke:red;
``` 

