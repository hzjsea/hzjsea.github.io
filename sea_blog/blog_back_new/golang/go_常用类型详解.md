---
title: go_常用类型详解
date: 2020-12-31 14:48:40
categories: [golang,]  
tags: [golang,]
---

<!-- more -->

# go_常用类型详解

参考书单


## 静态数组
数组具有`固定的长度`,拥有零个或多个`相同数据类型元素`的序列
在go中,定义静态数组如下
```golang
func main(){
	var arrary1 = [3]string{"xx","yy","zz"}
	var arrary2 = [3]int{1,2,3}
}
```


## go字符串
字符串本身是一种静态的的数组，也就是一种只读的切片，或者说是在一块连续的内存区域。
<div style='font-size:10px;color:red'>
在go中字符串以utf-8的编码形式出现，utf-8是unicode的一种编码形式，具体后面会讲到
</div>

> 定义一个不可变的字符串

```golang
func main(){
	var str = "helloworld";
	str1:="helloworld"
	
}
```

> 按照格式定义字符串

使用"`"可以按照格式来定义字符串
```golang
func main(){
	var str = "xx";
	var str1 = `
				x
				xml
				x
	` 
}
```

### 字符串的使用

> 获取长度 variable.len()

```golang
func main(){
	var yy = "helloworld"
	var yy_len = len(yy)
	fmt.Println(yy_len) // 10
	xx := "中"
	var xx_len = len(xx)
	fmt.Println(xx_len) // 3
}
```
这里的长度获取的是整个变量中所包含的字节的数量，举一个例子，"helloworld"这个字符串中所有的的字母数量，但是对于其他文字来说，比如说"中"这个汉字，首先要转换为unicode 再去计算字节的数量，这里中占三个字节。可以用在线工具测试一下
![](https://noback.upyun.com/2021-06-02-15-03-45.png!)
如上,go中默认使用utf-8编码形式，因此会中是占3个字节的

https://tool.chinaz.com/tools/unicode.aspx 转换中文到unicode
https://www.sojson.com/hexadecimal.html 转换中文到16进制
![](https://noback.upyun.com/2021-06-02-15-05-35.png!)
汉字`中`已经超出了ASCII编码的范围，用Unicode编码是十进制的`20013`，二进制的`01001110 00101101` 如果我们全部用unicode来编写的话，在ASCII编码范围的内容，比如说`A` 二进制就可以表示为`00000000 01000001`，但如果我们整片文章都是用ASCII表中的内容来写的话， 原本1字节的就要变成2字节的 用Unicode编码比ASCII编码需要多一倍的存储空间，在存储和传输上就十分不划算，于是为了节省内容，utf-8编码定义 所有ASCII编码中的内容定义保持原有的内容，ASCII表外的字符,第一个字节的前n位设置为1，第n+1位设置为0 后面的每个字节前两位都设置为10。

因此 `中`最后的结果是3

> 修改大小写

```golang
func main(){
	var lowercase  = "helloworld";
	var uppercase = "HELLOWORLD";
	if lowercase == uppercase{
		fmt.Println(true)
	} else {
		lowercase := strings.ToUpper(lowercase)
		if lowercase == uppercase{
			fmt.Println(true)
		} else if strings.ToLower(uppercase) == lowercase{
			fmt.Println(true)
		} else {
			fmt.Println(false)
		}
	}
}
```

> 比较字符串

```golang
func main(){
	lowercase  = "helloworld";
	uppercase = "HELLOWORLD";
	// 忽略大小写比较
	if strings.EqualFold(lowercase,uppercase) == true{
		fmt.Println(true)
	} else {
		fmt.Println("equalfold is false")
	}

	if strings.Compare(lowercase,uppercase) == 0{
		fmt.Println(true)
	} else {
		fmt.Println("compare is false")
	}
}
```


## 切片

https://ueokande.github.io/go-slice-tricks/
![](https://noback.upyun.com/2021-06-03-00-29-17.png!)
除去静态数组以外，就是动态数组了，动态数组比静态数组多了一个属性，也就是cap容量，保证在运行过程中能够扩容，也就是为后续的元素预留位置,创建方式如下

```golang
func main(){
	// 形式0
	var slice []int; // len = 0 cap = 0
	if slice == nil{
		fmt.Println("slice is nil")
	}
	
	// 形式1
	var slice = make([]string,10,20)
	slice = append(slice,"xx")
	// 形式2
	slice = []string{"hello"}
	slice = append(slice,"world")
	// 形式3
	var list = [3]int{1,2,3}
	var new_slice = list[:]
	fmt.Println(new_slice)
	// 形式4 创建一个cap == len的切片
	ss := [...]int{1,2,3,4,5}
	fmt.Println(cap(ss)) // 5
}
```
形式1中创建slice切片，在初始化阶段，会被要求设置容量和长度，如下是切片的源码定义
```golang
type slice struct {
	array unsafe.Pointer // 元素指针 // 指向底层数组，
	len   int // 长度  // 表示切片可用元素的个数，也就是说使用下标对 slice 的元素进行访问时，下标不能超过 slice 的长度
	cap   int // 容量 // 底层数组的元素个数，容量 >= 长度。在底层数组不进行扩容的情况下，容量也是 slice 可以扩张的最大限度
}
```

> 切片数据共享

![](https://noback.upyun.com/2021-06-02-16-32-55.png!)
注意到这里并没有对slice做什么处理，因此底层数组可以被多个slice同时指向，会互相影响
```golang
func main(){
	var vector = []int{1,2,3,4,56}
	var vv = vector[1:3];
	var vv1 = vector[1:2];
	vv1[0] = 100000
	fmt.Println(vector,vv,vv1)
	/*
	[1 2 3]
	[1 2 3]
	[1 100000 3 4 56] [100000 3] [100000]
	*/
}
```

> nil slice 与 empty slice

事实上，这两个slice中 capacity 和 length 都是为空的，但是他们两个却是不想的，主要是empty slice还有一个地址存在

```golang
package main

import "fmt"

func main(){
	var slice []int; // 长度为0 容量为0
	fmt.Printf("%d, %d, %p \n",len(slice), cap(slice), slice) // 这里的nil切片是没有地址的 0x0

	// true nil切片
	if slice == nil{
		fmt.Println(true)
	}

	var slice1 = make([]int,0,0)
	fmt.Printf("%d, %d, %p \n",len(slice1), cap(slice1), slice1) // 这里的空切片是有地址的 0x1196a98


	// false 空切片
	if slice1 == nil{
		fmt.Println(true)
	} else {
		fmt.Println(false)
	}
}
```

> 切片从静态数组衍生过来的实例图

![](https://noback.upyun.com/2021-06-03-00-39-20.png!)
切片中的第一个属性 指针指向底层数组的元素

### 切片的操作

> 遍历切片

```golang
for i := 0; i < len(summer); i++ {
    fmt.Println("summer[", i, "] =", summer[i]) 
}

for k,v := range summer {
	fmt.Println("%s%s",k,v)
}
```


> 动态增加元素 append

切片比数组更强大之处在于支持动态增加元素，甚至可以在容量不足的情况下自动扩容。在切片类型中，元素个数和实际可分配的存储空间是两个不同的值，元素的个数即切片的实际长度，而可分配的存储空间就是切片的容量

- 对于基于数组和切片创建的切片而言，默认容量是从切片起始索引到对应底层数组的结尾索引；
- 对于通过内置 make 函数创建的切片而言，在没有指定容量参数的情况下，默认容量和切片长度一致

```golang
	s1:=[10]int{1,2,3,4,5,6,7,8,9,10}
	var s2_slice = s1[1:2]
	fmt.Println(s2_slice,len(s2_slice),cap(s2_slice)) // [2] 1  9  
```
注意 这里从静态数组上获取下来的切片的容量是起始的索引到底层数组的结尾索引

<div style='font-size:10px;color:red'>同时append方法(当满足容量不够的时候),继续添加新的元素时,会重新开辟一块新的内存</div> 这样之前所担心的切片间数据共享的问题就会被解决了

```golang
slice1 := make([]int, 4)
slice2 := slice1[1:3]
slice1 = append(slice1, 0) // 重新分配内存
slice1[1] = 2
slice2[1] = 6
fmt.Println("slice1:", slice1)
fmt.Println("slice2:", slice2)
```

![](https://noback.upyun.com/2021-06-03-08-51-35.png!)


> 特殊状态下的切片

切片本身是一种动态且元素类型一致性的数组,在go中切片会存在三种特殊的状态，分别是`zero slice` 、`empty slice` and `nil slice`
https://juejin.cn/post/6844903712654098446



## 结构体与json

结构体struct是一种复合型类型，而不是引用类型。在传递值的过程中，复合类型采用的是值传递，而引用类型则是引用传递

> 值类型的特点

值类型的特点是内存紧凑，大小固定，对 GC 与内存访问来说都比较友好
值类型结构体内存分布图
![](https://noback.upyun.com/2021-06-03-14-54-26.png!)
![](https://noback.upyun.com/2021-06-03-14-55-00.png!)

但值类型也会有缺点，如果赋值、传参等等操作多了，内存就会出现明显的浪费，这时候纪要介入引用来缓解一些内存上的不必要的赋值操作，直接操作指针目的地就可以实现`赋值`,可以看看下面的指针结构体

https://liujiacai.net/blog/2020/03/14/go-struct-interface/

> 创建结构体

```golang
type Member struct {
    id          int
	name 		string
	email 		string
	gender 		int
	age 		int
}
```

> 空结构体

如果结构体本身没有任何字段，则作为`空结构体`, 空结构体主要使用在并发编程中 channel通信的信号量,切空结构体并不占任何内存

```golang
ch := make(chan struct{})
ch <- struct{}{}

	tt1 := struct
	fmt.Println(unsafe.Sizeof(tt1)) // 0
```


> 指针结构体

因为go中的struct与数组一样是值传递，在给函数中的形参赋值的时候，会在内存池上拷贝一份副本然后交给形参，所以为了提高性能，一般不会把数组直接传递给函数，而是使用切片(引用类型)代替，而把结构体传给函数时，可以使用指针结构体
指针结构体，即一个指向结构体的指针,声明结构体变量时，在结构体类型前加*号，便声明一个指向结构体的指针。这样在赋值给形参的时候,只会将形参的指针指向同一块内存地址
```golang
func main(){
	tt := Name{
		"hello",
		"world",
	}
	fmt.Printf("%p\n",&tt) // 0xc00008c020
	getField(tt)
	getField2(&tt)

}

func getField(n1 Name){ // 值传递
	fmt.Printf("%p\n", &n1) // 0xc00008c040
	fmt.Println(n1.LastName)
}

func getField2(n1 *Name)  { // 引用传递
	fmt.Printf("%p\n",n1) // 0xc00008c020
	fmt.Println(n1.LastName)
}
```

> 初始化结构体

new()函数可以初始化结构体，并且分配内存
```golang
var m2 = new(Member)
	tt1 := new(Member)
	fmt.Println(unsafe.Sizeof(tt1)) // 8
m2.name = "小红"
```

> 可见性

对于同一个包内的结构体中的字段，大小写不区分，但是对于跨包来说，存在以下可见性规则
go的结构体需要区分大小写，大写表示向外开放，小写表示为在该包中的私有字段 


> tags

在json里面再说



### json

json由struct转换而来，同时在go中可以给struct定义一些规则，在json转换的过程中可以被实现。

> json的官方包

```golang
package main

import (
	"encoding/json"
	"fmt"
	"reflect"
)

type User struct {
	Id        int       `json:"id"`
	Name      string    `json:"name"`
	Bio       string    `json:"about,omitempty"`
	Active    bool      `json:"active"`
	Admin     bool      `json:"-"`
}


func main(){
	user := User{
		Id:     0,
		Name:   "xx",
		Bio:    "xx",
		Active: false,
		Admin:  false,
	}


	// to_json
	if user_json,err := json.Marshal(user); err == nil{
		fmt.Println(string(user_json))
		//{
		//  "id": 0,
		//  "name": "xx",
		//  "about": "xx",
		//  "active": false
		//}

		fmt.Println(user_json)
	//[123 34 105 100 34 58 48 44 34 110 97 109 101 34 58 34 120 120 34 44 34 97 98 111 117 116 34 58 34 120 120 34 44 34 97 99 116 105 118 101 34 58 102 97 108 115 101 125]
	}

	// from_json
	user_json1,err := json.Marshal(&user)
	if err != nil{
		panic("error")
	}
	fmt.Println(user_json1)
	fmt.Println(reflect.TypeOf(user_json1)) // []int8


	// user_struct
	// 必须先创建一个接收体，然后再Unamrshal中使用
	user_struct := new(User)
	err = json.Unmarshal(user_json1,&user_struct)
	if err != nil{
		panic("error")
	}
	fmt.Println(user_struct)

}
```
上面的代码中除了序列化和反序列化的展示，也表明了标签的作用，
- "-"表示在转换为json的时候不会被记录，
- "omitempty"表示当值为零值的时候，不会被记录


https://loesspie.com/2020/04/23/go-tag/
chrome-extension://ecabifbgmdmgdllomnfinbmaellmclnh/data/reader/index.html?id=3210&url=https%3A%2F%2Fblog.csdn.net%2Fzxy_666%2Farticle%2Fdetails%2F80173288

## 字典map
```go
package main

import "fmt"

func main() {
	// 初始化一个map
	ages := make(map[string]int)
	//
	ages2 := map[string]int{
		"alice":31,
		"charlie":34,
	}

	fmt.Println(ages,ages2)

	// 删除字典中的元素
	delete(ages2,"alice")
	fmt.Println(ages2)
}
```

## 接口

## golang博客
https://sanyuesha.com/categories/golang/


### io.writer

`io.writer`解控定义了Fprintf和调用者之间的约定. 一方面,这个约定要求调用者提供的具体类型包含一个与其签名和行为一致的Write方法

```go
// io.write
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

```go
// buffio.Write
type Writer struct {
	err error
	buf []byte
	n   int
	wr  io.Writer
}
func NewWriter(w io.Writer) *Writer {
	return NewWriterSize(w, defaultBufSize)
}

func (b *Writer) Write(p []byte) (nn int, err error) {
	for len(p) > b.Available() && b.err == nil {
		var n int
		if b.Buffered() == 0 {
			// Large write, empty buffer.
			// Write directly from p to avoid copy.
			n, b.err = b.wr.Write(p)
		} else {
			n = copy(b.buf[b.n:], p)
			b.n += n
			b.Flush()
		}
		nn += n
		p = p[n:]
	}
	if b.err != nil {
		return nn, b.err
	}
	n := copy(b.buf[b.n:], p)
	b.n += n
	nn += n
	return nn, nil
}
```
可以看出bufio这个库里面的Write实现了Write这个函数,也就是实现了Write这个接口,也就是说bufio中的Write实现了io.write这个类型


再比如
`*bytes.Buffer`也是一个`io.Writer`
因此`*Buffer`也是一个`io.Writer`类型
```go
type Buffer struct {
	buf      []byte // contents are the bytes buf[off : len(buf)]
	off      int    // read at &buf[off], write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly.
}

func (b *Buffer) Write(p []byte) (n int, err error) {
	b.lastRead = opInvalid
	m, ok := b.tryGrowByReslice(len(p))
	if !ok {
		m = b.grow(len(p))
	}
	return copy(b.buf[m:], p), nil
}
func NewBuffer(buf []byte) *Buffer { return &Buffer{buf: buf} }
```

试着手动实现一个


```go
package main

 
type Animaler interface {
	Run(speed int)(sd int,err error)
}

type Mammals struct {
	Name string
	Id int
}

func NewMammals(name string,id int) *Mammals{
	return &Mammals{
		Name: name,
		Id:   id,
	}
}
func (ms *Mammals) Run(speed int)(sd int,err error){
	return speed,nil
}


func main() {
	// 创建哺乳类动物实例
	ma := NewMammals("houzi",12)
	ma.Run(20)
}


```

### 接口值
一个接口类型的值有两部分,一个具体类型和该类型的一个值,分别是接口的`动态类型`和`动态值`
```bash
func main(){
	var w io.Writer
	w = os.Stdout
	w = new(bytes.Buffer)
	w = nil
}
```
看第二行,w被赋值了一个`io.Writer`类型的值


### golang接口总结
golang中有一种接口类型,比如io库里面定义了很多底层的接口,这些接口也常常被当做接口类型来使用

> 他于其他类型的不同之处

之前使用的切片,数组等等都有对应的操作那个,但是接口没有,接口本身是一种抽象的类型,你可以使用它来定义一些自己需要的方法以及动作等等

> 如何实现接口(接口类型)

一个接口类型定义了一套方法,就和我们使用切片类型一样都具有一套`.`的方法,在golang中只要实现了接口中的所有方法,那么这个类型就实现了该接口类型,同时也实现了该接口


> io库中所定义的底层接口以及底层接口类型

io库定义了很多的底层接口,比如`io.Reader`,`io.Writer`,`io.Closer`等等
`io.Writer`是一个广泛使用的接口,它负责所有可以写入字节的类型的抽象,包括文件 内存缓冲区 网络连接 HTTP客户端,打包器 散列器等等
`io.Reader`抽象了所有可以读取字节的类型
`io.Close`抽象了所有可以关闭的类型,比如文件 或者网络连接

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
type Closer interface {
    Close() error
}
```

因为golang中还支持接口的嵌套,于是会将上面的基本组合起来,如下
```go
type WriteCloser interface {
    Writer
    Closer
}
type ReadCloser interface {
    Reader
    Closer
}
```

> 实现接口的过程

如上说道,io库里面定义了很多这样的接口类型,要想使用这些接口类型只需要实现这个接口中的所有方法,
比如`os.File`
```go
type File struct {
    // contains filtered or unexported fields
}
func (f *File) Read(b []byte) (n int, err error)
```
如上就实现了Reader这个接口类型
再比如`bytes.buffer`
```go
type Buffer struct {
    // contains filtered or unexported fields
}
func (b *Buffer) Read(p []byte) (n int, err error)
```
如上就实现了Reader这个接口类型



> 带着上面的内容再来看几个库的使用


实现是`os.file`
这个库是用来做和文件有关的动作的
![2021-01-02-21-36-19](http://noback.upyun.com/2021-01-02-21-36-19.png)


## lib_net
这个库很多大佬都推荐阅读,应该是里面涉及的内容比较全,代码也写的比较好

### 定义一些基础变量
```go
// 常量
const (
    IPv4len = 4
    IPv6len = 16
)
// 变量v4
var (
    IPv4bcast     = IPv4(255, 255, 255, 255) // limited broadcast
    IPv4allsys    = IPv4(224, 0, 0, 1)       // all systems
    IPv4allrouter = IPv4(224, 0, 0, 2)       // all routers
    IPv4zero      = IPv4(0, 0, 0, 0)         // all zeros
)
// 变量v6
var (
    IPv6zero                   = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}
    IPv6unspecified            = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}
    IPv6loopback               = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1}
    IPv6interfacelocalallnodes = IP{0xff, 0x01, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x01}
    IPv6linklocalallnodes      = IP{0xff, 0x02, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x01}
    IPv6linklocalallrouters    = IP{0xff, 0x02, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x02}
)
```


```go
package main

import (
	"fmt"
	"log"
	"net"
	"reflect"
)

func main() {
	// JoinHostPort
	// 组合ip+port 类型
	addr_port:=net.JoinHostPort("127.0.0.1","80")
	fmt.Println(addr_port)
	// SplitHostPort
	// 拆分组合
	addr,port,err:= net.SplitHostPort(addr_port)
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(addr,port)


	// 解析硬件mac地址
	mac_address,err:= net.ParseMAC("01:23:45:67:89:ab")
	if err!=nil{
		log.Fatal(err)
	}
	fmt.Println(mac_address.String())


	// 返回系统的网络接口列表
	net_list,err := net.Interfaces()
	for index,value := range net_list{
		fmt.Println(index,value)
	}
	//0 {1 16384 lo0  up|loopback|multicast}
	//1 {2 1280 gif0  pointtopoint|multicast}
	//2 {3 1280 stf0  0}
	//3 {4 0 XHC20  0}
	//4 {5 0 XHC0  0}
	//5 {6 0 VHC128  0}
	//6 {8 1500 ap1 a6:83:e7:83:71:89 broadcast|multicast}
	//7 {9 1500 en0 a4:83:e7:83:71:89 up|broadcast|multicast}
	//8 {10 2304 p2p0 06:83:e7:83:71:89 up|broadcast|multicast}
	//9 {11 1484 awdl0 26:5e:1b:eb:07:be up|broadcast|multicast}
	//10 {12 1500 en1 a2:00:68:a8:8a:01 up|broadcast|multicast}
	//11 {13 1500 en2 a2:00:68:a8:8a:00 up|broadcast|multicast}
	//12 {14 1500 bridge0 a2:00:68:a8:8a:01 up|broadcast|multicast}
	//13 {15 2000 utun0  up|pointtopoint|multicast}
	//14 {7 1500 en3 ac:de:48:00:11:22 up|broadcast|multicast}
	// 返回 指定索引的网络接口
	net_interface,err := net.InterfaceByIndex(1) // lo 口
	// 网络接口名字
	fmt.Println(net_interface.Name) // lo
	//
	fmt.Println(net_interface.Index) // 序号
	//
	fmt.Println(net_interface.Flags) // 信息
	//
	fmt.Println(net_interface.Addrs()) //

	// 返回系统的网络接口的地址列表。
	addr_list,err := net.InterfaceAddrs()
	for index,value := range addr_list{
		fmt.Println(index,value)
	}


	// ******************************ipv4
	ip1:=net.IPv4(127,0,0,1)
	fmt.Println(reflect.TypeOf(ip1)) // net.IP
	fmt.Println(ip1.String()) // 转换为字符串吧
	fmt.Println(ip1.Equal(net.IPv4(127,0,0,1))) // 判断是否相等
	fmt.Println(ip1.DefaultMask()) // ff000000
	ip2 := net.ParseIP("10.0.6.55")
	if ip2 == nil{
		log.Fatal("ip2格式错误")
	}
	fmt.Println(ip2.DefaultMask()) // ff000000

	// 将IP地址转换为16字节
	fmt.Println(ip2.To16())
	// 将ip地址转换为4字节
	fmt.Println(ip2.To4())

	ip3 := net.ParseIP("192.168.20.1")
	fmt.Println(ip3.Mask(net.IPv4Mask(255,255,255,0))) // 192.168.20.0

	// ******************************ipnet

	ip1,ipnet,err:= net.ParseCIDR("192.168.20.1/24")
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(ip1)  // 192.168.20.1 网关
	fmt.Println(ipnet) // 192.168.20.0/24
	fmt.Println(ipnet.Mask) // 掩码 ffffff00
	fmt.Println(ipnet.IP) // 网络地址 192.168.20.0
	fmt.Println(ipnet.Contains(net.ParseIP("192.168.20.3"))) // 主机位中是否包含192.168.20.3
	fmt.Println(ipnet.Network()) // 返回网络类型名

	// ****************************** conn

	//type Addr interface {
	//	Network() string // 网络名
	//	String() string  // 字符串格式的地址
	//} // 这里只是拿来做接口类型使用的 ,没其他的用途 比如address
	// 拨号 ,单次的 不是双工的
	conn,err := net.Dial("tcp","10.0.6.55:80")
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(conn.LocalAddr()) // 127.0.0.1:53208
	fmt.Println(conn.LocalAddr().Network()) // 显示网络类型名
	fmt.Println(conn.LocalAddr().String()) // 转换成字符串
	fmt.Println(conn.RemoteAddr().String()) // 返回远端网络地址

	// 将读取到的内容读到b中区
	b := make([]byte,10)
	conn.Read(b)
	fmt.Println(b) // 因为没有接收任何消息,所以结果是[0 0 0 0 0 0 0 0 0 0]

	defer conn.Close() // 滞后关闭连接




	conn1,conn2:=net.Pipe()
	fmt.Println(conn1.RemoteAddr())
	fmt.Println(conn2.LocalAddr())

	go func() {
		var bc = []byte("send conn1")
		n, err := conn1.Write(bc)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println(n)
		conn1.Close()
	}()

	// 另一端读取
	var rc = make([]byte,10)
	n,err :=conn2.Read(rc)
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(n)
	fmt.Println(string(rc))
	conn2.Close()
	//



	// ****************************** Listener
	// 本地建立一个监听,监听一个端口
	listener,err := net.Listen("tcp","127.0.0.1:8080")
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(listener.Addr())
	defer listener.Close() // 程序结束之前关闭listener连接
	// 创建一个死循环,接收监听端口上的信息
	for{
		conn,err := listener.Accept()
		if err != nil{
			log.Fatal(err)
		}
		b=make([]byte,10)
		n,err := conn.Read(b)
		fmt.Println(n)

		/// ********
		/// 本地起一个服务端 发送数据就OK了
		/// nc 127.0.0.1 8080
		//  1111 >
		//  4 <

		/// *********

		//go func(c net.Conn) {
		//	b=make([]byte,10)
		//	n,_ := conn.Read(b)
		//	fmt.Println(n)
		//	// Echo all incoming data.
		//	io.Copy(c, c)
		//	// Shut down the connection.
		//	c.Close()
		//}(conn)
	}


	//// 对地址执行反向查找，并返回映射到给定地址的名称列表.
	//addr_list,err := net.LookupAddr("114.114.114.114")
	//if err !=nil{
	//	log.Fatal(err)
	//}
	//for index,addr := range addr_list{
	//	fmt.Println(index,addr)
	//}
	//
	//// 查找别名记录CNAME
	//cname,_ := net.LookupCNAME("blog.noback.top") // hzjsea.github.io
	//fmt.Println(cname)
	//
	//// dns解析查询
	//host_list,err := net.LookupHost("hzjsea.github.io")
	//if err != nil{
	//	log.Fatal(err)
	//}
	//fmt.Println(host_list)
	//
	////
	//
	//port,_ := net.LookupPort("tcp","http")
	//fmt.Println(port)
	//port,_ = net.LookupPort("tcp","telnet")
	//fmt.Println(port)
	//port,_ = net.LookupPort("tcp","ssh")
	//fmt.Println(port)
	//
	//
	//// 将域名（baidu.com）作为字符串，并返回DNS TXT记录列表作为字符串片段.
	//fmt.Println("domain text --------")
	//txts,_ := net.LookupTXT("baidu.com")
	//for _,v := range txts{
	//	fmt.Println(v)
	//}



}

```



## lib_strings
strings举例了一系列关于字符串操作方法,其他的方法可以用到再看, 主要看下他实现了`io.reader`的一些方法,主要是将字符串中的内容读取到IO

strings库可以高效的读取字符串,他会定义到读取到哪个位置
```bash
type Reader struct {
	s        string
	i        int64 // current reading index
	prevRune int   // index of previous rune; or < 0
}
```
```go
package main

import (
	"fmt"
	"log"
	"strings"
)

func main() {
	// 读取字符串中的内容到io
	in_io_s:=strings.NewReader("foobar")
	fmt.Println(in_io_s) //&{foobar 0 -1}

	in_io_s = strings.NewReader("中文")
	fmt.Println(in_io_s) // &{中文 0 -1}

	in_io_s = strings.NewReader("123456789")
	fmt.Println(in_io_s) // &{123456789 0 -1}
	// 将io中的字符串内容读取到byte数组当中
	p:=make([]byte,1)
	n,_ := in_io_s.Read(p)
	fmt.Println(n) //1
	fmt.Println(in_io_s) // &{123456789 1 -1}

	// 每次读取一个,指针不断向后移动
	for i:=0;i<in_io_s.Len();{
		p:=make([]byte,1)
		n,_:=in_io_s.Read(p)
		fmt.Printf("%d------%#v\n",n,in_io_s)
	}
	//1------&strings.Reader{s:"123456789", i:2, prevRune:-1}
	//1------&strings.Reader{s:"123456789", i:3, prevRune:-1}
	//1------&strings.Reader{s:"123456789", i:4, prevRune:-1}
	//1------&strings.Reader{s:"123456789", i:5, prevRune:-1}
	//1------&strings.Reader{s:"123456789", i:6, prevRune:-1}
	//1------&strings.Reader{s:"123456789", i:7, prevRune:-1}
	//1------&strings.Reader{s:"123456789", i:8, prevRune:-1}
	//1------&strings.Reader{s:"123456789", i:9, prevRune:-1}

	in_io_s = strings.NewReader("123456789")
	for i:=0;i<in_io_s.Len();{
		b,err := in_io_s.ReadByte()
		if err != nil{
			log.Fatal(err)
		}
		fmt.Printf("%d------%#v\n",b,in_io_s)
	}
	//49------&strings.Reader{s:"123456789", i:1, prevRune:-1}
	//50------&strings.Reader{s:"123456789", i:2, prevRune:-1}
	//51------&strings.Reader{s:"123456789", i:3, prevRune:-1}
	//52------&strings.Reader{s:"123456789", i:4, prevRune:-1}
	//53------&strings.Reader{s:"123456789", i:5, prevRune:-1}
	//54------&strings.Reader{s:"123456789", i:6, prevRune:-1}
	//55------&strings.Reader{s:"123456789", i:7, prevRune:-1}
	//56------&strings.Reader{s:"123456789", i:8, prevRune:-1}
	//57------&strings.Reader{s:"123456789", i:9, prevRune:-1}

	// 读取中文
	in_io_s = strings.NewReader("中文博大精dsds深")
	for i:=0;i<in_io_s.Len();{
		ch,size,err := in_io_s.ReadRune()
		if err!=nil{
			log.Fatal(err)
		}
		fmt.Println(string(ch),size)
	}
	//中 3
	//文 3
	//博 3
	//大 3
	//精 3
	//d 1
	//s 1
	//d 1
	//s 1
	//深 3
}

```

## lib_bytes
bytes和strings差不多,无非一个是字节数组,一个是字符串,两个都库的方法都

```go
 	var xx  = []byte{'1','2','3'}
	bfp:=bytes.NewReader(xx)
	fmt.Println(bfp) // &{[49 50 51] 0 -1}


	//b:=make([]byte,10)
	//bfp.Read(b)
	//fmt.Println(bfp) // &{[49 50 51] 3 -1}
	//fmt.Println(bfp.Size()) //3
	//fmt.Println(bfp.Len())
	//c,err:=bfp.ReadByte()
	//if err !=nil{
	//	log.Fatal(err)
	//}
	////fmt.Println(string(c))




	// 初始化一个buffer
	//bs:=new(bytes.Buffer)
	// 初始化一个buffer的同时 写入一个已经存在的数据
	//bf:=bytes.NewBuffer([]byte{'1','2'})
	//bf_str:=bytes.NewBufferString("helloworld");



	bf:=bytes.NewBuffer([]byte{})
	fmt.Println(bf.Len()) //0
	bf.WriteString("xx")
	fmt.Println(bf.String())
	fmt.Println(bf.Next(2))
	bf.WriteString( "zzzzz")
	fmt.Println(string(bf.Next(3)))
```


## lib_bufio
这个库是io库的升级版
bufio包实现了有缓冲的I/O。它包装一个`io.Reader`或`io.Writer`接口对象，创建另一个也实现了该接口，且同时还提供了缓冲和一些文本I/O的帮助函数的对象
如果对于一些小字符串的,频繁的使用io会增加负担,bufio会把内容读取到缓存当中(内存),然后再取读取需要的内容的时候，直接在缓存中读取,避免文件的`i/o`操作。同样，通过bufio写入内容，也是先写入到缓存中（内存），然后由缓存写入到文件。避免多次小内容的写入操作`I/o`操作。

```go

type Reader struct {
	buf          []byte
	rd           io.Reader // reader provided by the client
	r, w         int       // buf read and write positions
	err          error
	lastByte     int // last byte read for UnreadByte; -1 means invalid
	lastRuneSize int // size of last rune read for UnreadRune; -1 means invalid
}
```
如上是Rader对接口`io.Reader`的封装,其中
- rd就是底层的`io.Reader`
- r:从buf中读走的字节（偏移）；w:buf中填充内容的偏移； 
- w - r 是buf中可被读的长度（缓存数据的大小），也是Buffered()方法的返回值
- lastByte 最后一次读到的字节（ReadByte/UnreadByte)
- lastRuneSize  最后一次读到的Rune的大小(ReadRune/UnreadRune)

![2021-01-04-16-36-10](http://noback.upyun.com/2021-01-04-16-36-10.png)
分别对应 buf rd r w err lastByte 以及 lastRuneSize

```go
	brd := bufio.NewReader(strings.NewReader("helloworld"))
	fmt.Println(brd)
	fmt.Println(brd.Size())
	// NewReaderSize创建一个具有最少有size尺寸的缓冲、从r读取的*Reader。如果参数r已经是一个具有足够大缓冲的* Reader类型值，会返回r
	// 默认情况下是4096个byte
	brd = bufio.NewReaderSize(strings.NewReader("helloworld"),64)
	fmt.Println(brd.Size()
```

可以查看size 如果没有缓冲的话,直接使用`strings.reader` 则size大小是根据字符串的大小决定的,但是如果使用的是带有缓冲的io的话,则返回的是默认情况下4096个字节的size 当然也可以使用`bufio.newReaderSize`来定义字节数量的大小

看源码可以看出 `bufio.reader` 其实是调用了`bufio.NewReaderSize` ,只是加了一个初始化的参数4096
```go

const (
	defaultBufSize = 4096
)

func NewReader(rd io.Reader) *Reader {
	return NewReaderSize(rd, defaultBufSize)
}
func NewReaderSize(rd io.Reader, size int) *Reader {
	// Is it already a Reader?
	b, ok := rd.(*Reader)
	if ok && len(b.buf) >= size {
		ret
	}
	if size < minReadBufferSize {
		size = minReadBufferSize
	}
	r := new(Reader)
	r.reset(make([]byte, size), rd)
	return r
}
```


### bufio.Read的思路

![2021-01-04-18-07-05](http://noback.upyun.com/2021-01-04-18-07-05.png)
```go
	brd= bufio.NewReader(strings.NewReader("helloworld"))
	p:=make([]byte,10)
	n,_:=brd.Read(p)
	fmt.Println(n)
```
第三种情况
源码内容如下
```go
func (b *Reader) Read(p []byte) (n int, err error) {
	n = len(p)  // 获取p切片的大小
	if n == 0 {
		if b.Buffered() > 0 {
			return 0, nil
		}
		return 0, b.readErr()
	} // 如果没0,则返回报错
	// r:从buf中读走的字节（偏移）；w:buf中填充内容的偏移；
	if b.r == b.w { // 当前缓冲区里面没有内容
		if b.err != nil {
			return 0, b.readErr()
		}
		if len(p) >= len(b.buf) {
			// Large read, empty buffer.
			// Read directly into p to avoid copy.
			n, b.err = b.rd.Read(p)
			if n < 0 {
				panic(errNegativeRead)
			}
			if n > 0 {
				b.lastByte = int(p[n-1])
				b.lastRuneSize = -1
			}
			return n, b.readErr()
		}
		// One read.
		// Do not use b.fill, which will loop.
		b.r = 0
		b.w = 0
		n, b.err = b.rd.Read(b.buf)
		if n < 0 {
			panic(errNegativeRead)
		}
		if n == 0 {
			return 0, b.readErr()
		}
		b.w += n
	}

	// // copy缓存区的内容到p中（填充满p）
	// copy as much as we can
	n = copy(p, b.buf[b.r:b.w])
	b.r += n
	b.lastByte = int(b.buf[b.r-1])
	b.lastRuneSize = -1
	return n, nil
}

```

> readline,readString,ReadBytes
这三个都是有分隔符组成的,官方更推荐使用`bufio.ReadString`以及`bufio.ReadBytes`

- ReadLine 返回一行的数据,不包括行尾标志的字节,如果行太长超过缓冲,返回值isPrefix会被设为true，并返回行的前面一部分.该行剩下的部分将在之后的调用中返回。返回值isPrefix会在返回该行最后一个片段时才设为false。返回切片是缓冲的子切片，只在下一次读取操作之前有效。ReadLine要么返回一个非nil的line，要么返回一个非nil的err，两个返回值至少一个非nil。官方推荐使用 `bufio.ReadString('\n')` 或者`bufio.ReadBytes('\n')`来代替
- ReadString  ReadString读取直到第一次遇到delim字节，返回一个包含已读取的数据和`delim字节的字符串`。如果ReadString方法在读取到delim之前遇到了错误，它会返回在错误之前读取的数据以及该错误（一般是io.EOF）。当且仅当ReadString方法返回的切片不以delim结尾时，会返回一个非nil的错误。
- ReadBytes读取直到第一次遇到delim字节，返回一个包含已读取的数据和`delim字节的切片`。如果ReadBytes方法在读取到delim之前遇到了错误，它会返回在错误之前读取的数据以及该错误（一般是io.EOF）。当且仅当ReadBytes方法返回的切片不以delim结尾时，会返回一个非nil的错误。

```go
	brd = bufio.NewReader(strings.NewReader("helloworlddsdsdsdsdsds\ndsdsdsssd\ndsdsds\ndsdssd\nddsdsd\n"))
	one_line,isprefix,err:= brd.ReadLine()
	if err != nil{
		log.Fatal(err)
	}
	if isprefix == true{
		fmt.Println("太长了")
	}else {
		fmt.Println(one_line)
	}
	brd = bufio.NewReader(strings.NewReader("helloworlddsdsdsdsdsds\ndsdsdsssd\ndsdsds\ndsdssd\nddsdsd\n"))
	one_line,_=brd.ReadBytes('\n')
	fmt.Println(one_line)

	brd = bufio.NewReader(strings.NewReader("helloworlddsdsdsdsdsds\ndsdsdsssd\ndsdsds\ndsdssd\nddsdsd\n"))
	one_line_str,_ := brd.ReadString('\n')
	fmt.Println(one_line_str)
```


### bufio.writer
这个和`bufio.Reader`一样,也是对`io.Writer`的封装,产生了缓存的内容
Writer实现了为io.Writer接口对象提供缓冲。如果在向一个Writer类型值写入时遇到了错误，该对象将不再接受任何数据，且所有写操作都会返回该错误。在说有数据都写入后，调用者有义务调用Flush方法以保证所有的数据都交给了下层的io.Writer。

```go
func NewWriter(w io.Writer) *Writer {
	return NewWriterSize(w, defaultBufSize)
}
type Writer struct {
	err error   // 错误
	buf []byte  // 缓存
	n   int   // 读取到的字节序号
	wr  io.Writer // 底层io
}
func (b *Writer) Write(p []byte) (nn int, err error) {
	for len(p) > b.Available() && b.err == nil {
		var n int
		if b.Buffered() == 0 {
			// Large write, empty buffer.
			// Write directly from p to avoid copy.
			n, b.err = b.wr.Write(p)
		} else {
			n = copy(b.buf[b.n:], p)
			b.n += n
			b.Flush()
		}
		nn += n
		p = p[n:]
	}
	if b.err != nil {
		return nn, b.err
	}
	n := copy(b.buf[b.n:], p)
	b.n += n
	nn += n
	return nn, nil
}
```
实现了wirte,也就是实现了`io.writer`这个类型

> bufio.NewWriter 与 bufio.NewWriterSize

两个都是对底层的`io.Writer`进行分装,`bufio.NewWriter`调用的是`bufio.NewWriterSize`,只不过默认的buff大小是4096
![2021-01-05-09-55-46](http://noback.upyun.com/2021-01-05-09-55-46.png)

```go
	// 创建缓存 bwd buff-writer 会创建一个4096大小的缓存区域
	bwd := bufio.NewWriter(os.Stdout)
	bwd.Size() //查看缓存区的大小  4096
	bwd.WriteString("helloworld") // 写入缓存
	fmt.Fprint(bwd,"helloworld") // 写入缓存
	bwd.Flush() // 将缓存中的内容写入到底层的io.writer中 这个时候缓存区域的大小应该还是4096
	fmt.Println(bwd.Available()) // 查看缓存区剩余的大小  4096
	fmt.Println(bwd.Available()) // 查看缓存区写入的大小  0
	bwd.WriteString("xxx")
	fmt.Println(bwd.Size()) // 4096
	fmt.Println(bwd.Available()) // 4093
	fmt.Println(bwd.Buffered()) // 3

	// 创建缓存为8192大小的缓存区域
	bwd = bufio.NewWriterSize(os.Stdout,8192)
	fmt.Println(bwd.Size())

	// 将p数组中的内容写入到缓存当中
	r:=bufio.NewReader(strings.NewReader("helloworld"));
	p=make([]byte,20)
	// 将r缓存区中的内容读取到p当中
	r.Read(p)
	fmt.Println(p)
	// 创建缓存输出
	w:=bufio.NewWriter(os.Stdout)
	// 将内容写入到缓存输出当中
	w.WriteString("helloworld")
	w.Write(p)

	s := bytes.NewBufferString("start----")
	w = bufio.NewWriter(s)
	w.WriteString("mid-------")
	p=make([]byte,20)
	r=bufio.NewReader(strings.NewReader("end-----"))
	r.Read(p)
	w.Write(p)
	w.Flush()
	fmt.Println(s)

```





### bufio.scanner
golang中提供扫描的字段 很多字符串或者是文件都是一大串的字符串，其中也包含了很多的delim分隔符，比如像这样
```go
	ss:= bufio.NewReader(strings.NewReader("helloworld"))
	ss.ReadBytes()
```
如果解决这个内容google在之后go版本中加入了scanner的内容，他同样是先把内容读取到缓存当中，然后根据不同的分隔符进行切割后输出
Scanner是一种复杂的结构，并且它嵌入了一个缓冲区。缓冲区可以动态增长(取决于scan函数的请求)，最大可达64kB(MaxScanTokenSize)。

先来看看整个数据结构
```go
type Scanner struct {
	r            io.Reader // 底层io
	split        SplitFunc // 切割方法， 使用*Scanner.Split(split SplitFunc) 调用
	maxTokenSize int      // 最大切割值
	token        []byte    // 返回切割后的数组
	buf          []byte    // buff数组
	start        int       // First non-processed byte in buf.
	end          int       // End of data in buf.
	err          error     // Sticky error.
	empties      int       // Count of successive empty tokens.
	scanCalled   bool      // Scan has been called; buffer is in use.
	done         bool      // Scan has finished.
}
```

使用方法如下
```go
package main

import (
	"bufio"
	"fmt"
	"strings"
)

func main() {
	// 将底层io的内容放到缓存区到中去
	scanner:=bufio.NewScanner(
		strings.NewReader("ABCDEFG\nHIJKELM"),
	)

	// 默认情况下按照\n为分隔符
	for scanner.Scan(){
		fmt.Println(scanner.Text()) // 返回分割后的数组并转换成string
		fmt.Println(scanner.Bytes()) // 返回分割后的数组，直接返回bytes数组
	}

	scanner=bufio.NewScanner(
		strings.NewReader("ABCDEFG\ndsds"),
	)
	scanner.Split(bufio.ScanBytes) // 按照字节返回
	for scanner.Scan(){
		fmt.Println(scanner.Text())
	}

	scanner=bufio.NewScanner(
		strings.NewReader("dsds" +
			"dsds" +
			"dsds" +
			"dsds" +
			"dsds"),
			)

	scanner.Split(bufio.ScanLines) //按照行来分割
	for scanner.Scan(){
		fmt.Println(scanner.Text()) //
	}


	scanner = bufio.NewScanner(strings.NewReader("hello world"))

	scanner.Split(bufio.ScanWords)
	for scanner.Scan(){
		fmt.Println(scanner.Text())
	}


	fmt.Println("cutsom-func\n")
	scanner = bufio.NewScanner(
		strings.NewReader("hello-world"))

	cutsom_split_func := func(data []byte, atEOF bool) (advance int, token []byte, err error){
		fmt.Printf("%t\t%d\t%s\n", atEOF, len(data), data)
		return 0,nil,nil
	}
	scanner.Split(cutsom_split_func)
	buf := make([]byte,2) // buffer区默认为4096 这里定义为2 当第一次没有读取完所有数据的时候，他会翻倍
	// 知道读取完之后，他会返回atEOF = true
	scanner.Buffer(buf,bufio.MaxScanTokenSize)
	for scanner.Scan(){
		fmt.Println(scanner.Text())
	}
	if scanner.Err() != nil {
		fmt.Printf("error: %s\n", scanner.Err())  //当EOF == true的时候表示读取完了，这个时候scanner.Err() != nil
    }
	// res
	// false	2	he
	// false	4	hell
	// false	8	hello-wo
	// false	11	hello-world
	// true	11	hello-world
	// error: bad luck

}
```

源码分析阶段

```go
// Copyright 2013 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package bufio

import (
	"bytes"
	"errors"
	"io"
	"unicode/utf8"
)

// Scanner provides a convenient interface for reading data such as
// a file of newline-delimited lines of text. Successive calls to
// the Scan method will step through the 'tokens' of a file, skipping
// the bytes between the tokens. The specification of a token is
// defined by a split function of type SplitFunc; the default split
// function breaks the input into lines with line termination stripped. Split
// functions are defined in this package for scanning a file into
// lines, bytes, UTF-8-encoded runes, and space-delimited words. The
// client may instead provide a custom split function.
//
// Scanning stops unrecoverably at EOF, the first I/O error, or a token too
// large to fit in the buffer. When a scan stops, the reader may have
// advanced arbitrarily far past the last token. Programs that need more
// control over error handling or large tokens, or must run sequential scans
// on a reader, should use bufio.Reader instead.
//
type Scanner struct {
	r            io.Reader // The reader provided by the client.
	split        SplitFunc // The function to split the tokens.
	maxTokenSize int       // Maximum size of a token; modified by tests.
	token        []byte    // Last token returned by split.
	buf          []byte    // Buffer used as argument to split.
	start        int       // First non-processed byte in buf.
	end          int       // End of data in buf.
	err          error     // Sticky error.
	empties      int       // Count of successive empty tokens.
	scanCalled   bool      // Scan has been called; buffer is in use.
	done         bool      // Scan has finished.
}

// SplitFunc is the signature of the split function used to tokenize the
// input. The arguments are an initial substring of the remaining unprocessed
// data and a flag, atEOF, that reports whether the Reader has no more data
// to give. The return values are the number of bytes to advance the input
// and the next token to return to the user, if any, plus an error, if any.
//
// Scanning stops if the function returns an error, in which case some of
// the input may be discarded.
//
// Otherwise, the Scanner advances the input. If the token is not nil,
// the Scanner returns it to the user. If the token is nil, the
// Scanner reads more data and continues scanning; if there is no more
// data--if atEOF was true--the Scanner returns. If the data does not
// yet hold a complete token, for instance if it has no newline while
// scanning lines, a SplitFunc can return (0, nil, nil) to signal the
// Scanner to read more data into the slice and try again with a
// longer slice starting at the same point in the input.
//
// The function is never called with an empty data slice unless atEOF
// is true. If atEOF is true, however, data may be non-empty and,
// as always, holds unprocessed text.
type SplitFunc func(data []byte, atEOF bool) (advance int, token []byte, err error)

// Errors returned by Scanner.
var (
	ErrTooLong         = errors.New("bufio.Scanner: token too long")
	ErrNegativeAdvance = errors.New("bufio.Scanner: SplitFunc returns negative advance count")
	ErrAdvanceTooFar   = errors.New("bufio.Scanner: SplitFunc returns advance count beyond input")
)

const (
	// MaxScanTokenSize is the maximum size used to buffer a token
	// unless the user provides an explicit buffer with Scanner.Buffer.
	// The actual maximum token size may be smaller as the buffer
	// may need to include, for instance, a newline.
	MaxScanTokenSize = 64 * 1024

	startBufSize = 4096 // Size of initial allocation for buffer.
)

// NewScanner returns a new Scanner to read from r.
// The split function defaults to ScanLines.
func NewScanner(r io.Reader) *Scanner {
	return &Scanner{
		r:            r,
		split:        ScanLines,
		maxTokenSize: MaxScanTokenSize,
	}
}

// Err returns the first non-EOF error that was encountered by the Scanner.
func (s *Scanner) Err() error {
	if s.err == io.EOF {
		return nil
	}
	return s.err
}

// Bytes returns the most recent token generated by a call to Scan.
// The underlying array may point to data that will be overwritten
// by a subsequent call to Scan. It does no allocation.
func (s *Scanner) Bytes() []byte {
	return s.token
}

// Text returns the most recent token generated by a call to Scan
// as a newly allocated string holding its bytes.
func (s *Scanner) Text() string {
	return string(s.token)
}

// ErrFinalToken is a special sentinel error value. It is intended to be
// returned by a Split function to indicate that the token being delivered
// with the error is the last token and scanning should stop after this one.
// After ErrFinalToken is received by Scan, scanning stops with no error.
// The value is useful to stop processing early or when it is necessary to
// deliver a final empty token. One could achieve the same behavior
// with a custom error value but providing one here is tidier.
// See the emptyFinalToken example for a use of this value.
var ErrFinalToken = errors.New("final token")

// Scan advances the Scanner to the next token, which will then be
// available through the Bytes or Text method. It returns false when the
// scan stops, either by reaching the end of the input or an error.
// After Scan returns false, the Err method will return any error that
// occurred during scanning, except that if it was io.EOF, Err
// will return nil.
// Scan panics if the split function returns too many empty
// tokens without advancing the input. This is a common error mode for
// scanners.
func (s *Scanner) Scan() bool {
	if s.done {
		return false
	}
	s.scanCalled = true
	// Loop until we have a token.
	for {
		// See if we can get a token with what we already have.
		// If we've run out of data but have an error, give the split function
		// a chance to recover any remaining, possibly empty token.
		if s.end > s.start || s.err != nil {
			advance, token, err := s.split(s.buf[s.start:s.end], s.err != nil)
			if err != nil {
				if err == ErrFinalToken {
					s.token = token
					s.done = true
					return true
				}
				s.setErr(err)
				return false
			}
			if !s.advance(advance) {
				return false
			}
			s.token = token
			if token != nil {
				if s.err == nil || advance > 0 {
					s.empties = 0
				} else {
					// Returning tokens not advancing input at EOF.
					s.empties++
					if s.empties > maxConsecutiveEmptyReads {
						panic("bufio.Scan: too many empty tokens without progressing")
					}
				}
				return true
			}
		}
		// We cannot generate a token with what we are holding.
		// If we've already hit EOF or an I/O error, we are done.
		if s.err != nil {
			// Shut it down.
			s.start = 0
			s.end = 0
			return false
		}
		// Must read more data.
		// First, shift data to beginning of buffer if there's lots of empty space
		// or space is needed.
		if s.start > 0 && (s.end == len(s.buf) || s.start > len(s.buf)/2) {
			copy(s.buf, s.buf[s.start:s.end])
			s.end -= s.start
			s.start = 0
		}
		// Is the buffer full? If so, resize.
		if s.end == len(s.buf) {
			// Guarantee no overflow in the multiplication below.
			const maxInt = int(^uint(0) >> 1)
			if len(s.buf) >= s.maxTokenSize || len(s.buf) > maxInt/2 {
				s.setErr(ErrTooLong)
				return false
			}
			newSize := len(s.buf) * 2
			if newSize == 0 {
				newSize = startBufSize
			}
			if newSize > s.maxTokenSize {
				newSize = s.maxTokenSize
			}
			newBuf := make([]byte, newSize)
			copy(newBuf, s.buf[s.start:s.end])
			s.buf = newBuf
			s.end -= s.start
			s.start = 0
		}
		// Finally we can read some input. Make sure we don't get stuck with
		// a misbehaving Reader. Officially we don't need to do this, but let's
		// be extra careful: Scanner is for safe, simple jobs.
		for loop := 0; ; {
			n, err := s.r.Read(s.buf[s.end:len(s.buf)])
			s.end += n
			if err != nil {
				s.setErr(err)
				break
			}
			if n > 0 {
				s.empties = 0
				break
			}
			loop++
			if loop > maxConsecutiveEmptyReads {
				s.setErr(io.ErrNoProgress)
				break
			}
		}
	}
}

// advance consumes n bytes of the buffer. It reports whether the advance was legal.
func (s *Scanner) advance(n int) bool {
	if n < 0 {
		s.setErr(ErrNegativeAdvance)
		return false
	}
	if n > s.end-s.start {
		s.setErr(ErrAdvanceTooFar)
		return false
	}
	s.start += n
	return true
}

// setErr records the first error encountered.
func (s *Scanner) setErr(err error) {
	if s.err == nil || s.err == io.EOF {
		s.err = err
	}
}

// Buffer sets the initial buffer to use when scanning and the maximum
// size of buffer that may be allocated during scanning. The maximum
// token size is the larger of max and cap(buf). If max <= cap(buf),
// Scan will use this buffer only and do no allocation.
//
// By default, Scan uses an internal buffer and sets the
// maximum token size to MaxScanTokenSize.
//
// Buffer panics if it is called after scanning has started.
func (s *Scanner) Buffer(buf []byte, max int) {
	if s.scanCalled {
		panic("Buffer called after Scan")
	}
	s.buf = buf[0:cap(buf)]
	s.maxTokenSize = max
}

// Split sets the split function for the Scanner.
// The default split function is ScanLines.
//
// Split panics if it is called after scanning has started.
func (s *Scanner) Split(split SplitFunc) {
	if s.scanCalled {
		panic("Split called after Scan")
	}
	s.split = split
}

// Split functions

// ScanBytes is a split function for a Scanner that returns each byte as a token.
func ScanBytes(data []byte, atEOF bool) (advance int, token []byte, err error) {
	if atEOF && len(data) == 0 {
		return 0, nil, nil
	}
	return 1, data[0:1], nil
}

var errorRune = []byte(string(utf8.RuneError))

// ScanRunes is a split function for a Scanner that returns each
// UTF-8-encoded rune as a token. The sequence of runes returned is
// equivalent to that from a range loop over the input as a string, which
// means that erroneous UTF-8 encodings translate to U+FFFD = "\xef\xbf\xbd".
// Because of the Scan interface, this makes it impossible for the client to
// distinguish correctly encoded replacement runes from encoding errors.
func ScanRunes(data []byte, atEOF bool) (advance int, token []byte, err error) {
	if atEOF && len(data) == 0 {
		return 0, nil, nil
	}

	// Fast path 1: ASCII.
	if data[0] < utf8.RuneSelf {
		return 1, data[0:1], nil
	}

	// Fast path 2: Correct UTF-8 decode without error.
	_, width := utf8.DecodeRune(data)
	if width > 1 {
		// It's a valid encoding. Width cannot be one for a correctly encoded
		// non-ASCII rune.
		return width, data[0:width], nil
	}

	// We know it's an error: we have width==1 and implicitly r==utf8.RuneError.
	// Is the error because there wasn't a full rune to be decoded?
	// FullRune distinguishes correctly between erroneous and incomplete encodings.
	if !atEOF && !utf8.FullRune(data) {
		// Incomplete; get more bytes.
		return 0, nil, nil
	}

	// We have a real UTF-8 encoding error. Return a properly encoded error rune
	// but advance only one byte. This matches the behavior of a range loop over
	// an incorrectly encoded string.
	return 1, errorRune, nil
}

// dropCR drops a terminal \r from the data.
func dropCR(data []byte) []byte {
	if len(data) > 0 && data[len(data)-1] == '\r' {
		return data[0 : len(data)-1]
	}
	return data
}

// ScanLines is a split function for a Scanner that returns each line of
// text, stripped of any trailing end-of-line marker. The returned line may
// be empty. The end-of-line marker is one optional carriage return followed
// by one mandatory newline. In regular expression notation, it is `\r?\n`.
// The last non-empty line of input will be returned even if it has no
// newline.
func ScanLines(data []byte, atEOF bool) (advance int, token []byte, err error) {
	if atEOF && len(data) == 0 {
		return 0, nil, nil
	}
	if i := bytes.IndexByte(data, '\n'); i >= 0 {
		// We have a full newline-terminated line.
		return i + 1, dropCR(data[0:i]), nil
	}
	// If we're at EOF, we have a final, non-terminated line. Return it.
	if atEOF {
		return len(data), dropCR(data), nil
	}
	// Request more data.
	return 0, nil, nil
}

// isSpace reports whether the character is a Unicode white space character.
// We avoid dependency on the unicode package, but check validity of the implementation
// in the tests.
func isSpace(r rune) bool {
	if r <= '\u00FF' {
		// Obvious ASCII ones: \t through \r plus space. Plus two Latin-1 oddballs.
		switch r {
		case ' ', '\t', '\n', '\v', '\f', '\r':
			return true
		case '\u0085', '\u00A0':
			return true
		}
		return false
	}
	// High-valued ones.
	if '\u2000' <= r && r <= '\u200a' {
		return true
	}
	switch r {
	case '\u1680', '\u2028', '\u2029', '\u202f', '\u205f', '\u3000':
		return true
	}
	return false
}

// ScanWords is a split function for a Scanner that returns each
// space-separated word of text, with surrounding spaces deleted. It will
// never return an empty string. The definition of space is set by
// unicode.IsSpace.
func ScanWords(data []byte, atEOF bool) (advance int, token []byte, err error) {
	// Skip leading spaces.
	start := 0
	for width := 0; start < len(data); start += width {
		var r rune
		r, width = utf8.DecodeRune(data[start:])
		if !isSpace(r) {
			break
		}
	}
	// Scan until space, marking end of word.
	for width, i := 0, start; i < len(data); i += width {
		var r rune
		r, width = utf8.DecodeRune(data[i:])
		if isSpace(r) {
			return i + width, data[start:i], nil
		}
	}
	// If we're at EOF, we have a final, non-empty, non-terminated word. Return it.
	if atEOF && len(data) > start {
		return len(data), data[start:], nil
	}
	// Request more data.
	return start, nil, nil
}


```

## lib_time



```go

package main

import (
	"fmt"
	"reflect"
	"time"
)

func main() {

	fmt.Println(time.Wednesday.String()) // 周三
	fmt.Println(time.Sunday.String()) // 周日

	fmt.Println(time.February)
	fmt.Println(reflect.TypeOf(time.February)) // time.Month


	// 返回当前时间
	fmt.Println(time.Now())

	timing()


	// 时间戳10位
	fmt.Println(time.Now().Unix())
	// 时间戳19位
	fmt.Println(time.Now().UnixNano())

	retrun_day()

	// 将时间戳转换为时间
	timestamp:=time.Now().Unix() // 时间戳
	timess:=time.Unix(timestamp, 0) // 时间戳转换为时间
	fmt.Println(timess)
	fmt.Println(timess.Unix()) // 时间戳转换为时间

	// 添加时间
	tt:=time.Now()
	tt=tt.Add(time.Hour)
	fmt.Println(tt)

	// 比较两个时间在前在后
	t1 := time.Now()
	t2 := t1.Add(time.Hour)
	_=t1.Equal(t2) // 比较是否相等
	_=t1.Before(t2) //
	_=t1.After(t2) // 比较t1的时间是否在t2之后


	// 计时器
	tickDemo()


	// 时间格式化
	formattime()
}


// 计时
func timing(){
	 t:=time.Now()
	 fmt.Println("do some function")
	 fmt.Println(time.Since(t).String())
}

// 显示具体某一个点的时间
func retrun_day(){
	year := time.Now().Year()
	yearday:=time.Now().YearDay()
	fmt.Println(year,yearday)
}

// 计时器
func tickDemo(){
	// 每20s执行一遍输出当前时间
	ticker := time.Tick(time.Second*20) //定时20S
	for i := range  ticker{
		fmt.Println(i)
	}
}

// 时间格式化
func formattime(){
	now := time.Now()
	// 格式化的模板为Go的出生时间2006年1月2号15点04分 Mon Jan
	// 24小时制
	fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan"))
	// 12小时制
	fmt.Println(now.Format("2006-01-02 03:04:05.000 PM Mon Jan"))
	fmt.Println(now.Format("2006/01/02 15:04"))
	fmt.Println(now.Format("15:04 2006/01/02"))
	fmt.Println(now.Format("2006/01/02"))
}
```


## lib_io_copy
之前已经有讲过`io.writer` `io.reader`两种接口类型了，现在在说一个`io.copy`

用一个实例的来说用
```go

func DownFile() {
	url :="http://wx.qlogo.cn/Vaz7vE1/64"
	resp ,err := http.Get(url)
	if err != nil {
		fmt.Fprint(os.Stderr ,"get url error" , err)
	}

	defer resp.Body.Close()
	
	// 上面部分是获取网页中的内容

	data ,err := ioutil.ReadAll(resp.Body)  // 读取body中的所有内容到内存区当中
	if err != nil {
		panic(err)
	}

	// 把内存区当中的内容写入到文件当中
	 _ =ioutil.WriteFile("/tmp/icon_wx.png", data, 0755)
}
```
如上这个代码 ,data就是缓存区中的内容，源代码如下
```go
func ReadAll(r io.Reader) ([]byte, error) {
	return readAll(r, bytes.MinRead)
}

func readAll(r io.Reader, capacity int64) (b []byte, err error) {
	var buf bytes.Buffer
	// If the buffer overflows, we will get bytes.ErrTooLarge.
	// Return that as an error. Any other panic remains.
	defer func() {
		e := recover()
		if e == nil {
			return
		}
		if panicErr, ok := e.(error); ok && panicErr == bytes.ErrTooLarge {
			err = panicErr
		} else {
			panic(e)
		}
	}()
	if int64(int(capacity)) == capacity {
		buf.Grow(int(capacity))
	}
	_, err = buf.ReadFrom(r)
	return buf.Bytes(), err
}

```
`readAll`有一个好处就是他会自动扩容缓存区，也就是说只要内存够大，readAll可以读取无限大的文件到缓存当中去，但是现实中的机器一定是有内存上的限制的，如果文件太大，就会出现内存溢出的问题，这个时候就要使用`io.copy` ,它是将源复制到目标，并且是按默认的缓冲区32k循环操作的，不会将内容一次性全写入内存中,这样就能解决大文件的问题。


源码内容如下
```go

// Copy copies from src to dst until either EOF is reached
// on src or an error occurs. It returns the number of bytes
// copied and the first error encountered while copying, if any.
//
// A successful Copy returns err == nil, not err == EOF.
// Because Copy is defined to read from src until EOF, it does
// not treat an EOF from Read as an error to be reported.
//
// If src implements the WriterTo interface,
// the copy is implemented by calling src.WriteTo(dst).
// Otherwise, if dst implements the ReaderFrom interface,
// the copy is implemented by calling dst.ReadFrom(src).
func Copy(dst Writer, src Reader) (written int64, err error) {
	return copyBuffer(dst, src, nil) 
}

// CopyBuffer is identical to Copy except that it stages through the
// provided buffer (if one is required) rather than allocating a
// temporary one. If buf is nil, one is allocated; otherwise if it has
// zero length, CopyBuffer panics.
func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {
	if buf != nil && len(buf) == 0 {
		panic("empty buffer in io.CopyBuffer")
	}
	return copyBuffer(dst, src, buf)
}

// copyBuffer is the actual implementation of Copy and CopyBuffer.
// if buf is nil, one is allocated.
func copyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {
	// If the reader has a WriteTo method, use it to do the copy.
	// Avoids an allocation and a copy.
	if wt, ok := src.(WriterTo); ok {
		return wt.WriteTo(dst)
	}
	// Similarly, if the writer has a ReadFrom method, use it to do the copy.
	if rt, ok := dst.(ReaderFrom); ok {
		return rt.ReadFrom(src)
	}
	if buf == nil {
		size := 32 * 1024   // 32k的缓冲区间， 在文件a 文件b的指针上直接做操作
		if l, ok := src.(*LimitedReader); ok && int64(size) > l.N {
			if l.N < 1 {
				size = 1
			} else {
				size = int(l.N)
			}
		}
		buf = make([]byte, size)
	}
	for {
		nr, er := src.Read(buf)
		if nr > 0 {
			nw, ew := dst.Write(buf[0:nr])
			if nw > 0 {
				written += int64(nw)
			}
			if ew != nil {
				err = ew
				break
			}
			if nr != nw {
				err = ErrShortWrite
				break
			}
		}
		if er != nil {
			if er != EOF {
				err = er
			}
			break
		}
	}
	return written, err
}
```

改良后的代码
```go

func DownFile() {
	url :="http://wx.qlogo.cn/Vaz7vE1/64"
	resp ,err := http.Get(url)
	if err != nil {
		fmt.Fprint(os.Stderr ,"get url error" , err)
	}


	defer resp.Body.Close()
	
	out, err := os.Create("/tmp/icon_wx_2.png")
	wt :=bufio.NewWriter(out)
	
	defer out.Close()
	
	n, err :=io.Copy(wt, resp.Body)
	fmt.Println("write" , n)
	if err != nil {
		panic(err)
	}
	wt.Flush()
}
```