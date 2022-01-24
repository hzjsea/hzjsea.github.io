---
title: go_net
date: 2021-06-03 17:15:33
categories:  [golang]
tags: [golang]
---


<!--more-->


# go_net


## 网络请求协议

## http 客户端

### Response

响应体结构体如下
- 状态行
- 首部字段
- 响应体

![](https://noback.upyun.com/2021-06-15-11-46-55.png!)

```go
type Response struct {
	Status     string // e.g. "200 OK"
	StatusCode int    // e.g. 200
	Proto      string // e.g. "HTTP/1.0"
	ProtoMajor int    // e.g. 1
	ProtoMinor int    // e.g. 0

	Header Header  // http 头 type Header map[string][]string 

	Body io.ReadCloser // 可读写的body本体

	// ContentLength records the length of the associated content. The
	// value -1 indicates that the length is unknown. Unless Request.Method
	// is "HEAD", values >= 0 indicate that the given number of bytes may
	// be read from Body.
	ContentLength int64 // 返回的消息长度

	// Contains transfer encodings from outer-most to inner-most. Value is
	// nil, means that "identity" encoding is used.
    TransferEncoding []string // 传输编码 
    
	// Close records whether the header directed that the connection be
	// closed after reading Body. The value is advice for clients: neither
	// ReadResponse nor Response.Write ever closes a connection.
	Close bool

	// Uncompressed reports whether the response was sent compressed but
	// was decompressed by the http package. When true, reading from
	// Body yields the uncompressed content instead of the compressed
	// content actually set from the server, ContentLength is set to -1,
	// and the "Content-Length" and "Content-Encoding" fields are deleted
	// from the responseHeader. To get the original response from
	// the server, set Transport.DisableCompression to true.
	Uncompressed bool

	// Trailer maps trailer keys to values in the same
	// format as Header.
	//
	// The Trailer initially contains only nil values, one for
	// each key specified in the server's "Trailer" header
	// value. Those values are not added to Header.
	//
	// Trailer must not be accessed concurrently with Read calls
	// on the Body.
	//
	// After Body.Read has returned io.EOF, Trailer will contain
	// any trailer values sent by the server.
	Trailer Header

	// Request is the request that was sent to obtain this Response.
	// Request's Body is nil (having already been consumed).
	// This is only populated for Client requests.
	Request *Request // 请求

	// TLS contains information about the TLS connection on which the
	// response was received. It is nil for unencrypted responses.
	// The pointer is shared between responses and should not be
	// modified.
	TLS *tls.ConnectionState // https
}
```

> 测试代码
```golang
package main

import (
	"fmt"
	"net/http"
)

func main() {
	resp,err := http.Get("http://www.baidu.com")
	if err != nil{
		panic("error")
	}
	defer resp.Body.Close()
	fmt.Println(resp.Body)
	fmt.Println(resp.TLS)

	resp,err = http.Get("https://www.baidu.com")
	if err != nil{
		panic("error")
	}
	fmt.Println(resp.TLS)
	fmt.Println(resp.ContentLength)
	fmt.Println(resp.Close)
	fmt.Println(resp.Request)
}
```
正常情况下，我们普遍用的就是request,body,StatusCode这三个字段了



### Request

request结构体如下
![](https://noback.upyun.com/2021-06-15-11-00-03.png!)
- 包含请求行、请求头（首部字段）和请求实体（请求主体）三部分，
- 请求行中包含了请求方法、URL 和 HTTP 协议版本，
- 请求头中包含了 HTTP 请求首部字段，
- 请求体 对于 GET 请求来说，没有提交表单数据，所以请求实体为空，对于 POST 请求来说，会包含包括表单数据的请求实体

在go中定义的request结构体内容如下，几个主要字段
```golang
type Request struct {
	Method string // 请求方法
	URL *url.URL
	Proto      string // "HTTP/1.0" // 请求协议

	//	Host: example.com
	//	accept-encoding: gzip, deflate
	//	Accept-Language: en-us
	//	fOO: Bar
	//	foo: two
	//
	// then
	//
	//	Header = map[string][]string{
	//		"Accept-Encoding": {"gzip, deflate"},
	//		"Accept-Language": {"en-us"},
	//		"Foo": {"Bar", "two"},
	//	}
	Header Header // 请求头
	Body io.ReadCloser // 一个可以读取和关闭的请求体
	GetBody func() (io.ReadCloser, error)
	ContentLength int64 // 请求总长度
	TransferEncoding []string
	Close bool
	Host string
	Form url.Values
	PostForm url.Values
	RequestURI string
	TLS *tls.ConnectionState
	Cancel <-chan struct{} // channel关闭的字段
	Response *Response // 响应体
	ctx context.Context
}
```

Url的结构体如下
```golang
type URL struct {
	Scheme     string // http or  https
	Opaque     string    // encoded opaque data
	User       *Userinfo // username and password information
	Host       string    // host or host:port
	Path       string    // 请求路径参数
	RawPath    string    // encoded path hint (see EscapedPath method)
	ForceQuery bool      // append a query ('?') even if RawQuery is empty
	RawQuery   string    // 请求查询参数，也就是 URL 中 ? 之后的部分；
	Fragment   string    // 表示 URL 中的锚点信息，也就是 URL 中 # 之后的部分。
}
```

比如,对于 `https://xueyuanjun.com/books/golang-tutorials?page=2#comments`来说
- Scheme 的值是https
- host 的值是xueyuanjun.com
- Path 的值的是/books/golang-tutorials
- RawQuery 是 page=2


## http 服务端

![](https://noback.upyun.com/2021-06-11-15-22-20.png!)

服务端的主要作用就是调度收到的请求以及其请求路由与服务端本地资源之间的调度
![](https://noback.upyun.com/2021-06-07-08-47-05.png!)


服务端一般以`http.ListenAndServe`起手，开通一个接口作为服务端接口
```golang
// ListenAndServe listens on the TCP network address addr and then calls
// Serve with handler to handle requests on incoming connections.
// Accepted connections are configured to enable TCP keep-alives.
//
// The handler is typically nil, in which case the DefaultServeMux is used.
//
// ListenAndServe always returns a non-nil error.
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```
- addr 服务端地址
- handler 句柄 标准库中er结尾的话，就是一个接口类型

他的本质是初始化了server的实例,然后调用server.ListenAndServe去实现服务监听,
在server的结构体声明当中，有如下的定义: 
当handler的值为nil的时候，路由注册会自动转变成`http.DefaultServeMux`,在server的源码里的原话为
```golang
type Server struct {
	Addr    string  // TCP address to listen on, ":http" if empty
	...
	Handler Handler // handler to invoke, http.DefaultServeMux if nil
	...
}
```


### Server.ListenAndServe


```golang
func (srv *Server) ListenAndServe() error {
	if srv.shuttingDown() {
		return ErrServerClosed
	}
	addr := srv.Addr
	if addr == "" {
		addr = ":http"
	}
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(ln)
}
```
创建socket监听，并将监听正确后的长连接传递给server.Serve(ln) 运行

```golang
// Serve accepts incoming connections on the Listener l, creating a
// new service goroutine for each. The service goroutines read requests and
// then call srv.Handler to reply to them.
//
// HTTP/2 support is only enabled if the Listener returns *tls.Conn
// connections and they were configured with "h2" in the TLS
// Config.NextProtos.
//
// Serve always returns a non-nil error and closes l.
// After Shutdown or Close, the returned error is ErrServerClosed.
func (srv *Server) Serve(l net.Listener) error {
	if fn := testHookServerServe; fn != nil {
		fn(srv, l) // call hook with unwrapped listener
	}

	origListener := l
	l = &onceCloseListener{Listener: l}
	defer l.Close()

	if err := srv.setupHTTP2_Serve(); err != nil {
		return err
	}

	if !srv.trackListener(&l, true) {
		return ErrServerClosed
	}
	defer srv.trackListener(&l, false)

	var tempDelay time.Duration // how long to sleep on accept failure

	baseCtx := context.Background()
	if srv.BaseContext != nil {
		baseCtx = srv.BaseContext(origListener)
		if baseCtx == nil {
			panic("BaseContext returned a nil context")
		}
	}

	ctx := context.WithValue(baseCtx, ServerContextKey, srv)
	for {
		rw, e := l.Accept()
		if e != nil {
			select {
			case <-srv.getDoneChan():
				return ErrServerClosed
			default:
			}
			if ne, ok := e.(net.Error); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				srv.logf("http: Accept error: %v; retrying in %v", e, tempDelay)
				time.Sleep(tempDelay)
				continue
			}
			return e
		}
		if cc := srv.ConnContext; cc != nil {
			ctx = cc(ctx, rw)
			if ctx == nil {
				panic("ConnContext returned nil")
			}
		}
		tempDelay = 0
		c := srv.newConn(rw)
		c.setState(c.rwc, StateNew) // before Serve can return
		go c.serve(ctx)
	}
}
```
<div style='font-size:20px;color:red'>接收客户端请求并建立连接</div>

创建 Listen Socket 成功后，调用 Server 实例的 Serve(net.Listener) 方法，用来接收并处理客户端的请求信息。这个方法里面起了一个 for 循环，在循环体中首先通过 net.Listener（即上一步监听端口中创建的 Listen Socket）实例的 Accept 方法接收客户端请求，接收到请求后根据请求信息创建一个 conn 连接实例，最后单独开了一个 goroutine，把这个请求的数据当做参数扔给这个 conn 去服务,

用户的每一次请求都是在一个新的 goroutine 去服务，相互不影响。客户端请求的具体处理逻辑都是在 c.serve 中完成的


```golang

func (c *conn) serve(ctx context.Context) {
	c.remoteAddr = c.rwc.RemoteAddr().String()
	ctx = context.WithValue(ctx, LocalAddrContextKey, c.rwc.LocalAddr())
	defer func() {
		if err := recover(); err != nil && err != ErrAbortHandler {
			const size = 64 << 10
			buf := make([]byte, size)
			buf = buf[:runtime.Stack(buf, false)]
			c.server.logf("http: panic serving %v: %v\n%s", c.remoteAddr, err, buf)
		}
		if !c.hijacked() {
			c.close()
			c.setState(c.rwc, StateClosed)
		}
	}()

	if tlsConn, ok := c.rwc.(*tls.Conn); ok {
		if d := c.server.ReadTimeout; d != 0 {
			c.rwc.SetReadDeadline(time.Now().Add(d))
		}
		if d := c.server.WriteTimeout; d != 0 {
			c.rwc.SetWriteDeadline(time.Now().Add(d))
		}
		if err := tlsConn.Handshake(); err != nil {
			// If the handshake failed due to the client not speaking
			// TLS, assume they're speaking plaintext HTTP and write a
			// 400 response on the TLS conn's underlying net.Conn.
			if re, ok := err.(tls.RecordHeaderError); ok && re.Conn != nil && tlsRecordHeaderLooksLikeHTTP(re.RecordHeader) {
				io.WriteString(re.Conn, "HTTP/1.0 400 Bad Request\r\n\r\nClient sent an HTTP request to an HTTPS server.\n")
				re.Conn.Close()
				return
			}
			c.server.logf("http: TLS handshake error from %s: %v", c.rwc.RemoteAddr(), err)
			return
		}
		c.tlsState = new(tls.ConnectionState)
		*c.tlsState = tlsConn.ConnectionState()
		if proto := c.tlsState.NegotiatedProtocol; validNPN(proto) {
			if fn := c.server.TLSNextProto[proto]; fn != nil {
				h := initNPNRequest{ctx, tlsConn, serverHandler{c.server}}
				fn(c.server, tlsConn, h)
			}
			return
		}
	}

	// HTTP/1.x from here on.

	ctx, cancelCtx := context.WithCancel(ctx)
	c.cancelCtx = cancelCtx
	defer cancelCtx()

	c.r = &connReader{conn: c}
	c.bufr = newBufioReader(c.r)
	c.bufw = newBufioWriterSize(checkConnErrorWriter{c}, 4<<10)

	for {
		w, err := c.readRequest(ctx)
		if c.r.remain != c.server.initialReadLimitSize() {
			// If we read any bytes off the wire, we're active.
			c.setState(c.rwc, StateActive)
		}
		if err != nil {
			const errorHeaders = "\r\nContent-Type: text/plain; charset=utf-8\r\nConnection: close\r\n\r\n"

			switch {
			case err == errTooLarge:
				// Their HTTP client may or may not be
				// able to read this if we're
				// responding to them and hanging up
				// while they're still writing their
				// request. Undefined behavior.
				const publicErr = "431 Request Header Fields Too Large"
				fmt.Fprintf(c.rwc, "HTTP/1.1 "+publicErr+errorHeaders+publicErr)
				c.closeWriteAndWait()
				return

			case isUnsupportedTEError(err):
				// Respond as per RFC 7230 Section 3.3.1 which says,
				//      A server that receives a request message with a
				//      transfer coding it does not understand SHOULD
				//      respond with 501 (Unimplemented).
				code := StatusNotImplemented

				// We purposefully aren't echoing back the transfer-encoding's value,
				// so as to mitigate the risk of cross side scripting by an attacker.
				fmt.Fprintf(c.rwc, "HTTP/1.1 %d %s%sUnsupported transfer encoding", code, StatusText(code), errorHeaders)
				return

			case isCommonNetReadError(err):
				return // don't reply

			default:
				publicErr := "400 Bad Request"
				if v, ok := err.(badRequestError); ok {
					publicErr = publicErr + ": " + string(v)
				}

				fmt.Fprintf(c.rwc, "HTTP/1.1 "+publicErr+errorHeaders+publicErr)
				return
			}
		}

		// Expect 100 Continue support
		req := w.req
		if req.expectsContinue() {
			if req.ProtoAtLeast(1, 1) && req.ContentLength != 0 {
				// Wrap the Body reader with one that replies on the connection
				req.Body = &expectContinueReader{readCloser: req.Body, resp: w}
			}
		} else if req.Header.get("Expect") != "" {
			w.sendExpectationFailed()
			return
		}

		c.curReq.Store(w)

		if requestBodyRemains(req.Body) {
			registerOnHitEOF(req.Body, w.conn.r.startBackgroundRead)
		} else {
			w.conn.r.startBackgroundRead()
		}

		// HTTP cannot have multiple simultaneous active requests.[*]
		// Until the server replies to this request, it can't read another,
		// so we might as well run the handler in this goroutine.
		// [*] Not strictly true: HTTP pipelining. We could let them all process
		// in parallel even if their responses need to be serialized.
		// But we're not going to implement HTTP pipelining because it
		// was never deployed in the wild and the answer is HTTP/2.
		serverHandler{c.server}.ServeHTTP(w, w.req)
		w.cancelCtx()
		if c.hijacked() {
			return
		}
		w.finishRequest()
		if !w.shouldReuseConnection() {
			if w.requestBodyLimitHit || w.closedRequestBodyEarly() {
				c.closeWriteAndWait()
			}
			return
		}
		c.setState(c.rwc, StateIdle)
		c.curReq.Store((*response)(nil))

		if !w.conn.server.doKeepAlives() {
			// We're in shutdown mode. We might've replied
			// to the user without "Connection: close" and
			// they might think they can send another
			// request, but such is life with HTTP/1.1.
			return
		}

		if d := c.server.idleTimeout(); d != 0 {
			c.rwc.SetReadDeadline(time.Now().Add(d))
			if _, err := c.bufr.Peek(4); err != nil {
				return
			}
		}
		c.rwc.SetReadDeadline(time.Time{})
	}
}

```


这个的中间有一段代码如下，主要调用server.ServeHTTP
```golang
// HTTP cannot have multiple simultaneous active requests.[*]
// Until the server replies to this request, it can't read another,
// so we might as well run the handler in this goroutine.
// [*] Not strictly true: HTTP pipelining. We could let them all process
// in parallel even if their responses need to be serialized.
// But we're not going to implement HTTP pipelining because it
// was never deployed in the wild and the answer is HTTP/2.
serverHandler{c.server}.ServeHTTP(w, w.req)
```

ServeHTTP的源码如下
```golang
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}
	handler.ServeHTTP(rw, req)
}

type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
这里实现了ServeHTTP,也就实现了Handler这个类型


## 路由
ServeMux 是一个 HTTP 请求多路复用器`(HTTP Request multiplexer)`，总的来说，go会开启一个服务端监听地址，然后传入pattern以及handler去处理.
具体来说，它主要有以下两个功能:
- 路由功能，将请求的`URL`与一组已经注册的路由模式(pattern)进行匹配，并选择一个最接近模式，调用其处理函数.
- 它会清理请求`URL`与`Host`请求头，清除端口号，并对包含`..`和重复`/`的URL进行处理.


先来看一下ServeMux的结构
```golang
type ServeMux struct {
    mu    sync.RWMutex
    m     map[string]muxEntry
    es    []muxEntry 
    hosts bool  // 标记路由中是否带有主机名
}
```
- m 就是用来存储路由与处理函数映射关系的 map
- es 按照路由长度从大到小的存放处理函数
- ServeMux 为了方便，map存放的值其实是放有处理函数和路由路径的 muxEntry 结构体

muxEntry结构体存储的就是我们在一开始输入的pattern路径和handler处理函数
```golang
type muxEntry struct {
    h       Handler  // 处理函数
    pattern string   // 路由路径
}
```

ServeMux暴露的方法主要有如下4个
```golang
func (mux *ServeMux) Handle(pattern string, handler Handler)
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string)
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)
```

> Handle 方法通过将传入的路由和处理函数存入 ServeMux 的映射表`m`中来实现`"路由注册(register)"`
```golang
func (mux *ServeMux) Handle(pattern string, handler Handler) {
    mux.mu.Lock()
    defer mux.mu.Unlock()
    // 检查路由路径是否为空
    if pattern == "" {
    	panic("http: invalid pattern")
    }
    // 检查处理函数是否为空
    if handler == nil {
    	panic("http: nil handler")
    }
    // 检查该路由是否已经注册过
    if _, exist := mux.m[pattern]; exist {
    	panic("http: multiple registrations for " + pattern)
    }
    // 如果还没有任何路由注册，就为 mux.m 分配空间
    if mux.m == nil {
    	mux.m = make(map[string]muxEntry)
    }
    // 实例化一个 muxEntry
    e := muxEntry{h: handler, pattern: pattern}
    // 将该路由与该 muxEntry 的实例存到 mux.m 中
    mux.m[pattern] = e
    // 如果该路由路径以 "/" 结尾，就把该路由按照大到小的路径长度插入到 mux.e 中
    if pattern[len(pattern)-1] == '/' {
    	mux.es = appendSorted(mux.es, e)
    }
    // 如果该路由路径不以 "/" 开始，标记该 mux 中有路由的路径带有主机名
    if pattern[0] != '/' {
    	mux.hosts = true
    }
}
```
> HandleFunc 方法接收一个具体的处理函数将其包装成 Handler：

```golang
type HandlerFunc func(ResponseWriter, *Request)

func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    if handler == nil {
    	panic("http: nil handler")
    }
    mux.Handle(pattern, HandlerFunc(handler))
}
```

<div style='font-size:20px;color:red'>其中 HandlerFunc(f) 起到的作用就是在 HandlerFunc 中执行 f</div>


> hanlder方法
Handler 方法从传入的请求(Request)中拿到 URL 进行匹配，返回对应的处理函数和路由
在看 Handler 的实现前，先看看它调用的 handler 方法

```golang
func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
    mux.mu.RLock()
    defer mux.mu.RUnlock()

    // 若当前 mux 中注册有带主机名的路由，就用"主机名+路由路径"去匹配
    // 也就是说带主机名的路由优先于不带的
    if mux.hosts {
    	h, pattern = mux.match(host + path)
    }
    // 所以若没有匹配到，就直接把路由路径拿去匹配
    if h == nil {
    	h, pattern = mux.match(path)
    }
    // 若都没有匹配到，就默认返回 NotFoundHandler，该 Handler 会往
    // 响应里写上 "404 page not found"
    if h == nil {
    	h, pattern = NotFoundHandler(), ""
    }
    // 返回获得的 Handler 和路由路径
    return
}
```

> handler 方法

```golang
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) {
    // 去掉主机名上的端口号
    host := stripHostPort(r.Host)
    // 整理 URL，去掉 ".", ".."
    path := cleanPath(r.URL.Path)

    // redirectToPathSlash 在 mux.m 中查看 path+"/" 是否存在
    // 如果存在，RedirectHandler 就将该请求重定向到 path+"/"
    if u, ok := mux.redirectToPathSlash(host, path, r.URL); ok {
    	return RedirectHandler(u.String(), StatusMovedPermanently), u.Path
    }

    // 如果整理后的 URL 与请求中的路径不一样，先调用 handler 进行匹配
    // 在将请求里的 URL 改成整理后的 URL
    // 最后将该请求重定向到整理后的 URL
    if path != r.URL.Path {
    	_, pattern = mux.handler(host, path)
    	url := *r.URL
    	url.Path = path
    	return RedirectHandler(url.String(), StatusMovedPermanently), pattern
    }
    // 若以上条件都不满足则返回匹配结果
    return mux.handler(host, r.URL.Path)
}
```

> match 匹配路由

```golang
func (mux *ServeMux) match(path string) (h Handler, pattern string) {
    // 若 mux.m 中已存在该路由映射，直接返回该路由的 Handler，和路径
    v, ok := mux.m[path]
    if ok {
    	return v.h, v.pattern
    }

    // 找到路径能最长匹配的路由。
    for _, e := range mux.es {
    	if strings.HasPrefix(path, e.pattern) {
    	    return e.h, e.pattern
    	}
    }
    return nil, ""
}
```
注意这里是在 mux.es 中进行查找，而不是映射表 mux.m 中，而 mux.es 是存放所有以 "/" 结尾的路由路径的切片。因为只会在以 "/" 结尾的路由路径中才会出现需要选择最长匹配方案
那么当一个请求的 URL 为 /a/b/c 的时候，我们希望是由 ab 来处理这个请求。
另外，为了减少在 mux.es 中的查询时间， mux.es 中元素是按照它们的长度由大到小顺序存放的。这样就能在保证一致性的同时，保证时间


> serverHTTP 与 ServeMux 联动

```golang
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

type myHandler struct {}

func (h *myHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("This message is from myHandler."))
}

func main() {
    http.Handle("/", &helloHandler{})  // 路由注册
} 
```


ServeMux 是一个结构体，它的 ServeHTTP 方法要做的就是将每个请求派遣(dispatch)到它们对应的处理函数上。
但是 ServeMux 的多路处理实现并不支持请求方法判断，也不能处理路由嵌套和URL变量值提取的功能
```golang
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
	if r.RequestURI == "*" {
		if r.ProtoAtLeast(1, 1) {
			w.Header().Set("Connection", "close")
		}
		w.WriteHeader(StatusBadRequest)
		return
	}
	h, _ := mux.Handler(r)
	h.ServeHTTP(w, r)
}
```


### 注册名字固定的路径
ServeMux 注册路由模式的方式有两种,`注册名字固定的路径`与`注册根路径子树`

https://www.bwangel.me/2019/11/30/intro-servemux/


## 处理路由
如何处理路由,go通过handle来设置并处理路由

先来来一组Handle与HandleFunc

```golang
// Handle registers the handler for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
func Handle(pattern string, handler Handler) {
	DefaultServeMux.Handle(pattern, handler)
}

// HandleFunc registers the handler function for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```
源码注释的解释如下，注册地址到DefaultServeMux,
- 对于handle来说，必须要传入一个Handler类型,该类型实现了一个ServeHTTP的方法
- 对于handleFunc来说，必须要传入一个与一个匿名函数，该函数的实现方法与serveHTTP一样

```golang
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

handler func(ResponseWriter, *Request)
```

然后再跳上去看ServeMux的方法 最终都是调用的`ServeMux.Handle`方法 ,该方法会去调用HandlerFunc方法
```golang
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}
	mux.Handle(pattern, HandlerFunc(handler))
}
```

这个HandlerFunc会把传入的匿名参数实现一遍ServeHTTP
```golang
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```
实现之后就会实现了Handler类型，传入到`mux.Handle` 中解析路由，


### Handler
handler这个接口类型被定义在net库当中的server.go文件当中 85行
```golang
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

- 只要实现了ServeHTTP 就实现了Handler这个接口，也实现这个接口类型
- 接收一个response的写入和一个request的指针，其中request为收到的请求的内容

```golang
// ResponseWriter 接口类型
type ResponseWriter interface {
	Header() Header
	Write([]byte) (int, error)
	WriteHeader(statusCode int)
}

// Request 结构体
type Request struct {
    Method string
    ...
}
```

- Handler用于响应一个HTTP request
- 接口方法ServeHTTP应该用来将response header和需要响应的数据写入到ResponseWriter中，然后返回。返回意味着这个请求已经处理结束，不能再使用这个ResponseWriter、不能再从Request.Body中读取数据，不能并发调用已完成的ServerHTTP方法
- handler应该先读取Request.Body，然后再写ResponseWriter。只要开始向ResponseWriter写数据后，就不能再从Request.Body中读取数据


于是整个服务端的接口情况如下
```golang
type helloHandle struct{}

// 创建handle句柄
func (h helloHandle) ServeHTTP(w http.ResponseWriter, r *http.Request) {

	fmt.Println("r.Form",r.Form)
	// 是否能够正常解析
	parseError := r.ParseForm()
	if parseError != nil{
		panic("error")
	}
	
	fmt.Fprintf(w, "测试")  // 发送响应到客户端
}

func main() {
	fmt.Println("address listing on 'http://127.0.0.1:12345'")
    var hello helloHandle
    // 加载句柄
	http.ListenAndServe(":12345",hello)
}

```

![](https://noback.upyun.com/2021-06-08-14-38-22.png!)


整个的工作流程如下
1. 创建 Listen Socket，监听指定的端口，等待客户端请求到来
2. Listen Socket 接收客户端的请求，得到 Client Socket，接下来通过 Client Socket 与客户端通信
3. 处理客户端的请求，首先从 Client Socket 读取 HTTP 请求的协议头, 如果是 POST 方法, 还可能要读取客户端提交的数据，然后交给相应的 Handler（处理器）处理请求，Handler 处理完毕后装载好客户端需要的数据，最后通过 Client Socket 返回给客户端。

> HandlerFunc Handler Handle HandleFunc

```golang

type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}


type HandlerFunc func(ResponseWriter, *Request)
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}


func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```
这里值得琢磨一下，因为在go中，很多都可以作为一种类型来看，这里HandleFunc就是一种方法类型，严肃的讲应该是一种()

如果我们使用HandlerFunc来建立服务端的话，代码应该是这样的
```golang
type HelloHandle struct{}

func (hello *HelloHandle) ServeHTTP(w http.ResponseWriter,  r *http.Request) {
	fmt.Fprintln(w,"hellworld")
}

func main(){
	var hello HelloHandle
	fmt.Printf("listen at http://127.0.0.1:9999")
	http.ListenAndServe(":9999", &hello)
}
```
这样坏处就是在于我们每次要创建一个handle的时候都需要去创建一个struct的类型。
我们可以采用另外一种形式

```golang
func main(){
    fmt.Printf("listen at http://127.0.0.1:9998")
    
    // 1
	http.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
		fmt.Fprint(writer,"helloworld")
    })

    // 2
	server := http.Server{
		Addr: ":9998",
    }
	server.ListenAndServe()
}
```
这样就会方便很多，但这里有个问题，路由是怎么注册的，第一段和第二段都没什么交互，这里就要看handlefunc内部的源码了



## 请求分发与路由匹配

serverHandler{c.server}.ServeHTTP(w, w.req)

路由分发通过`mux.ServeHTTP`实现, 处理客户端请求时，会调用默认`ServeMux`实现的`ServeHTTP`方法
```golang
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    if r.RequestURI == "*" {
        w.Header().Set("Connection", "close")
        w.WriteHeader(StatusBadRequest)
        return
    }
    
    h, _ := mux.Handler(r)
    h.ServeHTTP(w, r)
}
```


## 新的路由处理方式 gorilla/mux

官方库里的http标准库，只支持基本的url路由匹配，并不支持路由的泛型匹配,
- 不支持参数设定，例如 /user/:uid 这种泛类型匹配
- 对 REST 风格接口支持不友好，无法限制访问路由的方法；
- 如果不采用动态绑定的方法的话，路由设置的代码会很繁杂

```golang

func sayHelloWorld(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)    // 设置响应状态码为 200
	fmt.Fprintf(w, "Hello, World!") // 发送响应到客户端
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/hello", sayHelloWorld)
	http.ListenAndServe(":9999", r)
}
```

> 支持多路由泛式匹配


设置查询参数
```golang
r.HandleFunc("/hello/{name}", sayHelloWorld)
r.HandleFunc("/hello/{name:[a-z]+}", sayHelloWorld)
```

解析路由参数 传回的是个map
```golang
func sayHelloWorld(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	fmt.Println(params) // map[name:xx]
	w.WriteHeader(http.StatusOK)    // 设置响应状态码为 200
	fmt.Fprintf(w, "Hello, World!") // 发送响应到客户端
}

func main() {

	// 创建路由
	r := mux.NewRouter()
	r.HandleFunc("/hello/{name}", sayHelloWorld)
	http.ListenAndServe(":9999", r)
}

```

同样gorilla也支持handle与handleFunc两种模式传递句柄
```golang
package main

import (
    "fmt"
    "github.com/gorilla/mux"
    "log"
    "net/http"
)

func sayHelloWorld(w http.ResponseWriter, r *http.Request)  {
    params := mux.Vars(r)
    w.WriteHeader(http.StatusOK)  // 设置响应状态码为 200
    fmt.Fprintf(w, "Hello, %s!", params["name"])  // 发送响应到客户端
}

// type HelloWorldHandler struct {}

// func (handler *HelloWorldHandler) ServeHTTP(w http.ResponseWriter, r *http.Request)  {
//     params := mux.Vars(r)
//     w.WriteHeader(http.StatusOK)  // 设置响应状态码为 200
//     fmt.Fprintf(w, "你好, %s!", params["name"])  // 发送响应到客户端
// }

func main()  {
    r := mux.NewRouter()
    r.HandleFunc("/hello/{name:[a-z]+}", sayHelloWorld)
    // r.Handle("/zh/hello/{name}", &HelloWorldHandler{})
    log.Fatal(http.ListenAndServe(":8080", r))
}
```



### 限定方法请求
```golang
func main(){
	r := mux.NewRouter()
	r.HandleFunc("/hello/{name:[a-z]+}", sayHelloWorld).Methods("GET", "POST")
	r.Handle("/zh/hello/{name}", &HelloWorldHandler{}).Methods("GET")
	log.Fatal(http.ListenAndServe(":8080", r))
}

```

### 限定域名
```golang
r.HandleFunc("/hello/{name:[a-z]+}",    sayHelloWorld).Methods("GET").Host("goweb.test")
// curl -X GET http://goweb.test::8080/hello/golang
```
只有在当请求正确域名的时候，才能正常使用句柄

### 限定请求方式
```golang
r.Handle("/zh/hello/{name}", &HelloWorldHandler{}).Methods("GET").Host("zh.goweb.test").Schemes("https")
// curl -X GET https://zh.goweb.test::8080/hello/golang
```


### 剩余的请看
https://laravelacademy.org/post/21365

