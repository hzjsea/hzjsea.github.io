---
title: golang标准库net
date: 2020-12-11 11:57:08
categories: [golang,]  
tags: [go,go-library]
---


<!-- more -->

# go库net


官方库 中文文档
https://studygolang.com/pkgdoc
官方库 英文文档
https://golang.org/pkg/net/#IP

## CIDR
CIDR工具
https://www.ipaddressguide.com/cidr
谈到IP地址 子网掩码 主机地址 就离不开CIDR(无类域间路由)
一开始的ipv4地址分配是根据A类 B类 C类地址来分配的，假设一个公司有256台主机，那么他们就会需要一段B类的IP地址，但是一个B类可以提供给65535台主机使用，一旦这个公司将这一段B类地址私有化，那么剩余的65000多个IP就会被浪费

使用子网掩码,CIDR基于变长子网掩码,这使得它可以定义任意长度的前缀，这种灵活性使其比旧系统更高效。
CIDR IP地址由两组数字组成d
- 网路地址
- 子网掩码

```bash
192.255.255.255/12
```
子网掩码所划分了两块区域，前12位是网络地址(子网掩码)，后20位主机地址  ps: 255.255.255.255 刚好32位 所以`网络位+主机位 = 32`

> 如何表示网络位和主机位

网络位 就是当网络位全为1 比如网络位是12 则表示为`11111111 11110000 00000000 00000000` 结果是`250.240.0.0`
主机位 就是主机位全为0 为第一个IP  主机位全为1 为第二个IP 

```bash
# 255.255.255.255 表示为
11111111 11111111 11111111 11111111
# 192.255.255.255 表示为
11000000 11111111 11111111 11111111

# 网络位12  250.240.0.0 
# 你别去看192.255.255.255 你要看255.255.255.255 转换后是全是1 然后前12为1 其他全为0
11111111 11110000 00000000 00000000

# 主机位20 第一个主机IP为主机位全为0  
11000000 11111111 00000000 00000000 # 192.240.0.0
# 主机位20 最后一个主机IP全为1
11000000 11111111 11111111 11111111 # 192.255.255.255
```

根据规则 主机地位中全0 和全1都去掉，剩下的就是可用的主机IP



## Net库
首先是net库下的方法,我们先过滤掉interface的内容,先来看实例类的生成也就是struct这一块

首先有以下几个功能实现
- IP地址
- 子网掩码
- IPNet网络
- 拨号dail
- 监听

需要了解整个包首先要区分整个net包中的结构体的名称

- IP地址 ip
- 子网掩码 mask
- 主机地址 ipNet (第一个主机地址(非可用),子网掩码)

![2020-12-15-15-44-41](http://noback.upyun.com/2020-12-15-15-44-41.png)


## IP地址
```bash
type IP []byte
func IPv4(a, b, c, d byte) IP
func ParseIP(s string) IP
```
实例化IP地址有如下两种方法
```go
func main(){
	ip2 := net.IPv4(10,0,5,41)
	fmt.Println(ip2) // 10.0.5.41
	
	ip3 := "127.0.0.1"
	res:=net.ParseIP(ip3)
	fmt.Println(res) //127.0.0.1
}
```
IP类型是代表单个IP地址的`[]byte`切片。本包的函数都可以接受4字节（IPv4）和16字节（IPv6）的切片作为输入。
注意，IP地址是IPv4地址还是IPv6地址是语义上的属性，而不取决于切片的长度：16字节的切片也可以是IPv4地址。


```golang
func main(){
	// 返回默认的子网掩码
	ip := net.ParseIP("10.0.6.55")
	mask := ip.DefaultMask()
	fmt.Println(mask.Size()) // 8 32

	// 判断两个ip是否相同
	ip2 := net.ParseIP("10.0.6.55")
	if ip.Equal(ip2){
		fmt.Println("相同")
	} else {
		fmt.Println("不相同")
	}

	fmt.Println(len(ip)) // 16
	// 将ip转化成4字节，如果ip不是v4或者v6的规范，会返回nil
	if ip = ip.To4();ip != nil{
		fmt.Println(ip,len(ip)) // 10.0.6.55 4
	}else {
		fmt.Println("ip不符合规范")
	}
}
```


## 掩码
```bash
type IPMask []byte
func IPv4Mask(a, b, c, d byte) IPMask
func CIDRMask(ones, bits int) IPMask
```
IPMask代表一个IP地址的掩码
IPv4Mask返回一个4字节格式的IPv4掩码a.b.c.d
CIDRMask返回一个IPMask类型值，该返回值总共有bits个字位，其中前ones个字位都是1，其余字位都是0

```go
func main(){
	mask:=net.IPv4Mask(255,255,255,0)
	fmt.Println(mask)
	mask2:=net.CIDRMask(20,32)
	fmt.Println(mask2) // fffff000
 	fmt.Println(mask.Size()) // 20 32
 	_ ,ipnet, _ := net.ParseCIDR("192.168.20.32/20")
	fmt.Println(ipnet.Mask.Size()) // 20 32
 	fmt.Println(ipnet.IP) // 192.168.16.0 
}
```


## IPNet
IPNet表示一个ip网络
```bash
type IPNet struct {
    IP   IP     // 网络地址
    Mask IPMask // 子网掩码
}
func ParseCIDR(s string) (IP, *IPNet, error)
func (n *IPNet) Contains(ip IP) bool
func (n *IPNet) Network() string
func (n *IPNet) String() string
```
ParseCIDR 返回一个IP地址和一个IPNET实例
Contains报告该网络是否包含地址ip。
Network返回网络类型名："ip+net"，注意该类型名是不合法的
String 字符串输出


```go
func main(){
	ip,ipadress,err := net.ParseCIDR("192.168.20.1/20")
	if err != nil{
		fmt.Println(nil)
	}
	fmt.Println("ip地址是",ip)
	fmt.Println("ipaddress是",ipadress)

	// 查看ip是否在ip段内
	ip2:=net.IPv4(10,0,6,55)
	res:=ipadress.Contains(ip2)
	fmt.Println(res)

	// 返回网络类型
	fmt.Println(ipadress.Network())

	//
	fmt.Println(ipadress.String())
}
```

## 拨号dail
```bash
type Conn interface {
    // Read从连接中读取数据
    // Read方法可能会在超过某个固定时间限制后超时返回错误，该错误的Timeout()方法返回真
    Read(b []byte) (n int, err error)
    // Write从连接中写入数据
    // Write方法可能会在超过某个固定时间限制后超时返回错误，该错误的Timeout()方法返回真
    Write(b []byte) (n int, err error)
    // Close方法关闭该连接
    // 并会导致任何阻塞中的Read或Write方法不再阻塞并返回错误
    Close() error
    // 返回本地网络地址
    LocalAddr() Addr
    // 返回远端网络地址
    RemoteAddr() Addr
    // 设定该连接的读写deadline，等价于同时调用SetReadDeadline和SetWriteDeadline
    // deadline是一个绝对时间，超过该时间后I/O操作就会直接因超时失败返回而不会阻塞
    // deadline对之后的所有I/O操作都起效，而不仅仅是下一次的读或写操作
    // 参数t为零值表示不设置期限
    SetDeadline(t time.Time) error
    // 设定该连接的读操作deadline，参数t为零值表示不设置期限
    SetReadDeadline(t time.Time) error
    // 设定该连接的写操作deadline，参数t为零值表示不设置期限
    // 即使写入超时，返回值n也可能>0，说明成功写入了部分数据
    SetWriteDeadline(t time.Time) error
}

func Dial(network, address string) (Conn, error)
func DialTimeout(network, address string, timeout time.Duration) (Conn, error)
func Pipe() (Conn, Conn)
```
在网络network上连接地址address，并返回一个Conn接口。可用的网络类型有：
"tcp"、"tcp4"、"tcp6"、"udp"、"udp4"、"udp6"、"ip"、"ip4"、"ip6"、"unix"、"unixgram"、"unixpacket"

```go
package main

import (
	"fmt"
	"log"
	"net"
	"time"
)

func main(){
	ip3:= net.ParseIP("10.0.6.55").String()
	fmt.Println(ip3)
	// 几种连接的方式
	// tcp 指定端口
	conn,err := net.Dial("tcp",ip3+":89")
	if err != nil{
		log.Fatal(err)
	}
	// udp 指定端口
	//conn,err = net.Dial("udp",ip3+":888")
	//if err!=nil{
	//	log.Fatal(err)
	//}
	//// icmp连接 不需要端口
	//conn,err = net.Dial("ip4:icmp",ip3)
	//if err != nil{
	//	log.Fatal(err)
	//}
	//
	//// icmp连接,使用协议编号
	//conn,err = net.Dial("ip4:1",ip3)
	//if err != nil{
	//	log.Fatal(err)
	//}
	// conn是一个实例化后的Conn实例
	// 关闭连接
	//defer conn.Close()
	// 本地地址
	fmt.Println(conn.LocalAddr())
	// 设置连接死亡时间
	err = conn.SetDeadline(time.Now().Add(time.Second*5))
	fmt.Println(err)
	// 设置连接读的死亡时间
	err = conn.SetReadDeadline(time.Now().Add(time.Second*5))
	// 设置连接写的死亡时间
	err = conn.SetWriteDeadline(time.Now().Add(time.Second*5))
	// 显示远端地址
	fmt.Println(conn.RemoteAddr())
	// 从连接中读取数据写入到变量当中去
	message := make([]byte,512)
	n,err  := conn.Read(message)
	if err != nil{
		fmt.Println("读取不到数据")
	}
	fmt.Println(n)
	// 像连接中写入数据
	n,err = conn.Write([]byte("helloworld"))
	if err != nil{
		fmt.Println("写不了数据")
	}
	fmt.Println(r)
}
```

## 监听
```bash
type Listener interface {
    // Addr返回该接口的网络地址
    Addr() Addr
    // Accept等待并返回下一个连接到该接口的连接
    Accept() (c Conn, err error)
    // Close关闭该接口，并使任何阻塞的Accept操作都会不再阻塞并返回错误。
    Close() error
}
func Listen(net, laddr string) (Listener, error)
type Addr interface {
    Network() string // 网络名
    String() string  // 字符串格式的地址
}
```

```go
package main

import (
	"fmt"
	"log"
	"net"
)

func main() {
	ip3 := net.ParseIP("10.0.6.55").String()
	fmt.Println(ip3)

	localhost:=net.ParseIP("127.0.0.1").String()
	fmt.Println(localhost)

	l,err := net.Listen("tcp",localhost+":8080")
	if err != nil{
		log.Fatal(err)
	}
	defer l.Close()

	// 建立长连接
	// 返回监听地址
	fmt.Println(l.Addr())
	// 返回监听地址网络协议
	fmt.Println(l.Addr().Network())
	// 返回监听地址字符串
	fmt.Println(l.Addr().String())

	for {
		// 返回监听地址
		conn,err := l.Accept()
		if err != nil{
			log.Fatal(err)
		}
		fmt.Printf("返回监听地址",l.Addr())
		fmt.Println(conn)

		fmt.Println(conn.LocalAddr())
		defer conn.Close()
	}

}
```

## 数值之间的比较
go里面最基本的就是以byte存在,他们之间的比较如下
```go
func doti( s string)(){
	for i:=0;i<len(s);i++ {
		//fmt.Println(s[i])
		fmt.Println(byte('0'))
		fmt.Println(byte('9'))

		if s[i] >= '0' || s[i] <= '9'{
			fmt.Println("存在于0-9之间")
		}
	}
}

func main(){
    doti("127")
}
// 48
// 57
// 存在于0-9之间
// 48
// 57
// 存在于0-9之间
// 48
// 57
// 存在于0-9之间
```

