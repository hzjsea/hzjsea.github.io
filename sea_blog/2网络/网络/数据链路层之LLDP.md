## 数据链路层之LLDP

转载( https://chars.tech/blog/data-link-layer-lldp/ )

随着网络技术的发展，接入网络的设备的种类越来越多，配置越来越复杂，来自不同设备厂商的设备也往往会增加自己特有的功能，这就导致在一个网络中往往会有很多具有不同特性的、来自不同厂商的设备，为了方便对这样的网络进行管理，就需要使得不同厂商的设备能够在网络中相互发现并交互各自的系统及配置信息。 

LLDP（Link Layer Discovery Protocol，链路层发现协议）就是用于这个目的的协议。LLDP 定义在 802.1ab 中，它是一个二层协议，它提供了一种标准的链路层发现方式。LLDP 协议使得接入网络的一台设备可以将其主要的**能力，管理地址，设备标识，接口标识**等信息发送给接入同一个局域网络的其它设备。当一个设备从网络中接收到其它设备的这些信息时，它就将这些信息以MIB的形式存储起来。

这些 MIB 信息可用于发现设备的物理拓扑结构以及管理配置信息。需要注意的是 LLDP 仅仅被设计用于进行信息通告，它被用于通告一个设备的信息并可以获得其它设备的信息，进而得到相关的 MIB 信息。它不是一个配置、控制协议，无法通过该协议对远端设备进行配置，**它只是提供了关于网络拓扑以及管理配置的信息，这些信息可以被用于管理、配置的目的，如何用取决于信息的使用者。**

LLDP框架结构

![data-link-layer-lldp-struct](C:\Users\alpaca\Desktop\script\img\data-link-layer-lldp-struct.png)

此图也表明 LLDP 就是一个信息发现与通告协议，LLDP 的实体主要维护了两个 MIB 库，一个 local system MIB，一个 remote system MIB。从其名字也可以看出，一个用于维护本地相关的设备 MIB 信息，一个用于维护远端设备 MIB 信息。

LLDP 通过与上图中右侧的几个 MIB 库交互来初始化并维护 local system MIB，并将本地的相关信息通告出去；同时当接收到来自其它设备的信息时就将其更新到remote system MIB 中。通过这种工作方式，一个设备就可以将自己的信息通告出去并获得网络中其它设备的相关信息，最终获得反应网络拓扑以及其它配置信息的两个 MIB 库。这两个库可以被其用户用来完成各种功能。需要说明的是**LLDP 信息的通告以及接收处理不受端口的STP状态的影响。**

 https://chars.tech/blog/data-link-layer-lldp/ 

 http://www.h3c.com/cn/d_201903/1157329_30005_0.htm#_Ref153340333 



## 翻译信息







## 交换机配置

 只有当全局和接口上都使能了LLDP功能后，LLDP才会发挥作用

```bash
# 全局开发lldp
lldp global enable
# 进入二/三层以太网接口视图、管理用以太网口视图、二/三层聚合接口视图或IRF物理端口视图 配置功能
interface interface-type interface-number
#看开启lldp功能
lldp enable
```







#### 其他配置

配置lldp桥模式

```bash
system-view 
lldp mode service-bridge
```





为什么给的配置文件中lldp都是undo lldp enable