##  # OSPF原理以及配置

#### ospf文档关键字
hello报文
lsr 链路状态请求
lsu 链路状态更新
lsack 链路状态确认
lsa 链路状态通告

## ospf简介

OSPF（Open Shortest Path First，开放最短路径优先）是IETF（Internet Engineering Task Force，互联网工程任务组）组织开发的一个基于链路状态的内部网关协议。目前针对IPv4协议使用的是OSPF Version 2。 

## ospf特点
...
## ospf报文类型

ospf协议报文直接封装成IP报文，协议号为89，并且ospf存在5种类型的协议报文

- hello报文
- DD:(database description)数据库描述
- LSR:(link state request) 链路状态请求
- LSU:(link state update) 链路状态更新
- LSAck:(link state Acknowledgement) 链路状态确认


Hello报文：周期性发送的报文，顾名思义hello打招呼，用来发现和维持ospf邻居关系，以及进行DR和BDR的选举

DD(database description 数据库描述)报文: 描述了本地LSDB(link state database 链路状态数据库)中每一条LSA(link state advertisement 链路状态通告)的摘要信息，用于两台路由器进行数据库同步

LSR（Link State Request，链路状态请求）报文：向对方请求所需的LSA。两台路由器互相交换DD报文之后，得知对端的路由器有哪些LSA是本地的LSDB所缺少的，这时需要发送LSR报文向对方请求所需的LSA

LSU（Link State Update，链路状态更新）报文：向对方发送其所需要的LSA

LSAck（Link State Acknowledgment，链路状态确认）报文：用来对收到的LSA进行确认


## ospf工作原理

首先当路由器开启ospf后，路由器之间就会相互发送hello报文，建立邻里关系，hello报文中包含一些路由器和链路的相关信息，当然hello报文最终的目的是维持邻居表，并不只是建立完成后就会消失，在默认情况下路由器没10s就会发送一次hello报文，以检测链路状态，保证链路时钟是正常的。hello报文不断的刷新和维持着邻里关系，这是ospf成功建立的基础

一旦邻居表建立完成，路由器之间就会相互发送LSA(链路状态通告) LSA会告诉自己的邻居路由器和自己相连的链路的状态，最后形成网络的拓扑表。ospf建立邻居表全过程( https://www.jianshu.com/p/da405886e70f ),最后形成LSDB(链路状态数据库，即多个拓扑表)最后通过spf算法，计算形成路由表

## LSA 链路状态通告类型

OSPF中对链路状态信息的描述都是封装在LSA中发布出去，常用的LSA有以下几种类型：

- Router LSA（Type-1）：由每个路由器产生，描述路由器的链路状态和开销，在其始发的区域内传播。 
- Network LSA（Type-2）：由DR产生，描述本网段所有路由器的链路状态，在其始发的区域内传播。
- Network Summary LSA（Type-3）：由**ABR**（Area Border Router，区域边界路由器）产生，描述区域内某个网段的路由，并通告给其他区域。 
- ASBR Summary LSA（Type-4）：由ABR产生，描述到ASBR（Autonomous System Boundary Router，自治系统边界路由器）的路由，通告给相关区域。   
- AS External LSA（Type-5）：由ASBR产生，描述到AS（Autonomous System，自治系统）外部的路由，通告到所有的区域（除了Stub区域和NSSA区域）。  
- NSSA External LSA（Type-7）：由NSSA（Not-So-Stubby Area）区域内的ASBR产生，描述到AS外部的路由，仅在NSSA区域内传播。
- Opaque LSA：用于OSPF的扩展通用机制，目前有Type-9、Type-10和Type-11三种。其中，Type-9 LSA仅在本地链路范围进行泛洪，用于支持GR（Graceful Restart，平滑重启）的Grace LSA就是Type-9的一种类型；Type-10 LSA仅在区域范围进行泛洪，用于支持MPLS TE的LSA就是Type-10的一种类型；Type-11 LSA可以在一个自治系统范围进行泛洪

## ospf区域

随着网络规模日益扩大，当一个大型网络中的路由器都运行OSPF协议时，LSDB会占用大量的存储空间，并使得运行SPF（Shortest Path First，最短路径优先）算法的复杂度增加，导致CPU负担加重。
在网络规模增大之后，拓扑结构发生变化的概率也增大，网络会经常处于“振荡”之中，造成网络中会有大量的OSPF协议报文在传递，降低了网络的带宽利用率。更为严重的是，每一次变化都会导致网络中所有的路由器重新进行路由计算。
OSPF协议通过将自治系统划分成不同的区域来解决上述问题。区域是从逻辑上将路由器划分为不同的组，每个组用区域号来标识。
 区域的边界是路由器，而不是链路。一个路由器可以属于不同的区域，但是一个网段（链路）只能属于一个区域，或说每个运行OSPF的接口必须指明属于哪一个区域。划分区域后，可以在区域边界路由器上进行路由聚合，以减少通告到其他区域的LSA数量，还可以将网络拓扑变化带来的影响最小化。 


#### 骨干区域,虚连接
在ospf中，如果存在区域，则必须存在一个are0的骨干区域，该区域的作用是   ，另外还要存在多个子区域，如area 1 2 3 4 5等。



## ospf路由类型
1.区域边界路由器(ABR)：用来连接Area0和其他区域的路由器
2.内部路由器 ： 保存自己区域的链路状态信息
3.自治边界路由器(ASBR)用来连接ospf的AS与外部其他的路由器，也就是说连接不是ospf协议的路由器
![](https://s1.51cto.com/images/blog/201808/02/94872d3db534043b3f1739a947998557.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

4.ospf路由的建邻过程
![](https://s1.51cto.com/images/blog/201808/02/cde03b45cf5bff8333dec8cbeea69c47.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

## 交换机配置ospf步骤

```bash
system-view 
- router id [router-id] # 配置全局router id
ospf [process-id] router-id [router-id] vpn-instance [instance-name] # 创建ospf 并进入ospf视图
- description [desc] # 描述
area area-id # 配置ospf区域，
- description [desc] # 描述
network [ip-address wildcard-mask] # 配置区域所包含的网段并在指定网段的借口上启用ospf  ip地址+反掩码

```

## 配置ospf接口网络类型为广播

<font color="red">ospf接口网络类型是什么</font>
```bash
system-view 
interface [interface-type][interface-number]
ospf network-type broadcast # 配置ospf接口网络类型为广播
- ospf dr-priority [priority] # 配置ospf接口的路由器优先级
```

## 配置ospf接口网络类型为p2p
```bash
system-view
interface [interface-type][interface-number]
ospf network-type p2p # 配置ospf接口的网络类型为p2p
```

## 配置ospf接口的开销值
```bash
system-view
interface [type][number]
ospf cost value #
```

ospf配置知识点
https://blog.51cto.com/13746824/2153847



## h3c ospf基本组网
![](http://img.noback.top/blog/imgh3c基本组网.png)

1.配置子区域内部交换机
\#switch C
```bash
system-view
router id 10.4.1.1
ospf 
area 1
network 10.2.1.0 0.0.0.255
network 10.4.1.0 0.0.0.255
quit
quit
```
\#switch D
```bash
system-view
router id 10.5.1.1
ospf
area 2 
network 10.3.1.0 0.0.0.255
network 10.5.1.0 0.0.0.255
quit
quit
```

2.配置边界路由器ABR
\#switch A
```bash
system-view
router id 10.2.1.1
ospf 
area 0
network 10.1.1.0 0.0.0.255
quit
area 1 
network 10.2.1.0 0.0.0.255
quit 
quit
```
\#switch B
```bash
system-view 
router id 10.3.1.1
ospf 
area 0 
network 10.3.1.0 0.0.0.255
quit 
area 1
network 10.1.1.0 0.0.0.255
quit
quit
```

## 检测

``bash
display ospf peer verbose # 查看ospf 邻居
display ospf routing
```