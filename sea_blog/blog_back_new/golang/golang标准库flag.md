---
title: golang标准库flag.md
date: 2021-02-05 11:10:15
categories:  [golang]
tags: [golang]
---

# golang标准库flag
在运行程序后跟随参数有两种方式
- 一种方式是直接跟随参数 我们可以把它叫做`列表参数`，没有任何的对应关系，只有顺序关系
- 另外一种方式是在输入参数之前需要输入对应的key ，我们把它叫做`字典参数`, 他们就像字典一样有对应的关系

列表参数和字典参数形式如下
```bash
# 列表参数
./xx.sh a b c d e 
# 字典参数
./xx.sh -name foo -age 12 --desc bar  
```

在go中,处理列表参数常用`os.Args`来处理，比较简单, 字典参数使用`flag`来处理


## 处理简单的列表参数
> `os.Args` 会返回运行程序后面的所有参数列表,默认第一个为运行程序的地址

```go
func main() {
	args := os.Args
	fmt.Println(args) // [/var/folders/1s/j9mtgsg14jv_xdqhrt1j42l80000gn/T/go-build424504941/b001/exe/lib_flag]
	argsUserful := os.Args[1:]
	fmt.Println(argsUserful) // []

	argsList := os.Args
	for index, value := range(argsList){
		fmt.Printf("%d---%s\n", index, value)
	}
	// 0---/var/folders/1s/j9mtgsg14jv_xdqhrt1j42l80000gn/T/go-build973842478/b001/exe/lib_flag
	// 1---xx
}
```

## 处理字典参数

### 形式1
仔细看flag库，会发现flag库其实有很大的复用性，这样做的目的只是为了适配的更多的变量类型
可以只看一个类型，然后举一反三推出其他类似内容的功能，这里先看`flag.String()`,

```golang
	params := flag.String("name","foo","input your name") // 这里返回的是一个存储参数地址
	flag.Parse()
	fmt.Println(*params) // bar
	fmt.Println(params) // 0xc000054220
```
接收三个参数，三个参数代表的内容如下
- "name" 字典参数的key
- "foo" 默认字典参数中的key 对应的value
- "input your name" 帮助列表中的内容 默认情况下 --help可以显示帮助

`flag.String`返回一个保存了该flag的值的指针
如上执行时候的内容应该是
```bash
go run xx.go -name bar
```

> 输入正常，但是无法返回指定的value 而是返回了默认值

这是因为所有定义好的字典参数的处理后，还需要做一步解析(`flag.Parse`)，这样go才会去解析获取到的参数列表

> 字典参数接收的输入类型

为了适配多种形式的输入，字典参数适配了四种类型的输入，如下

- -flag xxx （使用空格，一个-符号）
- --flag xxx （使用空格，两个-符号）
- -flag=xxx （使用等号，一个-符号）
- --flag=xxx （使用等号，两个-符号）

其中bool 类型必须以=来赋值，不然go只要解析到 --flag 就会默认为True
```golang
func main() {
	params_bool := flag.Bool("isexist",false,"help")
	flag.Parse()
	fmt.Println(*params_bool)
}
```
如上,
```bash
go run lib_flag.go  --isexist  			# true
go run lib_flag.go  --isexist  false 	# true
go run lib_flag.go  --isexist=false 	# false
```


### 形式2 

```golang
var (
	name string
	delay time.Duration
)

func main() {
	flag.StringVar(&name,"name","ds","input your name")
	flag.DurationVar(&delay,"time",0,"now time")

	flag.Parse()
	fmt.Println(name)
	fmt.Println(delay)
}
```

### 其他使用方法
- flag.Agrs() 返回除去字典参数外的其他参数，我们叫做额外参数
- flag.Arg(n int) 返回额外参数列表中的第n个参数
- flag.NArg() 返回额外列表的中参数的数量
- flag.NFlag() 返回字典参数列表中key-value的对数

```go
var (
	name string
	delay time.Duration
)
func main() {
	flag.StringVar(&name,"name","ds","input your name")
	flag.DurationVar(&delay,"time",0,"now time")

	// go run lib_flag.go -name foo -time 0 dsdsdsds dsds ds dsds
	flag.Parse()
	fmt.Println(name) // foo
	fmt.Println(delay) // 0s

	fmt.Println(flag.Args()) 		//  [dsdsdsds dsds ds dsds]
	fmt.Println(flag.Arg(2))        //  ds
	fmt.Println(flag.NArg())		//  4
	fmt.Println(flag.NFlag()) 		//  2
}
```

### command-help
flag默认情况下使用`--help`来表示输出命令的帮助信息，这有在源码中写道
```golang
// --- 
flag.Parse()
// -- 
func Parse() {
	CommandLine.Parse(os.Args[1:])
}
// ---
f.parseOne()
// ---
	m := f.formal
	flag, alreadythere := m[name] // BUG
	if !alreadythere {
		if name == "help" || name == "h" { // special case for nice help message.
			f.usage()
			return false, ErrHelp
		}
		return false, f.failf("flag provided but not defined: -%s", name)
	}
// ---
func (f *FlagSet) usage() {
	if f.Usage == nil {
		f.defaultUsage()
	} else {
		f.Usage()
	}
}
```
当识别到`name==help`或者`name==h`的时候，就回去读取flag中的usage这个属性，然后输出

```golang
var (
	name string
	delay time.Duration
)

func main() {
	flag.StringVar(&name,"name","hzj","output your name")
	flag.DurationVar(&delay,"time",0,"output now time")
	flag.Parse()
}
```

```bash
Usage of /var/folders/1s/j9mtgsg14jv_xdqhrt1j42l80000gn/T/go-build438323228/b001/exe/lib_flag:
  -name string
        output your name (default "hzj")
  -time duration
        output now time
exit status 2
```



## subcommand 子命令

```golang
    // 我们使用 `NewFlagSet` 函数声明一个子命令，
    // 然后为这个子命令定义一个专用的 flag。
    fooCmd := flag.NewFlagSet("foo", flag.ExitOnError)
    fooEnable := fooCmd.Bool("enable", false, "enable")
    fooName := fooCmd.String("name", "", "name")

    // 对于不同的子命令，我们可以定义不同的 flag。
    barCmd := flag.NewFlagSet("bar", flag.ExitOnError)
    barLevel := barCmd.Int("level", 0, "level")
```

```golang
package main

import (
	"flag"
	"fmt"
	"log"
	"os"
)

var (
	action string
)

var (
	addCmd = flag.NewFlagSet("add", flag.ExitOnError)
	delCmd = flag.NewFlagSet("del", flag.ExitOnError)
	updateCmd = flag.NewFlagSet("update", flag.ExitOnError)
)

func main() {

	if len(os.Args) < 2{
		log.Fatal("参数过短")
		os.Exit(1)
	}

	switch os.Args[1] {
	case "add":
		addCmd.Parse(os.Args[2:])
		fmt.Println("subcommand 'foo'")
	case "del":
		delCmd.Parse(os.Args[2:])
		fmt.Println("subcommand 'del'")
	case "update":
		updateCmd.Parse(os.Args[2:])
		fmt.Println("subcommadn 'update'")
	default:
		fmt.Println("expected 'foo' or 'bar' subcommands")
		os.Exit(1)
	}
}


```