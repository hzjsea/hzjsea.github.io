---
title: go_接口与空接口
date: 2021-05-18 18:09:15
categories:  [golang]
tags: [golang]
---


<!--more-->


# go_接口与空接口


golang中接口是一个特别的东西，


## duck typing



## 空接口


<div style='font-size:20px;color:red'>在将空接口之前还要来看一个golang中提供的 检验传入的值的类型</div>

```golang
func main(){
    if v,ok := i.(*http.Server);ok == true{
		fmt.Println(v," interface's type is *http.Server")
	} else {
		fmt.Println("error")
	}
}
```
其中v是interface的值内容， ok是bool类型，如果符合类型，则返回true 


根据接口的定义，只要实现了接口中的所有方法，就实现了该接口类型。那么好，现在有一个空接口，里面是没有任何方法的，所以任何类型都实现空接口类型

现在有这么一个空接口
```golang
func doSomething(v interface{}){  
    ...1
}
```
<div style='font-size:20px;color:red'>
这里的形参v要求接收一个实现空接口类型的,方便在调用的时候，传值后可以被类型转换成空接口， 那么是不是可以理解为，函数被调用时，在函数内部v代表任何类型。并不是这样，虽然函数的参数可以接受任何类型，并不表示 v 就是任何类型。
在函数 doSomething 内部 v 仅仅是一个 interface 类型，之所以函数可以接受任何类型是在 go 执行时传递到函数的任何类型都被自动转换成 interface{}。注意这里是做了类型转换的
</div>


<div style='font-size:20px;color:red'>空接口有类型吗?</div>

其实空接口是有类型的，他类似于一个结构体，结构体一段执行数据源，一段指向接收时的数据的类型，这边文章讲述了 interface中如何存储值和值的类型的 [Russ Cox 关于 interface 的实现](https://research.swtch.com/interfaces)

下面可以来看一段案例

```golang
package main

import (
	"fmt"
	"net/http"
)

func log(i interface{}) {
	// if v,ok := i.(*http.Server);ok == true{
		// fmt.Println(v," interface's type is *http.Server")
	// } else {
		// fmt.Println("error")
	// }

	fmt.Printf("%v, %v\n", i, i == nil)
}

func run(s *http.Server) {

	// 形参的类型转换
	// (*http.Server)(nil)
	fmt.Printf("%v\n",s) // 首先是nil进来 他的值就是nil 所以输出的是nil 但是类型是*http.Server
	log(s) // struct {  vlaue:nil  type:*http.Server} 进入log 之后被转成成interface接口类型，但是type 依旧是 *http.Server
	fmt.Println("---------")
	log(nil) // struct {value:nil  type:nil}
	fmt.Println("---------")
	log((*http.Server)(nil)) // 强制类型转换了，传入的时候 // struct {value:nil type:*http.Server}
	fmt.Println("---------")
}

func main() {
	run(nil)
}
```

可以用测试跑一下，结果是这样的。

```bash
<nil>
<nil>, false
---------
<nil>, true
---------
<nil>, false
---------
```


> []interface{} 可以正常接收 类似于 []string 的内容嘛

比如说你这样干
```golang
func printAll(vals []interface{}) { //1
	for _, val := range vals {
		fmt.Println(val)
	}
}

func main(){
    name := [3]string{"stanley", "david", "oscar"}
    printAll(name) // error Cannot use 'names' (type [3]string) as type []interface{}
	names := []string{"stanley", "david", "oscar"}
	printAll(names) // error  Cannot use 'names' (type []string) as type []interface{}
}
```



首先`[]string`是一个数组的切片slice，本身他的长度就是动态的，`[]interface{}`的长度也是动态的
前者的长度是是`n*2`,后者的长度是`N*sizeof(T)` 两种 slice 实际存储值的大小是有区别的



https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/
https://github.com/golang/go/wiki/InterfaceSlice
https://github.com/kamipo/mysql-build
https://blog.csdn.net/whatday/article/details/107918399
https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array-and-slice/