---
title: rust_package
date: 2021-06-22 17:03:55
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust_package

记录一些rust中包出现的问题，以及对应的解决办法

## 清除cargo缓存

一般情况下你cargo下的包，会保存在`~/cargo/registry`下面,删除项目中的target文件夹之后重新编译，如果本地是存在一些依赖库的他会直接去调用，而不是重新下载。如果本身的包就有问题的话就会报错。

实际案例
```rust
error: expected `::`, found `,`
  --> /Users/alpaca/.cargo/registry/src/mirrors.ustc.edu.cn-61ef6e0cd06fb9b8/tokio-nsq-0.12.0/src/consumer.rs:66:22
   |
66 |     topic: <NSQTopic>,
   |                      ^ expected `::`
```
这里下了一个tokio-nsq的库，但是呢编译运行的时候报错，说是源码里面有些问题,我多次删除了target文件夹重新编译也解决不了这个问题，后面是需要清除缓存才行，直接删

```bash
rm -r  /Users/alpaca/.cargo/registry/src/mirrors.ustc.edu.cn-61ef6e0cd06fb9b8/tokio-nsq-0.12.0/
rm -r target
cargo c ## cargo check

```
