---
title: go_package_io
date: 2021-06-17 14:36:27
categories:  [golang,go_package]
tags: [golang,go_package]
---


<!--more-->


# go_package_io

io标准库地址 https://golang.org/pkg/io/
io包为`I/O`原语提供了基本的接口。它主要包装了这些原语的已有实现。
在io包中最重要的是两个接口:`Reader`和`Writer`接口，以及一个输入流结尾`io.EOF` , 所以后续更高级的接口，都围绕这三个接口进行扩展，具体的扩展图如下

![2020-10-23-17-11-28](http://noback.upyun.com/2020-10-23-17-11-28.png)

<div style='font-size:20px;color:red'>但是io所提供的接口是无法提供缓存的，只提供了最底层的接口, 无缓存的为`io.reader`或者`io.writer` 有缓存的则是`bufio.reader`,`bufio.writer`</div>

## 无缓存读写

### Reader接口
```go
type Reader interface {
    Read(p []byte) (n int, err error)
    // 解析，将内容读取到p数组，返回n个数量的字节，以及err错误信息
    // 错误信息中 如果是因为读取完了 则返回的是io.EOF错误
    // 返回的是[] EOF
}
```
接受一个byte数组 返回n字节以及err ,没有错误时，err为nil


> Read 将 len(p) 个字节读取到 p 中。它返回读取的字节数 n（0 <= n <= len(p)） 以及任何遇到的错误。即使 Read 返回的 n < len(p)，它也会在调用过程中占用 len(p) 个字节作为暂存空间。若可读取的数据不到 len(p) 个字节，Read 会返回可用数据，而不是等待更多数据。
当 Read 在成功读取 n > 0 个字节后遇到一个错误或 EOF (end-of-file)，它会返回读取的字节数。它可能会同时在本次的调用中返回一个non-nil错误,或在下一次的调用中返回这个错误（且 n 为 0）。 一般情况下, Reader会返回一个非0字节数n, 若 n = len(p) 个字节从输入源的结尾处由 Read 返回，Read可能返回 err == EOF 或者 err == nil。并且之后的 Read() 都应该返回 (n:0, err:EOF)。
调用者在考虑错误之前应当首先处理返回的数据。这样做可以正确地处理在读取一些字节后产生的 I/O 错误，同时允许EOF的出现。

这里Reader接口中只有一个Read的方法函数，所有实现了 `Read` 方法的类型都满足 `io.Reader` 接口，也就是说，在所有需要`io.Reader`的地方，可以传递实现了`Read()`方法的类型的实例

### ReadFrom方法
https://golang.org/pkg/io/
ReadFrom函数接收一个io.Reader的接口类型对象，和一个num的读取的字节数
```go
func ReadFrom(reader io.Reader, num int) ([]byte, error) {
    p := make([]byte, num)
    n, err := reader.Read(p)
    if n > 0 {
        return p[:n], nil
    }
    return p, err
}
```

从io.Reader接口类型中读取内容，他们都实现了Read方法，不信的话我们可以来看看
```go
// 从标准输入中读取
data,err := ReadFrom(os.Stdin, 11)
// 从文件中读取
data,err := ReadFrom(file,9)
// 从字符串中读取
data,err := ReadFrom(strings.NewReader("from string"), 12)
```

这边看`os.Stdin`这个函数
```golang
// 实质是去创建一个/dev/stdin的临时文件
var Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
// 这个file呢是实现了read方法的
func NewFile(fd uintptr, name string) *File {
	kind := kindNewFile
	if nb, err := unix.IsNonblock(int(fd)); err == nil && nb {
		kind = kindNonBlock
	}
	return newFile(fd, name, kind)
}
type File struct {
	*file // os specific
}
func (f *File) read(b []byte) (n int, err error) {
	n, err = f.pfd.Read(b)
	runtime.KeepAlive(f)
	return n, err
}
```


### Writer 接口
```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```
获取byte数组，返回n个字节和err=nil

> Write 将 `len(p)` 个字节从 p 中写入到基本数据流中。它返回从 p 中被写入的字节数 `n(0 <= n <= len(p))`以及任何遇到的引起写入提前停止的错误。若 Write 返回的 `n < len(p)`，它就必须返回一个 `非nil` 的错误。



### Fprintln方法
我们常用的`fmt.println`的方法如下
```go
func Println(a ...interface{}) (n int, err error) {
    return Fprintln(os.Stdout, a...)
}
```
其中Fprintln的实现如下
```go
func Fprintln(w io.Writer, a ...interface{}) (n int, err error) {
	p := newPrinter()
	p.doPrintln(a)
	n, err = w.Write(p.buf) // 实现了Writer接口的唯一方法 Write 也就是实现了io.Writer接口
	p.free()
	return
}
```
这里可以看出Fprintln是实现了`io.Writer`接口的 ,并且他接收多个参数，返回n个字节以及一个error




###  ReaderAt 和 WriterAt 接口
来看下他的源码
```go
// ReaderAt
type ReaderAt interface {
    ReadAt(p []byte, off int64) (n int, err error)
}

// WriterAt
type WriterAt interface {
    WriteAt(p []byte, off int64) (n int, err error)
}
```
他们比之前的就多了一个off  而这个off就是偏移量,主要作用是将`ReadAt`从`基本输入源`的偏移量 `off` 处开始，将 `len(p)` 个字节读取到 p 中。
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// 创建一个输入源
	var qq = strings.NewReader("helloworld")
	// 创建一个输入源
	var list = make([]byte,20)
	n,err := qq.ReadAt(list, 2)
	if err == nil{
		fmt.Println(n)
	}
	fmt.Printf("%s, %d\n", list, n)
}

// result 
// lloworld, 8
```
`WriteAt` 从 `p` 中将 `len(p)` 个字节写入到`偏移量` off 处的`基本输出源`中
```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	// 创建一个本文
	file,err := os.Create("writeAt.txt")
	if err !=nil{
		panic(err)
	}
	defer file.Close()
	file.WriteString("Golang中文社区——这里是多余")
	n, err = file.WriteAt([]byte("Go语言中文网"), 24)
	if err != nil {
		panic(err)
	}
	fmt.Println(n)
}
```


###  ReaderFrom 和 WriterTo 接口
先看下源码
```go
type ReaderFrom interface {
    ReadFrom(r Reader) (n int64, err error)
}
```
接口说明
ReadFrom 从 `r` 中读取数据，直到 `EOF` 或`发生错误`。其返回值n为读取的字节数。除`io.EOF`之外，在读取过程中遇到的任何错误也将被返回。




## 有缓存的读写

有缓存的读写经常出现在一下集中情况当中

- net.Conn: 网络
- os.Stdin, os.Stdout, os.Stderr: console终端标准输出,err
- os.File: 网络,标准输入输出,文件的流读取
- strings.Reader: 把字符串抽象成Reader
- bytes.Reader: 把[]byte抽象成Reader
- bytes.Buffer: 把[]byte抽象成Reader和Writer
- bufio.Reader/Writer: 抽象成带缓冲的流读取（比如按行读写）

特点，Unix 下有一切皆文件的思想,Golang 把这个思想贯彻到更远,因为本质上我们对文件的抽象就是一个可读可写的一个对象, 也就是实现了io.Writer 和 io.Reader 的对象我们都可以称为文件,


### strings  Reader
strings这个库里面reader 是io.reader的扩展，结构体如下
但strings是没有实现一个writer的

```go
type Reader struct {
	s        string  // 内容
	i        int64 // 游标，记录当前位置
	prevRune int   // 中文字符的游标
}
```

他实现了  Read 这个函数,也就实现了io.read

```go
func (r *Reader) Read(b []byte) (n int, err error) {
	if r.i >= int64(len(r.s)) {
		return 0, io.EOF
	}
	r.prevRune = -1
	n = copy(b, r.s[r.i:])
	r.i += int64(n)
	return
}
```

另外实现了一个writeTo方法，将内容写入到支持写入的对象当中去
```go
func (r *Reader) WriteTo(w io.Writer) (n int64, err error) {
	r.prevRune = -1
	if r.i >= int64(len(r.s)) {
		return 0, nil
	}
	s := r.s[r.i:]
	m, err := io.WriteString(w, s)
	if m > len(s) {
		panic("strings.Reader.WriteTo: invalid WriteString count")
	}
	r.i += int64(m)
	n = int64(m)
	if m != len(s) && err == nil {
		err = io.ErrShortWrite
	}
	return
}
```

### 外加方法

`strings.reader`继承自`io.reader`, 并附加了很多其他的方法，官方库如下

```go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package strings

import (
	"errors"
	"io"
	"unicode/utf8"
)

// A Reader implements the io.Reader, io.ReaderAt, io.Seeker, io.WriterTo,
// io.ByteScanner, and io.RuneScanner interfaces by reading
// from a string.
// The zero value for Reader operates like a Reader of an empty string.
type Reader struct {
	s        string
	i        int64 // current reading index
	prevRune int   // index of previous rune; or < 0
}

// Len returns the number of bytes of the unread portion of the
// string.
func (r *Reader) Len() int {
	if r.i >= int64(len(r.s)) {
		return 0
	}
	return int(int64(len(r.s)) - r.i)
}

// Size returns the original length of the underlying string.
// Size is the number of bytes available for reading via ReadAt.
// The returned value is always the same and is not affected by calls
// to any other method.
func (r *Reader) Size() int64 { return int64(len(r.s)) }

func (r *Reader) Read(b []byte) (n int, err error) {
	if r.i >= int64(len(r.s)) {
		return 0, io.EOF
	}
	r.prevRune = -1
	n = copy(b, r.s[r.i:])
	r.i += int64(n)
	return
}

func (r *Reader) ReadAt(b []byte, off int64) (n int, err error) {
	// cannot modify state - see io.ReaderAt
	if off < 0 {
		return 0, errors.New("strings.Reader.ReadAt: negative offset")
	}
	if off >= int64(len(r.s)) {
		return 0, io.EOF
	}
	n = copy(b, r.s[off:])
	if n < len(b) {
		err = io.EOF
	}
	return
}

func (r *Reader) ReadByte() (byte, error) {
	r.prevRune = -1
	if r.i >= int64(len(r.s)) {
		return 0, io.EOF
	}
	b := r.s[r.i]
	r.i++
	return b, nil
}

func (r *Reader) UnreadByte() error {
	if r.i <= 0 {
		return errors.New("strings.Reader.UnreadByte: at beginning of string")
	}
	r.prevRune = -1
	r.i--
	return nil
}

func (r *Reader) ReadRune() (ch rune, size int, err error) {
	if r.i >= int64(len(r.s)) {
		r.prevRune = -1
		return 0, 0, io.EOF
	}
	r.prevRune = int(r.i)
	if c := r.s[r.i]; c < utf8.RuneSelf {
		r.i++
		return rune(c), 1, nil
	}
	ch, size = utf8.DecodeRuneInString(r.s[r.i:])
	r.i += int64(size)
	return
}

func (r *Reader) UnreadRune() error {
	if r.i <= 0 {
		return errors.New("strings.Reader.UnreadRune: at beginning of string")
	}
	if r.prevRune < 0 {
		return errors.New("strings.Reader.UnreadRune: previous operation was not ReadRune")
	}
	r.i = int64(r.prevRune)
	r.prevRune = -1
	return nil
}

// Seek implements the io.Seeker interface.
func (r *Reader) Seek(offset int64, whence int) (int64, error) {
	r.prevRune = -1
	var abs int64
	switch whence {
	case io.SeekStart:
		abs = offset
	case io.SeekCurrent:
		abs = r.i + offset
	case io.SeekEnd:
		abs = int64(len(r.s)) + offset
	default:
		return 0, errors.New("strings.Reader.Seek: invalid whence")
	}
	if abs < 0 {
		return 0, errors.New("strings.Reader.Seek: negative position")
	}
	r.i = abs
	return abs, nil
}

// WriteTo implements the io.WriterTo interface.
func (r *Reader) WriteTo(w io.Writer) (n int64, err error) {
	r.prevRune = -1
	if r.i >= int64(len(r.s)) {
		return 0, nil
	}
	s := r.s[r.i:]
	m, err := io.WriteString(w, s)
	if m > len(s) {
		panic("strings.Reader.WriteTo: invalid WriteString count")
	}
	r.i += int64(m)
	n = int64(m)
	if m != len(s) && err == nil {
		err = io.ErrShortWrite
	}
	return
}

// Reset resets the Reader to be reading from s.
func (r *Reader) Reset(s string) { *r = Reader{s, 0, -1} }

// NewReader returns a new Reader reading from s.
// It is similar to bytes.NewBufferString but more efficient and read-only.
func NewReader(s string) *Reader { return &Reader{s, 0, -1} }

```



> 使用实例

```go
    // 一个可读对象
    reader_no_buffer := strings.NewReader("helloworld"); 
    
    // 一个缓存通道，长度为1
    p := make([]byte,1)
    // 开一个循环，将内存通过通道p读取
	for  {
		// 将内容写入到p数组
		n,err := reader_no_buffer.Read(p)
		if err != nil{
			if err == io.EOF{
				fmt.Println("读取完了")
				break
			}else {
				fmt.Println("异常错误")
				panic(err)
			}
		}
		fmt.Printf("一共读取到%d个字符,内容为%s\n",n,string(p[:n]))
	}
// result 
// 一共读取到1个字符,内容为h
// 一共读取到1个字符,内容为e
// 一共读取到1个字符,内容为l
// 一共读取到1个字符,内容为l
// 一共读取到1个字符,内容为o
// 一共读取到1个字符,内容为w
// 一共读取到1个字符,内容为o
// 一共读取到1个字符,内容为r
// 一共读取到1个字符,内容为l
// 一共读取到1个字符,内容为d
// 读取完了




    // 将内容写入到缓冲区
    // 一个可读取对象
    reader_no_buffer := strings.NewReader("helloworld"); 
    
    // 读取缓存
    var writer bytes.Buffer
    
    // 将内存读取到缓存当中
	reader_no_buffer.WriteTo(&writer)
	fmt.Println(writer.String()) // helloworld

	var xx string  = "helloworld"
	fmt.Println(len(xx))  //10
	reader_no_buffer := strings.NewReader(xx);

	// Size 返回底层字符串的原始长度
	n := reader_no_buffer.Size()
	fmt.Println("底层字符串长度为%d",n) //10

	// 重置读取缓存区的内容
	reader_no_buffer.Reset("abcd")
	fmt.Println(reader_no_buffer.Size()) // 4
	// 读取字节数
	for  {
		bb,err := reader_no_buffer.ReadByte()
		if err != nil {
			if err == io.EOF {
				fmt.Println("读取结束")
				break
			} else {
				fmt.Println("异常错误")
			}
		}
		fmt.Printf("当前读到的内容为 %s\n",string(bb))
	}
	// 返回未读部分的字节数
	reader_no_buffer.Reset("helloworld") // 10
	reader_no_buffer.Read(make([]byte,1)) // 10 - 1 = 9
	fmt.Println(reader_no_buffer.Len()) // 9

	// 指针偏移距离
	reader_no_buffer.Reset("hellowrold")
	reader_no_buffer.Seek(2,1)// 从第一个位置开始偏移两个位置
	for  {
		bb,err := reader_no_buffer.ReadByte()
		if err != nil{
			break
		}
		fmt.Printf("%s",string(bb)) // llowrold
	}
```



### bytes Reader

跟strings Reader一样,自己也构造一个Reader,并且实现了io.read
```go
type Reader struct {
	s        []byte
	i        int64 // current reading index
	prevRune int   // index of previous rune; or < 0
}

func (r *Reader) Read(b []byte) (n int, err error) {
	if r.i >= int64(len(r.s)) {
		return 0, io.EOF
	}
	r.prevRune = -1
	n = copy(b, r.s[r.i:])
	r.i += int64(n)
	return
}

func NewReader(b []byte) *Reader { return &Reader{b, 0, -1} }
```


### bytes Buffer
但他同时也构造了一个缓存 buffer 支持读写,都分别实现了io.read 以及 io.write
```go

type Buffer struct {
	buf      []byte // contents are the bytes buf[off : len(buf)]
	off      int    // read at &buf[off], write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly.
}

func (b *Buffer) Read(p []byte) (n int, err error) {
	b.lastRead = opInvalid
	if b.empty() {
		// Buffer is empty, reset to recover space.
		b.Reset()
		if len(p) == 0 {
			return 0, nil
		}
		return 0, io.EOF
	}
	n = copy(p, b.buf[b.off:])
	b.off += n
	if n > 0 {
		b.lastRead = opRead
	}
	return n, nil
}

func (b *Buffer) Write(p []byte) (n int, err error) {
	b.lastRead = opInvalid
	m, ok := b.tryGrowByReslice(len(p))
	if !ok {
		m = b.grow(len(p))
	}
	return copy(b.buf[m:], p), nil
}

```
> 使用示例如下
```go
	var writer bytes.Buffer
	// 写入内容
	writer.Write([]byte{1,2,3,45})
	// 转换成string类型
	fmt.Println(writer.String())
	// 按照字节读取
	for {
		bb,err := writer.ReadByte()
		if err != nil {
			break
		}
		fmt.Println(string(bb))
	}
	// 重置
	fmt.Println(writer.Len())
	writer.Reset()
	//  查看未读取的内容
	fmt.Println(writer.Len())
```

### bufio Reader

同上面两个结构体一样，单独实现了一个Reader和一个Writer, 并且实现了`io.write` 和 `io.read`

Reader实例
```go
type Reader struct {
    buf          []byte        // 缓存
    rd           io.Reader    // 底层的io.Reader
    // r:从buf中读走的字节（偏移）；w:buf中填充内容的偏移；
    // w - r 是buf中可被读的长度（缓存数据的大小），也是Buffered()方法的返回值
    r, w         int
    err          error        // 读过程中遇到的错误
    lastByte     int        // 最后一次读到的字节（ReadByte/UnreadByte)
    lastRuneSize int        // 最后一次读到的Rune的大小 (ReadRune/UnreadRune)
}


func (b *Reader) Read(p []byte) (n int, err error) {
	n = len(p)
	if n == 0 {
		if b.Buffered() > 0 {
			return 0, nil
		}
		return 0, b.readErr()
	}
	if b.r == b.w {
		if b.err != nil {
			return 0, b.readErr()
		}
		if len(p) >= len(b.buf) {
			// Large read, empty buffer.
			// Read directly into p to avoid copy.
			n, b.err = b.rd.Read(p)
			if n < 0 {
				panic(errNegativeRead)
			}
			if n > 0 {
				b.lastByte = int(p[n-1])
				b.lastRuneSize = -1
			}
			return n, b.readErr()
		}
		// One read.
		// Do not use b.fill, which will loop.
		b.r = 0
		b.w = 0
		n, b.err = b.rd.Read(b.buf)
		if n < 0 {
			panic(errNegativeRead)
		}
		if n == 0 {
			return 0, b.readErr()
		}
		b.w += n
	}

	// copy as much as we can
	n = copy(p, b.buf[b.r:b.w])
	b.r += n
	b.lastByte = int(b.buf[b.r-1])
	b.lastRuneSize = -1
	return n, nil
}


func (b *Reader) WriteTo(w io.Writer) (n int64, err error) {
	n, err = b.writeBuf(w)
	if err != nil {
		return
	}

	if r, ok := b.rd.(io.WriterTo); ok {
		m, err := r.WriteTo(w)
		n += m
		return n, err
	}

	if w, ok := w.(io.ReaderFrom); ok {
		m, err := w.ReadFrom(b.rd)
		n += m
		return n, err
	}

	if b.w-b.r < len(b.buf) {
		b.fill() // buffer not full
	}

	for b.r < b.w {
		// b.r < b.w => buffer is not empty
		m, err := b.writeBuf(w)
		n += m
		if err != nil {
			return n, err
		}
		b.fill() // buffer is empty
	}

	if b.err == io.EOF {
		b.err = nil
	}

	return n, b.readErr()
}

```

<div style='font-size:20px;color:red'>
但这里要注意的是 bytes 和 strings 都是接收一个strings 或者一个切片转换成一个reader ，但是bufio 是将`io.read`的类型转换成bufio.reader
</div>

```go
// bytes
func NewBuffer(buf []byte) *Buffer { return &Buffer{buf: buf} }
func NewReader(b []byte) *Reader { return &Reader{b, 0, -1} }
// strings
func NewReader(s string) *Reader { return &Reader{s, 0, -1} }
// bufio
func NewWriter(w io.Writer) *Writer {
	return NewWriterSize(w, defaultBufSize)
}
func NewReader(rd io.Reader) *Reader {
	return NewReaderSize(rd, defaultBufSize)
}

```

> 使用示例如下

```go
// 实现内容，将reader 中的东西读取到数据P到中去
	reader := bufio.NewReader(strings.NewReader("hello"))
	p:=make([]byte,4)
	for  {
		m,err := reader.Read(p)
		if err!=nil {
			if err == io.EOF {
				fmt.Println("读取结束了")
				break
			} else {
				fmt.Println("异常退出")
				os.Exit(1)
			}
		}
		fmt.Printf("读取到的字节为%d,字节内容为 %s\n",m,string(p[:m]))

		//读取到的字节为4,字节内容为 hell
		//读取到的字节为1,字节内容为 o
		//读取结束了
	}
```


## ioutil
io库作为一个底层库，由于太底层，实现的功能接口很少,比如要读取一个文件中的内容，io包就很难用，后面官方出了一个ioutil的子库

主要是在io库上做了一层新的封装,封装出更加实用的方法,`主要体现在对文件测读写操作上`
https://studygolang.com/static/pkgdoc/pkg/io_ioutil.htm

- Discard
- ReadAll
- ReadFile
- Writefile
- ReadDir
- TempDir
- TempFile
  

Discard是一个io.Writer接口，对它的所有Write调用都会无实际操作的成功返回。相当于linux中的/dev/null吧 所有东西都丢到黑洞里面去
```bash
var Discard io.Writer = devNull(0)
```

### ReadAll 
```bash
func ReadAll(r io.Reader) ([]byte, error)
```
ReadAll从r读取数据直到EOF或遇到error，返回读取的数据和遇到的错误。成功的调用返回的err为nil而非EOF。因为本函数定义为读取r直到EOF，它不会将读取返回的EOF视为应报告的错误。

### ReadFile 
```bash
func ReadFile(filename string) ([]byte, error)
```
ReadFile 从filename指定的文件中读取数据并返回文件的内容。成功的调用返回的err为nil而非EOF。因为本函数定义为读取整个文件，它不会将读取返回的EOF视为应报告的错误。

###  WriteFile
```bash
func WriteFile(filename string, data []byte, perm os.FileMode) error
```
函数向filename指定的文件中写入数据。如果文件不存在将按给出的权限创建文件，否则在写入数据之前清空文件。

### ReadDir
```bash
func ReadDir(dirname string) ([]os.FileInfo, error)
```
返回dirname指定的目录的目录信息的有序列表。

### TempDir
```bash
func TempDir(dir, prefix string) (name string, err error)
```
在dir目录里创建一个新的、使用prfix作为前缀的临时文件夹，并返回文件夹的路径。如果dir是空字符串，TempDir使用默认用于临时文件的目录（参见os.TempDir函数）。 不同程序同时调用该函数会创建不同的临时目录，调用本函数的程序有责任在不需要临时文件夹时摧毁它。

### TempFile
```bash
func TempFile(dir, prefix string) (f *os.File, err error)
```
在dir目录下创建一个新的、使用prefix为前缀的临时文件，以读写模式打开该文件并返回os.File指针。如果dir是空字符串，TempFile使用默认用于临时文件的目录（参见os.TempDir函数）。不同程序同时调用该函数会创建不同的临时文件，调用本函数的程序有责任在不需要临时文件时摧毁它。

## package_os
```go
package main

import (
	"fmt"
	"io/ioutil"
	"os"
)

func main() {

	// 获取命令行变量 数组中第一个一定是程序名
	fmt.Println(os.Args) // [/var/folders/1s/j9mtgsg14jv_xdqhrt1j42l80000gn/T/go-build302297154/b001/exe/testos xx]
	// 返回hostname,error
	fmt.Println(os.Hostname())
	// 获取进程
	fmt.Println(os.Getpid())

	// 获取所有环境变量
	env := os.Environ()
	for k,v := range env{
		fmt.Println(k,"---",v)
	}


	// 获取一条环境变量
	fmt.Println(os.Getenv("PATH"))
	// 获取当前目录
	fmt.Println(os.Getwd())

	// 创建目录
	err := os.Mkdir("./hello/",os.ModePerm) // 0777
	if err != nil{
		fmt.Println(err)
	}
	// 递归创建目录
	err = os.MkdirAll("./hello/world/test",os.ModePerm)
	if err !=nil{
		fmt.Println(err)
	}

	// 创建临时目录
	tmp_dir := os.TempDir()
	fmt.Println(tmp_dir)


	// 创建文件
	file,_ := os.Create("hello.txt")
	// 显示文件名字
	fmt.Println(file.Name())
	// 显示文件信息
	fmt.Println(file.Stat())
	// 打开文件用于读取
	gwd,_:= os.Getwd()
	file,err=os.Open(gwd+"/xx/xx.go")
	if err !=nil{
		fmt.Println(err)
	}else {
		fmt.Println(file.Name())
	}

	// 递归获取目录信息
	file_info_list,err := ioutil.ReadDir(gwd+"/xx/")
	if err != nil{
		fmt.Println(err)
	}else {
		for i:=0;i<len(file_info_list);i++ {
			fmt.Println(file_info_list[i].Name())
		}
	}

	// 从file中读取最多len(b)字节数据并写入b
	file,err=os.Open(gwd+"/xx/xx.go")
	var bb []byte = make([]byte,32)
	if err !=nil {
		fmt.Println(err)
	}else {
		file.Read(bb)
	}
	fmt.Println(string(bb))

	// 向文件中写入字符串
	filepath,err := os.Getwd() // 获取当前文件地址
	if  err != nil{
		fmt.Println(err)
	}
	filepath = filepath+"/xx/helloworld.txt" // 拼接文件地址
	new_file,_ := os.Create(filepath) // 创建新的文件
	fmt.Println(new_file.Name()) // 查看创建文件夹的名字
	n,err := new_file.WriteString("hellowrold");
	if err != nil{
		fmt.Println(err)
	}else {
		fmt.Println(n)
	}

	// 关闭当前文件，使文件不能用于读写
	file.Close()
	new_file.Close()
	// 一般都滞后关闭，也就是程序结束之前关闭
	//defer file.Close()


	// 终止当前程序
	var i int = 1
	for {
		if i == 20 {
			os.Exit(1)
		}
		i++
		fmt.Println(i)
	}

}
```


## golang文件信息
os包里面可以用stat来查看某一个文件的文件信息，或者使用 ioutil.ReadDir来递归批量获取文件夹下面所有文件的信息

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 获取文件地址
	filepath,err := os.Getwd();
	if err != nil {
		fmt.Println(err)
	}else {
		filepath = filepath+"/xx/"
	}

	// 查看文件信息
	fileinfo := filepath+"helloworld.txt"
	file_info,err := os.Stat(fileinfo)
	if err != nil{
		fmt.Println(err)
	}else {
		fmt.Println(file_info)
	}

	// 文件名字
	fmt.Println(file_info.Name())
	// 文件大小
	fmt.Println(file_info.Size())
	// 文件权限
	fmt.Println(file_info.Mode())
	//文件创建时间
	fmt.Println(file_info.ModTime())
	// 文件是否是文件夹
	fmt.Println(file_info.IsDir())
	// 基础数据源
	fmt.Println(file_info.Sys())
}

```

```go
func main() {
	// 获取文件地址
	filepath,err := os.Getwd();
	if err != nil {
		fmt.Println(err)
	}else {
		filepath = filepath+"/xx/"
	}

	//
	fileinfoList,err := ioutil.ReadDir(filepath)
	if err != nil{
		panic(err)
	}else {
		for i:=0;i<len(fileinfoList);i++{
			fmt.Println(fileinfoList[i].Name())
		}
	}
}

```

