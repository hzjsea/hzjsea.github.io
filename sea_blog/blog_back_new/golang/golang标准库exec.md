---
title: golang标准库exec
date: 2021-02-06 16:03:11
categories:  [golang]
tags: [golang,golibrary]
---


<!--more-->


# golang标准库exec


## os-exec

os-exec包用来执行外部的命令,他创建了一个cmd实例
```golang
type Cmd struct {
    // Path是将要执行的命令的路径。
    //
    // 该字段不能为空，如为相对路径会相对于Dir字段。
    Path string
    // Args保管命令的参数，包括命令名作为第一个参数；如果为空切片或者nil，相当于无参数命令。
    //
    // 典型用法下，Path和Args都应被Command函数设定。
    Args []string
    // Env指定进程的环境，如为nil，则是在当前进程的环境下执行。
    Env []string
    // Dir指定命令的工作目录。如为空字符串，会在调用者的进程当前目录下执行。
    Dir string
    // Stdin指定进程的标准输入，如为nil，进程会从空设备读取（os.DevNull）
    Stdin io.Reader
    // Stdout和Stderr指定进程的标准输出和标准错误输出。
    //
    // 如果任一个为nil，Run方法会将对应的文件描述符关联到空设备（os.DevNull）
    //
    // 如果两个字段相同，同一时间最多有一个线程可以写入。
    Stdout io.Writer
    Stderr io.Writer
    // ExtraFiles指定额外被新进程继承的已打开文件流，不包括标准输入、标准输出、标准错误输出。
    // 如果本字段非nil，entry i会变成文件描述符3+i。
    //
    // BUG: 在OS X 10.6系统中，子进程可能会继承不期望的文件描述符。
    // http://golang.org/issue/2603
    ExtraFiles []*os.File
    // SysProcAttr保管可选的、各操作系统特定的sys执行属性。
    // Run方法会将它作为os.ProcAttr的Sys字段传递给os.StartProcess函数。
    SysProcAttr *syscall.SysProcAttr
    // Process是底层的，只执行一次的进程。
    Process *os.Process
    // ProcessState包含一个已经存在的进程的信息，只有在调用Wait或Run后才可用。
    ProcessState *os.ProcessState
    // 内含隐藏或非导出字段
}
```
使用command生成
```bash
func Command(name string, arg ...string) *Cmd
```
```go
	cmd :=exec.Command("tr","a-z","A-Z")
    cmd.Stdin = strings.NewReader("some")
    
	var out bytes.Buffer
	cmd.Stdout = &out
    
    err := cmd.Run()
	if err != nil{
		log.Fatal()
	}
    
    fmt.Println(out.String())
```


```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"os/exec"
	"strings"
	"unsafe"
)


//https://studygolang.com/static/pkgdoc/pkg/os_exec.htm#example-Cmd-Output

func displaydir(){
	// 返回一个对象实例
	cmd := exec.Command("find", ".", "-name" ,"xx")

	// 指定cmd的输出
	var out bytes.Buffer
	cmd.Stdout = &out
	err := cmd.Run()
	if err !=nil{
		log.Fatal(err)
	}
	fmt.Println(out.String())

	fmt.Println(cmd.String()) // /usr/bin/find . -name xx


}


func pipecmd(){
	cmd := exec.Command("echo","-n",`{"Name": "Bob", "Age": 32}`)
	var out bytes.Buffer
	cmd.Stdout = &out
	err := cmd.Run()
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(out.String())
	//fmt.Println(cmd.Output())
}

func cmdstratanddwait(){
	cmd := exec.Command("sleep","5")
	err := cmd.Start() // 不会进行阻塞ned
	if err != nil{
		log.Fatal(err)
	}
	err = cmd.Wait() // 阻塞进行
	if err != nil{
		log.Fatal(err)
	}

}

func cmdoutput(){
	out,err := exec.Command("date").Output()
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(out)

	// 两种[]byte转换string的方法
	fmt.Println(string(out[:])) // 写的简单,但是效率慢
	stringout := *(*string)(unsafe.Pointer(&out))
	fmt.Println(stringout) // 效率高
}

func cmdCombinedOutput(){
	out,err := exec.Command("data").Output()
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(string(out[:]))
}

func main() {


	cmd :=exec.Command("tr","a-z","A-Z")

	cmd.Stdin = strings.NewReader("some")

	var out bytes.Buffer
	cmd.Stdout = &out
	err := cmd.Run()

	if err != nil{
		log.Fatal()
	}

	fmt.Println(out.String())

	displaydir()
	pipecmd()
	cmdoutput()
	cmdCombinedOutput()
	cmdstratanddwait()

}
```


##  lib_os

os这个包主要是针对系统进程上的一些处理，



### lib_os_exec

```go

type Cmd struct {
    // Path是将要执行的命令的路径。
    //
    // 该字段不能为空，如为相对路径会相对于Dir字段。
    Path string
    // Args保管命令的参数，包括命令名作为第一个参数；如果为空切片或者nil，相当于无参数命令。
    //
    // 典型用法下，Path和Args都应被Command函数设定。
    Args []string
    // Env指定进程的环境，如为nil，则是在当前进程的环境下执行。
    Env []string
    // Dir指定命令的工作目录。如为空字符串，会在调用者的进程当前目录下执行。
    Dir string
    // Stdin指定进程的标准输入，如为nil，进程会从空设备读取（os.DevNull）
    Stdin io.Reader
    // Stdout和Stderr指定进程的标准输出和标准错误输出。
    //
    // 如果任一个为nil，Run方法会将对应的文件描述符关联到空设备（os.DevNull）
    //
    // 如果两个字段相同，同一时间最多有一个线程可以写入。
    Stdout io.Writer
    Stderr io.Writer
    // ExtraFiles指定额外被新进程继承的已打开文件流，不包括标准输入、标准输出、标准错误输出。
    // 如果本字段非nil，entry i会变成文件描述符3+i。
    //
    // BUG: 在OS X 10.6系统中，子进程可能会继承不期望的文件描述符。
    // http://golang.org/issue/2603
    ExtraFiles []*os.File
    // SysProcAttr保管可选的、各操作系统特定的sys执行属性。
    // Run方法会将它作为os.ProcAttr的Sys字段传递给os.StartProcess函数。
    SysProcAttr *syscall.SysProcAttr
    // Process是底层的，只执行一次的进程。
    Process *os.Process
    // ProcessState包含一个已经存在的进程的信息，只有在调用Wait或Run后才可用。
    ProcessState *os.ProcessState
    // 内含隐藏或非导出字段
}
```


> 寻找可执行文件，返回绝对路径或相对路径

```go

	path,err := exec.LookPath("ls") // /bin/ls   这里不能有很多空格 比如"ls   "这样是识别不了的
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(path)

	path,_ = exec.LookPath("go") // 找可执行文件go  //  /usr/local/bin/go
	fmt.Println(path)
```

> 执行命令并获取结果

执行命令使用`exec.Command(name string, arg ...string)` ,前面的是可执行文件，后面的是各种参数
```go
	cmd := exec.Command("ls","-al")
	fmt.Println(cmd.Output())
```


> 命令执行流程

1. 首先使用`exec.Command` 返回一个 `*cmd`实例，
2. `cmn.Run()`的时候一次会去执行 `cmd.start()` `cmd.Wait()`
3. `cmd.start()`会先去找有没有这个可执行文件，如果不存在则直接返回报错，如果存在的话会去调用`os.process`来做一系列的调度
4. `cmd.Wait()`

- `exec.Start`开始执行c包含的命令，但并不会等待该命令完成即返回。Wait方法会返回命令的返回状态码并在命令返回后释放相关的资源。
- `exec.Wait`,Wait会阻塞直到该命令执行完成，该命令必须是被Start方法开始执行的。
- `exec.Output`执行命令并返回标准输出的切片。


```go

// exec.Run
func (c *Cmd) Run() errorfunc (c *Cmd) Run() error {
	if err := c.Start(); err != nil {
		return err
	}
	return c.Wait()
}

// exec.Start() 准备一系列 启动一个程序需要准备的内容 如进程号
// exec.Wait() 检测一系列准备是否准备完成，完成的会执行返回信息

	//
	cmd := exec.Command("ls","-altr")
	fmt.Printf("%#v",cmd)
	//&exec.Cmd{Path:"/bin/ls", Args:[]string{"ls", "-altr"}, Env:[]string(nil), Dir:"",
	//Stdin:io.Reader(nil), Stdout:io.Writer(nil), Stderr:io.Writer(nil), ExtraFiles:[]*os.File(nil), SysProcAttr:(*syscall.SysProcAttr)(nil),
	//Process:(*os.Process)(nil),
	//ProcessState:(*os.ProcessState)(nil), ctx:context.Context(nil), lookPathErr:error(nil),
	//finished:false, childFiles:[]*os.File(nil), closeAfterStart:[]io.Closer(nil),
	//closeAfterWait:[]io.Closer(nil), goroutine:[]func() error(nil), errch:(chan error)(nil), waitDone:(chan struct {})(nil)}%

	cmd.Run()
	fmt.Printf("%#v",cmd)
	//&exec.Cmd{Path:"/bin/ls", Args:[]string{"ls", "-altr"}, Env:[]string(nil), Dir:"", Stdin:io.Reader(nil),
	//Stdout:io.Writer(nil), Stderr:io.Writer(nil), ExtraFiles:[]*os.File(nil), SysProcAttr:(*syscall.SysProcAttr)(nil),
	//Process:(*os.Process)(0xc00001c090), ProcessState:(*os.ProcessState)(0xc00000c0e0), ctx:context.Context(nil),
	//lookPathErr:error(nil), finished:true, childFiles:[]*os.File{(*os.File)(0xc00000e028), (*os.File)(0xc00000e030),
	//(*os.File)(0xc00000e038)}, closeAfterStart:[]io.Closer{(*os.File)(0xc00000e028), (*os.File)(0xc00000e030), (*os.File)(0xc00000e038)},
	//closeAfterWait:[]io.Closer(nil), goroutine:[]func() error(nil), errch:(chan error)(nil), waitDone:(chan struct {})(nil)}%
```

可以看到有些地址值是在变换的，比如进程号 process

> 返回标准输出和错误输出合并的切片

```go

	 cmd := exec.Command("ls","-altr")
	 res,err := cmd.CombinedOutput()
	 if err != nil{
	 	log.Fatal(err)
	 }
	 fmt.Println(res)

	 cmd = exec.Command("ls","-altr")
	 res = com.Output() // 直接返回输出
	 fmt.Println(res)
```

> 命令执行过程中获取输出

```go

func copyAndCapture(w io.Writer, r io.Reader) ([]byte, error) {
	var out []byte
	buf := make([]byte, 1024, 1024)
	for {
		n, err := r.Read(buf[:])
		if n > 0 {
			d := buf[:n]
			out = append(out, d...)
			os.Stdout.Write(d)
		}
		if err != nil {
			// Read returns io.EOF at the end of file, which is not an error for us
			if err == io.EOF {
				err = nil
			}
			return out, err
		}
	}
	// never reached
	panic(true)
	return nil, nil
}
func main() {
	cmd := exec.Command("ls", "-lah")
	var stdout, stderr []byte
	var errStdout, errStderr error
	stdoutIn, _ := cmd.StdoutPipe()
	stderrIn, _ := cmd.StderrPipe()
	cmd.Start()
	go func() {
		stdout, errStdout = copyAndCapture(os.Stdout, stdoutIn)
	}()
	go func() {
		stderr, errStderr = copyAndCapture(os.Stderr, stderrIn)
	}()
	err := cmd.Wait()
	if err != nil {
		log.Fatalf("cmd.Run() failed with %s\n", err)
	}
	if errStdout != nil || errStderr != nil {
		log.Fatalf("failed to capture stdout or stderr\n")
	}
	outStr, errStr := string(stdout), string(stderr)
	fmt.Printf("\nout:\n%s\nerr:\n%s\n", outStr, errStr)
}
```

```go

func main() {
	var stdoutBuf, stderrBuf bytes.Buffer
	cmd := exec.Command("ls", "-lah")
	stdoutIn, _ := cmd.StdoutPipe()
	stderrIn, _ := cmd.StderrPipe()
	var errStdout, errStderr error
	stdout := io.MultiWriter(os.Stdout, &stdoutBuf)
	stderr := io.MultiWriter(os.Stderr, &stderrBuf)
	err := cmd.Start()
	if err != nil {
		log.Fatalf("cmd.Start() failed with '%s'\n", err)
	}
	go func() {
		_, errStdout = io.Copy(stdout, stdoutIn)
	}()
	go func() {
		_, errStderr = io.Copy(stderr, stderrIn)
	}()
	err = cmd.Wait()
	if err != nil {
		log.Fatalf("cmd.Run() failed with %s\n", err)
	}
	if errStdout != nil || errStderr != nil {
		log.Fatal("failed to capture stdout or stderr\n")
	}
	outStr, errStr := string(stdoutBuf.Bytes()), string(stderrBuf.Bytes())
	fmt.Printf("\nout:\n%s\nerr:\n%s\n", outStr, errStr)
}
```

> 使用管道

```go
package main
import (
    "bytes"
    "io"
    "os"
    "os/exec"
)
func main() {
    c1 := exec.Command("ls")
    c2 := exec.Command("wc", "-l")
    r, w := io.Pipe() 
    c1.Stdout = w
    c2.Stdin = r
    var b2 bytes.Buffer
    c2.Stdout = &b2
    c1.Start()
    c2.Start()
    c1.Wait()
    w.Close()
    c2.Wait()
    io.Copy(os.Stdout, &b2)
}
```
```go
package main
import (
    "os"
    "os/exec"
)
func main() {
    c1 := exec.Command("ls")
    c2 := exec.Command("wc", "-l")
    c2.Stdin, _ = c1.StdoutPipe()
    c2.Stdout = os.Stdout
    _ = c2.Start()
    _ = c1.Run()
    _ = c2.Wait()
}
```
```go
package main
import (
	"fmt"
	"os/exec"
)
func main() {
	cmd := "cat /proc/cpuinfo | egrep '^model name' | uniq | awk '{print substr($0, index($0,$4))}'"
	out, err := exec.Command("bash", "-c", cmd).Output()
if err != nil {
		fmt.Printf("Failed to execute command: %s", cmd)
	}
	fmt.Println(string(out))
}
```