---
title: go_并发编程
date: 2021-06-04 11:49:34
categories:  [golang]
tags: [golang]
---


<!--more-->


# go_并发编程
看go源码的时候回经常看到一个`context.Background`,这个是用来在多个goruntine之间通信的
go 1.17版本之后,context被纳入到官方库里面, context 定义了如下4种状态，并且context 本身也是作为一种接口类型

```golang
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    // 返回的第一个值是 截止时间，到了这个时间点，Context 会自动触发 Cancel 动作。
    // 返回的第二个值是 一个布尔值，true 表示设置了截止时间，false 表示没有设置截止时间，如果没有设置截止时间，就要手动调用 cancel 函数取消 Context

    Done() <-chan struct{}
    // 返回一个只读的通道（只有在被cancel后才会返回），类型为 struct{}。当这个通道可读时，意味着parent context已经发起了取消请求，根据这个信号，开发者就可以做一些清理动作，退出goroutine。

    Err() error
    // 返回 context 被 cancel 的原因

    Value(key interface{}) interface{}
    // 返回被绑定到 Context 的值，是一个键值对，所以要通过一个Key才可以获取对应的值，这个值一般是线程安全的
}
```


常见的关闭goroutine的方法有三种
- goroutine 自己跑完结束退出
- 主进程crash退出，goroutine 被迫退出
- 通过通道发送信号，引导协程的关闭

> 第一种正常关闭的

```golang
func main(){
    go func() {
		for i:=0;i<10;i++{
			fmt.Println(i)
		}
	}()
	time.Sleep(10*time.Second)
}
```

> 第三种是通过select+信号标记的方式来控制runtine的

```golang
func main(){
	stop := make(chan  bool) // 停止的信号
	message := make(chan string) // 消息通道

	go func() {
		for {
			select {
			case <-stop:
				fmt.Printf("监控退出")
				return
			case value:=<-message :
				fmt.Println(value)
			default:
				fmt.Println("default")
				time.Sleep(2*time.Second)
			}
		}
	}()
	for i:=0;i<10;i++{
		message<-"helloworld"
	}
	time.Sleep(10*time.Second)
	fmt.Println("开始")
	stop <-true
	time.Sleep(5*time.Second)
	fmt.Println("停止")
}

```

缺点: 这样的场景只能针对于单个goruntine来使用，如果在多个runtine的场景下就要使用多个信号标志来处理这个runtine。并且runtine还支持多个一层一层的嵌套，这样使用信号标记控制runtine就会很复杂
```golang
func main(){
    	stop := make(chan  bool) // 停止的信号
	stop_nums :=0
	go func() {
		for {
			select {
			case <-stop:
				stop_nums += 1
				if stop_nums >= 2 {
					fmt.Println(stop_nums)
					fmt.Printf("监控退出")
					return
				} else {
					fmt.Println(stop_nums)
					go func(){
						fmt.Println("break")
						return
					}()
				}
			default:
				fmt.Println("default")
				time.Sleep(1*time.Second)
			}
		}
	}()
	time.Sleep(3*time.Second)
	fmt.Println("开始")
	stop <-true
	time.Sleep(1*time.Second)
	stop <-true
	time.Sleep(100*time.Second)
}
```
```golang
package main

import (
    "fmt"
    "time"
)

func monitor(ch chan bool, number int)  {
    for {
        select {
        case v := <-ch:
            // 仅当 ch 通道被 close，或者有数据发过来(无论是true还是false)才会走到这个分支
            fmt.Printf("监控器%v，接收到通道值为：%v，监控结束。\n", number,v)
            return
        default:
            fmt.Printf("监控器%v，正在监控中...\n", number)
            time.Sleep(2 * time.Second)
        }
    }
}

func main() {
    stopSingal := make(chan bool)

    for i :=1 ; i <= 5; i++ {
        go monitor(stopSingal, i)
    }

    time.Sleep( 1 * time.Second)
    // 关闭所有 goroutine
    close(stopSingal)

    // 等待5s，若此时屏幕没有输出 <正在监控中> 就说明所有的goroutine都已经关闭
    time.Sleep( 5 * time.Second)

    fmt.Println("主程序退出！！")

}
```

于是后面出现了context这个库，主要是用来提供给每个routine一个消息传递的通道,使用context之前先要了解几个和他有关的函数,根据context的四种状态，定义了四种方法。
- func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
- func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
- func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
- func WithValue(parent Context, key, val interface{}) Context

这四个函数都必须要接收一个父的context,
通过一次继承，就多实现了一个功能，比如使用 WithCancel 函数传入 根context ，就创建出了一个子 context，该子context 相比 父context，就多了一个 cancel context 的功能。
如果此时，我们再以上面的子context（context01）做为父context，并将它做为第一个参数传入WithDeadline函数，获得的子子context（context02），相比子context（context01）而言，又多出了一个超过 deadline 时间后，自动 cancel context 的功能

类似于一种建造者的模式，每一次的继承都会返回一个新的context 同时带上一层新的属性

## 根context
什么是根context，看context这个源码的时候会发现，官方已经定义好了两个context
```golang
type emptyCtx int // int类型，可能是处于内存考虑把

var (
	background = new(emptyCtx) // 大部分都是用这个，基本上都用这个，跑后台的时候用这个context 
	todo       = new(emptyCtx) // 当你不知道用什么的时候，就用这个todo
)

func Background() Context {
	return background
}

func TODO() Context {
	return todo
}
```

接下来是实现
```golang
func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return // 返回一个deadline 一个ok  分别是各自的默认值
}

func (*emptyCtx) Done() <-chan struct{} {
    return nil // 返回一个空的struct
}

func (*emptyCtx) Err() error {
	return nil // 返回Nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
	return nil // 返回Nil
}
```
这样一来 emptyCtx 也就是context.Background 以及 context.TODO 就实现了Context这个接口

常用结构图
![](https://noback.upyun.com/2021-06-07-07-54-27.png!)

## 使用context

> WithDeadline

```golang
package main

import (
    "context"
    "fmt"
    "time"
)


func monitor(ctx context.Context, number int)  {
    for {
        select {
        case <- ctx.Done():
            fmt.Printf("监控器%v，监控结束。\n", number)
            return
        default:
            fmt.Printf("监控器%v，正在监控中...\n", number)
            time.Sleep(2 * time.Second)
        }
    }
}

func main() {
    // 首先是使用四个函数创建一个context 
    // 这里对context 继承了两次，也就是说ctx02 有两个属性 一个是done  一个是deadline
    ctx01, cancel := context.WithCancel(context.Background())
    // 当前时间的1s钟之后会被关掉
    ctx02, cancel := context.WithDeadline(ctx01, time.Now().Add(1 * time.Second))

    defer cancel()

    for i :=1 ; i <= 5; i++ {
        go monitor(ctx02, i)
    }


    time.Sleep(5  * time.Second)
    if ctx02.Err() != nil {
        fmt.Println("监控器取消的原因: ", ctx02.Err())
    }

    fmt.Println("主程序退出！！")
}
```

## WithValue
通过Context我们也可以传递一些必须的元数据，这些数据会附加在Context上以供使用。
元数据以 `Key-Value` 的方式传入，Key 必须有可比性，Value 必须是线程安全的。
还是用上面的例子，以 ctx02 为父 context，再创建一个能携带 value 的ctx03，由于他的父context 是 ctx02，所以 ctx03 也具备超时自动取消的功能。

```golang
func monitor(ctx context.Context, number int)  {
	for {
		select {
		case <- ctx.Done():
			fmt.Printf("监控器%v，监控结束。\n", number)
			return
		default:
            // 获取传递过来的参数  
            value := ctx.Value("item")
			fmt.Printf("监控器%v，正在监控 %v \n", number, value)
			time.Sleep(2 * time.Second)
		}
	}
}

func main() {
	// 首先是使用四个函数创建一个context
	// 这里对context 继承了两次，也就是说ctx02 有两个属性 一个是done  一个是deadline
	ctx01, cancel := context.WithCancel(context.Background())
	ctx02, cancel := context.WithDeadline(ctx01, time.Now().Add(1 * time.Second))
	ctx03 := context.WithValue(ctx02,"item","CPU")
	ctx04 := context.WithValue(ctx03,"item","CPU2")

	defer cancel()

	for i :=1 ; i <= 5; i++ {
		go monitor(ctx04, i)
	}

	time.Sleep(1  * time.Second)
	if ctx02.Err() != nil {
		fmt.Println("监控器取消的原因: ", ctx02.Err())
	}

	fmt.Println("主程序退出！！")
}
```

<div style='font-size:20px;color:red'>注意事项</div>

- 不要把原本可以由函数参数来传递的变量，交给 Context 的 Value 来传递。
- 当一个函数需要接收一个 Context 时，但是此时你还不知道要传递什么 Context 时，可以先用 context.TODO 来代替，而不要选择传递一个 nil。
- 当一个context被cancel掉之后， 继承自该context的所有子goruntine都会被释放掉


## context适用场景
想象一个并发的场景，为了解决的更多的请求,go在处理request的时候开启了并发的模式，也就是说一条goruntine的请求下面嵌套了多层的行为，我们称它为一组请求，当在请求的过程中如果客户端关闭了浏览器也就是手动的关闭了请求，这样后续的操作其实是不用再进行了，因为即使你返回也不会有人接收，于是有了context，当检测到浏览器关闭后，context 就会执行cancel， 关闭整组请求。
![](https://noback.upyun.com/2021-06-07-08-07-16.png!)

## 扩展连接
https://zhuanlan.zhihu.com/p/136664236
https://zhuanlan.zhihu.com/p/145882778
https://i6448038.github.io/archives/