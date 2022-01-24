---
title: go_数据类型
date: 2021-06-15 17:31:12
categories:  [golang]
tags: [golang]
---


<!--more-->


# go_数据类型


## 数据类型转换
数据类型转换，强转 一个是自带的strconv包 一个是unknow/com 包, 个人感觉第二个包并不好用

```golang
package main

import (
	"fmt"
	"github.com/unknwon/com"
        "strconv"
)

func main() {
	test1()
}

func StringToInt(ctx *gin.Context) {
   a := "123"
   b := com.StrTo(a).MustInt() //string 转 int
   b2 := com.StrTo(a).MustInt64() //string 转 int
   fmt.Printf("%T %s\n", a, a) //string 123
   fmt.Printf("%T %d\n", b,b) //int 123
   fmt.Printf("%T %d\n", b2,b2) //int64 123

   var c string = "123"
   c2,_:= strconv.ParseInt(c,10,64)  //string 转 int
   fmt.Printf("%T %s\n", c, c) //string 123
   fmt.Printf("%T %d\n", c2,c2) //int64 123

   d := "33"
   d1 := com.StrTo(d).MustFloat64() //string 转 float64
   fmt.Printf("%T %s\n", d,d) //string 33
   fmt.Printf("%T %.2f\n", d1,d1) //float64 33.00

   e,_:= com.StrTo(a).Float64() //string 转 float64
   fmt.Printf("%T %.2f\n", e,e) //float64 123.00

   g:=strconv.Itoa(int(50)) //int转string
   fmt.Printf("%T %.2f\n", g,g) //string 50

   f:=strconv.FormatInt(int64(50), 10) //int64转string
   fmt.Printf("%T %.2f\n", f,f) //string 50



   //res := testSwitch(a)
   //httpext.SuccessExt(c, res)
}
```

## 数据类型判断
go中数据类型判断有两种方法， 一种是用空接口，另外一种是使用反射，推荐和常用的都是前者
```go
func m_type(i interface{}) {
    switch i.(type) {
    case string:
        //...
    case int:
        //...
    }
    return
}
```

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x int32 = 20
    fmt.Println("type:", reflect.TypeOf(x))
}
```

<div style='font-size:20px;color:red'>
区别:  第一种方法需要预先知道有哪几种类型
        第二种方法可以对任意不清楚类型的变量使用
</div>

这里可以举一个例子, 对错误的处理，errors包里面定义了错误的结构体如下
```go
type Error struct {
	Code       int    `json:"code"`  // 错误code
	Message    string `json:"msg"`   // 错误信息
	RequestID  string `json:"id"`    // 请求id
	Operation  string                // 行为
	StatusCode int                   
	Header     http.Header                
	Body       []byte
}
```
于是我们对错误的处理可以这样做
错误处理
```go
func main(){
	if config.LocalPath != "" {
		var fd *os.File
		if fd, err = os.Open(config.LocalPath); err != nil {
			return errorOperation("open file", err) // 读取文件错误
		}
		defer fd.Close()
		config.Reader = fd
        }
}
```
```go
func errorOperation(op string, err error) error {
	if err == nil {
		return errors.New(op)
        }
        // 判断是否是错误类型，
        // 如果是的话,填入具体引导错误的操作
	ae, ok := err.(*Error)
	if ok {
		ae.Operation = op
		return ae
	}
	return fmt.Errorf("%s: %w", op, err)
}
```