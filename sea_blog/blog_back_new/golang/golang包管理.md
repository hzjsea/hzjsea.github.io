---
title: golang包管理
categories:
  - golang
tags:
  - golang
abbrlink: 842f1cd7
date: 2020-05-17 01:18:04
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [golang包管理](#golang包管理)
  - [golang包管理历史](#golang包管理历史)
  - [使用合适的包管理工具](#使用合适的包管理工具)
    - [包管理工具 go mod](#包管理工具-go-mod)
    - [包下载代理  goproxy](#包下载代理-goproxy)
  - [常用的命令](#常用的命令)

<!-- /code_chunk_output -->


<!-- more -->

# golang包管理


## golang包管理历史
在旧版本中的golang包管理一直是一个没有被官方重视的地址
- 在 1.5 版本之前，所有的依赖包都是存放在 GOPATH 下，没有版本控制。这个类似 Google 使用单一仓库来管理代码的方式。这种方式的最大的弊端就是无法实现包的多版本控制，比如项目 A 和项目 B 依赖于不同版本的 package，如果 package 没有做到完全的向前兼容，往往会导致一些问题
- 1.5 版本推出了 vendor 机制。所谓 vendor 机制，就是每个项目的根目录下可以有一个 vendor 目录，里面存放了该项目的依赖的 package。go build 的时候会先去 vendor 目录查找依赖，如果没有找到会再去 GOPATH 目录下查找
- 1.9 版本推出了实验性质的包管理工具 dep，这里把 dep 归结为 Golang 官方的包管理方式可能有一些不太准确。关于 dep 的争议颇多，比如为什么官方后来没有直接使用 dep 而是弄了一个新的 modules，具体细节这里不太方便展开。
- 1.11 版本推出 modules 机制，简称 mod。modules 的原型其实是 vgo，关于 vgo，可以自行搜索


另外社区为了方便管理golang的包，一直存在几个活跃的包管理工具
- godep
- glide
- govendor


## 使用合适的包管理工具
前面讲了一大堆的包管理工具的发展以及如何使用，现在我们来讲讲面对新版本的golang语言，我们应该有什么包管理工具来处理

### 包管理工具 go mod
Golang 版本：1.12.3。在 1.12 版本之前，使用 Go modules 之前需要环境变量 GO111MODULE:
在1.13版本以后 go mod是默认开启的。不在需要设置环境变量
```
- GO111MODULE=off: 不使用 modules 功能，查找vendor和GOPATH目录
- GO111MODULE=on: 使用 modules 功能，不会去 GOPATH 下面查找依赖包。
- GO111MODULE=auto: Golang 自己检测是不是使用 modules 功能，如果当前目录不在$GOPATH 并且 当前目录（或者父目录）下有go.mod文件，则使用 GO111MODULE， 否则仍旧使用 GOPATH mode

  export GO111MODULE=on //linux
```
创建项目
```go
  package main
  ​
  import "fmt"
  import "github.com/astaxie/beego"
  func main() {
      fmt.Println("hello")
      beego.Run()
  }
```
初始化go module 
```go
go mod init test
```
这里他会生成一个mod文件，和python里面的pyenv库一样，这个mod文件用于存放下载的包 另外还会产生一个go.sum 文件，这个文件用来存放依赖包对应版本的下载地址和哈希值这些 package 并不是直接存储到 GOPATH/src，而是存储到 GOPATH/pkg/mod 下面，不同版本并存的方式。
运行项目，go会自动检测使用的包,并且下载


### 包下载代理  goproxy
有时候我们下包会出现超时，网络方面的问题，这是因为go的一些包会被丢在国外，然后无法透过网络，传到国内，这个时候就要用一些代理来完成这个操作，这里的代理值得是一些ss或者说国内的镜像源
```go
// 切换镜像源
export GOPROXY=https://goproxy.cn
// export GOPROXY=https://mirrors.aliyun.com/goproxy/
```
![2020-05-17-01-46-43](http://noback.upyun.com/2020-05-17-01-46-43.png)



## 常用的命令
```go
// 初始化modules
go mod init filename
// 更新所有依赖
go list -u -m all
```

