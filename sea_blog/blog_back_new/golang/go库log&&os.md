---
title: go库shell
categories:
  - golang
tags:
  - golang
abbrlink: 41c23ec
date:
---


<!-- more -->

# golang


## 执行命令捕捉错误

```go
package main

import (
	"fmt"
	"log"
	"os/exec"
)

func main() {
	cmd1 := exec.Command("ls","-al")

	// 捕捉错误
	if err := cmd1.Run();err != nil{
	log.Fatal(err)
	}

}
```

来看一下RUN和Command的源码
RUN源码
```go
func (c *Cmd) Run() error {
	if err := c.Start(); err != nil {
		return err
	}
	return c.Wait()
}

```
这里在实现接口中的Run方法，如果无法start则返回一个error 


## 执行命令获取返回值
```go
package main

import (
	"fmt"
	"log"
	"os/exec"
)

func main() {
	cmd1 := exec.Command("ls","-al")

	// 获取返回值
	if output,err := cmd1.CombinedOutput(); err != nil{
		log.Fatal(err)
	}else {
		fmt.Println(string(output))
	}

}
```
来看一下CombinedOutput的源码
```go
func (c *Cmd) CombinedOutput() ([]byte, error) {
	if c.Stdout != nil {
		return nil, errors.New("exec: Stdout already set")
	}
	if c.Stderr != nil {
		return nil, errors.New("exec: Stderr already set")
	}
	var b bytes.Buffer
	c.Stdout = &b
	c.Stderr = &b
	err := c.Run()
	return b.Bytes(), err
}
```