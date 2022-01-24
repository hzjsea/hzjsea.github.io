## H3c交换机状态检测命令

## 风扇
查看风扇状态
```bash
system-view 
display fan
```
损坏状态:
```text
[ONI-KC-I06-A02]display fan
 Slot 1
      FAN    1
      State    : Fault
```
正常运行状态
```text
[ONI-KC-I05-A01]display fan
 Slot 1
      FAN    1
      State    : Normal
```
## arp表项
查看arp表项
```bash
system-view
display arp
```
结果:
```text
  Type: S-Static   D-Dynamic   O-Openflow   R-Rule   M-Multiport  I-Invalid
IP address      MAC address    VID        Interface/Link ID        Aging Type 
10.0.0.62       0080-91b5-ae52 1000       GE1/0/48                 402   D    
10.0.0.130      84d9-317f-57c4 1000       GE1/0/48                 1197  D    
10.0.0.133      70f9-6d0d-e1c3 1000       GE1/0/48                 1078  D    
10.0.0.134      70f9-6d0d-e1c0 1000       GE1/0/48                 611   D    
10.0.0.135      5cdd-707a-2215 1000       GE1/0/48                 1078  D    
10.0.0.136      5cdd-707a-2212 1000       GE1/0/48                 609   D    
10.0.0.137      487a-da9c-21aa 1000       GE1/0/48                 559   D    
10.0.0.138      487a-da9c-20d2 1000       GE1/0/48                 213   D    
10.0.0.150      70f9-6ddc-7719 1000       GE1/0/48                 612   D    
10.0.0.161      5cdd-7008-82d8 1000       GE1/0/48                 864   D    
10.0.0.162      3ce5-a6af-fb39 1000       GE1/0/48                 612   D    
10.0.0.163      70f9-6d4a-d07d 1000       GE1/0/48                 610   D    
10.0.0.164      5cdd-7095-0c41 1000       GE1/0/48                 609   D    
10.0.0.165      7425-8ad9-0d84 1000       GE1/0/48                 611   D    
10.0.0.166      7425-8ad9-0924 1000       GE1/0/48                 611   D    
10.0.0.167      70f9-6d4a-c4ed 1000       GE1/0/48                 610   D    
10.0.0.168      5cdd-70a5-bc53 1000       GE1/0/48                 612   D    
10.0.0.169      50da-00f2-0c06 1000       GE1/0/48                 610   D    
10.0.0.170      845b-125c-381b 1000       GE1/0/48                 1185  D    
10.0.0.171      586a-b106-39f4 1000       GE1/0/48                 1071  D    
10.0.0.173      586a-b106-6994 1000       GE1/0/48                 1071  D
```
其中包括IP地址 mac地址等

## 接口信息
```bash
system-view 
display interface brief 
```

## 环路检测显示和维护
```bash
system-view
display loopback-detection
```

## 时间显示
```bash
display clock
```
**修改时间**
1.获取NTP,由网络时钟服务器获取时间
clock protocol ntp mdc [mdc-ip]
2.不设置时钟服务器
clock protoco none

```bash
<H3C>clock protocol none　　　　　　　　　　　　　　#关闭protocol ,缺省情况下，默认开启，由缺省MDC获取系统时间

<H3C>clock datetime hh:mm:ss year/month/day　　　　 #设置时间

[H3C]clock timezone beijing add 8　　　　　　　　　　#设置时区 北京 在UTC偏移8小时

[H3C]quit
```



##  显示Switch A上全局和所有接口的LLDP状态信息。 

```bash
[SwitchA] display lldp status
```

结果:

```bash
LLDP status information of port 1 [Ten-GigabitEthernet1/0/1]:
LLDP agent nearest-bridge:
Port status of LLDP            : Enable
Admin status                   : RX_Only
Trap flag                      : No
MED trap flag                  : No
Polling interval               : 0s
Number of LLDP neighbors       : 1
Number of MED neighbors        : 1
Number of CDP neighbors        : 0
Number of sent optional TLV    : 21
Number of received unknown TLV : 0
```

## FIB表的信息(ip转发信息表)

```bash
[hzj_test]display fib 
```

结果：

```bash
[hzj_test]display  fib 

Destination count: 17 FIB entry count: 17

Flag:
  U:Useable   G:Gateway   H:Host   B:Blackhole   D:Dynamic   S:Static
  R:Relay     F:FRR 

Destination/Mask   Nexthop         Flag     OutInterface/Token       Label
0.0.0.0/32         127.0.0.1       UH       InLoop0                  Null       
10.0.0.0/20        10.0.5.124      U        Vlan1000                 Null       
10.0.0.0/32        10.0.5.124      UBH      Vlan1000                 Null       
10.0.1.19/32       10.0.1.19       UH       Vlan1000                 Null       
10.0.1.54/32       10.0.1.54       UH       Vlan1000                 Null       
10.0.1.70/32       10.0.1.70       UH       Vlan1000                 Null       
10.0.1.71/32       10.0.1.71       UH       Vlan1000                 Null       
10.0.2.56/32       10.0.2.56       UH       Vlan1000                 Null       
10.0.5.124/32      127.0.0.1       UH       InLoop0                  Null       
10.0.15.255/32     10.0.5.124      UBH      Vlan1000                 Null       
127.0.0.0/8        127.0.0.1       U        InLoop0                  Null       
127.0.0.0/32       127.0.0.1       UH       InLoop0                  Null       
127.0.0.1/32       127.0.0.1       UH       InLoop0                  Null       
127.255.255.255/32 127.0.0.1       UH       InLoop0                  Null       
224.0.0.0/4        0.0.0.0         UB       NULL0                    Null       
224.0.0.0/24       0.0.0.0         UB       NULL0                    Null       
255.255.255.255/32 127.0.0.1       UH       InLoop0                  Null    
```

FIB表中包含了下列关键项：

- Destination：目的地址。用来标识IP报文的目的地址或目的网络。
- Mask：网络掩码。与目的地址一起来标识目的主机或交换机所在的网段的地址。将目的地址和网络掩码“逻辑与”后可得到目的主机或交换机所在网段的地址。例如：目的地址为192.168.1.40、掩码为255.255.255.0的主机或交换机所在网段的地址为192.168.1.0。掩码由若干个连续“1”构成，既可以用点分十进制法表示，也可以用掩码中连续“1”的个数来表示。
- NextHop：转发的下一跳地址。
- Flag：路由的标志。
- OutInterface：转发接口。指明IP报文将从哪个接口转发。
- Token：LSP（Label Switched Path，标签交换路径）索引号。
- Label：内层标签值


## 风扇
查看风扇状态
```bash
system-view 
display fan
```
损坏状态:
```text
[ONI-KC-I06-A02]display fan
 Slot 1
      FAN    1
      State    : Fault
```
正常运行状态
```text
[ONI-KC-I05-A01]display fan
 Slot 1
      FAN    1
      State    : Normal
```
## arp表项
查看arp表项
```bash
system-view
display arp
```
结果:
```text
  Type: S-Static   D-Dynamic   O-Openflow   R-Rule   M-Multiport  I-Invalid
IP address      MAC address    VID        Interface/Link ID        Aging Type 
10.0.0.62       0080-91b5-ae52 1000       GE1/0/48                 402   D    
10.0.0.130      84d9-317f-57c4 1000       GE1/0/48                 1197  D    
10.0.0.133      70f9-6d0d-e1c3 1000       GE1/0/48                 1078  D    
10.0.0.134      70f9-6d0d-e1c0 1000       GE1/0/48                 611   D    
10.0.0.135      5cdd-707a-2215 1000       GE1/0/48                 1078  D    
10.0.0.136      5cdd-707a-2212 1000       GE1/0/48                 609   D    
10.0.0.137      487a-da9c-21aa 1000       GE1/0/48                 559   D    
10.0.0.138      487a-da9c-20d2 1000       GE1/0/48                 213   D    
10.0.0.150      70f9-6ddc-7719 1000       GE1/0/48                 612   D    
10.0.0.161      5cdd-7008-82d8 1000       GE1/0/48                 864   D    
10.0.0.162      3ce5-a6af-fb39 1000       GE1/0/48                 612   D    
10.0.0.163      70f9-6d4a-d07d 1000       GE1/0/48                 610   D    
10.0.0.164      5cdd-7095-0c41 1000       GE1/0/48                 609   D    
10.0.0.165      7425-8ad9-0d84 1000       GE1/0/48                 611   D    
10.0.0.166      7425-8ad9-0924 1000       GE1/0/48                 611   D    
10.0.0.167      70f9-6d4a-c4ed 1000       GE1/0/48                 610   D    
10.0.0.168      5cdd-70a5-bc53 1000       GE1/0/48                 612   D    
10.0.0.169      50da-00f2-0c06 1000       GE1/0/48                 610   D    
10.0.0.170      845b-125c-381b 1000       GE1/0/48                 1185  D    
10.0.0.171      586a-b106-39f4 1000       GE1/0/48                 1071  D    
10.0.0.173      586a-b106-6994 1000       GE1/0/48                 1071  D
```
其中包括IP地址 mac地址等

## 接口信息
```bash
system-view 
display interface brief 
```

## 环路检测显示和维护
```bash
system-view
display loopback-detection
```