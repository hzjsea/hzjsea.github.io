---
title: go_gotty
date: 2021-04-30 10:19:49
categories:  [golang]
tags: [golang]
---


<!--more-->


# gotty


gotty tty
https://hellogithub.com/periodical/volume/#category-11
https://zhuanlan.zhihu.com/p/26590894


## 不同类型大小之间的区别

在 golang 里面，有 int64,int32,int 这 3 种数据类型。
int64: 占 64 位的整数数据类型
int32: 占 32 位的整数数据类型
int: 在 64 位处理器上占 64 位，在 32 位处理器上面占 32 位

int 本身是个动态类型，在64位上会被转化成Int64, 在32位上会被转化成int32,这样就导致在编译的时候会对这些内容做一些判断，因此尽可能的指定好对应的数据类型

> 实际上，在不需要做存储和通讯协议数据对齐的场景下，long int 是在跨平台情况下效率最高的。
跨平台存储和通讯协议数据对齐场景下，4 字节就固定 int32，8 字节就固定 int64

另外还有一个是内存对齐的问题




> go的多个数值类型，应该选哪个

Go 内置很多种数值类型，往往初学者不知道编写程序如何选择，使用哪种数值类型更有优势。

内置的数值类型有：uint8、 uint16、 uint32、 uint64、 uint、 int8、 int16、 int32、 int64、 int。

从类型名称上可以很好了解到类型的大小，这个非常直观，uint 和 int 这两种类型是不带大小的，那么它们的大小会根据编译参数 GOARCH=amd64 平台决定的。

我最早设计的一个go的项目，当时设计系统使用采用最小类型原则，几乎使用了大多数数值类型，很少使用 uint 和 int 类型，后来遇到很多问题，标准库和三方库函数都接收 int、 uint、 int64、uint64, 一些代码生成工具, 比如 protobuf 生成类型是 int32，一些三方系统大多数也是 int 类型，这时候与其它组件件的交互就需要 类型转换, 类型转换成本是很高的，导致程序性能并没有预期的好。

上面一个小故事(事故)警醒大家不要一味的根据数据的大小选择数值类型，而要考虑数值的用来做什么，后面会有哪些交互，需要调用哪些函数等等，是不是选择数值具体使用什么类型很复杂呢？并不是这样，考虑的越少，选择越简单，下面有一些近些年的总结。

需要原子操作的数值根据数据大小选择 int32、 int64、 uint32、uint64。因为原子类型的操作包天生支持这些类型。
需要与代码生成的交互的数据，可以看生成的代码具体使用哪种类型，做一下参考。
需要调用大多数标准库函数进行处理，选这个 int (我们的程序大多数跑在64位系统上，如果运行在32系统，且类型可能会超过 int32 可以选择 int64) 。
有些时候可能我们需要一个无符号数据且比较大优先选用 uint 和 uint64 。
只和自己的函数交互以及一些不关注具体类型的包(json、fmt)交互式时，按数值使用范围选择最小类型。
我现在写代码一些特殊场景如原子操作会针对使用的包选择具体类型，偶尔会使用uint64，往往是一些按位做一些复杂计算的数据，也都局限在局部逻辑上，与其它模块或者系统交互的都会使用 int 类型，这样可以大幅度降低数值类型的类型转换问题，从而从空间换取时间，获得更好的程序性能。

不得不说说 Go 语言神奇的 int 类型，为什么需要这样一个编程是无法确定具体长度的类型呢，而需要在编译时确定呢，有什么好处呢。

往往我们写程序是不太关注数值类型的，或者说我们程序中很多数值不会超过 int32 的最大值（往往我们的程序运行在 32 或 64位平台上），这个时候很多三方库都可以使用 int 作为交互类型，不用把一个函数为每种类型数值都写一遍，能简化标准库。我们也能写出更容易维护、简洁的系统


## 变量间类型转换
类型转换一共分为三种
- 强制类型转换 
- 隐式类型转换 动态语言偏多
- 借助第三方库或者官方标准库做的类型转换

在golang中`:=`会发生隐式类型转换
```golang
package main

import "fmt"

func main() {
	var v int = 32.0
	fmt.Printf("%T",v) // int

	vv := 32.0
	fmt.Printf("%T",vv) // float64 
}
```
除去隐式类型转换,再来看看强制类型转换

```golang
package main

import "fmt"

func main() {
	var v int = 32.0
	fmt.Printf("%T",v) // int

	vv := 32.0
	fmt.Printf("%T",vv) // float64


	vvv := 32.0
	fmt.Printf("%T", int(vvv)) // int
}

```
但是强制类型转换往往会失去一定的精度上的计算，不同的语言处理方式不同，比如在golang中在强制类型转换之后，就会舍去小数点后的内容

```golang
package main

import "fmt"

func main() {
	var v int = 32.0
	fmt.Printf("%T\n",v) // int

	vv := 32.0
	fmt.Printf("%T\n",vv) // float64


	vvv := 32.6
	fmt.Printf("%d\n%T\n",int(vvv),int(vvv)) // 32 int // look here
}

```

第三方库转换，对于字符串类型，可以用第三方库`strconv`或者`parse类函数`,`Format类函数`进行转换

```golang
package main
import "fmt"

func main(){
    
    var xx int = 32
	fmt.Println(strconv.Itoa(xx)) // int to string

	var yy string = "32"
	if res,err := strconv.Atoi(yy); err == nil{  // string to int,error
		fmt.Println(res)
	} else if err != nil {
		fmt.Println("converted failed")
    }
    
    b,_ := strconv.ParseBool("true")
	fmt.Println(b)

	f,_ := strconv.ParseFloat("32.1",64) // string to float64
	fmt.Println(f)

    i,_ := strconv.ParseInt("32",10,64) // int10 to int int64
    fmt.Printf(i)

}
```

```golang
func ParseInt(s string, base int, bitSize int) (i int64, err error)
```
当base=0的时候,golag就会根据s的前缀去判断进制，0x则表示16进制, 0表示8进制, 其他都就会以10进制去处理 
bitSize表示要转换成多少位的数字，比如bitSize=8 则转换后就是 int8

<div style='font-size:20px;color:red'>parse和format是两个相反的函数</div>
Format类函数主要是将各种基础类型转换成string类型

```golang
	flag := true
	flag_bool := strconv.FormatBool(flag)
	fmt.Printf("%T\n",flag_bool) // string
```


## unicode与utf-8
了解过unicdeo和utf-8之后，就可以来看看golang当中字符串长度的问题了，对于一些静态语言，字符串长度中如果出现了中英文，计算出来的长度往往不好理解，在golang当中，他的长度是基于utf-8来做的，比如一个`中`字，在utf-8中是占3个自己，那算出来的结果就是3个字节的 `	fmt.Println(len("中")) // 3`

golang中string底层是通过byte数组([]byte()  []u8())实现的。中文字符在unicode下占2个字节，在utf-8编码下占3个字节，而golang默认编码正好是utf-8。
如果我们需要把它转换成unicode来看的话，可以通过rune, rune的底层类型是 int32 四个字节

```golang
func main(){
	fmt.Println(len([]rune("中"))) // 1 int32
	fmt.Println(len("中")) // 3  uint8
	fmt.Println(len([]rune("x"))) // 1 int32
	fmt.Println(len("x")) // 1 uint8
}
```
看似对单字节字符的长度没什么影响，但是在内存占用上是多了3个字节的，因为一个是int32 一个是uint8 , 所以对于rune 酌情使用

byte 等同于int8，常用来处理ascii字符
rune 等同于int32,常用来处理unicode或utf-8字符