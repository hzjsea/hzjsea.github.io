---
title: golang协程
categories:
  - golang
tags:
  - golang
abbrlink: 18fd54e2
date: 2020-09-14 12:07:47
---


<!-- more -->

# golang协程


## 进程线程协程

https://www.cnblogs.com/lxmhhy/p/6041001.html
CPU密集型与IO密集型
https://blog.csdn.net/youanyyou/article/details/78990156
https://juejin.cn/post/6844904056918376456#heading-3

总的结果就是每个程序会产生一个或多个进程，进程的调度是很消耗内存的，如果每一个程序之间的通信是在进程间完成的,那么进程的产生与销毁就能够消耗大量的内存了，于是后面产生了线程，一个进程会有多个线程，同一个进程之间线程是共享变量的，线程的缺点是啥，后面再补充
在后面就是协程 ，协程是一种用户态拟线程，是交给程序来调度的


## golang协程示例
协程是通过程序来调度的，因此也被叫做用户态线程，比如现在有如下一个程序
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	echoMessage()
	end()
}

func end()  {
	fmt.Println("end process")
}

func echoMessage()  {
	// 一共睡眠20s
	for i:=0;i<20;i++{
		fmt.Println("sleep...")
		time.Sleep(time.Second*1)
	}
}

```
如上程序，这个程序会在先执行`echoMessage`这个方法，等全部结束之后，才会执行`end()`这个程序如下
![2021-01-20-14-54-59](http://noback.upyun.com/2021-01-20-14-54-59.png)

现在给`end`加一个询问,代码如下
```go
package main

import (
	"fmt"
	"time"
)

var flag bool = false

func main() {
	echoMessage()
	end()
}

func end()  {

	for !flag {
		fmt.Println("end?")
		time.Sleep(time.Second*2)
	}
	fmt.Println("end process")
}

func echoMessage()  {
	// 一共睡眠20s
	for i:=0;i<20;i++{
		fmt.Println("sleep...")
		time.Sleep(time.Second*1)
	}
	flag = true
}

```
![2021-01-20-15-01-11](http://noback.upyun.com/2021-01-20-15-01-11.png)
依旧是会先去执行`echoMessage()` 等运行结束之后才会去运行end,现在继续优化，希望在执行`echoMessage()`的时候回去执行`end`询问是否结束,只需要把`echoMessage`变成协程调用就可以了，调用的方式是如下 `go echoMessage`,golang会把他放到放把一个线程分为多个协程，然后取一个去调用这个`echoMessage`,如下
```go
package main

import (
	"fmt"
	"time"
)

var flag bool = false

func main() {
	go echoMessage()
	end()
}

func end()  {

	for !flag {
		fmt.Println("end?")
		time.Sleep(time.Second*2)
	}
	fmt.Println("end process")
}

func echoMessage()  {
	// 一共睡眠20s
	for i:=0;i<20;i++{
		fmt.Println("sleep...")
		time.Sleep(time.Second*1)
	}
	flag = true
}

```
![2021-01-20-15-04-44](http://noback.upyun.com/2021-01-20-15-04-44.png)


## golang channel
如上，在golang中我们可以创建一个协程来完成异步的工作,channel可以保证在异步协程的同时在多个协程之间完成通信
```go
// 创建通信
cc := make(chan int) // 无缓冲通道
ccc := make(chan int ,3 ) // 缓冲通道
```

### 无缓冲通道(同步通道)
默认情况下，通信通道是无缓冲的，这种特性导致通道的发送/接收操作，在对方准备好之前是阻塞的，因此无缓冲通道很容易出现以下的错误

> all goroutines are asleep - deadlock!

- 对于同一个通道，发送操作在接收者准备好之前是阻塞的。如果通道中的数据无人接收，就无法再给通道传入其他数据。新的输入无法在通道非空的情况下传入，所以发送操作会等待channel再次变为可用状态，即通道值被接收后。
- 对于同一个通道，接收操作是阻塞的，直到发送者可用。如果通道中没有数据，接收者就阻塞了
如下就会出现死锁
```go
	cc := make(chan int)
	cc <- 20
```
解决方法就是给他一个接受者,注意这里的阻塞和非阻塞，可以理解为这个通道是否有缓冲区，如果没有缓冲区，则需要建立一个协程来异步执行，如果没有异步执行，依旧还会出现死锁，如下
```go
	cc := make(chan int)
	cc <- 20
	fmt.Println(<-cc)
```
添加到协程之后就可以正常使用了,如下
```go
	cc := make(chan int)
	go func() {
		cc <- 20
	}()
	fmt.Println(<-cc
```
另外一种解决方法就是给channel添加缓冲区
```go
	cc := make(chan int,1)
	cc <- 20
	fmt.Println(<-cc)
```

> 再举一个会出现死锁的状况

```go
func main() {

	cc := make(chan int)
	go func() {
		for i:=0;i<20;i++{
			cc <- i
		}
	}()

	for value := range  cc{
		time.Sleep(time.Second*2)
		fmt.Println(value)
	}
```
出现死锁的原因是因为，协程里面输完0-19就结束了，但是外面的for还一直在从通道中拿数据，于是就出现死锁了,解决方法如下
```go
func main() {

	cc := make(chan int,19)
	go func() {
		for i:=0;i<20;i++{
			cc <- i
		}
	}()

	go func() {
		for value := range cc {
			fmt.Println(value)
		}
	}()

	time.Sleep(3 * 1e9)

}
```

### channel中的变量类型
上面有讲到创建不同类别的channel的两种通用方法，但是都需要指定变量类型，假设想要指定一个自定义的变量类型，可以使用struct 重新构造一个,如下

```go
type IntString struct {
	str string
}

func main() {

	cc := make(chan IntString)
	go func() {
		cc <- IntString{"helloworld"}
		cc <- IntString{"helloworld"}
		cc <- IntString{"helloworld"}
		cc <- IntString{"helloworld"}
		cc <- IntString{"helloworld"}
		cc <- IntString{"helloworld"}
	}()
	fmt.Println(<-cc)
}
```

> channel零值为nil


## 单向通道与管道

单向通道与管道很好的证明了 协程中channel可以进行通信

```go
package main

import "fmt"

func main() {
	// 管道 也就是单向通道

	// chan
	in := make(chan int)
	in_level2 := make(chan int)
	// in
	go func() {
		for  {
			in <- 20
		}
	}()

	go func() {
		for {
			// 从上一个channel取出来
			x := <- in
			// 修改后进入另一个channel
			in_level2 <- x*x
		}
	}()

	// 从in_level2中取出
	for x := range in_level2 {
		fmt.Println(x)
	}

}

```

使用close来关闭channel
```go
package main

import "fmt"

func main() {
	// 管道 也就是单向通道

	// chan
	in := make(chan int)
	in_level2 := make(chan int)

	flag := true
	// in
	go func() {
		for i:=0;i<20;i++ {
			if i > 10{
				flag = false
				break
			}
			in <- i
		}
		close(in)
	}()

	go func() {
		for {
			// 从上一个channel取出来
			x := <- in
			// 修改后进入另一个channel
			in_level2 <- x*x
			if !flag{
				break
			}
		}
		close(in_level2)
	}()

	// 从in_level2中取出
	for x := range in_level2 {
		fmt.Println(x)
	}

}
```

> 单向channel改写

```go
package main

import (
	"fmt"
)

func main() {
	// 管道 也就是单向通道

	// chan
	in := make(chan int)
	in_level2 := make(chan int)

	go Channelleve1(in)
	go Channelleve2(in,in_level2)
	echoMessage(in_level2)

}

func Channelleve1(c chan <-int)  {
	for i:=0;i<100;i++{
		c<-i
	}
	close(c)
}

func Channelleve2(in <-chan int,out chan<- int)  {
	for value := range in {
		out <- value
	}
	close(out)
}

func echoMessage(out <-chan int)  {
	for value := range out{
		fmt.Println(value)
	}
}
```

## 缓冲通道
```go
package main

import "fmt"

func main() {
	in := make(chan int,3)
	for i:=0;i<3;i++{
		in<-i
	}
	fmt.Println(cap(in)) //3
	fmt.Println(len(in)) //3
	in <- 3 // fatal error: all goroutines are asleep - deadlock!
}
```
如上创建了一个缓冲区为3的通道,再往里面加就会出现死锁

> channel竞速

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	cc := make(chan string,3 )
	// 竞速
	go EchoMessage(20,cc,"sleep 20s")
	go EchoMessage(5,cc,"sleep 5s")
	go EchoMessage(5,cc,"sleep 5s,so fast")
	fmt.Println(<-cc)
}

func EchoMessage( duration time.Duration ,cc chan<- string,message string)  {
	time.Sleep(duration*time.Second)
	cc <- message
}
```
官方文档中有些，如果使用的是一个无缓冲的通道的话，两个较慢的goroutime将被卡主，因为他们发送响应结果到通道的时候没有goroutine来接收，这个情况叫做goroutine泄露，属于一个bug

## 多路复用
多路复用的本质，是对channel的动作以及channel通信的内容做识别，并执行对应的动作，比如
```go
package main

import (
	"fmt"
	"os"
	"time"
)

func main() {
	fmt.Println("cc")
	tick := time.Tick(1*time.Second)
	go func(){
	for countdown := 10; countdown >0 ;countdown-- {
		fmt.Println(countdown)
		<-tick
	}
	}()

	abort := make(chan struct{})
	go func() {
		os.Stdin.Read(make([]byte,1))
		abort <- struct{}{}
		// <- abort 这个注释取消掉就不会中断
	}()

	select {
	case <-time.After(10 * time.Second):
	case <-abort:
		fmt.Println("xx")
		return
	}

}
```
上面的程序中，执行了两个协程，并且都往channel中写入了数据，使用select进行处理,如果是等待10s 则不执行任何操作那个，如果是abort这个channel中有内容输入 则case 为true，则输出"xx" 并返回终止，上面的注释部分取消掉就不会终止，这是因为他内容输入后就被输出了 则case为false

> 缓冲select

```go

package main

import "fmt"

func main() {

	ch := make(chan int,1)
	for i:=0;i<10;i++{
		select {
		case x:= <- ch:
			fmt.Println(x)
		case ch <-i:
		}
	}
}
```
如上，执行顺序为
i等于0的时候，x=0 输出0
i等于1的时候 ch接收1 ，此时len=1
i等于2的时候， x=1 ，输出1

## 多线程抢占执行
在多线程环境下，多个线程并发抢占会使得打印不是按照顺序来，那么我们如何确保子线程全部结束完之后主线程再停止呢？主要有两种方式
### 一使用阻塞channel
```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	// 输出当前系统的CPU数量
	fmt.Println(runtime.NumCPU())
	// 设置当前程序执行使用的CPU数量
	runtime.GOMAXPROCS(runtime.NumCPU()/2)

	// 创建一个channel
	c1 := make(chan  bool)

	for i:=0;i<10;i++{
		go goRun(c1,i)
	}
	for i := 0; i < 10; i++ {
		<- c1
	}
}


func goRun(c chan bool,index int ) {
	sum := 0
	for i:=0;i<100000000;i++{
		sum+=i
	}
	fmt.Println("线程序号:", index, sum)
	c <- true
}

```
输出结果
```
8
线程序号: 1 4999999950000000
线程序号: 4 4999999950000000
线程序号: 0 4999999950000000
线程序号: 9 4999999950000000
线程序号: 6 4999999950000000
线程序号: 2 4999999950000000
线程序号: 5 4999999950000000
线程序号: 7 4999999950000000
线程序号: 3 4999999950000000
线程序号: 8 4999999950000000
```

从打印结果可以看出，多线程环境下运行代码打印和顺序没有关系，由 CPU 调度自己决定，多运行几次打印结果一定不会一样，就是这个道理。

### 二使用同步机制
```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	fmt.Println("当前系统核数：", runtime.NumCPU())
	runtime.GOMAXPROCS(runtime.NumCPU()) //设置当前程序执行使用的并发数
	/**
	  waitGroup即任务组，它的最要作用就是用来添加需要工作的任务，没完成一次任务就标记一次Done，这样任务组的待完成量会随之减1
	  那么主线程就是来判断任务组内是否还有未完成任务，当没有未完成当任务之后主线程就可以结束运行，从而实现了与阻塞队列类似的同步功能
	  这里创建了一个空的waitGroup(任务组)
	*/
	wg := sync.WaitGroup{}
	wg.Add(10)  //添加10个任务到任务组中
	//这里启动10个线程运行
	for i :=0; i < 10; i++ {
		go goRun(&wg, i)
	}
	wg.Wait()
}

/**
这里需要传入引用类型不能传入值拷贝，因为在子线程中是需要执行Done操作，类似与我们修改结构体中的int变量主词递减，如果是只拷贝的话是不会影响原类型内的数据
这样就会发生死循环导致死锁程序奔溃，报错异常为：fatal error: all goroutines are asleep - deadlock!
*/
func goRun(wg *sync.WaitGroup, index int) {
	a := 1
	//循环叠加1千万次并返回最终结果
	for i := 0; i < 10000000; i++ {
		a += i
	}
	fmt.Println("线程序号:", index, a)

	wg.Done()
}
```

## 使用Select处理多个channel
```go
package main

import "fmt"

func main() {
	// 创建多个channel
	c1,c2 := make(chan int),make(chan  string)
	go func() {
		/**
		  创建一个无限循环语句,使用select进行处理
		  我们一般都是使用这种方式来处理不断的消息发送和处理
		*/
		for {
			select {
			case v, ok := <- c1:
				if !ok {
					break
				}
				fmt.Println("c1:", v)
			case v, ok := <- c2:
				if !ok {
					break
				}
				fmt.Println("c2:", v)
			}
		}
	}()

	c1 <- 1
	c2 <- "liang"
	c1 <- 2
	c2 <- "xuli"

	//关闭channel
	close(c1)
	close(c2)
}
```




