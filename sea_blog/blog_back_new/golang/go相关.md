---
title: go相关
date: 2021-03-01 16:30:57
categories:  [golang]
tags: [golang]
---


<!--more-->


# go相关

## golang单项目多个包运行时报错
golang中在同包下正产是可以使用所有.go文件中的func的  没有需要导包的条件
但是现在的问题是在iterm等第三方终端 无法运行，
解决方法
1. 编译运行  go build .
2.  运行所有go文件 go run *.go  会自动去找main.go 作为入口文件运行

## go文件写入的注意点

- 用带缓存的方法写入,必须要刷新
- 不带缓存的方法写入，不用刷新
- 按照格式写入

基础代码
```go
package main

import (
	"fmt"
	"log"
	"os"
	"path/filepath"
	"strconv"
)

func main() {
	currPath, err := os.Getwd()
	if err!= nil{
		log.Fatal(err)
	}
	backUpDir := filepath.Join(currPath+"/Backup/")
	_, err = os.Stat(backUpDir)
	if err != nil{
		os.Mkdir(backUpDir,0777)
	}
	for i:=0;i<10;i++{
        w,err := os.OpenFile(backUpDir+"/"+strconv.Itoa(i)+".txt",os.O_APPEND|os.O_RDWR|os.O_CREATE,0666)
        if err != nil{
            log.Fatal(err)
        }
        defer w.Close()
		for j:=0;j<10000;j++{
			if err != nil{
				log.Fatal(err)
			}
			//buf := bufio.NewWriter(w)
			//buf.WriteString(strconv.Itoa(j))
			//buf.Flush()   // 带缓存的写入 需要flush刷新
            fmt.Fprint(w,strconv.Itoa(j)) // 写入
            fmt.Fprintf(w,"%s\n",strconv.Itoa(j)) // 带格式的写入
		}
	}

}


func DeleteDir(p string){
	status,err:= os.Stat(p)
	if err != nil{
		log.Fatal("删除失败， 文件不存在")
	}
	if status.IsDir(){
		os.RemoveAll(p)
	}
}

```

上面的代码如果带缓存的写入 但是不刷新的话， 文件夹会空


## 结构体被嵌套接口
之前看了snmp这个库,snmp里面这个struct调用了conn interface
```golang
type GoSNMP struct {
	mu sync.Mutex

	// Conn is net connection to use, typically established using GoSNMP.Connect().
	Conn net.Conn
	...
}
```
想要看下这个内容
```golang
type Conn interface {
	// Read reads data from the connection.
	// Read can be made to time out and return an Error with Timeout() == true
	// after a fixed time limit; see SetDeadline and SetReadDeadline.
	Read(b []byte) (n int, err error)

	// Write writes data to the connection.
	// Write can be made to time out and return an Error with Timeout() == true
	// after a fixed time limit; see SetDeadline and SetWriteDeadline.
	Write(b []byte) (n int, err error)

	// Close closes the connection.
	// Any blocked Read or Write operations will be unblocked and return errors.
	Close() error

	// LocalAddr returns the local network address.
	LocalAddr() Addr

	// RemoteAddr returns the remote network address.
	RemoteAddr() Addr

	// SetDeadline sets the read and write deadlines associated
	// with the connection. It is equivalent to calling both
	// SetReadDeadline and SetWriteDeadline.
	//
	// A deadline is an absolute time after which I/O operations
	// fail with a timeout (see type Error) instead of
	// blocking. The deadline applies to all future and pending
	// I/O, not just the immediately following call to Read or
	// Write. After a deadline has been exceeded, the connection
	// can be refreshed by setting a deadline in the future.
	//
	// An idle timeout can be implemented by repeatedly extending
	// the deadline after successful Read or Write calls.
	//
	// A zero value for t means I/O operations will not time out.
	//
	// Note that if a TCP connection has keep-alive turned on,
	// which is the default unless overridden by Dialer.KeepAlive
	// or ListenConfig.KeepAlive, then a keep-alive failure may
	// also return a timeout error. On Unix systems a keep-alive
	// failure on I/O can be detected using
	// errors.Is(err, syscall.ETIMEDOUT).
	SetDeadline(t time.Time) error

	// SetReadDeadline sets the deadline for future Read calls
	// and any currently-blocked Read call.
	// A zero value for t means Read will not time out.
	SetReadDeadline(t time.Time) error

	// SetWriteDeadline sets the deadline for future Write calls
	// and any currently-blocked Write call.
	// Even if write times out, it may return n > 0, indicating that
	// some of the data was successfully written.
	// A zero value for t means Write will not time out.
	SetWriteDeadline(t time.Time) error
}
```
这个是Connet的inteface  然后再库的下面创建了一个conn的结构体来实现上面的接口中的方法
```golang
type conn struct {
	fd *netFD
}

func (c *conn) ok() bool { return c != nil && c.fd != nil }

// Implementation of the Conn interface.

// Read implements the Conn Read method.
func (c *conn) Read(b []byte) (int, error) {
	if !c.ok() {
		return 0, syscall.EINVAL
	}
	n, err := c.fd.Read(b)
	if err != nil && err != io.EOF {
		err = &OpError{Op: "read", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return n, err
}

// Write implements the Conn Write method.
func (c *conn) Write(b []byte) (int, error) {
	if !c.ok() {
		return 0, syscall.EINVAL
	}
	n, err := c.fd.Write(b)
	if err != nil {
		err = &OpError{Op: "write", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return n, err
}

// Close closes the connection.
func (c *conn) Close() error {
	if !c.ok() {
		return syscall.EINVAL
	}
	err := c.fd.Close()
	if err != nil {
		err = &OpError{Op: "close", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return err
}

// LocalAddr returns the local network address.
// The Addr returned is shared by all invocations of LocalAddr, so
// do not modify it.
func (c *conn) LocalAddr() Addr {
	if !c.ok() {
		return nil
	}
	return c.fd.laddr
}

// RemoteAddr returns the remote network address.
// The Addr returned is shared by all invocations of RemoteAddr, so
// do not modify it.
func (c *conn) RemoteAddr() Addr {
	if !c.ok() {
		return nil
	}
	return c.fd.raddr
}

// SetDeadline implements the Conn SetDeadline method.
func (c *conn) SetDeadline(t time.Time) error {
	if !c.ok() {
		return syscall.EINVAL
	}
	if err := c.fd.SetDeadline(t); err != nil {
		return &OpError{Op: "set", Net: c.fd.net, Source: nil, Addr: c.fd.laddr, Err: err}
	}
	return nil
}

// SetReadDeadline implements the Conn SetReadDeadline method.
func (c *conn) SetReadDeadline(t time.Time) error {
	if !c.ok() {
		return syscall.EINVAL
	}
	if err := c.fd.SetReadDeadline(t); err != nil {
		return &OpError{Op: "set", Net: c.fd.net, Source: nil, Addr: c.fd.laddr, Err: err}
	}
	return nil
}

// SetWriteDeadline implements the Conn SetWriteDeadline method.
func (c *conn) SetWriteDeadline(t time.Time) error {
	if !c.ok() {
		return syscall.EINVAL
	}
	if err := c.fd.SetWriteDeadline(t); err != nil {
		return &OpError{Op: "set", Net: c.fd.net, Source: nil, Addr: c.fd.laddr, Err: err}
	}
	return nil
}

// SetReadBuffer sets the size of the operating system's
// receive buffer associated with the connection.
func (c *conn) SetReadBuffer(bytes int) error {
	if !c.ok() {
		return syscall.EINVAL
	}
	if err := setReadBuffer(c.fd, bytes); err != nil {
		return &OpError{Op: "set", Net: c.fd.net, Source: nil, Addr: c.fd.laddr, Err: err}
	}
	return nil
}

// SetWriteBuffer sets the size of the operating system's
// transmit buffer associated with the connection.
func (c *conn) SetWriteBuffer(bytes int) error {
	if !c.ok() {
		return syscall.EINVAL
	}
	if err := setWriteBuffer(c.fd, bytes); err != nil {
		return &OpError{Op: "set", Net: c.fd.net, Source: nil, Addr: c.fd.laddr, Err: err}
	}
	return nil
}

// File returns a copy of the underlying os.File.
// It is the caller's responsibility to close f when finished.
// Closing c does not affect f, and closing f does not affect c.
//
// The returned os.File's file descriptor is different from the connection's.
// Attempting to change properties of the original using this duplicate
// may or may not have the desired effect.
func (c *conn) File() (f *os.File, err error) {
	f, err = c.fd.dup()
	if err != nil {
		err = &OpError{Op: "file", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return
}
```

我一直在想snmp 中的 Conn也没去实现对应的方法呀，怎么能够使用里面的内容了，后来看下下文，其实这个Conn在初始化的时候并不会使用到，
在第一个connet的时候回去调用netConnet方法， 返回的对象中实现了Connect
```golang
func (x *GoSNMP) Connect() error {
	return x.connect("")

func (x *GoSNMP) netConnect() error {
	var err error
	addr := net.JoinHostPort(x.Target, strconv.Itoa(int(x.Port)))

	switch transport := x.Transport; transport {
	case "udp", "udp4", "udp6":
		if x.UseUnconnectedUDPSocket {
			x.uaddr, err = net.ResolveUDPAddr(transport, addr)
			if err != nil {
				return err
			}

			// As far as I know, this should not be needed in production but only to
			// work around tests: in tests we are opening fake destination with ends up
			// being ipv4:0.0.0.0. You can't send packets from :: to 0.0.0.0.
			if addr4 := x.uaddr.IP.To4(); addr4 != nil {
				x.uaddr.IP = addr4
				transport = "udp4"
			}
			x.Conn, err = net.ListenUDP(transport, nil)
			return err
		}
	}
	dialer := net.Dialer{Timeout: x.Timeout}
	x.Conn, err = dialer.DialContext(x.Context, x.Transport, addr)   // 注意看这里 对x.Conn赋值了
	
	return err
}
```