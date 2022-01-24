# 网络隧道技术

协议： http://www.h3c.com/cn/d_201311/804105_30005_0.htm#feedback 

GRE（Generic Routing Encapsulation，通用路由封装）协议用来对某些网络层协议（如IP和IPX）的数据报文进行封装，使这些被封装的数据报文能够在另一个网络层协议（如IP）中传输。封装后的数据报文在网络中传输的路径，称为GRE隧道。GRE隧道是一个虚拟的点到点的连接，其两端的设备分别对数据报文进行封装及解封装。 

## GRE的封装

<font color="red">GRE整个协议只是在网络层进行的吗？是的</font>

![img](C:\Users\alpaca\Desktop\script\img\clip_image001.png)

GRE封装的报文格式：
- 净荷数据(Payload packet)指的就是需要封装和传输的数据报文。
- GRE头(GRE header):  系统收到净荷数据后，在净荷数据上添加GRE头，使其成为GRE报文。对净荷数据进行封装的GRE协议，称为封装协议(Encapsulation Protocol)
-  传输协议的报文头（Delivery header）：负责转发封装后报文的网络协议，称为传输协议(Delivery Protocol或者Transport Protocol)。在GRE报文上需要增加传输协议的报文头，以便传输协议对封装后的报文进行转发处理。 

比如现在我们需要IPV6 over IPV4的 则乘客协议为IPV6 封装协议为GRE  传输协议为IPV4

- GRE over IPv4：传输协议为IPv4，乘客协议为任意网络层协议。
- GRE over IPv6：传输协议为IPv6，乘客协议为任意网络层协议

## 加封装的过程

首先我们需要了解下目的，X协议要通过隧道达到

![](http://www.h3c.com/cn/res/201311/19/20131119_1709876_image005_804105_30005_0.png)

这里介绍一下整个隧道从开始到结束的过程，

**加封装**

首先DeviceA收到Group1的接口的X协议报文后，首先交由X协议处理；

X协议根据报文头中的目的地址，确定如何路由此包

若报文的目的地址要经过GRE隧道才能到达，则Device A将此报文发给相应的Tunnel接口；

Tunnel接口收到此报文后先进行GRE封装，再封装IP报文头；

Device A根据封装的IP报文头的目的地址，查找路由表，对封装后的报文进行转发

通俗的讲，为了让A端的信息传递到B端，必须要在他的外层包装一层B端的通行证，为了识别这个通行证是谁加上去的，还要在他的内层加入GRE的头部

**解封装**

Device B从Tunnel接口收到IP报文，检查目的地址；

如果目的地是本设备，且IP报文头中的协议号为47（表示封装的报文为GRE报文），则Device B剥掉此报文的IP报头，交给GRE协议处理；

GRE协议完成相应的处理后，剥掉GRE报头，将报文交给X协议进行后续的转发处理。

## 交换机配置流程

```bash
system-view
interface Tunnel interface-number mode gre # 创建隧道tunnel，模式为gre  创建GRE over IPv4
interface Tunnel interface-number mode gre ipv6 # 创建GRE over IPv6
# 为了保证隧道的通畅，在隧道的两端应配置相同的隧道模式，否则可能会造成报文传输失败
# 设置乘客协议，如果乘客协议为没有设置
source {ip | interface-type interface-number}# 设置隧道的源地址或源接口
destination ip # 设置隧道的目的端地址
- keepalive [interval[times]] # 开启GRE的keepalive功能
- tunnel dfbit enable # 设置封装后隧道报文的不分片标志
- tunnel discard ipv4-compatible-packet # 设置丢弃含有IPv4兼容IPv6地址的IPv6报文
```

## GRE显示和维护

```bash
display interface [tunnel [number] ][brief] # ipv4
display ipv6 interface [tunnel [number] ][brief] # ipv4
```

