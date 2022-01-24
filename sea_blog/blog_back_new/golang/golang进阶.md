---
title: golang进阶
categories:
  - golang
tags:
  - golang
abbrlink: 484c947
date: 2020-05-09 23:43:33
---
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [golang](#golang)
	- [go point](#go-point)
		- [函数指针](#函数指针)
	- [go struct](#go-struct)
		- [go结构体嵌套和匿名成员](#go结构体嵌套和匿名成员)
	- [go list](#go-list)
		- [语句转换](#语句转换)
		- [访问与赋值](#访问与赋值)
		- [数组取值循环 range](#数组取值循环-range)
		- [判断元素是否存在数组当中](#判断元素是否存在数组当中)
	- [go slice](#go-slice)
		- [切片中的nil值](#切片中的nil值)
		- [数组中嵌套struct](#数组中嵌套struct)
		- [nil切片零值](#nil切片零值)
		- [make创建动态数组](#make创建动态数组)
		- [数组追加元素append](#数组追加元素append)
	- [go map](#go-map)
		- [创建一个基本的map](#创建一个基本的map)
		- [利用map来获取相同key出现的次数](#利用map来获取相同key出现的次数)
		- [映射值](#映射值)
		- [使用map统计单词数量](#使用map统计单词数量)
	- [go function](#go-function)
		- [基本函数问题](#基本函数问题)
		- [匿名函数](#匿名函数)
		- [go必包](#go必包)
		- [函数返回多个值](#函数返回多个值)
	- [go method](#go-method)
		- [函数与方法中的指针](#函数与方法中的指针)
		- [函数与方法的区别](#函数与方法的区别)
	- [接口](#接口)
		- [实现一个简单的接口](#实现一个简单的接口)
		- [接口类型值](#接口类型值)
		- [为什么要使用接口](#为什么要使用接口)
		- [实现值类型接口与指针类型接口](#实现值类型接口与指针类型接口)
		- [实现接口与接口之间的嵌套](#实现接口与接口之间的嵌套)
		- [空接口](#空接口)
		- [接口值与接口类型](#接口值与接口类型)
		- [利用断言来判断接口类型](#利用断言来判断接口类型)
		- [接口与抽象类进阶](#接口与抽象类进阶)
	- [类型断言](#类型断言)
	- [go错误检查](#go错误检查)
		- [使用error.New创建一个错误提示](#使用errornew创建一个错误提示)
		- [常用错误解决方法](#常用错误解决方法)
	- [go reflect](#go-reflect)
	- [panic、recover、defer 的用法](#panicrecoverdefer-的用法)
	- [defer 向下抛到最后运行](#defer-向下抛到最后运行)
	- [golang test测试](#golang-test测试)
	- [go类型判断](#go类型判断)
	- [空接口](#空接口-1)
		- [空接口中存在什么](#空接口中存在什么)
	- [闭包](#闭包)

<!-- /code_chunk_output -->


<!-- more -->


# golang

## go point
go拥有指针，指针保存了值得内存地址
任何类型的指针的零值都是nil。如果p指向某个有效变量, 则`p!=nil` 其实就是变量的内存地址不能为空，当变量没有被赋值的时候，指针的地址就为nil
另外`*int` 表示指向int类型的指针

golang中的指针没有像C/C++中的那样复杂，他不支持">"操作,golang中只有如下两个操作那个
- & 取地址
- \* 根据(指针)地址取值 也就是&的反向操作
```go
func main() {
	v:="hzj"
	ptr := &v
	fmt.Println("指针的地址    ",ptr)
	fmt.Println("指针的所指向的地址上面的值", *ptr)
}
```

### 函数指针
golang中可以把指针作为返回值，其实就是返回内存值
```golang
package main

import "fmt"

func main() {
	pp := f()
	fmt.Println(pp)
}

func f() *int{
	p:= 22
	return &p
}
```
如果我们不断的修改指针指向的变量的值，他的地址也会不断的改变
```go
package main

import "fmt"

func main() {
	pp := f()
	fmt.Println(pp)

	for i:=0;i<=10;i++{
		var ss int = ff(&i)
		fmt.Println(ss,&ss)
	}
}

// 要求返回一个int类型的指针(地址)
func f() *int{
	p:= 22
	return &p // 取完地址返回
}

// 这里要求传入一个int类型的指针
func ff(p *int) int {
	*p++
	return *p
}

// result
// 0xc00001c0a0
// 1 0xc00001c0a8
// 3 0xc00001c0c8
// 5 0xc00001c0d8
// 7 0xc00001c0e8
// 9 0xc00001c0f8
// 11 0xc00001c108

```


## go struct

- Go 中的struct与C中的struct非常相似，并且Go没有class
- 使用 `type <Name> struct{}` 定义结构，名称遵循可见性规则(即首字母大写对外可见)。 type person struct{}
- 支持指向自身的指针类型成员，支持匿名结构，可用作成员或定义成员变量
- 匿名结构也可以用于struct的值，可以使用字面值对结构进行初始化
- 允许直接通过指针来读写结构成员
- 相同类型的成员可进行直接拷贝赋值
- 支持 == 与 != 比较运算符，但不支持 > 或 <
- 支持匿名字段，本质上是定义了以某个类型名为名称的字段
- 嵌入结构作为匿名字段看起来像继承，但不是继承
- 可以使用匿名字段指针


结构体内的变量最好都是大写的,一方面是方便转换成json(go中的json库默认不会解析struct中的小写字段)
创建结构体的语法以及调用结构体中的字段如下
```go
type Demo struct{
	Age int
	Name string
}
func main()  {
	// 实例化结构体
	var Demo1 = Demo{1,"hello"}
	var Demo2 = Demo{Age: 1,Name: "hello"}
	// 访问实例内容
	fmt.Println(Demo1.Name)
	// 修改实例内容
	Demo1.Name = "hzj"
	// 通过实例地址访问值来修改内容
	*&Demo1.Age =3
	fmt.Println(Demo1)
	// 通过结构体的(指针)地址来访问实例
	p2 := &Demo2
	fmt.Println(p2)
	p2.Age = 30
	fmt.Println(Demo2)

}
```
### go结构体嵌套和匿名成员
golang中的结构体支持嵌套和匿名成员，这里的匿名成员其实就是一组结构体，为了简写方便
```go
package main

import "fmt"

type Demo struct{
	Age int
	Name string
	Desc
}

type Desc struct {
	From string
	Dest string
	Date string
}

func main() {

	// 全都指定字段的写法
	var Desc1 = Desc{"1","2","3"}
	var Demo1 = Demo{Age: 1,Name: "hzj",Desc:Desc1}
	fmt.Println(Demo1)

	// 全都不指定字段
	var Demo2 = Demo{ 2, "3",Desc{"1","2","3"}} 
	fmt.Println(Demo2)
}

```




## go list
golang中的数组是一个静态数组，也就是在创建初期，长度=容量，golang支持两种方式对数组进行初始化，一种是在创建初期就对数组的长度进行显示的指定`var list = [3]int{1,2,3}`。另一种则是在创建的时候，根据后面的元素数量来决定数组的长度`[...]T`的形式进行指定
后一种声明方式在编译期间就会被『转换』成为前一种，这也就是编译器对数组大小的推导

两种不同形式的声明方式会导致编译器做出完全不同的处理

1. 定义数组的格式：var name [num]type  -->  var a [10]int  (定义1个长度为10类型为int的数组，数组名为 a)
2. 数组长度也是类型的一部分，因此具有不同长度的数组为不同的类型，这点很重要
3. 数组在Go中为值类型，故数组之间可以使用 ==、!= 进行比较，但不可以使用 <、> 进行判断
4. 可以使用 new 来创建数组，此方法返回一个指向数组的指针

来看下第四条内容,`new`的方法如下,但是new不经常使用，使用的通常是make函数
```go
func new(Type) *Type // 返回一个类型的指针(地址)
```
```go
	list := new([]int)
	fmt.Println("返回指针类型的地址     ",&list)  //   0xc00000e030
	fmt.Println("返回指针类型    ",list)  // &[]
	// 上面这两个取其实没有什么意义 
	fmt.Println(*list) //根据地址取值  // []

	//使用new赋值
	var a *int // 创建一个int指针类型的变量a
	a = new(int)
	// 根据指针类型取值
	*a = 10
	fmt.Println(*a)

	var b = make([]int,10)
	fmt.Println(b)
```

make函数的源码如下
```go
func make(t Type, size ...IntegerType) Type
```
make函数接收type类型,以及一系列属性，比如静态数组则需要定义len  切片则需要定义 len和cap ,返回一个类型的本身，而不是像new一样，返回的是一个类型指针



```go
package main

import "fmt"

func main() {
	// 创建数组 此时容量为2 长度为0
	var a[2] string
	//数组赋值
	a[0] = "hello"
	a[1] = "world"

	// 数组打印
	fmt.Println(a[0])
	fmt.Println(a)

	// 创建数组并且固定大小+固定元素
	arrary1 := [10]int{1,2,3,4,5,6,7,8,9,10}
	fmt.Println(arrary1)

	// 创建数组的时候不固定大小
	arrary2 := [...]int{1,2,3}  // 在编译器中被叫做DDDArray
	arrary2[0] = 3
	arrary2[1] = 4
	fmt.Println(arrary2)
}
```
两种数组创建时，数据长度上限推导
第一种在创建的时候就固定大小
https://github.com/golang/go/blob/616c39f6a636166447bdaac4f0871a5ca52bae8c/src/cmd/compile/internal/types/type.go#L473-L481
会采用上面的这种方式提取出来
https://github.com/golang/go/blob/b7d097a4cf6b8a9125e4770b54d33826fa803023/src/cmd/compile/internal/gc/typecheck.go#L2755-L2961
而不固定大小的方式则会以上面这种方式进行推导


### 语句转换
由字面量组成的数组，会根据数组中元素的个数实现优化
- 元素数量小于或等于4个的时候，会讲数组中的元素放到栈上
- 元素数量大于4个的时候，会讲数组中的元素放到静态区并运行时取出
![2020-08-21-14-42-54](http://noback.upyun.com/2020-08-21-14-42-54.png)

### 访问与赋值
无论是在栈上还是静态存储区，数组在内存中其实就是一连串的内存空间，表示数组的方法就是一个指向数组开头的指针、数组中元素的数量以及数组中元素类型占的空间大小，如果我们不知道数组中元素的数量，访问时就可能发生越界，而如果不知道数组中元素类型的大小，就没有办法知道应该一次取出多少字节的数据，如果没有这些信息，我们就无法知道这片连续的内存空间到底存储了什么数据

数组越界访问的错误处理
在rust中数组越界主要是通过Option来处理的
而在golang中会使用特定的函数在编译期中有静态检查完成
https://github.com/golang/go/blob/b7d097a4cf6b8a9125e4770b54d33826fa803023/src/cmd/compile/internal/gc/typecheck.go#L327-L2081
![2020-08-21-14-46-13](http://noback.upyun.com/2020-08-21-14-46-13.png)

报错的内容分别是
- 访问数组的索引是非整数时会直接报错 —— non-integer array index %v；
- 访问数组的索引是负数时会直接报错 —— "invalid array index %v (index must be non-negative)"；
- 访问数组的索引越界时会直接报错 —— "invalid array index %v (out of bounds for %d-element array)"；


### 数组取值循环 range
数组取值循环可以使用range来遍历其中的元素,取出来的数据主要包括`元素的序号`和`当前序号所对应的值`
```go
package main

import "fmt"

func main() {
	var listN = [...]int{1,2,3,4,5}
	for i,v := range listN{
		fmt.Println(i,v)
	}


	// 对于不需要取出来的值，可以进行_操作
	for _,v := range listN{
		fmt.Println(v)
	}
}
```

### 判断元素是否存在数组当中
因为go中有些内容并没有内置的函数，要自己写。这个是经常会使用到的函数
```go
package main

import "fmt"

// golang判断元素是否存在数组当中

func IsContain(items []string,item string) bool{
	for _,eachItem := range items{
		if eachItem  == item{
			return true
		}
	}
	return false
}
func main() {
	ss:="hello"
	var sentence = []string{"my","words","hello"}
	if IsContain(sentence,ss){
		fmt.Println("true")
	}else {
		fmt.Println("false")
	}

}
```

## go slice
golang中切片的使用率大于静态数组的使用率,切片表示一个拥有相同类型元素的可变长度
slice 类型的零值是nil,值为nil的slice没有底层数组，并且长度和容量都为0
- 其本身并不是数组，它指向底层的数组
- 作为变长数组的替代方案，可以关联底层数组的局部或全部
- slice 类型为引用类型
- 可以直接创建或从底层数组获取生存
- 使用 len() 获取元素个数，cap() 获取容量
- 一般使用 make() 创建
- 如果多个 slice 指向相同底层数组，其中一个的值改变会影响全部
- make([]T, len, cap)  其中 cap 可以省略，则和 len 的值相同，len 表示元素的个数，cap 表示当前最大可以存放的容量

```go
package main

import "fmt"

func main() {
	// 创建数组
	arrary1 := [10]int{1,2,3,4,5,6,7,8,9,10}
	fmt.Println(arrary1)
	//数组切片
	fmt.Println(arrary1[1:4])
}
```

切片数组反转 
```go
package main

import "fmt"

func main() {
	listN := []int{1,2,3,5,6,7}
	for i,j := 0,len(listN)-1 ; i<j ;i,j = i+1,j-1{
			listN[i],listN[j] = listN[j],listN[i]
	}
	fmt.Println(listN)

}
```

### 切片中的nil值
不同的创建形式，切片的值也会不同
```go
package main

import "fmt"

func main() {
	var s []int  // len(s) == 0 ;s == nil
	s = nil  // len(s) == 0 ; s == nil
	s = []int(nil) // len(s) == 0; s == nil
	s = []int{} // len(s) == 0; s!=nil
}
```
如果len(s) == 0 则slice 为空，
但是s == nil 时候， slice 不一定为空




### 数组中嵌套struct
数组可以有很多种，其中初始化可以这样初始化
因为数组规定了类型必须是一样的，为了让他存储多种类型的值，可以定义一个struct来存储
```go
q(被赋值的变量) := [](定义数组元素个数) int(数组中的变量类型) {}(数组元素集合)
```
因此我们可以这样来写
```go
package main

import "fmt"

func main() {
	
//  定义数据元素为int 类型
	q := []int{1,2,3,4,5,5}
	fmt.Println(q)

//  定义数据元素为string 类型
	r := [] string{"hzj","alpaca","code","cy"}
	fmt.Println(r)

// 数组中嵌套struct
	s := []struct{
		i int
		q string
	}{
		{2,"hzj"},
		{3,"hzj"},
	}

}
```

### nil切片零值
默认情况下go的切片的值为nil 也就是当数组为空的时候，他的值就是nil
```go
package main

import "fmt"

func main() {

	var s []int
	fmt.Println(s,len(s),cap(s))
	if s == nil{
		fmt.Println("nil")
	}
}
```

### make创建动态数组
内置make可以创建一个具有指定元素类型、长度和容量的slice,其中容量参数可以省略，在这种情况下 slice的长度和容量相等，另外在golang中slice 可以通过`len()`以及`cap()` 分别来获取slice的长度和容量

使用make 创建的slice 动态数组返回的是一个slice 
```go
package main

import "fmt"

func main() {
	// 创建一个长度为5的动态数组
	// 静态数组因为长度是固定不变的，所以不需要传入参数，即使一开声明过程中
	// 固定了数组的长度，也是需要填满的
	a := make([]int, 5)
	printslice("a", a)
	// res: [0 0 0 0 0]

	// 创建一个长度为0 容量为5的动态数组
	b := make([]int, 0, 5)
	printslice("b", b)

	c := b[:2]
	printslice("c", c)

	d := c[2:5]
	printslice("d", d)
}

func printslice(s string, x []int) {
	fmt.Printf("数组为%s,长度为%d 容量为%d %v\n",
		s, len(x), cap(x), x)
}

//res
// 数组为a,长度为5 容量为5 [0 0 0 0 0]
// 数组为b,长度为0 容量为5 []
// 数组为c,长度为2 容量为5 [0 0]
// 数组为d,长度为3 容量为3 [0 0 0]
```

```go
package main

import "fmt"

func main() {
	listN := make([]int,20)
	fmt.Println("长度是",len(listN))
	fmt.Println("容量是",cap(listN))
	//长度是 20
	//容量是 20

	listZ := make([]int ,20 ,30)
	fmt.Println("长度是",len(listZ))
	fmt.Println("容量是",cap(listZ))
	//长度是 20
	//容量是 30

	listError := make([]int ,30 ,20)
	fmt.Println("长度是",len(listError))
	fmt.Println("容量是",cap(listError))
	// 这里就会报错，因为创建的slice 容量小于长度
}
```

### 数组追加元素append
内置函数append用来将元素追加到slice后面
```go
package main

import "fmt"

func main() {
	var s []int
	printSlice(s)

	// 添加一个空切片
	s = append(s, 0)
	printSlice(s)

	// 这个切片会按需增长
	s = append(s, 1)
	printSlice(s)

	// 可以一次性添加多个元素
	s = append(s, 2, 3, 4)
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```

## go map
map是一个无序的基于key-value的数据结构，go语言中的map是一个引用类型，必须初始化后才能使用，没有初始化的时候，他就是一个Nil值
go语言中定义语法如下，可以他把看成字典，也就是keytype表示Key的类型  valuetype表示value

- 类似其它语言中的哈希表活着字典，以 key-value 形式存储数据
- key 必须是支持 == 或 != 比较运算的类型，不可以是函数、map 或 slice
- map 查找比线性搜索快很多，但比使用索引访问数据的类型慢100倍
- map使用 make() 创建，支持 := 这种简写方式
- make([keyType]valueType, cap)，cap表示容量，可省略。超出容量时会自动扩容，但尽量提供一个合理的初始值  make([int]string,10)
- 使用 len() 获取元素个数
- 键值对儿不存在时自动添加，使用 delete() 删除某键值对儿，使用 for range 对 map 和 slice 进行迭代操作



```go
map[KeyType]ValueType
```
- KeyType表示键类型
- ValueType表示值类型

### 创建一个基本的map
```go
package main

import "fmt"

func main() {
	// 创建一个空的map
	// { "age": 1 } 类似于这样的
	var map1 map[string]int
	// 这里返回值是nil 因为这就是个空的
	fmt.Println(map1)

	// 创建一个map， 并且初始化
	var map2 = make(map[string]int, 8)
	// 这里初始化了，但是值还是Nil 因为这里只定义了容量,这样做的好处是，防止后面再去动态添加内存
	fmt.Println(map2)

	// 给map2赋值
	map2["age"] = 2
	map2["name"] = 3
	fmt.Println(map2)

	//map[]
	//map[]
	//map[age:2 name:3]

	// 声明map的同时，初始化

	map3 := map[string]int{
		"name": 2,
		"age":  3,
	}
	fmt.Println(map3)
	//map[age:3 name:2]

	//判断值是否存在于map中
	// 类似于python中的
	// for key,value in ites_dict.items():
	//	if "xx" in key:
	//       return true
	// 这里取值，返回value 以及是否存在的表示flag
	v, flag := map3["name"]
	fmt.Println(v, flag)

	vv, flag := map3["class"]
	if flag == false {
		fmt.Println("不存在class这个值")
	} else {
		fmt.Println("存在class这个值，值为", vv)
	}

	// 遍历map组
	// map是无序的 键值对和添加的顺序无关
	for k, v := range map3 {
		fmt.Println(k, v)
	}
	// 只遍历key
	for k := range map3 {
		fmt.Println(k)
	}
	// 只遍历vlaue
	for _, v := range map3 {
		fmt.Println(v)
	}

	// 根据键删除值
	delete(map3, "name")
	for v, k := range map3 {
		fmt.Println(v, k)
	}
}

```

### 利用map来获取相同key出现的次数
```go
func getmax3() int {
	need := make(map[string]int)
	t:="hello" // 类型 unit8 byte类型  fmt.Println(reflect.TypeOf(t[1]))
	for i:=0 ;i<len(t);i++{
		need[string(t[i])]++
	}
	fmt.Println(need)
	return 0
}


// result   map[e:1 h:1 l:2 o:1]

func getmax3() int {
	need := make(map[byte]int)
	t:="hello" // 类型 unit8 byte类型  fmt.Println(reflect.TypeOf(t[1]))
	for i:=0 ;i<len(t);i++{
		need[t[i]]++
	}
	fmt.Println(need)
	return 0
}


// result map[101:1 104:1 108:2 111:1]

```

### 映射值
映射的零值为 nil 。nil 映射既没有键，也不能添加键。
make 函数会返回给定类型的映射，并将其初始化备用。
```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	fmt.Println(m["Bell Labs"])
}
```


### 使用map统计单词数量
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var words = "how are you"
	fmt.Println(words)

	words_list := strings.Split(words, " ")
	fmt.Println(words_list)

	var wordsCount = make(map[string]int, 10)
	for _, value := range words_list {
		v, ok := wordsCount[value]

		if ok {
			wordsCount[value] = v + 1
		} else {
			wordsCount[value] = 1
		}
	}

	for v, ok := range wordsCount {
		fmt.Println(v, ok)
	}

}

```

## go function
-  Go 函数 不支持 嵌套、重载 和 默认参数
-  Go 函数支持一下特性：
   - 无需声明原型
   - 不定长度变参
   - 可以返回多个值
   - 命名返回值参数
   - 匿名函数
   - 闭包
-  定义函数使用关键字 func，且左大括号不能另起一行
-  函数也可以作为一种类型使用
### 基本函数问题
```go
package main

import "fmt"

func main() {
	havefunc(1, 2, 3, 5)
	havefunc1(1, 2)
	res := havefunc2()
	fmt.Println(res)
	fixed_res := havefunc2_fixed()
	fmt.Println(fixed_res)
}

// 固定参数函数
func havefunc1(x int, y int) {
	fmt.Println(x + y)
}

// 有返回值的函数
func havefunc2() string {
	return "havefunc2"
}

// 命名函数返回值
func havefunc2_fixed() (res string) {
	// 另外这里提醒一下，上面固定了返回类型，这个res的变量已经存在了
	//res := "xxx" 如果这样写的话，其实会报错说没有新的变量在:=左边
	res = "这是一个固定的返回变量"
	return res
}

// 可变参数的函数
// 这里的参数为可变参数列表，其实它就是一个slice
// 强制规定：可变参数必须放在参互列表的最后位，如果还有其他参数请将其他参数放置其前面
func havefunc(x ...int) {
	for i := range x {
		fmt.Println(i)
	}

}

```

### 匿名函数
```go
func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))
}

```

### go必包
- 闭包（closure）是 javascript 的一大难点，也是它的特色。很多高级应用都是依靠闭包来实现的。
- 变量的作用域
要理解闭包，首先要理解javascript的特殊的变量作用域。变量的作用域无非就两种：全局变量 和 局部变量。
注意语法规则：函数内部可以直接读取全局变量，但是在函数外部无法读取函数内部的局部变量。

![2020-09-11-17-12-13](http://noback.upyun.com/2020-09-11-17-12-13.png)
受益 https://www.cnblogs.com/liang1101/p/6606667.html


- 使用闭包注意事项
由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。
闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。
另外闭包与匿名函数也经常一起出现
```go
func func_outer() func(int2 int) int{ // 后面一段归为一类看
	return func(i int) int {
		return i
	}
}
```
func_outer 返回一个函数(这个函数为返回一个传入一个int类型，返回一个int类型的函数)

```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}

```

### 函数返回多个值
```go
package main

import "fmt"

func main() {
	listString :=  []string{"1","2","3"}
	count,endwords :=returnManyWords(listString)
	fmt.Println(count,endwords)
}

// 返回多个值
func returnManyWords(listString []string)(count int , endwords string){
	for i,v := range listString{
		fmt.Println(i,v)
		count ++
	}
	return count , listString[len(listString)-1]
}
```


## go method
方法 method
- Go 中虽没有 class，但依旧有 method
- 通过显示说明 receiver 来实现与某个类型的组合
- 只能为同一个包中的类型定义方法
- Receiver 可以是类型的值或者指针
- 不存在方法重载
- 可以使用值或指针来调用方法，编译器会自动完成转换
- 从某种意义上来说，方法是函数的语法糖，因为 receiver 其实就是方法所接收的第1个参数（Method Value vs. Method Expression）
- 如果外部结构和嵌入结构存在同名方法，则优先调用外部结构的方法
- 类型别名不会拥有底层类型所附带的方法
- 方法可以调用结构中的非公开字段

```go
package main

import "fmt"

type A struct {
    Name string
}

type B struct {
    Name string
}

//引用传递得道到是指针到拷贝，修改会同步修改结构体内到内容
func (a *A) Print() {
    a.Name = "AA"
    fmt.Println("A")
}

//值传递只是得到结构体内容到拷贝
func (b B) Print()  {
    b.Name = "BB"
    fmt.Println("B")
}

func main() {
    a := A{}
    a.Print() // AA
    fmt.Println(a.Name) // AA

    b := B{}
    b.Print() // BB
    fmt.Println(b.Name) // 空字符串
}
```

Go 还有一个特别灵活的地方，就是它可以为任何一个声明类型绑定定义一个方法，如下：
```go
package main

import "fmt"

/**
Go到一个灵活到地方就是可以为type声明到任何类型绑定一个方法
例如下面到例子，定义了类型为整型到变量Va，为其绑定一个输出函数并在主方法中调用
 */
type Va int

func (a *Va) Print() {
    fmt.Println("Va")
}

func main() {
    var a Va
    a.Print()
}
```



### 函数与方法中的指针
在之前 函数与方法两种写法不一样，调用的方法也不一样，如下
```go
package main

import "fmt"

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) { // 方法
	v.X = v.X * f
	v.Y = v.Y * f
}

func ScaleFunc(v *Vertex, f float64) {  // 函数
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(2)
	ScaleFunc(&v, 10)

	p := &Vertex{4, 3}
	p.Scale(3)
	ScaleFunc(p, 8)

	fmt.Println(v, p)
}

```

### 函数与方法的区别
方法更加对象抽象化
```go
type Rectangle struct {
	width, height float64
}

//　　这里就是函数即func,其声明：
//   func funcName(parameters) (results)
func area(r Rectangle) float64 {
    return r.width*r.height
}

// 这里就是方法即method，其声明：
// func (r ReceiverType) funcName(parameters) (results)

func (r Rectangle) area() float64 {
    return r.width*r.height
}


func main() {
	c1 := Rectangle{20.0,20.0}
	c1.area() //方法
	area(c1) //函数

}
```



## 接口
接口类型 是由一组方法签名定义的集合，就是定义了一组方法，这组方法可以不关心属性，只关心实现方法 类似于python中的类，接口就是一种抽象的类
如果说goroutine和channel是Go并发的两大基石，那么接口是Go语言编程中数据类型的关键。在Go语言的实际编程中，几乎所有的数据结构都围绕接口展开，接口是Go语言中所有数据结构的核心。
Go不是一种典型的OO语言，它在语法上不支持类和继承的概念。
go语言接口U的独特之处在于他是隐式实现，对于一个具体的类型，无须声明它实现了哪些接口，只要提供接口所必须的方法即可

接口的本质就是引入一个新的中间层，调用方可以通过接口与具体实现分离，解除上下游的耦合，上层的模块不再需要依赖下层的具体模块，只需要依赖一个约定好的接口。
![golang-interface](https://img.draveness.me/golang-interface.png)


- 接口是一个或多个方法签名的集合
- 只要某个类型拥有该接口的所有方法签名，即算实现该接口，无需显示声明实现了哪个接口，这称为 `Structural typing`
- 接口只有方法声明，没有实现，没有数据字段
- 接口可以匿名嵌入其它接口，或切入到结构中去
- 将对象赋值给接口时，会发生拷贝，而接口内部存储的是指向这个复制品到指针，既无法修改复制品的状态，也无法获取指针
- 只有当接口存储的类型和对象都为 nil 时，接口才等于 nil
- 接口调用不会做 receiver 的自动转换
- 接口同样支持匿名字段方法
- 接口也可实现类似 OOP 中的多态
- 空接口可以作为任何类型数据的容器

### 实现一个简单的接口
```go
package main

import (
	"fmt"
)

// 声明接口类型的方式
/*

type Name interface {
	Method1(param_list) return_type
	Method2(param_list) return_type
...
}

*/

// 声明一个接口类型
// 这里声明了一个area的接口类型,接口中包含了一个不带参数、返回值为 string 的方法 func_area()
// 任何实现了方法 Area() 的类型 T，我们就说它实现了接口 Shape。
type area interface {
	func_area() string
}

func main() {
	// 创建接口实例
	var ar area
	// 使用接口 这里会返回一个零值得
	fmt.Println("value of s is", ar)
	fmt.Printf("type of s is %T\n",ar)
}

// res:
// value of s is <nil>
// type of s is <nil>


```

### 接口类型值
变量的类型在声明时指定、且不能改变，称为静态类型。接口类型的静态类型就是接口本身。接口没有静态值，它指向的是动态值。接口类型的变量存的是实现接口的类型的值。该值就是接口的动态值，实现接口的类型就是接口的动态类型。

在实现最简单的接口类型的时候，看上面的代码块。接口返回的值和和类型都是nil值.这是因为我们并没有给这个接口实现具体的类型，有因为接口没有静态值，静态类型是其本身，也就是area 
但他们的动态值和动态类型都是nil，现在我们来给他们的动态值和动态类型来赋值

```go
package main

import "fmt"

type area2 interface {
	func_area() string
}

// 创建一个空结构体
type int_type1 struct{}
type int_type2 struct{}

// 实现接口
func (int_type1) func_area() string { return "hello int_type1" }
func (int_type2) func_area() string { return "hello int_type2" }

func main() {
	// 创建接口实例
	var ar2 area2 = int_type1{}
	// 返回类型
	fmt.Printf("%T\n", ar2)
	//
	fmt.Printf("%v\n", ar2)

	ar2 = int_type2{}

	fmt.Printf("%T\n", ar2)
	//
	fmt.Printf("%v\n", ar2)
}


 // res
 // main.int_type1
 // {}
 // main.int_type2
 // {}

```

### 为什么要使用接口
接口就是一组具体方法的组合，就好比一根电缆，每一个方法就是里面的一根光纤，接入光纤的时候，与家庭中的任何属性都无关，光纤只实现它自己的方法
```go
package main

import "fmt"

// 首先我们来试着用struct

type animal struct{}

func (a animal) say() {
	fmt.Println("i am animal")
}

// 引入接口
type fruit struct{}

func (f fruit) say() {
	fmt.Println("i am fruit")
}

// 定义接口
// 在接口中声明say方法
type Interfaces interface {
	say()
}

type personal struct {
	name string
	age  int
	desc string
}

func (p personal) say() {
	fmt.Println("hello world ", p.name)
}

func main() {
	a := animal{}
	a.say()

	i := Interfaces(fruit{})
	//fmt.Println(i.say())
	i.say()

	q := Interfaces(personal{"hzj", 12, "hello"})
	q.say()

}

```
这里我们用空结构体来实现对象，其中有一个animal的对象我们只是简单的实现了调用方法
```go
package main

import "fmt"

// 首先我们来试着用struct

type animal struct{}

func (a animal) say() {
	fmt.Println("i am animal")
}


func main() {
	a := animal{}
	a.say()
}

```
如上是实现了一个空结构体对象实现的一个say方法,接下来是用接口实现
```go
package main
import "fmt"

type fruit struct{}

func (f fruit) say() {
	fmt.Println("i am fruit")
}

type personal struct {
	name string
	age  int
	desc string
}

func (p personal) say() {
	fmt.Println("hello world ", p.name)
}

// 定义接口
// 在接口中声明say方法
type Interfaces interface {
	say()
}


func main() {
	i := Interfaces(fruit{})
	//fmt.Println(i.say())
	i.say()
	
	q := Interfaces(personal{"hzj", 12, "hello"})
	q.say()

}

```
这样我们只需要对该接口传入我们的对象就可以实现方法了，但是接口并不是这么用的
接口主要的作用是用来做解耦，就像上面的，虽然上面的方法中我们定义了接口，并且看上去是使用了接口，但其实不然，并没有达到解耦的效果
在声明同一个接口的时候，我们传入了不同的struct对象，返回的也是不同的inter接口对象，但其实我们可以对这一步再进行一个封装
```go
package main

import "fmt"

// 首先我们来试着用struct

type animal struct{}

func (a animal) say() {
	fmt.Println("i am animal")
}

// 引入接口
type fruit struct{}

func (f fruit) say() {
	fmt.Println("i am fruit")
}


type personal struct {
	name string
	age  int
	desc string
}

func (p personal) say() {
	fmt.Println("hello world ", p.name)
}

// 定义接口
// 在接口中声明say方法
type Interfaces interface {
	say()
}


func DotypeInter(i Interfaces){
	i.say()
}

func main() {


	ii := fruit{}
	DotypeInter(ii)

	qq := animal{}
	DotypeInter(qq)

	pp := personal{"hzj",12,"hellowrold"}
	DotypeInter(pp)
}

```
如上，我们又一步的封装了接口.
或者你也可以这样写，因为之前我们所有只要实现了这个接口中的所有方法，你就等于实现了这个接口
```go
package main

import "fmt"

// 首先我们来试着用struct

type animal struct{}

func (a animal) say() {
	fmt.Println("i am animal")
}

// 引入接口
type fruit struct{}

func (f fruit) say() {
	fmt.Println("i am fruit")
}


type personal struct {
	name string
	age  int
	desc string
}

func (p personal) say() {
	fmt.Println("hello world ", p.name)
}


// 定义一个没有实现任何方法的struct
type empty_struct struct {

}

func main() {
	// 定义一个接口
	var i Interfaces
	// 创建一个空结构体
	zz := fruit{}
	// 因为这个结构体实现了i这个接口中的所有方法，也就是实现了这个接口，所以可以直接赋值过去
	i = zz
	// 调用方法
	i.say()

	// 这样写就会报错，因为没有实现接口中的方法
	// es := empty_struct{}
	// i = es
	// i.say()
}

```
这样做的好处就是，下一次我不像再用fruit这个对象了，我想使用personal这个对象了，其他的都不用变，只需要改变传入参数就行了

```go
func main() {
	var i Interfaces
	zz := fruit{}
	i = zz
	i.say()

	// example
	pp := personal{"hzj",12,"hello"}
	pp.say()
}
```

### 实现值类型接口与指针类型接口
无论是值类型接口还是指针类型接口，都会被保存在接口的变量当中
```go
package main

import "fmt"

type valueInt struct {
	name string
}

func (vi valueInt) say (){
	fmt.Printf("hi %s, say your words\n",vi.name)
}

type pointInt struct{
	name string
}

func (pi *pointInt) say() {
	fmt.Printf("hi %s,say your words\n",pi.name)
}

// 创建一个接口
type interface_pi interface {
	say()
}

func main(){
	// 声明接口
	var ipp interface_pi

	// 创建值类型实例
	vi := valueInt{"hzj"}
	ipp = vi
	ipp.say()

	// 创建指针类型实例
	pi := pointInt{"hzj"}
	ipp = &pi
	ipp.say()
}
```
### 实现接口与接口之间的嵌套
```go
package main

import "fmt"

// 先实现一个say接口
type sayword interface {
	say()
}
// 再实现一个walk接口
type walkdone interface {
	walk()
}

// 实现一个聚合接口包含say walk
type SayAndWalkdone interface {
	sayword
	walkdone
}


// 创建一个personal 实现say walk 方法
type personal struct {

}

func (p personal) say()  {
	fmt.Println("i can say")
}

func (p personal) walk() {
	fmt.Println("i can walk")
}

// 创建一个animal 仅实现walk 方法
type animal struct {

}

func (a animal) walk () {
	fmt.Println("i can walk")
}



func main(){
	// 创建聚合接口
	var sad SayAndWalkdone
	// 创建personal实例
	p1 := personal{}
	sad = p1
	sad.walk()
	sad.say()

	// 创建say接口
	var walkdone walkdone
	a1 := animal{}
	walkdone = a1
	walkdone.walk()
}
```

### 空接口
空接口就是没有定义任何方法的接口，所以任何类型都实现了空接口
空接口类型变量可以存储任意类型的变量

```go
package main

import "fmt"

func main()  {
	var x interface{}
	x = "helloworld"
	fmt.Println(x)

	// Println的实现方法
	//func Println(a ...interface{}) (n int, err error) {
	//	return Fprintln(os.Stdout, a...)
	//}

	// 利用空接口实现一个map 实现value值为任意值

	// 创建一个普通的map
	var map1 = make(map[string]int)
	map1["test1"] = 1
	map1["test2"] = 2
	for k,v := range(map1){
		fmt.Println(k,v)
	}


	// 创建一个任意类型value的map
	var map2 = make(map[string]interface{})
	map2["test"] = 1
	map2["test2"] = "hello"
	map2["list"] = []int{1,2,3,4,5}
	for k,v := range(map2){
		fmt.Println(k,v)
	}

}
```

### 接口值与接口类型
一个接口的值是由一个具体的值和一个具体的类型决定的 他们分别被叫做接口的动态值和动态类型
```bash
package main

import "fmt"

func main()  {
	var x interface{}
	fmt.Printf("接口动态类型 %T\n",x)
	fmt.Printf("接口动态值 %v\n",x)


	x = "hello"
	fmt.Printf("接口动态类型 %T\n",x)
	fmt.Printf("接口动态值 %v\n",x)

	x = true
	fmt.Printf("接口动态类型 %T\n",x)
	fmt.Printf("接口动态值 %v\n",x)
}

# 接口动态类型 <nil>
# 接口动态值 <nil>
# 接口动态类型 string
# 接口动态值 hello
# 接口动态类型 bool
# 接口动态值 true

```
### 利用断言来判断接口类型
```go
package main

import "fmt"

func main()  {
	x = true
	fmt.Printf("接口动态类型 %T\n",x)
	fmt.Printf("接口动态值 %v\n",x)


	_,ok := x.(bool)
	if ok {
		fmt.Println("这是一个bool值")
	}else {
		fmt.Println("这不是一个布尔值")
	}
}

//这是一个bool值

```

### 接口与抽象类进阶
在使用接口的过程中，发现接口和抽象类有点类似，问了一下周围的人发现大部分人都在用接口做项目，抽象类用的比较少，但类似于生成数据库表的时候会涉及到抽象类的使用
首先接口和抽象类的设计目的就是不一样的。接口是对动作的抽象，而抽象类是对根源的抽象。对于抽象类，比如男人，女人这两个类，那我们可以为这两个类设计一个更高级别的抽象类--人。对于接口，我们可以坐着吃饭，可以站着吃饭，可以用筷子吃饭，可以用叉子吃饭，甚至可以学三哥一样用手抓着吃饭，那么可以把这些吃饭的动作抽象成一个接口--吃饭。所以在高级语言中（如Java,C#），一个类只能继承一个抽象类（因为你不可能同时是生物又是非生物）。但是一个类可以同时实现多个接口，比如开车接口，滑冰接口，啪啪啪接口，踢足球接口，游泳接口。也就是说抽象类的使用更加高级，但没有必要设计抽象类的设计在普通项目中
抽象类的功能应该要远多于接口，但是定义抽象类的代价较高。因为高级语言一个类只能继承一个父类，即你在设计这个类的时候必须要抽象出所有这个类的子类所具有的共同属性和方法；但是类（接口）却可以继承多个接口，因此每个接口你只需要将特定的动作方法抽象到这个接口即可。也就是说，接口的设计具有更大的可扩展性，而抽象类的设计必须十分谨慎。

另外接口总是与方法一起出现 不是与函数一起出现 来看一下函数与方法的区别
`func (a type) getMax() int {}` 这个是方法
`func getMax(a type) int{}` 这个是函数


来看一段代码，涉及到了接口的使用
```go
package main

import "fmt"

/*
定义一个结构体
*/

type Run struct{
	leg string
	arm string
}

type  Speak struct {
	mouth string
	arm string
	eye string
}

type People struct {
	Speak
	Run
	name string
	age string
	habit string
	sex string
}

// 使用普通方法
func CreatePeople() *People{
	run := Run{arm:"fast",leg: "fast"}
	speak := Speak{arm: "fast",mouth: "fast",eye:"eyes is fast"}
	people := People{speak,run,"hzj","12","linke","man"}
	return &people
}
func RunFunc(p1 *People) {
	fmt.Println(p1.Run)
}

func SpeakFunc(p1 *People){
	fmt.Println(p1.Speak)
}

// 使用接口
type Runer interface {
	RunImpl()
}

type UseSpeaker interface {
	SpeakImpl()
	SpeakNameImpl()
}

// 给具体的数据类型实现接口中的方法
func (p1 People) RunImpl() {
	fmt.Println("running ")
}

func (p1 People) SpeakImpl() {
	fmt.Println("speaking ")
}

func (p1 People) SpeakNameImpl(){
	fmt.Println("my name is",p1.name)
}

func main() {
	p1 := CreatePeople()
	RunFunc(p1)
	SpeakFunc(p1)
	p1.SpeakImpl()
	p1.SpeakNameImpl()
	p1.RunImpl()
	
}
```
在接口的使用过程中，只要某一个实例实现了接口中定义的全部方法，则表示实现了该接口
如上 所有的people类型都实现了Runer接口和UseSpeaker接口
另外接口与接口之间也可以实现嵌套

```go
package main

import (
	"fmt"
)

/*
定义一个结构体
*/

type Run struct{
	leg string
	arm string
}

type  Speak struct {
	mouth string
	arm string
	eye string
}

type People struct {
	Speak
	Run
	name string
	age string
	habit string
	sex string
}

// 使用普通方法
func CreatePeople() *People{
	run := Run{arm:"fast",leg: "fast"}
	speak := Speak{arm: "fast",mouth: "fast",eye:"eyes is fast"}
	people := People{speak,run,"hzj","12","linke","man"}
	return &people
}

// 使用接口
type Runer interface {
	RunImpl()
	UseSpeaker
}

type UseSpeaker interface {
	SpeakImpl()
	SpeakNameImpl()
}

func (p1 People) RunImpl() {
	fmt.Println("running ")
}

func (p1 People) SpeakImpl() {
	fmt.Println("speaking ")
}

func (p1 People) SpeakNameImpl(){
	fmt.Println("my name is",p1.name)
}



func main() {
	p1 := CreatePeople()
	p1.SpeakImpl()
	p1.SpeakNameImpl()
	p1.RunImpl()
	var p2 UseSpeaker = UseSpeaker(*p1) // 强制转换接口
	p2.SpeakImpl()
	p2.SpeakNameImpl() // 只能实现两种方法

	var p3 Runer = Runer(*p1)
	p3.SpeakNameImpl()
	p3.SpeakImpl()
	p3.RunImpl()
}

```
- 强制转换接口，只能从高到低转换`var p2 UseSpeaker = UseSpeaker(*p1)`


## 类型断言
类型断言格式：`usb.(要判断的对象)`，判断传入的是否为Runer类型
关键词 interface switch

类型断言 x.(T) 其实就是判断 T 是否实现了 x 接口，如果实现了，就把 x 接口类型具体化为 T 类型；而 x.(type) 这种方式的类型断言，就只能和 switch 搭配使用，因为它需要和多种类型比较判断，以确定其具体类型。


```go
func checkType(args ...interface{}) {
   for _, arg := range args {
      switch arg.(type) {
      case int:
         fmt.Println(arg, "is an int value.")
      case string:
         fmt.Println(arg, "is a string value.")
      case int64:
         fmt.Println(arg, "is an int64 value.")
      default:
         fmt.Println(arg, "is an unknown type.")
      }
   }
}
```
golang里面所有的数据类型都都是一个空接口
```go
// 接口类型就可以
var name interface{} = 1
v,ok := name.(int)

fmt.Println(v,ok) 

// result  1 int
```

```go
func Disconnect(runner Runer) {
	/**
	  这里是通过类型断言来判定当前是哪个对象调用并打印
	  类型断言格式：usb.(要判断的对象)，判断传入的是否为Runer类型
	*/
	if k, ok := runner.(Runer); ok {
		k.SpeakImpl()
		return
	}
}

/**
Go语言中类型定义只要符合定义的接口那么它就实现来该接口，在java语言中都有一个顶级父类叫 Object。那么在Go语言中有吗？
当然有的，通过接口概念可以了解到当我们定义一个空接口的时候，任何类型都会继承它。那么针对Disconnect方法我们就可以定义一种更加广泛的应用方式
*/
func disconnectAll(NilInter interface{}){
	if k,ok := NilInter.(Runer); ok{
		k.SpeakNameImpl()
		return
	}
	fmt.Println("error")

}


/**
在Go语言中可以用另外一种方式进行断言，叫：type switch
*/
func DisconnectSwitch(usb interface{}) {
	switch v := usb.(type) {
	case PhoneConnecter:
		fmt.Println("DisconnectSwitch：", v.name)
	default:
		fmt.Println("Unknown decive.")
	}
}
```


## go错误检查
golang错误检查，源码如下
```go
type error interface {
    Error() string
}
```

### 使用error.New创建一个错误提示
```go
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // 其他逻辑实现
}
```
### 常用错误解决方法
```go
result, err:= Sqrt(-1) // 产生一个错误

if err != nil {  // 如果错误存在 则执行解决方法
   fmt.Println(err)
}
```


## go reflect
https://studygolang.com/articles/24406
反射reflection
- 反射可以大大的提高程序的灵活性，使得 interface{} 有更大的发挥余地
- 反射使用 TypeOf 和 ValueOf 函数从接口中获取目标对象信息
- 反射会将匿名字段作为独立字段（匿名字段本质）
- 想要利用反射修改对象状态，前提是 interface.data 是 settable，即 pointer-interface
- 通过反射可以“动态”调用方法

```go

type User struct {
	Id int
	Name string
	Age int
}

func (u User) hello(){
	fmt.Println("helloworld")
}

func Info(o interface{}){
	t := reflect.TypeOf(o)   // 获取接受到接口的类型
	fmt.Println(t.Name()) // 输出类型名字

	// 判断是哪种类型
	//reflect.Int
	//reflect.String 等等
	if k:= t.Kind();k != reflect.Struct{
		fmt.Println("传入的类型有误，请检查")
		return
	}

	// 获取接口到的接口的类型+内容(属性字段和方法)
	v := reflect.ValueOf(o)
	fmt.Println("Fields:")
	for i:=0;i<t.NumField();i++{
		fmt.Println(i,t.Field(i),v.Field(i).Interface()) // 输出字段，各个属性
		f:=t.Field(i)
		fmt.Println(f.Name,f.Type)
	}
	// 通过接口类型.NumMethod 获取当前类型所有方法的个数
	for i:=0;i<t.NumMethod();i++{
		m:=t.Method(i)
		fmt.Println(m.Name,m.Type)
	}

}

func main() {
	u1 := User{1,"hzj",2}
	u1.hello()
	Info(u1)
}


```

## panic、recover、defer 的用法
- 它的执行方式类似其他语言中的折构函数，在函数体执行结束后按照调用顺序的 相反顺序 逐个执行
- 即使函数发生 严重错误 也会被执行，类似于 java 中 try{...} catch(){} finally{} 结构的 finally
- 支持匿名函数的调用
- 常用于资源清理、文件关闭、解锁以及记录时间等善后操作
- 通过与匿名函数配合可在 return 之后修改函数计算结果
- 如果函数体内某个变量作为 defer 时匿名函数的参数，则在定义 defer 时即已经获得了拷贝，否则则是引用某个变量的地址
- 需要注意，Go 没有异常机制，但有 panic／recover 模式来处理错误
- Panic 可以在任何地方引发，但 recover 只有在 defer 调用的函数中有效

## defer 向下抛到最后运行
```go
package main

import "fmt"

func main() {
    fmt.Println("a")
    defer fmt.Println("b")
    defer fmt.Println("c")
    defer fmt.Println("d")
}
```

```go
package main

import "fmt"

func main(){
    defer func(){ // 必须要先声明defer，否则不能捕获到panic异常
        fmt.Println("c")
        if err:=recover();err!=nil{
            fmt.Println(err) // 这里的err其实就是panic传入的内容，55
        }
        fmt.Println("d")
    }()

    f()
}

func f(){
    fmt.Println("a")
    panic(55)
    fmt.Println("b")
    fmt.Println("f")
}
```

## golang test测试
```bash
cd /myGoProject/src/go-test/utils
go test -v

```
单独测试某个方法
```bash
go test -v -run='TestTimestamp2StandardTimeStr'
```


## go类型判断
go类型判断一共两种原则，一种是使用空接口的功能,另外一种使用发射来获取变量属性
```go
	var xx interface{} = "hellowrold" // is string
	var xxx interface{} = 'x'  // is rune
	switch xxx.(type) {
	case string:
		fmt.Println("is string")
	case rune:
		fmt.Println("is rune")
	case byte:
		fmt.Println("is byte")
	default:
		fmt.Println("dont know")

	}

	var xx  = 20;
	fmt.Println(reflect.TypeOf(xx)) // int
	var qq = "hzj"
	var qqq = 'h'
	fmt.Println(reflect.TypeOf(qq),reflect.TypeOf(qqq)) // string int32
	fmt.Println(reflect.TypeOf(xxx)) // int32 == rune
```
都是接收一个空的接口也就是变量类型

## 空接口
https://studygolang.com/articles/24407
什么是空接口，接口是两件事务
- 一组方法
- 一种类型

因为golang语言设计的特殊性，只要实现了接口中的所有方法，就代表这个变量实现了这个接口, 在java中则需要继承接口，然后重写接口中的方法。
根据上面的内容所述，golang中存在一个空的接口，也就是说他没有任何的方法，那么所有的变量都实现了这个接口
因此，空接口作为参数的方法可以接受任何类型。Go 将继续转换为接口类型以满足这个函数

### 空接口中存在什么
```go
func main(){
	var i byte = 1
	read(i)
}

func read(i interface{}){
	println(i) // (0x10ab540,0x1167e41)  
}
```

`(0x10ab540,0x1167e41)` 
- 一个指向类型的相关信息的指针
- 一个指向数据的相关信息的指针


## 闭包

```go
package main

import "fmt"

// 闭包的作用就是将函数看做变量来使用
//  如下

// 计算 两个整数变量相加的和
func  cal(a int ,b int) int {
	return a+b
}

// 2. 闭包的使用，将函数当做变量使用 ，作用是保护局部变量不被外部变量所改变
func incr()	func() int{
	var x int = 10
	return func() int {
		x++
		return x
	}
}

func main() {
	// 普通使用
	sum := cal(10,20)
	fmt.Println(sum) //30

	// 把函数当做变量来使用
	sumfunc := cal   //  <- 这里进行变量赋值
	fmt.Println(sumfunc(1,2)) // 3
	fmt.Println(sumfunc(2,3)) // 5

	fmt.Println(sumfunc)  // 0x10994d0
	fmt.Println(cal) // 0x10994d0


	//  使用闭包
	sumx := incr()
	fmt.Println(sumx()) // 11
	fmt.Println(sumx()) // 12
	fmt.Println(sumx()) // 13
	fmt.Println(sumx()) // 14
	fmt.Println(sumx()) // 15
}

```

```go
package main

import (
	"fmt"
)

func main() {
	f1,f2 := calc(10)
	fmt.Println(f1(1),f2(2)) // 11 9
	fmt.Println(f1(1),f2(2)) // 10 8
	fmt.Println(f1(1),f2(2)) // 9 7
	fmt.Println(f1(1),f2(2)) // 8 6
	fmt.Println(f1(1),f2(2)) // 7 5

	var base int = 10
	fmt.Println(calc2(base)) // 11 9
	fmt.Println(calc2(base)) // 11 9
	fmt.Println(calc2(base)) // 11 9
	fmt.Println(calc2(base)) // 11 9
	fmt.Println(calc2(base)) // 11 9
	fmt.Println(calc2(base)) // 11 9



}

func calc(base int)(func(int) int,func(int) int) {
	add := func(i int) int {
		base += i
		return base
	}

	sub := func(i int) int {
		base -= i
		return base
	}
	return add,sub
}

func calc2(base int) (int,int) {
	add := base +1
	sub :=  base -1
	return add,sub
}



```