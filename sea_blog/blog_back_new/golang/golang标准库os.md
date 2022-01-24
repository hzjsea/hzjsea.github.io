---
title: golang标准库os
date: 2020-12-16 18:28:38
categories: [golang,]  
tags: [golang,golibrary]
---


<!-- more -->

# go库os

os这个库主要实现了以下几个部分的功能

- 获取系统变量
- 文件操作
- 获取进程状态

## 系统指令
```bash
hostname # 获取主机名
getpagesize # 内存页尺寸
environ # 环境变量
getenv # 根据key获取变量
setenv # 设置环境变量
clearenv # 清除所有环境变量
exit # 以当前状态码退出程序 0表示成功  非0表示出错
```


## 文件操作

### 文件信息

```golang
type FileInfo interface {
    Name() string       // 文件的名字（不含扩展名）
    Size() int64        // 普通文件返回值表示其大小；其他文件的返回值含义各系统不同
    Mode() FileMode     // 文件的模式位
    ModTime() time.Time // 文件的修改时间
    IsDir() bool        // 等价于Mode().IsDir()
    Sys() interface{}   // 底层数据来源（可以返回nil）
}
```

> 返回文件信息，不存在则报错 stat() 与 lstat()

stat() 碰到软硬连接的文件的时候，会跳转
lstat() 碰到连接文件 不会跳转

```golang
func main(){
	file, err := os.Stat("./go.mod")
	if err != nil{
		log.Fatal("err")
	}
	fmt.Println(file.Mode().String()) // -rw-r--r--
}
```

> 判断是否为分隔符号

```golang
fmt.Println(os.IsPathSeparator('/'))
```

> 文件夹操作

```golang
func main(){
	// 返回当前工作目录的路径
	curren_path, err := os.Getwd()
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(curren_path) // /Users/alpaca/GolandProjects/workspace_golang

	// 修改工作目录
	err = os.Chdir("/Users/alpaca/GolandProjects/")
	if err != nil{
		log.Fatal(err)
	}
	curren_path,_ = os.Getwd()
	fmt.Println(curren_path) // /Users/alpaca/GolandProjects

	// 使用指定权限创建文件
	curren_path = curren_path + "/workspace_golang"
	if err:=os.Mkdir(curren_path+"/test/",os.ModePerm);err!=nil{
		fmt.Println("创建失败")
	}

	// 递归创建文件夹
	if err := os.MkdirAll(curren_path+"/test/demo/text",os.ModePerm); err != nil{
		fmt.Println("创建失败")
	}

	// 重命名文件
	curren_file := curren_path+"/xx.tx"
	stat,err := os.Stat(curren_file)
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(stat.Name())
	if err := os.Rename(curren_file,curren_path+"./yy.txt");err != nil{
		log.Fatal("更换失败")
	}
}

```

> 文件操作

```golang
func main(){
	// 获取当前工作目录
	currentPath,err := os.Getwd()
	if err != nil{
		log.Fatal("无法获取工作目录")
	}
	newFilePath := currentPath +"/xx.txt"
	newFile, err := os.Create(newFilePath) // 如果文件存在，文件会被清空
	if err != nil{
		log.Fatal("无法创建文件")
	}
	fmt.Println(newFile.Name())
	defer newFile.Close()

	// 只读模式打开文件
	newfile, err := os.Open(newFilePath)
	if err != nil{
		log.Fatal("无法打开文件")
	}
	fmt.Println(newfile)
	defer newfile.Close()

	// 也可以打开目录
	path,err := os.Open("./")
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(path.Name())

	// 自定义权限打开文件
	newfile, err = os.OpenFile(newFilePath,os.O_RDONLY|os.O_RDWR|os.O_TRUNC,os.ModePerm)
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(newfile.Name())
	fmt.Println(newfile.Fd())
	defer newfile.Close()
}
```

> 获取当前目录下面的所有文件信息

有两种实现方法
```golang
func main(){
	path,err := os.Open("./")
	if err != nil{
		log.Fatal(err)
	}
	if list,err := path.Readdir(-1);err == nil{
		for index,value := range list{
			fmt.Println(index,value.Name())
		}
	}else {
		fmt.Println(err)
	}
}
```

```golang
fucn main(){
	list,err := path.Readdir(-1)
	defer path.Close()
	if err != nil{
		fmt.Println(err)
	}

	sort.Slice(list, func(i, j int) bool { return list[i].Name() < list[j].Name() })
	fmt.Println(list)
}
```
```golang
	files,err := ioutil.ReadDir("./")
	if err != nil{
		log.Fatal(err)
	}
	for index,value := range files{
		fmt.Println(index,value)
	}
```
```golang
files, _ := filepath.Glob("*")
    fmt.Println(files)
```
`ioutil.ReadDir`其实就是调用了上面的方法
`filepath`是调用了`path/filepath`这个库，这个库输出结果类似于这个
```golang
func main(){
	path, err := os.Open("./")
	if err != nil{
		log.Fatal(err)
	}
	files,err := path.Readdirnames(-1)
	if err != nil{
		log.Fatal(err)
	}
	for index,value := range files{
		fmt.Println(index,value)
	}
}
```

### 文件权限
文件操作就离不开文件权限
```bash
const (
    # 单字符是被String方法用于格式化的属性缩写。
    ModeDir        FileMode = 1 << (32 - 1 - iota) // d: 目录
    ModeAppend                                     // a: 只能写入，且只能写入到末尾
    ModeExclusive                                  // l: 用于执行
    ModeTemporary                                  // T: 临时文件（非备份文件）
    ModeSymlink                                    // L: 符号链接（不是快捷方式文件）
    ModeDevice                                     // D: 设备
    ModeNamedPipe                                  // p: 命名管道（FIFO）
    ModeSocket                                     // S: Unix域socket
    ModeSetuid                                     // u: 表示文件具有其创建者用户id权限
    ModeSetgid                                     // g: 表示文件具有其创建者组id的权限
    ModeCharDevice                                 // c: 字符设备，需已设置ModeDevice
    ModeSticky                                     // t: 只有root/创建者能删除/移动文件
    // 覆盖所有类型位（用于通过&获取类型位），对普通文件，所有这些位都不应被设置
    ModeType = ModeDir | ModeSymlink | ModeNamedPipe | ModeSocket | ModeDevice
    ModePerm FileMode = 0777 // 覆盖所有Unix权限位（用于通过&获取类型位）
)
```
这个文件权限对应的就是linux当中的权限分配
`rwxrwxrwx`


- 查看是否是一个目录 FileMode.IsDir
- 查看是否是一个文件 FileMode.IsRegular
- 查看权限位  FileMode.Perm
- 转换成字符串类型 FileMode.String()  -rw-r--r--




## os读取文件

```golang
package main

import (
   "os"
   "io/ioutil"
   "fmt"
)

func main() {
   file, err := os.Open("../log.txt") // 只读模式打开
   if err != nil {
      panic(err)
   }
   defer file.Close()
   content, err := ioutil.ReadAll(file)
   fmt.Println(string(content))
}
```

```golang

package main

import (
   "os"
   "io/ioutil"
   "fmt"
)

func main() {
   filepath := "./log.txt"
   content ,err :=ioutil.ReadFile(filepath) 
   if err !=nil {
      panic(err)
   }
   fmt.Println(string(content))
}
```





## 文件指令 FileMode



```bash
isdir # 是否是个目录
isregular # 是否是一个普通文件
perm # 返回权限位
String # 字符串格式化

```

## 文件对象  FileInfo

```bash
type FileInfo interface {
    Name() string       // 文件的名字（不含扩展名）
    Size() int64        // 普通文件返回值表示其大小；其他文件的返回值含义各系统不同
    Mode() FileMode     // 文件的模式位
    ModTime() time.Time // 文件的修改时间
    IsDir() bool        // 等价于Mode().IsDir()
    Sys() interface{}   // 底层数据来源（可以返回nil）
}
```
获取文件对象
```bash
os.stat
os.lstat
os.isexist
os.isnoexist
os.isPermission
os.getwd
os.chdir
os.chmod
os.chowd
os.mkdir
os.mkdirall
os.rename
os.remove
os.removeall
```


打开一个文件对象
```bash
type File struct {
    // 内含隐藏或非导出字段
}

os.create
os.open

file.Name
file.stat
file.Size
file.fd
file.chdir
file.chmod
file.chowd

```


```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
)

func main() {
	fmt.Println(os.Getpagesize())
	pwd,err:= os.Getwd()
	if err != nil{
		log.Fatal()
	}
	fmt.Println(pwd)


	// 获取文件信息  stat检测的如果是链接文件会跳转过去,但是lstat不会
	fileinfo,err := os.Stat(pwd+"/client.go")
	if err != nil{
		log.Fatal()
	}
	fmt.Println(fileinfo)
	fmt.Println(fileinfo)

	fileinfo2,err := os.Lstat(pwd+"/client.go")
	if err != nil{
		log.Fatal()
	}
	fmt.Println(fileinfo2)

	// 判断一个错误 是不是因为文件或目录不存在 而导致的
	fileinfo,err = os.Lstat("ds")
	if err != nil{
	 	fmt.Println(os.IsExist(err))
	}else {
		fmt.Println(fileinfo.Name())
	}

	// 返回一个布尔值说明该错误是否表示因权限不足要求被拒绝
	fileinfo,err = os.Lstat("ds")
	if err != nil{
		fmt.Println(os.IsPermission(err))
	}else{
		fmt.Println(fileinfo.Name())
	}

	// Getwd返回一个对应当前工作目录的根路径
	pwd,_ = os.Getwd()
	fmt.Println(pwd)

	// 修改工作目录
	//pwd,_ = os.Getwd()
	//fmt.Println(pwd)
	//os.Chdir("/etc/")
	//pwd,_ = os.Getwd()
	//fmt.Println(pwd)

	// 修改文件权限
	//os.Chmod()
	// 修改用户组
	//os.Chown()

	// 创建文件夹
	//pwd,_= os.Getwd()
	//fmt.Println(pwd)
	//os.Chdir("/")
	pwd,_ = os.Getwd()
	fmt.Println(pwd)
	os.Mkdir(pwd+"/helloword.txt",os.ModePerm)

	// 递归创建文件 相当于 mkdir -p
	pwd,_ = os.Getwd()
	fmt.Println(pwd)
	os.MkdirAll(pwd+"/xx/helloworld.txt",os.ModePerm)
	listfile,err := os.Stat(pwd)
	if err !=nil{
		log.Fatal()
	}
	fmt.Println(listfile.Name())

	// 删除文件
	os.Remove(pwd+"/helloworld.txt")

	// 递归删除文件
	os.RemoveAll(pwd+"/xx/helloworld.txt")

	// TempDir返回一个用于保管临时文件的默认目录。
	//temdir := os.TempDir()
	//fmt.Println(temdir)
	//temdir_info,err := os.Stat(temdir)
	//fmt.Println(temdir_info.Name())


	// 创建一个模式为0666的文件,可读写不可运行
	file,err:=os.Create("helloworld")
	fmt.Println(file.Name())
	// 读取一个文件 或者一个文件夹
	file,err = os.Open("helloworld")

	// 返回文件名字
	fmt.Println(file.Name())
	// 返回Unix文件描述符
	fmt.Println(file.Fd())

	file,err = os.Open("xx")
	fmt.Println(file.Name())
	file.Chdir()
	pwd,err = os.Getwd()
	fmt.Println(pwd)



	// 进程
	// StartProcess使用提供的属性、程序名、命令行参数开始一个新进程

	// FindProcess根据进程id查找一个运行中的进程
}
```