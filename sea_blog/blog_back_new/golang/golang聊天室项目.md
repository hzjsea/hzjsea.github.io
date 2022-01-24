---
title: golang聊天室项目
categories:
  - golang
tags:
  - golang
abbrlink: 81d00d15
date: 2020-09-17 10:55:11
---


<!-- more -->
# golang聊天室项目

## 单项点对点

:::  tip 
server
:::

```go
package main

import (
	"bufio"
	"fmt"
	"net"
)



func process (conn net.Conn){
	// 关闭连接
	defer conn.Close()
	// 针对当前连接做发送和接受操作
	//input := bufio.NewReader(os.Stdin)
	for {
		reader := bufio.NewReader(conn) // 创建缓冲区对象
		var buf [128]byte // 创建缓冲区大小
		n,err := reader.Read(buf[:]) // 将内容写入到buf缓冲区
		if err != nil{
			fmt.Printf("read from conn faild,err:%v\n",err)
			break
		}
		recv := string(buf[:n]) // 取出接收到的内容
		fmt.Printf( "服务端收到数据:%v\n",recv)
		// 将接收到的数据返回给客户端

		//mess,_ := input.ReadString('\n') // 每次遍历读取一遍回车 将bytes转换成string
		//mess = strings.TrimSpace(mess)
		//if strings.ToUpper(mess) == "Q" {
		//	return
		//}
		_,err = conn.Write([]byte("ok")) // 将ok写入缓存
		if err != nil{
			fmt.Printf("write from conn faild,err %v",err)
			break
		}
	}

}




func main() {
	// 创建监听节点
	listen,err := net.Listen("tcp","127.0.0.1:9090")
	if err != nil{
		fmt.Printf("listen faild,err : %v",err)
		return
	}

	for {
		// 节点接受监听，返回连接节点
		conn,err := listen.Accept()
		if err != nil{
			fmt.Printf( "accept faild,err%v\n",err)
			continue
		}
		// 传入节点
		go process(conn)
	}
}

```


:::  tip 
Client
:::

```go
package main

import (
	"bufio"
	"fmt"
	"net"
	"os"
	"strings"
)

func main() {
	// 主动发起请求与服务端产生连接
	conn,err := net.Dial("tcp","127.0.0.1:9090")
	if err != nil{
		fmt.Printf("conn server faild,err: %v",err)
		return
	}
	input := bufio.NewReader(os.Stdin) // 创建一个缓存 将写入的内容写到缓冲中区
	for {
		s,_ := input.ReadString('\n') // 每次遍历读取一遍回车 将bytes转换成string
		s = strings.TrimSpace(s) // 去除空格
		if strings.ToUpper(s) == "Q"{  // 如果是q 直接跳出
			return
		}
		_,err = conn.Write([]byte(s))  // 将数据写入发送到server端
		if err != nil{
			fmt.Println("error")
			return
		}

		// 从服务端收回消息
		var buf [1024]byte   // 从服务端接收消息
		n,err := conn.Read(buf[:])  // 将消息写入
		if err !=nil{
			fmt.Println("error")
			return
		}

		fmt.Printf("客户端发送信息%v,收到服务器消息%v\n",s,string(buf[:n])) // string 格式化
	}
}

```


## 点对多