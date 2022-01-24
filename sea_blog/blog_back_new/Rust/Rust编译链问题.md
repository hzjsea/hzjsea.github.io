---
title: Rust编译链问题
date: 2021-05-07 10:00:12
categories:  [rust]
tags: [rust]
---


<!--more-->


# Rust编译链问题

在rust的使用过程当中，遇到了一个交叉编译的问题，赶紧现在的交叉编译并不完善，没有像go那样好用

> 方案1
<div style='font-size:20px;color:red'>
1. 我在mac端写好对应的程序,并编译(没有用交叉编译)
2. 在linux端无法执行编译后的程序  
# 这是正常的 ，因为我没有使用交叉编译，但是mac的交叉编译，我在下载编译链的时候，电脑风扇哗哗哗的转，于是我就放弃了这个选择
</div>

> 方案2
<div style='font-size:20px;color:red'>
1. 在mac端编写好程序，使用对应系统的机器编译，再使用
2. 但是这里有另外一个问题，首先我编译的机器是centos7 或者Ubuntu20，运行的机器系统是 centos6(因为特殊原因不想破坏centos6的环境)
3. 这样就会导致centos6无法跑centos7编译好后的程序
</div>

> 方案3
<div style='font-size:20px;color:red'>
1. 在mac端编写好程序，使用对应系统对应版本的机器进行编译，再运行
2. 在centos6上安装好环境，发现版本不对，但是怎么升级编译链都升级不上去
</div>

这里要提一句，rust的版本控制主要看两个，  一个是rustc 一个是cargo ，但是在更新上主要使用rustup来升级
```bash
# 查看当前rustc 的版本
rustc --version
# 查看当前cargo 的版本
cargo --version
```
使用rustup升级
```bash
# 先升级本身
rustup self update
# 再升级工具链
rustup update 
# 这时候就可以看当前有哪些工具链了,可以看下图
rustup show 
# 这个时候并没有正式的替换，需要使用default来更改默认编译的版本
rustup default xxxx-unknown-linux-gnu


## 示例 安装nightly版本
rust self update
rust update
rustup install nightly  
rustup default nightly 

## 示例 安装其他的版本
rust self update 
rust update
rustup install nightly-2020-03-19
rustup default nightly-2020-03-19
```
![](https://noback.upyun.com/2021-05-07-10-13-27.png!)
