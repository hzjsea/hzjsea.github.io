## loopback接口与Null接口简介

 LoopBack接口是一种虚拟接口。LoopBack接口创建后，除非手工关闭该接口，否则其物理层永远处于up状态。鉴于这个特点，LoopBack接口的应用非常广泛，主要表现在： 

1） 该接口的地址常被配置为设备产生的IP报文的源地址。因为LoopBack接口地址稳定且是单播地址，所以通常将LoopBack接口地址视为设备的标志。在认证或安全等服务器上设置允许或禁止携带LoopBack接口地址的报文通过，就相当于允许或禁止某台设备产生的报文通过，这样可以**简化报文过滤规则**。但需要注意的是，将LoopBack接口地址用于IP报文源地址时，需借助路由配置来确保LoopBack接口到对端的路由可达。另外，<font color="blue">任何送到LoopBack接口的IP报文都会被认为是送往设备本身的，设备将不再转发这些报文 </font>

2） 该接口常用于动态路由协议。比如：在一些动态路由协议中，当没有配置Router ID时，将选取所有LoopBack接口上数值最大的IP地址作为Router ID；在BGP协议中，为了使BGP会话不受物理接口故障的影响，可将发送BGP报文的源接口配置成LoopBack接口 

## LoopBack接口

```bash
system-view # 进入系统视图
interface loopback [number] # 创建loopback接口并进入loopback接口视图
description text # 接口描述
bandwith [value]  # 配置接口的期望带宽
default #恢复接口配置
undo shutdown  # 开启loopback接口
```

## loopback环路检测配置
网络连接错误或配置错误都容易导致二层网络中出现转发环路，使设备对广播、组播以及未知单播报文进行重复发送，造成网络资源的浪费甚至导致网络瘫痪。为了能够及时发现二层网络中的环路，以避免对整个网络造成严重影响，需要提供一种检测机制，使网络中出现环路时能及时通知用户检查网络连接和配置情况，这种机制就是环路检测机制。当网络中出现环路时，环路检测机制通过生成日志信息来通知用户，并可根据用户事先的配置来选择是否关闭出现环路的端口。

#### 环路检测配置
全局环路检测
```bash
system-view
# 环路检测
loopback-detection global enable vlan all
```
端口环路检测
```bash
system-view
interface interface-type interface-number
loopback-detection enable vlan all
```
检测时间
```bash
loopback-detection interval-time interval
```
全局配置环路检测处理模式
```bash
system-view
loopback-detection global action shutdown
```
环路检测和显示
```bash
display loopback-detection
```



## NULL接口

 NULL接口是一种虚拟接口。它永远处于up状态，但不能转发报文，也不能配置IP地址和链路层协议。Null接口为设备提供了一种过滤报文的简单方法——将不需要的网络流量发送到NULL接口，从而免去配置ACL的复杂工作。比如，在路由中指定到达某一网段的下一跳为NULL接口，则任何送到该网段的网络数据报文都会被丢弃。 

```bash
system-view
interface null 0 # 进入null接口视图
description text # 描述
default  # 重置
```

