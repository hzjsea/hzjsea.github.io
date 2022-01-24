---
title: go第三方库gosnmp
date: 2021-03-09 10:27:48
categories:  [golang]
tags: [golang]
---


<!--more-->


# go第三方库gosnmp

## 什么是snmp
SNMP：“简单网络管理协议”，用于网络管理的协议。SNMP用于网络设备的管理。
SNMP的工作方式：被SNMP操作的机器会会设置SNMP权限  `Community` 和 `ContextEngineID`
管理员需要向设备获取数据，管理员可以像设备进行三类操作
- `读操作`:管理员需要向设备执行设置操作，
- `写操作`:设备需要在重要状况改变的时候，
- `Trap操作`:向管理员通报事件的发生

> SNMP统一性

大部分内容统一，少部分功能独有作为生产厂家设备特性

SNMP 被设计为工作在TCP/IP协议族上。SNMP基于TCP/IP协议工作，对网络中支持SNMP协议的设备进行管理。所有支持SNMP协议的设备都提供 SNMP这个统一界面，使得管理员可以使用统一的操作进行管理，而不必理会设备是什么类型、是哪个厂家生产的。如下图
![](http://noback.upyun.com/2021-03-09-13-47-30.png!)


> 如何通信

SNMP通过MIB进行通信
管理站与代理端通过MIB进行接口 统一，MIB定义了设备中的被管理对象。管理站和代理都实现了相应的MIB对象，使得双方可以识别对方的数据，实现通信。管理站向代理申请MIB中定义的 数据，代理识别后，将管理设备提供的相关状态或参数等数据转换为MIB定义的格式，应答给管理站，完成一次管理操作

> 什么是MIB

MIB管理信息数据库，可以理解为管理者和设备之间通信的语言，
IETF 规定的管理信息库MIB（由中定义了可访问的网络设备及其属性，由对象识别符（OID：Object Identifier）唯一指定。MIB是一个树形结构，SNMP协议消息通过遍历MIB树形目录中的节点来访问网络中的设备。下图给出了NMS系统中 SNMP可访问网络设备的对象识别树（OID：Object Identifier）结构
![](https://noback.upyun.com/2021-03-09-13-58-35.png!)

> SNMP中的消息类型

SNMP中定义了五种消息类型：Get-Request、Get-Response、Get-Next-Request、Set-Request、Trap

- Get-Request 、Get-Next-Request与Get-Response
SNMP 管理站用Get-Request消息从拥有SNMP代理的网络设备中检索信息，而SNMP代理则用Get-Response消息响应。Get- Next-Request用于和Get-Request组合起来查询特定的表对象中的列元素。如：首先通过下面的原语获得所要查询的设备的接口数：
{iso org(3) dod(6) internet(1) mgmt(2) mib(1) interfaces(2) ifNumber(2)}
后再通过下面的原语，进行查询（其中第一次用Get-Request，其后用Get-Next-Request）：
{iso org(3) dod(6) internet(1) mgmt(2) mib(1) interfaces(2) ifTable(2)}
- Set-Request
SNMP管理站用Set-Request 可以对网络设备进行远程配置（包括设备名、设备属性、删除设备或使某一个设备属性有效/无效等）。
- Trap
SNMP代理使用Trap向SNMP管理站发送非请求消息，一般用于描述某一事件的发生。

https://www.cnblogs.com/lsgxeva/p/9220949.html


> PDU

Protocol Data Unit(协议数据单元)，它是网络中传送的数据包。每一种SNMP操作，物理上都对应一个PDU
可以理解为SNMP发送MIB后返回一段PDU信息

如下是一段go的代码，连接设备获取pdu信息
```golang
package main

import (
	"fmt"
	g "github.com/gosnmp/gosnmp"
	"log"
	"time"
)

func main() {

	var ctx = &g.GoSNMP{
		Port:               161,
		Transport:          "udp",
		Community:          Community,
		Version:            g.Version2c,
		Timeout:            time.Duration(2) * time.Second,
		Retries:            3,
		ExponentialTimeout: true,
		MaxOids:            g.MaxOids,
		Target:				Target,
		ContextEngineID:   	ContextEngineID,
		Conn:  nil,
	}

	err := ctx.Connect() // 不连接的话, Conn就会置空
	if err != nil{
		log.Fatal("connect error")
	}

	defer ctx.Conn.Close()
	oids := []string{
		//"1.3.6.1.2.1.1.5.0",
		"1.3.6.1.2.1.80.1.3.1.4",
	}

	err = ctx.BulkWalk(oids[0], func(dataUnit g.SnmpPDU) error {
		fmt.Println(dataUnit)
		return nil
	})

	if err != nil{
		log.Fatal(err)
	}
}

// result = {.1.3.6.1.2.1.80.1.3.1.4.3.117.112.120.11.104.117.97.119.101.105.99.108.111 Gauge32 7}
```


## linux版本-snmpwalk
```bash
snmpwalk -v 2c -c ${Community} 172.16.33.4 1.3.6.1.2.1.80.1.3.1.5
```


## gosnmp解析 
go里面的一个snmp操作库
```go
go get github.com/gosnmp/gosnmp
```
### snmp结构体
```golang
type GoSNMP struct {
	Conn net.Conn // 一个实现了net.Conn接口的对象
	Target string // 目标机器的IP地址
	Port uint16 // 目标机器的端口
	Transport string // 使用的协议 tcp udp 一般使用udp
	Community string  // 相当于秘钥
    Version SnmpVersion // snmp 版本 默认是v2c 还是v3   v1的太老了
	Context context.Context // 上下文
	Timeout time.Duration // snmp 超时
	Retries int // 重播次数
	ExponentialTimeout bool 
	Logger Logger
	PreSend func(*GoSNMP)
	OnSent func(*GoSNMP)
	OnRecv func(*GoSNMP)
	OnRetry func(*GoSNMP)
	OnFinish func(*GoSNMP)
	MaxOids int
	MaxRepetitions uint32
	NonRepeaters int
	UseUnconnectedUDPSocket bool
	AppOpts map[string]interface{}
	MsgFlags SnmpV3MsgFlags
	SecurityModel SnmpV3SecurityModel
	SecurityParameters SnmpV3SecurityParameters
	ContextEngineID string
	ContextName string
}
```

> 初始化一个默认配置的SNMP对象
```golang
// 源码
var Default = &GoSNMP{
	Port:               161,
	Transport:          udp,
	Community:          "public",
	Version:            Version2c,
	Timeout:            time.Duration(2) * time.Second,
	Retries:            3,
	ExponentialTimeout: true,
	MaxOids:            MaxOids,
}
// 使用
func main(){
    var ctx = g.Default
}

// 重新声明
import (
	"fmt"
	g "github.com/gosnmp/gosnmp"
	"time"
)

func main() {

	var ctx = &g.GoSNMP{
		Port:               161,
		Transport:          "udp",
		Community:          "public",
		Version:            g.Version2c,
		Timeout:            time.Duration(2) * time.Second,
		Retries:            3,
		ExponentialTimeout: true,
		MaxOids:            g.MaxOids,
		Target:				"172.16.33.4",
		ContextEngineID:    "public",
	}
}
```

### BulkWalk
使用方法如下
```golang
func main() {

	var ctx = &g.GoSNMP{
		Port:               161,
		Transport:          "udp",
		Community:          Community,
		Version:            g.Version2c,
		Timeout:            time.Duration(2) * time.Second,
		Retries:            3,
		ExponentialTimeout: true,
		MaxOids:            g.MaxOids,
		Target:				Target,
		ContextEngineID:   	ContextEngineID,
		Conn:  nil,
	}

	err := ctx.Connect() // 不连接的话, Conn就会置空
	if err != nil{
		log.Fatal("connect error")
	}

	defer ctx.Conn.Close()
	oids := []string{
		"1.3.6.1.2.1.1.5.0",
		"1.3.6.1.2.1.80.1.3.1.4",
	}

	err = ctx.BulkWalk(oids[0], func(dataUnit g.SnmpPDU) error {
		fmt.Println(dataUnit)
		return nil
	})

	if err != nil{
		log.Fatal(err)
	}
}
```
支持单个执行，调用匿名函数执行

### BulkWalkAll
如果节点数靠前，返回的是一大串的信息，可以用BulkWalkAll
```golang
package main

import (
	"fmt"
	g "github.com/gosnmp/gosnmp"
	"log"
	"time"
)

func main() {

	var ctx = &g.GoSNMP{
		Port:               161,
		Transport:          "udp",
		Community:          Community,
		Version:            g.Version2c,
		Timeout:            time.Duration(2) * time.Second,
		Retries:            3,
		ExponentialTimeout: true,
		MaxOids:            g.MaxOids,
		Target:				Target,
		ContextEngineID:   	ContextEngineID,
		Conn:  nil,
	}

	err := ctx.Connect() // 不连接的话, Conn就会置空
	if err != nil{
		log.Fatal("connect error")
	}

	defer ctx.Conn.Close()
	oids := []string{
		//"1.3.6.1.2.1.1.5.0",
		"1.3.6.1.2.1.80.1", // 返回一串结果
		"1.3.6.1.2.1.80.1.3.1.5", // 返回单个结果

	}
	reslut,err := ctx.BulkWalkAll(oids[0])
	for i,value := range reslut{
		fmt.Println(i, value)
	}
	if err != nil{
		log.Fatal(err)
	}
}
```



> 请求的包

```bash
// Get sends an SNMP GET request
func (x *GoSNMP) Get(oids []string) (result *SnmpPacket, err error) {
	oidCount := len(oids)
	if oidCount > x.MaxOids {
		return nil, fmt.Errorf("oid count (%d) is greater than MaxOids (%d)",
			oidCount, x.MaxOids)
	}
	// convert oids slice to pdu slice
	var pdus []SnmpPDU
	for _, oid := range oids {
		pdus = append(pdus, SnmpPDU{oid, Null, nil})
	}
	// build up SnmpPacket
	packetOut := x.mkSnmpPacket(GetRequest, pdus, 0, 0)
	return x.send(packetOut, true)

```