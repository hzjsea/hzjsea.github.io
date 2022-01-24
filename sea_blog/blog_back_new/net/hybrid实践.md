---
title: hybrid实践
categories:
  - 网络
tags:
  - 网络
abbrlink: 8819fa2
date: 2020-04-01 15:57:43
---

learn net
<!-- more -->
http://blog.sina.com.cn/s/blog_64aac6750101q5hi.html

# 交换机连接
连接的方式有一下几种，可以看一下，或者直接搜二层交换机
https://blog.noback.top/posts/9bb9b12b/

交换机连接一共有三种模式，分别是acess trunk 以及hybrid模式

- acess也就是直连 服务器 PC机连接交换机的时候，大都使用acess连接 这是直连，是没有tag信息(vlan信息)的连接
- trunk 就是一个给数据信息打标签的过程，  首先服务器发送信息以acess的形式发送到交换机，交换机根据vlan划分，将端口上的数据转发到各自的vlan下面， 从交换机下联到 另外一个交换机上联的时候， 需要对发出去的数据包打标签
```bash
port trunk permit vlan 1000 # 打标签的过程 打上1000
```
- hybrid


## 三种模式接发方式

端口报文发送

```bash
access 剥掉报文所携带的VLAN标记，进行转发  -> unTag
trunk  首先判断报文所携带的VLAN标记是否和端口的PVID相等。如果相等，则剥掉报文所携带的VLAN标记，进行转发；否则报文将携带原有的VLAN标记进行转发  -> Tag
hybrid 首先判断报文所携带的VLAN标记在本端口需要做怎样的处理。如果是untagged方式转发，unTag  如果是tagged方式转发 Tag 同Trunk一样
```


<font color='red'>untag tag只在发送的时候起作用</font>


端口接收
Access端口  
- 有vlan信息: 丢弃该报文  
- 无vlan信息: 为该报文打上VLAN标记为本端口的PVID
Trunk端口   
- 有vlan信息： 查看是否存在相同vlan 存在转发 ,不存在转发
- 无vlan信息  打上pvid
Hybrid端口  
- 同 trunk        


## 什么是交换机上下联
接收信息的那端是上联
发送信息的那端是下联



<font color='red'>?</font>


## 什么是tag pvid 
数据信息进入交换机以后 被规划到某一个vlan下面，为了识别他们 就要对他们进行标识，标识的信息就是tag头部标识，如果没有tags信息 就说明没有vlan信息

另外一个是pvid  PVID英文为Port-base VLAN ID，是表示网络通信中基于端口的VLAN ID，一个端口可以属于多个VLAN，但是只能有一个PVID，收到一个不带tag头的数据包时，会打上PVID所表示的VLAN号，视同该VLAN的数据包处理。

## 实例
![2020-04-01-15-58-28](http://noback.upyun.com/2020-04-01-15-58-28.png)

这里access的时候 发出去的时候是tag 然后对应上联也有个口，port access vlan 1333 这个时候他也有acess默认，打上pvid 1333



## 实例
```bash
配置端口E0/3为Hybrid端口，能够接收VLAN10和20发过来的报文
[SwitchA]interface Ethernet 0/3
[SwitchA-Ethernet0/3]port link-type hybrid
[SwitchA-Ethernet0/3]port hybrid vlan 10 20 untagged

配置端口G2/1为Hybrid端口，能够接收并透传VLAN10和20发过来的报文
[SwitchA]interface GigabitEthernet 2/1
[SwitchA-GigabitEthernet2/1]port link-type hybrid
[SwitchA-GigabitEthernet2/1]port hybrid vlan 10 20 tagged
```
注意上面那个例子是只能接收，因为他传出去的是untag的，上联直接丢弃了
下面那个能够透传  是因为传出去的时候带有vlan10 和vlan20的vlan信息 
