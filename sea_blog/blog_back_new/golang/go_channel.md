---
title: golangRuntime
date: 2021-01-11 14:48:32
categories:  [golang]
tags: [golang]
---


<!--more-->


# golangRuntime

## Channel
通道分为无缓冲通道和缓冲通道，创建方式如下


```go
	ch := make(chan int)
	var x int = 1
	ch <- x
	// 赋值
	//var y int =  <- ch
	//fmt.Println(y)
	<- ch // 丢弃
	// 关闭通道
	close(ch)
	fmt.Println(ch)


	ch = make(chan  int)
	// 无缓冲通道
	ch = make(chan  int ,0)
	// 容量为3的缓冲通道
	ch = make(chan  int, 3)
```





## 程序

> 回声服务器

Server
```go
package main

import (
	"io"
	"log"
	"net"
	"time"
)

func main() {
	get_conn()
}


// 启动一个监听端口
// 监听本机的端口，收到信息后则处理当前信息
// 处理内容为输出当前时间

func get_conn(){
	host:= net.JoinHostPort("","9090")
	listener,err := net.Listen("tcp",host)
	if err != nil{
		log.Fatal(err)
	}
	// 监听到tcp连接之后就进行长连接
	for {
		conn,err := listener.Accept()
		if err != nil{
			log.Fatal(err)
			continue
		}
		go HandleConn(conn)
	}
}


func HandleConn(conn net.Conn){
	// 结束之后 抛掉连接
	defer conn.Close()
	for {
		_,err := io.WriteString(conn,time.Now().Format("15:04:05\n"))
		if err != nil{
			return
		}
		time.Sleep(1*time.Second)
	}
}


```

Client
```go
package main

import (
	"io"
	"log"
	"net"
	"os"
)

func main() {

	conn, err := net.Dial("tcp","localhost:8888")
	if err != nil{
		log.Fatal(err)
	}
	defer conn.Close()
	go mustCopy(os.Stdout,conn) // 输出到终端 这一只程序走另一个子进程
	mustCopy(conn,os.Stdin)
}

func mustCopy(dst io.Writer, src io.Reader){
	if _,err := io.Copy(dst,src);err != nil{
		log.Fatal(err)
	}
}


```