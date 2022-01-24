---
title: python并发编程
categories:
  - 高级特性
tags:
  - 高级特性
abbrlink: 3194b181
date: 2020-09-28 18:10:37
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->

# python并发编程
提到并发编程,就离不开线程的使用，先来介绍一下线程
多进程和多线程都可以执行多个任务，线程是进程的一部分。线程的特点是线程之间可以共享内存和变量，资源消耗少，缺点是线程之间的同步和加锁比较麻烦。


## 进程的使用
使用os fork 创建多进程
Unix/Linux操作系统提供了一个`fork()`系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是`fork()`调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。如下
```py
import os
print('Process (%s) start...' % os.getpid())
pid = os.fork()
print(pid)

# res
Process (87128) start...
87129
0
```
复制了一份到子进程中返回，也就是子进程返回0  父进程返回87129  87129是复制出来的子进程在87128这个父进程管控下的pid

```py
import os

print('Process (%s) start...' % os.getpid())
# Only works on Unix/Linux/Mac:
pid = os.fork()
if pid == 0:
    print('I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid()))
else:
    print('I (%s) just created a child process (%s).' % (os.getpid(), pid))

# res
Process (876) start...
I (876) just created a child process (877).
I am child process (877) and my parent is 876.
```

使用  multiprocessing 创建多进程
```py
from multiprocessing import Process
import os

# 子进程要执行的代码
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('Child process will start.')
    p.start()
    p.join()
    print('Child process end.')

# res
Parent process 928.
Child process will start.
Run child process test (929)...
Process end.
```

## 线程的使用
```py
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: occur.py.py
@time: 2020/9/28 5:52 下午
'''

import time

def countdown(n):
    while n > 0:
        print("T-minus",n)
        n -= 1
    time.sleep(5)

from threading import  Thread
t = Thread(target=countdown(10))

# Python中的线程会在一个单独的系统级线程中执行（比如说一个 POSIX 线程或者一个 Windows 线程），这些线程将由操作系统来全权管理
t.start()  # 开启线程

# 获取如下
t1 = Thread(target=countdown,args=(10,))
t1.start()
```

- 线程的其他信息
```py
# 创建线程的其他信息
t1 = Thread(target=countdown,args=(10,))
t1.start()
# 后台运行
t1 = Thread(target=countdown,args=(10,),daemon=True)
t1.start()
# 是否存活
t1.is_alive()
# 线程名字
t1.name or t1.getname()
# 是否后台运行
t1.daemon
```
- join阻塞函数
join的原理就是依次检验线程池中的线程是否结束，没有结束就阻塞直到线程结束，如果结束则跳转执行下一个线程的join函数
如果没有没有设置join的话，他会一直执行到主线程结束，也就是并没有等到子线程结束之后再去执行主线程,如下
```py
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: occur.py.py
@time: 2020/9/28 5:52 下午
'''
import time

def countdown(n):
    while n > 0:
        print("T-minus",n)
        n -= 1
        time.sleep(10)

def nextCountDown(n):
    print("is running",n)

from threading import  Thread
t = Thread(target=countdown,args=(5,),daemon=False)
t.start()
for i in range(2):
    print("helloworld")

# res
T-minus 5
Thread-1 False Thread-1
helloworld
helloworld
T-minus 4
T-minus 3
T-minus 2
T-minus 1
```
加上join阻塞之后
```py
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: occur.py.py
@time: 2020/9/28 5:52 下午
'''
import time
def countdown(n):
    while n > 0:
        print("T-minus",n)
        n -= 1
        time.sleep(1)
def nextCountDown(n):
    print("is running",n)
from threading import  Thread
t = Thread(target=countdown,args=(5,),daemon=False)
t.start()
print(t.name,t.daemon,t.getName())
t.join()
for i in range(2):
    print("helloworld")

# res
T-minus 5
Thread-1 False Thread-1
T-minus 4
T-minus 3
T-minus 2
T-minus 1
helloworld
helloworld

```


## Python线程的限制
由于全局解释锁（GIL）的原因，Python 的线程被限制到同一时刻只允许一个线程执行这样一个执行模型。所以，Python 的线程更适用于处理I/O和其他需要并发执行的阻塞操作（比如等待I/O、等待从数据库获取数据等等），而不是需要多处理器并行的计算密集型任务。在python中如果说起多线程肯定是会被认为是伪多线程，因为python在1989年圣诞节诞生的时候计算机硬件并没有那么发达，计算机也都是单核存在，所以自然而然都是以单线程去运行程序， 等到多核的出现，python开始支持多线程了， 那么解决多线程之间的数据完整性和状态同步最简单的方法就是加锁, 所以GIL(Global Interperter Lock， 全局解释器锁)出现了,顾名思义它并不是在python语言特性上的东西， 而是在python解释器上, GIL给线程加锁， 在同一个进程中，同一时间只允许一个线程运行，当线程切换的时候同时释放又重新加锁，这么一想反而多线程变得更加的麻烦，那么python的多线程真的是鸡肋吗？



## 线程间通信
线程间通信主要使用队列来实现，t1线程往队列q中不断写入数据,t2队列从队列中不断读取数据，代码如下
```py
#!/usr/bin/env python3
# encoding: utf-8
from queue import Queue
from threading import  Thread
def producer(out ):
    while True:
        out.put("111")
def consumer(in_q):
    while True:
        data = in_q.get()
q = Queue()

t1 = Thread(target=consumer, args=(q,))
t2 = Thread(target=producer, args=(q,))
t1.start()
t2.start()
```
上面使用了生产者消费者模式

## 线程间通信终止
在队列中主要使用一个特殊的值来表示终止线程
