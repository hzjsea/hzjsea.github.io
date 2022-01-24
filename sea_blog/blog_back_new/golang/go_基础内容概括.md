---
title: golang基础
categories:
  - golang
tags:
  - golang
abbrlink: 7e99cc4e
date: 2020-09-14 17:31:54
---


<!-- more -->


# golang(一)


## 安装环境

### mac环境安装
```bash
# 安装homebrew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# 使用brew安装golang
brew install go
# 查看go版本信息
brew info go
go env
# 配置go开发环境，宣告环境 写入.bash_profile里面
export GOPATH=/Applications/JD/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
```

> 配置Go路径环境
- GOROOT： go安装目录
- GOPATH：go工作目录
- GOBIN：go可执行文件目录
- PATH：将go可执行文件加入PATH中，使GO命令与我们编写的GO应用可以全局调用

编辑我们的.bash_profile文件
```bash
GOROOT=/usr/local/Cellar/go/1.10.1/libexec
export GOROOT
export GOPATH=/Users/jiangqiaowei/mygo
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN:$GOROOT/bin
```
```bash
source ~/.bash_profile
```

### 第一个helloworld
```go
package main

import "fmt"

func main(){
    fmt.Println("hello world")
}
```

## go中的特殊性
### go底层默认执行函数 func init(){}
golang里的main函数是程序的入口函数，main函数返回后，程序也就结束了。golang还有另外一个特殊的函数init函数，先于main函数执行，实现包级别的一些初始化操作。

- init函数先于main函数自动执行，不能被其他函数调用；
- init函数没有输入参数和返回值
- 每个包可以有多个init函数
- 同一个包的init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序
- 不同包的init函数按照包导入的依赖关系决定执行顺序

```go
package main
import "fmt"
func main() {
   println("world")
}

func init() {
   fmt.Println("hello")
}

// hello
// world
```

>默认执行函数以及包之间的执行顺序
先会去引用包中运行pkg中的程序 ，再运行init()函数，最后运行main函数，结束时候exit退出

![2020-09-11-14-31-17](http://noback.upyun.com/2020-09-11-14-31-17.png)

### 开启本地文档
golang可以在本地开启文档中心，内容如下
```go
godoc -http=localhost:6060
// 访问127.0.0.1:6060
```


## 字符串
Go中的字符串都是采用UTF-8字符集编码。字符串是用一对双引号("")或者一对反引号(` `)定义
在Go中，字符串是不可变的 immutable ,字符串本身就是一种只读的静态数组，是一段连续的内存空间。
![](https://noback.upyun.com/2021-06-01-08-53-32.png!)
```go
var s string = "hello"
s[0] = "c" // 这样就会报错
```
如果需要更换的话，需要将string转换成byte切片后在进行更换
```go
func main() {
	s:="helloworld"
	bytess := []byte(s)
	//[104 101 108 108 111 119 111 114 108 100]
	fmt.Println(bytess)
	bytess[0] = 'c'
	s = string(bytess)
	fmt.Println(s)
	//celloworld
}
```

扩展阅读
https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-string/



## golang变量

golang中的基本变量如下，其中 byte表示单字节
```go
bool  //布尔类型
string // 字符串类型
int  int8  int16  int32  int64   // 整型
uint uint8 uint16 uint32 uint64 uintptr
byte // uint8 的别名
rune // int32 的别名 // 表示一个 Unicode 码点
float32 float64  // 浮点类型
complex64 complex128
```

### 短变量赋值与申明变量赋值
变量的初始化有两种位置，一种是在方法外面的全局变量，另一种则是在方法内的局部变量
在声明局部变量的时候，我们可以使用`短变量赋值`的形式进行变量声明，
`短变量赋值`的好处在于 go编译器会根据`:=`后面的数据类型自动判别数据类别 在函数外面也可以使用var达到这样的效果。
但是如果没有进行变量赋值的话，那必须声明变量类型

```go
package main
import "fmt"
//全局批量变定义
var c,python,java bool
// 不限定变量类型，编译的时候会自动判断
// 批量定义
var flag,words = true,"helloworld"
// 常量声明
const Pi = 3.14

func main() {
	// 短变量声明
	k:=3 		 // var k = 3 => var k int = 3
	j:="string"  // var j = "string" = > var j string = "string"
	fmt.Println(j,k)
}
```

<font color='red'>简洁赋值语句 := 可在类型明确的地方代替 var 声明。
但是函数外的每个语句都必须以关键字开始（var, func 等等），因此 := 结构不能在函数外使用。</font>



### golang String字符串

计算机如何表示字符
1. 计算机是二进制的，字符最终也是转换成二进制保存起来的 字符集就是定义字符对应的数值。
2. Unicode是一个字符集，为每个字符规定一个用来表示该字符的数字，但是并没有规定该数字的二进制保存方式，utf8规定了对于unicode值的二进制保存方式。
3. utf8是可变长度字符编码，不同的字符会对应不同大小的存储方式，比如"a"字符(unicode值97)用1个字节，而"中"字符(unicode值20013）则用3个字节。字符的unicode值决定了字符需要用多少字节表示


golang中字符类型有如下两种
- 一种是 uint8 类型，或者叫 byte 型，代表了 ASCII 码的一个字符。
- 另一种是 rune 类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 rune 类型。rune 类型等价于 int32 类型

需要知晓的是rune类型的底层类型是int32类型，而byte类型的底层类型是int8类型，这决定了rune能比byte表达更多的数
golang中string底层是通过byte数组实现的。中文字符在unicode下占2个字节，在utf-8编码下占3个字节，而golang默认编码正好是utf-8。

```go
package main

import "fmt"

func main() {

	var s= "中"
	// 中文字符在unicode下占2个字节，在utf-8编码下占3个字节，而golang默认编码正好是utf-8
	fmt.Println(len(s)) // 3
	var ss = "z"
	fmt.Println(len(ss))  // 1

	// 将byte数组转换成rune数组
	var sss = "中国"
	fmt.Println(len(sss)) // 6
	fmt.Println(len([]rune(sss))) // 2
	fmt.Println([]rune(sss))  // [20013 22269]
}
```

Go语言中，string就是只读的采用utf8编码的字节切片(slice) 因此用len函数获取到的长度并不是字符个数，而是字节个数。 for循环遍历输出的也是各个字节

```go
package main

import (
   "fmt"
   "reflect"
)

func main() {
   ss:="China"
   for _,value := range ss{
      fmt.Println(value,reflect.TypeOf(value))
   }
   //67 int32
   //104 int32
   //105 int32
   //110 int32
   //97 int32
   for i:=0;i<len(ss);i++{
      fmt.Println(ss[i],reflect.TypeOf(ss[i]))
   }
   //67 uint8
   //104 uint8
   //105 uint8
   //110 uint8
   //97 uint8
   ss="中国"
   for _,value := range ss{
      fmt.Println(value,reflect.TypeOf(value))
   }
   //20013 int32
   //22269 int32
   for i:=0;i<len(ss);i++{
      fmt.Println(ss[i],reflect.TypeOf(ss[i]))
   }
   //228 uint8
   //184 uint8
   //173 uint8
   //229 uint8
   //155 uint8
   //189 uint8
}
```

Golang rune类型
rune是int32的别名，代表字符的Unicode编码，采用4个字节存储，将string转成rune就意味着任何一个字符都用4个字节来存储其unicode值，这样每次遍历的时候返回的就是unicode值，而不再是字节了，这样就可以解决乱码问题了


## golang range函数
range主要用来遍历数组、map、slice 等等集合

对于数组和切片这些顺序表，range返回的是序号和元素，而对于数组来说 range返回的是key和value 


```go
package main

import "fmt"

func main() {
	var list = []int{1,2,3,4,56}
	for index,value := range list{
		fmt.Println(index,value)
	}
	var sliceList = list[:2]
	for index,value := range sliceList{
		fmt.Println(index,value)
	}
	var mapd = map[string]int{"age":12,"name":13}
	for key,value := range mapd{
		fmt.Println(key,value)
	}
}
```

## golang函数
函数的形式如下
```go
package main

import "fmt"

//定义了一个函数，并且规定了传入的参数类型
// 因为golang是静态编译语言，所以在编译的时候
// 会对参数类型做检验，如果不符合数据类型，则会编译不通过
// 另外定义一个返回的类型为int的res
// 在python中也有类似的内容，它叫做类型注释
func add(x int, y int) int {
	return x + y
}

// 当连续两个或多个函数的已命名形参类型相同时，除最后一个返回结果的类型以外，其它都可以省略
func cut(x, y int) int {
	return x - y
}

func main() {
	fmt.Println(add(20, 13))
	fmt.Println(cut(20, 3))
}

```

### 函数多值返回

```go
package main

import "fmt"

// 返回多个结果
func multip(x, y int) (int, int, int) {
	return x * y, x, y
}

func main() {
	res, x, y := multip(6, 7)
	fmt.Print(x, y, res)
}

```

### 函数命名返回
也就是返回在函数定义的时候所声明的返回的内容，如下返回一个(x,y)的元组
```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}

//res:
// 7 10
```

### 返回默认值
go中的函数支持返回默认值
```golang
func main(){
	one,two,three := xx()
	fmt.Println(one,two,three) // false   0
}


func xx()(one bool,two string,three int) {
	return
}
```

## golang逻辑控制

### for普通遍历
```go
func main() {
	sum :=20
	for i := 0; i < sum; i++ {
		fmt.Println(i)
	}
	// while true
	for {
		fmt.Println(i)
	}
}
```

### if语句
if语句看起来和其他语言差不多，就是在判断语句前后不再需要括号了
```go
package main

import "fmt"
import "math"

func main() {
	fmt.Println(sqrt(2),sqrt(-4))
}



func sqrt(x float64) string{
	if x < 0 {
		return sqrt(-x) + "i"
	}
	return fmt.Sprint(math.Sqrt(x))
}
```

### if支持短赋值语句
```go
func main() {
	if a :=20; a<30{
		a+=1
	}else{
		a-=1
	}
	fmt.Println(a) // 报错

	var a int
	if a = 20;a<30{
		a +=1
	}
	fmt.Println(a)
}
```

### switch语句
switch语句有点类似于模式匹配
```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}

```

### defer语句
defer语句会将当前内容 推迟 到外层函数返回后执行
推迟调用的函数其参数会立即求值，但直到外层函数返回前函数都不会被调用
```go
package main

import "fmt"

func main() {
	defer  fmt.Println("world")
	fmt.Println("helloworld")
}
```

### defer语句2
```go
package main

import "fmt"

func main() {
	name:="hzj"
	defer say_hellos(name)
	fmt.Println("i will run first")
}

func say_hellos(name string){
	fmt.Println("hello world,"+name)
}
```
如上面的内容，在main函数中执行了一个say_hello函数，以及输出一句话
另外在say_hello中也会输出一句话
在不加defer的时候 会先执行say_hello 
加上defer之后 会先调用参数，但不执行，之后执行输出语句，离开main函数之后，再输出say_hellos

### defer语句3
> defer栈 推迟的函数调用会被压入一个栈中，当外层函数返回的时候，被推迟的函数会按照后进后出的顺序被调动


```go
package main

import "fmt"

func main() {
	for i:=0 ; i<10; i++ {
		defer return_res(i)
	}
	fmt.Println("start")
}
func return_res(i int ){
	fmt.Println(i)
}

// result:
// start
// 9
// 8
// ...
// 2
// 1
// 0
```

## make 和new的区别
- make的作用是初始化数据结构，比如slice,hashmap,channel等等
- new的作用是根据传入的类型分配一片内存空间并返回指向这片内存空间的指针

示例代码如下
```golang
slice := make([]int, 0, 100)
hash := make(map[int]bool, 10)
ch := make(chan int, 5)
```


## golang中int和string互相转换
```golang
// string转成int：
int, err := strconv.Atoi(string)
// string转成int64：
int64, err := strconv.ParseInt(string, 10, 64)
// int转成string：
string := strconv.Itoa(int)
// int64转成string：
string := strconv.FormatInt(int64,10)
```