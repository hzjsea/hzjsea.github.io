---
title: golang测试testing
categories:
  - golang
tags:
  - golang
abbrlink: 66a16cf8
date: 2020-09-29 17:00:40
---


<!-- more -->

# golang测试


## 初始化项目包
- 创建项目
- 使用Go mod 初始化项目


项目文件夹如下
```bash
🏃:Module/ $ tree   
.
├── go.mod
├── hello.go
└── hello_test.go

0 directories, 3 files

🏃:GolandProjects/ $ more Module/go.mod 
module Module/hello

go 1.13



# 创建Module
mkdir Module && cd Module
# 初始化包管理
go mod init Module/hello
# 创建hello.go项目
touch hello.go
touch hello_test.go
```

编写程序
```go
// hello.go
package Module

import "fmt"

func sayhello() string {
	fmt.Println("helloworld")
	return "helloworld"
}
```
```go
// hello_test.go
package Module

import (
	"fmt"
	"testing"
)

func helloTest(t *testing.T){
	want := "helloworld"
	if got:=sayhello();got !=want{
		fmt.Println("is error ")
		return
	}
	fmt.Println("is true")
}
```

运行`go test`

```bash
🏃:Module/ $ go test
testing: warning: no tests to run
PASS
ok      Module/hello    0.005s
```

## 初始化项目包2
项目结构如下
```bash
🏃:GolandProjects/ $ tree mm 
mm
├── go.mod
└── src
    ├── hello.go
    └── hello_test.go

1 directory, 3 files

🏃:GolandProjects/ $ more mm/go.mod 
module mm/src

go 1.13


```

构建项目

```bash
mkdir -p mm/src
cd mm/
go mod init mm/src/hello
touch src/hello.go
touch src/hello_test.go
```

程序如下
```go
// hello.go
package src

import "fmt"

func sayhello() string {
	fmt.Println("helloworld")
	return "helloworld"
}
```
```go
// hello_test.go
package src

import (
	"fmt"
	"testing"
)

func HelloTest(t *testing.T){
	if got:=sayhello();got != "helloworld"{
		fmt.Println("None")
	}
	fmt.Println("yes")
}

```
运行 ` cd mm/src/ &&  go test`
```go
🏃:src/ $ go test               
testing: warning: no tests to run
PASS
ok      mm/src/src      0.005s
```


## 错误
```bash
go: cannot find main module; see 'go help modules'
```
因为项目根目录下面没有go.mod文件，创建这个文件，这个文件里面用来管理module的