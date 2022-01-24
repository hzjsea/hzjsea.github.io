---
title: golang标准库log
date: 2021-01-29 12:24:00
categories:  [golang]
tags: [golang,golibrary]
---


<!--more-->


# golang标准库log
日志是分析项目流程以及是否完善的主要工具之一，对于可能错误或需要提示的地方进行日志打印，以便进行分析

golang中自带的日志引擎组成部分如下
```golang
type Logger struct {
	mu     sync.Mutex // ensures atomic writes; protects the following fields // 高并发锁
	prefix string     // 日志的头部 前缀，可以用来表示app 【mail server】
	flag   int        // 日志输出的形式，go中定义了7中形式 
	out    io.Writer  // 日志输出地址，var buf bytes.Buffer
	buf    []byte     // 用于累积要写入的文本
}
```

## 7中flag的形式
支持多种形式，比如
```golang
log.SetFlags(log.Ldate | log.Ltime)
```
```golang
const (
	Ldate         = 1 << iota     // the date in the local time zone: 2009/01/23
	Ltime                         // the time in the local time zone: 01:23:23
	Lmicroseconds                 // microsecond resolution: 01:23:23.123123.  assumes Ltime.
	Llongfile                     // full file name and line number: /a/b/c/d.go:23
	Lshortfile                    // final file name element and line number: d.go:23. overrides Llongfile
	LUTC                          // if Ldate or Ltime is set, use UTC rather than the local time zone
	LstdFlags     = Ldate | Ltime // initial values for the standard logger
)
```

```golang
func New(out io.Writer, prefix string, flag int) *Logger
var buf bytes.Buffer;
newlog := log.New(&buf,"[Header]",1)
newlog.SetFlags(log.Ldate) // 2021/01/29
newlog.SetFlags(log.Ltime) // 13:59:19
newlog.SetFlags(log.Lmicroseconds) // 13:59:30.746728
newlog.SetFlags(log.Llongfile) // /Users/alpaca/Regex/xx.go:18:
newlog.SetFlags(log.Lshortfile)// xx.go:19
newlog.SetFlags(log.LUTC) //
newlog.SetFlags(log.LstdFlags) // 2021/01/29 14:00:53
fmt.Println(&buf)
```


## 创建log实例以及修改属性
```golang
var buf bytes.Buffer
logger :=  log.New(&buf,"log header:",log.Ldate)
fmt.Println(logger.Prefix()) // log header: // 输出头部的设置
fmt.Println(logger.Flags()) // 1 log.Late
// 重新设置头部信息
logger.SetPrefix("log header2:")
// 重新设置输出格式 也就是 [mail server] 2021/02/06 panic 中间这一部分
logger.SetFlags(2)
// 重新设置日志输出的目标点
var buf2 bytes.Buffer
log.SetOutput(&buf2)
```

## 输出日志
输出日志有有两种方法
- 调用output()
- 调用println()等方法

第二种的本事也是调用output(),源码内容如下
```golang
func (l *Logger) Println(v ...interface{}) { l.Output(2, fmt.Sprintln(v...)) }
```
所以我们可以先看Output()的源码
```golang
func (l *Logger) Output(calldepth int, s string) error {
	now := time.Now() // get this early.
	var file string
	var line int
	l.mu.Lock()
	defer l.mu.Unlock()
	if l.flag&(Lshortfile|Llongfile) != 0 {
		// Release lock while getting caller info - it's expensive.
		l.mu.Unlock()
		var ok bool
		_, file, line, ok = runtime.Caller(calldepth)
		if !ok {
			file = "???"
			line = 0
		}
		l.mu.Lock()
	}
	l.buf = l.buf[:0]
	l.formatHeader(&l.buf, now, file, line)
	l.buf = append(l.buf, s...)
	if len(s) == 0 || s[len(s)-1] != '\n' {
		l.buf = append(l.buf, '\n')
	}
	_, err := l.out.Write(l.buf)
	return err
}
```
默认情况下calldepath就是2,官方文档里面也没写清楚，s表示我们日志需要打印的内容，
这段源码里面为了支持runtime加入了锁，formatHeader部分格式化了日志的输出内容

> 打印内容到日志文件下面

```golang
func main() {
	//var buf bytes.Buffer
	//logger :=  log.New(&buf,"log header:",log.Ldate)
	//fmt.Println(logger.Prefix()) // log header: // 输出头部的设置
	//fmt.Println(logger.Flags()) // 1 log.Late
	//
	//// 重新设置头部信息
	//logger.SetPrefix("log header2:")
	//// 重新设置输出格式 也就是 [mail server] 2021/02/06 panic 中间这一部分
	//logger.SetFlags(2)
	//// 重新设置日志输出的目标点
	//var buf2 bytes.Buffer
	//log.SetOutput(&buf2)


	f, err := os.OpenFile("./xx.txt",os.O_RDWR|os.O_CREATE|os.O_APPEND, 0666)
	if err !=  nil{
		log.Fatal(err)
	}
	logger := log.New(f,"[mail server]", log.Ldate | log.Lshortfile) 
	logger.Println("panic")
	logger.Output(2,"output")
	defer f.Close()

	ff, err := os.Open("./xx.txt")
	if err != nil{
		log.Fatal(err)
	}
	content, err := ioutil.ReadAll(ff)
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(string(content))
	defer ff.Close()
}
```
输出信息如下
```
[mail server]2021/02/06 lib_log.go:30: panic
[mail server]2021/02/06 proc.go:203: output
[mail server]2021/02/06 lib_log.go:30: panic
[mail server]2021/02/06 proc.go:203: output

```


## 参考链接
https://www.flysnow.org/2017/05/06/go-in-action-go-log.html
http://legendtkl.com/2016/03/11/go-log/