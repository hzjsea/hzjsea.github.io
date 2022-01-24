---
title: go第三方库homedir
categories:
  - golang
tags:
  - golang
abbrlink: f18da52a
date: 2020-10-27 15:29:39
---



<!-- more -->
# go第三方库homedir

https://segmentfault.com/a/1190000021585053

主要是两个功能 一个是获取用户主目录
另一个是将路径中的第一个`~`扩展成用户主目录

```go
import (
  "fmt"
  "log"
    
  "github.com/mitchellh/go-homedir"
)
func test() {
	// 获取用户主目录
	path,err := homedir.Dir()
	if err != nil{
		log.Fatal("not  dir")
	}
	fmt.Println(path)

    // 将主目录替换成config
	path,err = homedir.Expand("config")
	if err != nil{
		log.Fatal("config is not exit")
	}

	fmt.Println(path)
}
```