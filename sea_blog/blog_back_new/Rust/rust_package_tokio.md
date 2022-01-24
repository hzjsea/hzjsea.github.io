---
title: rust_package_tokio
date: 2021-06-21 11:31:19
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust_package_tokio

> A runtime for writing reliable, asynchronous, and slim applications.
用于编写可靠、异步和超薄应用程序的运行时
Tokio 是一个事件驱动的非阻塞 I/O 平台，用于使用 Rust 编程语言编写异步应用程序。在高层次上，它提供了几个主要组件：

Tokio库主要提供了一下功能模块

> Tools for working with asynchronous tasks, including synchronization primitives and channels and timeouts, delays, and intervals.
APIs for performing asynchronous I/O, including TCP and UDP sockets, filesystem operations, and process and signal management.
A runtime for executing asynchronous code, including a task scheduler, an I/O driver backed by the operating system's event queue (epoll, kqueue, IOCP, etc...), and a high performance timer.

- 用于处理异步任务的工具，包括同步原语和通道以及超时、延迟和间隔。
- 用于执行异步 I/O 的 API，包括 TCP 和 UDP 套接字、文件系统操作以及进程和信号管理。
- 用于执行异步代码的运行时，包括任务调度程序、由操作系统事件队列（epoll、kqueue、IOCP 等）支持的 I/O 驱动程序和高性能计时器。


## 引用包

rust为了减小编译包的大小，对库中的内容做了功能模块的分类，通过导入包时标注的flags标记，来引用对应的功能模块。 
Tokio库默认不支持任何功能, 有很多，这里列举几个常用的，实在不行就先用`full`，后续要考虑包大小的时候，再逐个删减
- full 引用全部
- tcp 引用 Tokio::net::tcp部分
- fs 引用tokio::fs 部分
- dns 引用异步的tokio::net::ToSocketAddrs

引用示范
```bash
tokio={version="0.2",features= ["full"] }
```

## rust 协程模型

rust本身不像go一样支持协程，但是有一个tokio的包，它实现了包括一个task系统，一个runtime，以及异步编码的一些基本库。

其中runtime就是一个线程，task就是一个协程

- task不会阻塞，它是全异步的
- task的执行不会被打断，也就是如果task不主动退出，是不会被切换走的


如果一个task中依赖了某个底层资源，runtime执行这个task的时候，根据底层资源是否ready，可以分为两种情况处理：1，底层资源已经ready，那么task可以立即返回需要的结果；2，底层资源还没有准备好，那么task返回NotReady，并再次进入调度队列。task系统此时会记录下这个task和相应底层资源的关系，等到底层资源ready后，会再次将task交给runtime来执行

如果task中没有依赖底层资源，runtime执行这个task的时候，会立即返回结果。需要注意的是，如果task中的计算逻辑过重，占用cpu时间过长，会影响到其他task的执行。因为tokio的runtime没有给task分配执行时间片，而是会一直将这个task执行完成，这个过程中，其他的task可能会长时间得不到执行，这是一种非预期的情形。这也是为什么task中不能写太重的计算逻辑的原因。


对于线程来说，创建runtime的时候，我们可以指定两种形式的runtime
- local thread 单线程
- thread pool  多线程

## Moudle Runtime

rust runtime相当于一个线程调度的功能，默认情况下如果不需要额外的runtime，可以在main方法上添加`[tokio::main]`创建一个runtime,比如像这样
```rust
use tokio::net::TcpListener;
use tokio::prelude::*;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut listener = TcpListener::bind("127.0.0.1:8080").await?;

    loop {
        let (mut socket, _) = listener.accept().await?;

        tokio::spawn(async move {
            let mut buf = [0; 1024];

            // In a loop, read data from the socket and write the data back.
            loop {
                let n = match socket.read(&mut buf).await {
                    // socket closed
                    Ok(n) if n == 0 => return,
                    Ok(n) => n,
                    Err(e) => {
                        println!("failed to read from socket; err = {:?}", e);
                        return;
                    }
                };

                // Write the data back
                if let Err(e) = socket.write_all(&buf[0..n]).await {
                    println!("failed to write to socket; err = {:?}", e);
                    return;
                }
            }
        });
    }
}
```

也可以自己建立一个runtime

```rust
fn main() -> Result<(), Box<dyn std::error::Error>>{
    let mut rt = Runtime::new()?; // create runtime
    rt.block_on(async {
        let mut listener = TcpListener::bind("127.0.0.1:8080").await?;

        loop {
            let (mut socket, _) = listener.accept().await?;

            tokio::spawn(async move {
                let mut buf = [0; 1024];

                // In a loop, read data from the socket and write the data back.
                loop {
                    let n = match socket.read(&mut buf).await {
                        // socket closed
                        Ok(n) if n == 0 => return,
                        Ok(n) => n,
                        Err(e) => {
                            println!("failed to read from socket; err = {:?}", e);
                            return;
                        }
                    };

                    // Write the data back
                    if let Err(e) = socket.write_all(&buf[0..n]).await {
                        println!("failed to write to socket; err = {:?}", e);
                        return;
                    }
                }
            });
        }
    })
}
```

简化版如下，创建一个线程 在线程中运行多个协程，协程以task的形式出现，
```rust
fn main() -> Result<(),Box<dyn std::error::Error>>{

    let mut  rt = tokio::runtime::Runtime::new()?;

    rt.block_on(async {
       let mut num = 0;


       // 添加任务
       tokio::spawn(async move{
               num += 1;
               println!("{:?}",num); // 1 down
       });

       println("{:?}",num); // 0, because num in task will be moved
       // 添加任务
       tokio::spawn(async move{
           num += 2;
           println!("{:?}",num); // 2 down 
       })
    });

    Ok(())
}

```


### tokio::spawn 添加任务

向当前的runtime中添加任务


### task没有分配时间切片
用过其他语言的协程的可能会知道，一个线程里面有多个协程的时候，会给他们分配一个时间切片， 也就是运行一段a协程之后就会调度去运行一段b协程,再过一段时间之后就会去运行a协程等等。
但是tokio 中的task并没有分配时间切片，因此当第一个task始终在loop中或者说计算力大的时候，就会出现之后加入的task都不会执行。

```rust
fn main() -> Result<(),Box<dyn std::error::Error>>{

    let mut  rt = tokio::runtime::Runtime::new()?;

    rt.block_on(async {
       let mut num = 0;
       tokio::spawn(async move{
           loop {
               num += 1;
               println!("{:?}",num);
           }
       });


       tokio::spawn(async move{
           println!("****************************"); // 2 down 
           println!("****************************"); // 2 down 
           println!("****************************"); // 2 down 
           println!("****************************"); // 2 down 
           println!("****************************"); // 2 down 
           println!("****************************"); // 2 down 
           println!("****************************"); // 2 down 
           println!("****************************"); // 2 down 
       })
    });

    Ok(())
}

```


## Module spawn

任务调度


## Module timer

这个模块 通常要和rust标准库中的time模块一起使用，有一下三个功能

- sleep 阻塞一段时间后运行
- Interval 固定周期运行
- Timeout 设置运行执行时间量的上线，如果超出时间则会返回错误。


延时1秒后发送
```rust
use tokio::time::sleep;
#[tokio::main]
async fn main(){
    sleep(std::time::Duration::from_secs(1));
    println("send");
}
```

计算运行时间，不得超过1s 不然就报错

```rust
use tokio::time::{timeout, Duration};

async fn long_future() {
    // do work here
}

let res = timeout(Duration::from_secs(1), long_future()).await;

if res.is_err() {
    println!("operation timed out");
}
```


间隔运行
```rust
use core::fmt;
use std::{os::unix::raw::time_t, time::SystemTime};

use chrono::Local;

fn main() -> Result<(),Box<dyn std::error::Error>>{

    let mut  rt = tokio::runtime::Runtime::new()?;

    rt.block_on(async{
            let mut interval = tokio::time::interval(std::time::Duration::from_secs(10));
            loop {
                interval.tick().await;
                println!("{:?}",chrono::Local::now());
            }
    });

    Ok(())
}
```

超时发送，这里用一个单通道接发做测试
```rust
fn main() -> Result<(),Box<dyn std::error::Error>>{

    let mut  rt = tokio::runtime::Runtime::new()?;

    rt.block_on(async{
        let (tx,rx) = tokio::sync::oneshot::channel();
        tokio::spawn(async move{
            tx.send("helloworld".to_string()).unwrap();
            if let Err(_) = tokio::time::timeout_at(tokio::time::Instant::now() + tokio::time::Duration::from_millis(10), rx).await{
                println!("timeout")
            } else {
                println!("{:?}",rx.await.unwrap())
            }
        })
    });
    
    Ok(())
}
```

## Module sync

https://docs.rs/tokio/1.7.1/tokio/sync/index.html

这个模块主要用于多个task之间的消息传递,
Tokio 程序中最常见的同步形式是消息传递。两个任务独立运行，相互发送消息进行同步。这样做的好处是避免共享状态。
消息传递主要依靠channel实现，消息传递根据传递方式的不同有如下几种形式

> oneshot 单次通信

```rust


// 设置消息
fn set_message() -> String{
    "hello".to_string()
}
fn main() -> Result<(),Box<dyn std::error::Error>>{

    let rt = tokio::runtime::Runtime::new()?;

    rt.block_on(async{
        // 创建管道
        let (tx,rx) = tokio::sync::oneshot::channel();
        // 设置间隔时间
        let interval = tokio::time::interval(std::time::Duration::from_secs(10));

        // 不能放loop里面，只是一次性通信
        tokio::spawn(async move{
            let msg = set_message();
            tx.send("helloworld".to_string()).unwrap();
        });
    });
    
    rt.block_on(async {
        let (t,r) = tokio::sync::oneshot::channel();
        tokio::spawn(async move{
            let res = "hellworold".to_string();
            t.send(res).unwrap();
        });
        
        println!("{:?}",r.await.unwrap()
        )
    });
    Ok(())
}
```

> mpsc 多生产者向单个消费者发送许多值，

```rust

// 设置消息
fn set_message() -> String{
    "hello".to_string()
}
fn main() -> Result<(),Box<dyn std::error::Error>>{

    let rt = tokio::runtime::Runtime::new()?;

    rt.block_on(async{
        // 创建管道, 设置管道长度,
        let (tx,mut rx) = tokio::sync::mpsc::channel(100);
        // 向通道中不断的发送消息，间隔为10s
        tokio::spawn(async move{
            // println!("{:?}",chrono::Local::now());
           match tx.send("helloworld".to_string()).await{
               Ok(value) => println!("{:?}",value),                
               Err(e) => println!("{:?}",e),
           }
        });

        // 不设置接收的话，会报错的
        while let Some(Result) = rx.recv().await {
            println!("{:?}",Result)
        }
    });
    
    Ok(())
}

```