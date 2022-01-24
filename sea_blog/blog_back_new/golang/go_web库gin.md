---
title: golangWeb库gin
categories:
  - golang
tags:
  - golang
  - gin
  - goweb
abbrlink: b035d942
date: 2020-09-21 12:05:07
---

# golangWeb库gin

## 项目初始化
gin没有现成的初始化web工具，在创建过程中可能会出现没有智能提示的问题
在golnd的中解决方法是用module创建,并添加库的下载地址

![2021-01-22-02-48-37](http://noback.upyun.com/2021-01-22-02-48-37.png)

```go
// get框架
go get github.com/gin-gonic/gin
// 同步整个项目中的库，没有安装的会被安装，多余的库会被删除
go mod tidy
```

## 热更新

- fresh 
- gin

两个都是热更新 原理应该都是监督文件变化重启来做到热更新的 
主要是把颜色去掉了，感觉灵魂么得了。。。



```go
# 安装 fresh
$ go get github.com/pilu/fresh

# 跳转到项目目录,例如项目名为‘myapp’
$ cd /path/to/myapp

# 启动
$ fresh

```
fresh 是根据配置文件来格式化输出内容的我们可以修改配置文件
```bash
touch runner.conf
cat << EOF > ./runner.conf
root:              .   
tmp_path:          ./tmp
build_name:        runner-build
build_log:         runner-build-errors.log
valid_ext:         .go, .tpl, .tmpl, .html
no_rebuild_ext:    .tpl, .tmpl, .html
ignored:           assets, tmp
build_delay:       600
colors:            1
log_color_main:    cyan
log_color_build:   yellow
log_color_runner:  green
log_color_watcher: magenta
log_color_app:
EOF
```

```go
# 安装 fresh
$ go get github.com/codegangsta/gin

# 验证gin是否安装成功
$ gin -h

# 启动
$ gin run main.go
```


## 路由对象

创建路由对象
```go
package  main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	// 创建路由控制器
	r := gin.Default()
	r.GET("/ping", func(context *gin.Context) {
		context.String(200,"pong")
	})

	// 启动
	r.Run()
}
```

> 函数与路由分离

```go
package  main

import (
	"github.com/gin-gonic/gin"
)

func Pong(context *gin.Context) {
	context.String(200,"pong")
}
func main() {
	// 创建路由控制器
	r := gin.Default()
	r.GET("/ping",Pong)
	// 启动
	r.Run()
}
```

设置404代码，可以设置一个静态页面返回404请求
```golang
r.LoadHTMLGlob("./views/*")
r.NoRoute(func(c *gin.Context) {
		c.HTML(http.StatusNotFound, "404.html", nil)
})
```

> 源码部分

- 首先是`Default`创建了一个`type Engine struct`的结构体实例，该结构体实现了各种各样的接口。作为路由对象
- GET 方法
```go
// GET is a shortcut for router.Handle("GET", path, handle).
func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes {
	return group.handle(http.MethodGet, relativePath, handlers)
}
```
接收一个相对路径，以及一个匿名函数

- RUN函数
```go
func (engine *Engine) Run(addr ...string) (err error) {
	defer func() { debugPrintError(err) }()

	address := resolveAddress(addr)
	debugPrint("Listening and serving HTTP on %s\n", address)
	err = http.ListenAndServe(address, engine)
	return
}

func resolveAddress(addr []string) string {
	switch len(addr) {
	case 0:
		if port := os.Getenv("PORT"); port != "" {
			debugPrint("Environment variable PORT=\"%s\"", port)
			return ":" + port
		}
		debugPrint("Environment variable PORT is undefined. Using port :8080 by default")
		return ":8080"
	case 1:
		return addr[0]
	default:
		panic("too many parameters")
	}
}

```
接收IP+PORT的地址形式或者是`:8080`单端口的形式


## 路由参数
```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()

	// 这个handler 将会匹配 /user/john 但不会匹配 /user/ 或者 /user
	router.GET("/user/:name", func(c *gin.Context) {
		name := c.Param("name") // 获取参数，这里name=john
		c.String(http.StatusOK, "Hello name is %s", name)
	})

    // 给多余参数一个值 action
    // /user/hzj/helloworld/xx  res = /helloworld/xx
	router.GET("/user/:name/*action", func(c *gin.Context) {
		name := c.Param("name")
		action := c.Param("action")
		message := name + " is " + action
		c.String(http.StatusOK, message)
	})

	//// 对于每个匹配的请求，上下文将保留路由定义
	//router.POST("/user/:name/*action", func(c *gin.Context) {
	//	c.FullPath() == "/user/:name/*action" // true
	//})

	router.Run(":8080")
}
```

## 查询参数

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()

	// Query string parameters are parsed using the existing underlying request object.
	// The request responds to a url matching:  /welcome?firstname=Jane&lastname=Doe
	router.GET("/welcome", func(c *gin.Context) {
		firstname := c.DefaultQuery("firstname", "Guest")   
		midname := c.DefaultQuery("your real name","hzj")
		lastname := c.Query("lastname") // shortcut for c.Request.URL.Query().Get("lastname")

		c.String(http.StatusOK, "Hello %s %s %s", firstname, midname,lastname)
	})
	router.Run(":8080")
}
```
```go
package  main

import "github.com/gin-gonic/gin"

type msg struct {
	Name    string `json:"user"`
	Message string
	Age     int
}
func main() {
	r := gin.Default()
	// 查询参数的使用
	// 比如 http://127.0.0.1:8080/userinfo?username=hzj&&password=upyun

	r.GET("/userinfo", func(c *gin.Context) {
		// 如果找不到则为默认的值
		username := c.DefaultQuery("username","hzj")

		password := c.Query("password")
		c.JSON(200,gin.H{
			"username":username,
			"password":password,
		})
	})
}
```
- 查询参数默认值 DefaultQuery
- 设置查询参数值 Query
- 查询参数地址值  /welcome?firstname=Jane&lastname=Doe


## 参数绑定
gin为了提高开发效率，添加了一个`ShouldBind`预绑定的方法，会根据`Content-type`类型+反射机制来自动提取请求中的内容
其实很多请求方法都有对应的方法和提取方法的
- querystring 查询参数，也就是在url中使用`?` 拼接，则使用`c.query`提取

![2021-01-24-11-21-15](http://noback.upyun.com/2021-01-24-11-21-15.png)
- form表单 对应的content-type 是 `multipart/form-data`

![2021-01-24-11-21-59](http://noback.upyun.com/2021-01-24-11-21-59.png)
- json类型 对应的content-type 是 `application/json`

![2021-01-24-11-22-21](http://noback.upyun.com/2021-01-24-11-22-21.png)
- xml 类型 对应的content-type 是 `text/xml`

![2021-01-24-11-27-40](http://noback.upyun.com/2021-01-24-11-27-40.png)


ShouldBind会按照下面的顺序进行请求中的数据完成绑定
1. 使用Get请求，只使用form绑定引擎(query)
2. 如果是 POST 请求，首先检查 content-type 是否为 JSON 或 XML，然后再使用 Form（form-data）

代码如下

```golang
package  main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)


type Login struct {
	User     string `form:"user" json:"user" binding:"required"`
	Password string `form:"password" json:"password" binding:"required"`
}

func main() {
	r := gin.Default()

	// 绑定JSON的示例 ({"user": "q1mi", "password": "123456"})
	r.POST("/loginJson", func(c *gin.Context) {
		var login Login

		if err:=c.ShouldBind(&login);err == nil{
			c.JSON(200,gin.H{
				"user":login.User,
				"password":login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest,gin.H{"err":err.Error()})
		}

	})

	r.POST("/loginForm", func(c *gin.Context) {
		var login Login
		// ShouldBind()会根据请求的Content-Type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})

	// 绑定QueryString示例 (/loginQuery?user=q1mi&password=123456)
	r.GET("/loginFormQuery", func(c *gin.Context) {
		var login Login
		// ShouldBind()会根据请求的Content-Type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})

	r.GET("/loginJsonGet", func(c *gin.Context) {
		var login Login
		if err := c.ShouldBind(&login);err != nil{
			c.JSON(200,gin.H{
				"user":login.User,
				"password":login.Password,
			})
		}
		c.JSON(200,&login)

	})

	r.POST("/loginXml", func(c *gin.Context) {
		var login Login
		if err := c.ShouldBind(&login);err != nil{
			c.JSON(200,gin.H{
				"user":login.User,
				"password":login.Password,
			})
		}
		c.JSON(200,&login)
	})

	r.Run()
}

```

## 表单
```go
func main() {
	router := gin.Default()

	router.POST("/form_post", func(c *gin.Context) {
		message := c.PostForm("message") // 表单key
		nick := c.DefaultPostForm("nick", "anonymous")  // 表单key-value

		c.JSON(200, gin.H{
			"status":  "posted",
			"message": message,
			"nick":    nick,
		})
	})
	router.Run(":8080")
}
```

![2020-09-21-16-43-28](http://noback.upyun.com/2020-09-21-16-43-28.png)


## 查询参数与表单验证
```go
func main() {
	router := gin.Default()

	router.POST("/post", func(c *gin.Context) {

		id := c.Query("id")
		page := c.DefaultQuery("page", "0")
		name := c.PostForm("name")
		message := c.PostForm("message")

		fmt.Printf("id: %s; page: %s; name: %s; message: %s", id, page, name, message)
	})
	router.Run(":8080")
}

```
```go
POST /post?id=1234&page=1 HTTP/1.1   // 查询参数
Content-Type: application/x-www-form-urlencoded

name=manu&message=this_is_great  // 查询参数
```


![2020-09-21-16-47-53](http://noback.upyun.com/2020-09-21-16-47-53.png)


## 查询参数与表单验证 map

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)
func main() {
	router := gin.Default()

	router.POST("/post", func(c *gin.Context) {

		ids := c.QueryMap("ids")
		names := c.PostFormMap("names")

		fmt.Printf("ids: %v; names: %v", ids, names)
	})
	router.Run(":8080")
}
```


![2020-09-21-16-50-42](http://noback.upyun.com/2020-09-21-16-50-42.png)


## 上传文件

```go

package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"log"
	"net/http"
)

func main() {
	router := gin.Default()
	// Set a lower memory limit for multipart forms (default is 32 MiB)
	router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// single file
		file, _ := c.FormFile("file")
		log.Println(file.Filename)

		// Upload the file to specific dst.
		c.SaveUploadedFile(file, dst)

		c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
	})
	router.Run(":8080")
}
```

```bash
curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/appleboy/test.zip" \
  -H "Content-Type: multipart/form-data"
```

## 上传各类文件

```go
func main() {
	router := gin.Default()
	// Set a lower memory limit for multipart forms (default is 32 MiB)
	router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// Multipart form
		form, _ := c.MultipartForm()
		files := form.File["upload[]"]

		for _, file := range files {
			log.Println(file.Filename)

			// Upload the file to specific dst.
			c.SaveUploadedFile(file, dst)
		}
		c.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
	})
	router.Run(":8080")
}
```
```bash
curl -X POST http://localhost:8080/upload \
  -F "upload[]=@/Users/appleboy/test1.zip" \
  -F "upload[]=@/Users/appleboy/test2.zip" \
  -H "Content-Type: multipart/form-data"
```

## 重定向
gin支持重定向
```golang
r.GET("/test", func(c *gin.Context) {
	c.Redirect(http.StatusMovedPermanently, "http://www.sogo.com/")
})
```

> 使用路由重定向

```go
r.GET("/test", func(c *gin.Context) {
    // 指定重定向的URL
    c.Request.URL.Path = "/test2"
    r.HandleContext(c)
})
r.GET("/test2", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"hello": "world"})
})
```


## 路由分组
```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router := gin.Default()

	// Simple group: v1
	v1 := router.Group("/v1")
	{
		v1.POST("/login", loginEndpoint)
		v1.POST("/submit", submitEndpoint)
		v1.POST("/read", readEndpoint)
	}

	// Simple group: v2
	v2 := router.Group("/v2")
	{
		v2.POST("/login", loginEndpoint)
		v2.POST("/submit", submitEndpoint)
		v2.POST("/read", readEndpoint)
	}

	router.Run(":8080")
}

func loginEndpoint(c *gin.Context) {
	//c.String(http.StatusOK,"helloworld")
	fmt.Println("hellowrld")
}

func submitEndpoint(c *gin.Context){
	c.String(http.StatusOK,"is ok")
}

func readEndpoint(c *gin.Context){
	c.String(http.StatusOK,c.FullPath())
}

```


### 路由组嵌套
```golang
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	r:=gin.Default()

	usergroup := r.Group("/user")
	searchUserGroup := usergroup.Group("/search")

	searchUserGroup.GET("/", func(c *gin.Context) {
		fmt.Println("the url is http://127.0.0.1:8080/user/search/")
		c.String(200,"hhhh")
	})

	r.Run()
}
```
使用group来做路由嵌套,使用的路由是`fmt.Println("the url is http://127.0.0.1:8080/user/search/")`


## 自定义中间件
router被定义成为一个结构体，默认可以使用gin.Default()或者gin.New()创建。区别在于gin.Default也适用于gin.New()创建router实例，但是会默认使用logger和Recover中间件
- Logger是负责进行打印并输出日志的中间件，方便开发者进行程序调试
- Recovery中间件的作如果程序执行过程中遇到panc中断了服务,则 Recovery会恢复程序执行,并返回服务器500内误。
通常情况下,我们使用默认的gin.Defaul()创建 router实例

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"io"
	"net/http"
	"os"
	"time"
)

func main() {
	// Creates a router without any middleware by default
	r := gin.New()

	// Global middleware
	// Logger middleware will write the logs to gin.DefaultWriter even if you set with GIN_MODE=release.
	// By default gin.DefaultWriter = os.Stdout
	r.Use(gin.Logger())

	//gin.DefaultErrorWriter = io.MultiWriter(f)


	r.Use(gin.Recovery())

	// Per route middleware, you can add as many as you desire.
	r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

	// Authorization group
	// authorized := r.Group("/", AuthRequired())
	// exactly the same as:
	authorized := r.Group("/")
	// per group middleware! in this case we use the custom created
	// AuthRequired() middleware just in the "authorized" group.
	authorized.Use(AuthRequired())
	{
		authorized.POST("/login", loginEndpoint)
		authorized.POST("/submit", submitEndpoint)
		authorized.POST("/read", readEndpoint)

		// nested group
		testing := authorized.Group("testing")
		testing.GET("/analytics", analyticsEndpoint)
	}

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}

func analyticsEndpoint(context *gin.Context) {
	context.String(http.StatusOK,"qqq")
}

func benchEndpoint(context *gin.Context) {
	context.String(http.StatusOK,"xxx")
}


// 中间件2  返回body
func MyBenchLogger() gin.HandlerFunc {
	return func(cc *gin.Context){
		fmt.Println(cc.Request.Body)
	}
}

// 中间件1 返回各类信息
func AuthRequired() gin.HandlerFunc {
	return func(context *gin.Context) {
		host := context.Request.Host
		url := context.Request.URL
		method := context.Request.Method
		fmt.Println("-----------------------------")
		fmt.Printf("%s::%s \t %s \t %s ", time.Now().Format("2006-01-02 15:04:05"), host, url, method)
		context.Next()
		fmt.Println("------------------------------")
		fmt.Println(context.Writer.Status())
	}
}

// 路由
func loginEndpoint(c *gin.Context){
	c.String(http.StatusOK,"helloworld")
}
// 路由
func submitEndpoint(c *gin.Context){
	c.String(http.StatusOK,"word")
}
// 路由
func readEndpoint(c *gin.Context){
	c.String(http.StatusOK,"xx")
}


```


如上 `MyBenchLogger` `AuthRequired` 是两个中间件
中间件有两种使用方法，分别是
给单独一个Api使用
```go
r.GET("/benchmark", MyBenchLogger(), benchEndpoint)
```
给一个API组使用
```go
authorized := r.Group("/")
	// per group middleware! in this case we use the custom created
	// AuthRequired() middleware just in the "authorized" group.
authorized.Use(AuthRequired())
{
	authorized.POST("/login", loginEndpoint)
	authorized.POST("/submit", submitEndpoint)
	authorized.POST("/read", readEndpoint)
	// nested group
	testing := authorized.Group("testing")
	testing.GET("/analytics", analyticsEndpoint)
}
```


### 全局日志文件
```go

router := gin.New()
// 添加自定义的 logger 中间件
router.Use(middleware.Logger(), gin.Recovery())


func Logger() gin.HandlerFunc {
	return func(cc *gin.Context){
		fmt.Println(cc.Request.Body)
	}
}

```


### 局部中间件
主要用于权限操作

```go
package middleware

import "github.com/gin-gonic/gin"

// 定义中间件
func Auth() gin.HandlerFunc {
	return func(context *gin.Context) {
		println("已经授权")
		context.Next()
	}
}

// 在执行某个接口的使用 执行中间街
userRouter.GET("/profile/", middleware.Auth(), handler.UserProfile)
userRouter.POST("/update", middleware.Auth(), handler.UpdateUserProfile)
```

## 日志处理
```go
func main(){
    // Creates a router without any middleware by default
	r := gin.New()

	// Global middleware
	// Logger middleware will write the logs to gin.DefaultWriter even if you set with GIN_MODE=release.
	// By default gin.DefaultWriter = os.Stdout
	//r.Use(gin.Logger())

	// Recovery middleware recovers from any panics and writes a 500 if there was one.
	gin.DisableConsoleColor()

	f,_:= os.Create("gin.log")
	gin.DefaultWriter = io.MultiWriter(f)
}
```
这时候会将输出日志输出到gin.log下面


### 日志输出格式化
```go
func main() {
	router := gin.New()

	// LoggerWithFormatter middleware will write the logs to gin.DefaultWriter
	// By default gin.DefaultWriter = os.Stdout
	router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {

		// your custom format
		return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
				param.ClientIP,  // 客户端地址
				param.TimeStamp.Format(time.RFC1123), // 时间格式化
				param.Method, // 请求方法
				param.Path, // 请求路径
				param.Request.Proto, // 请求协议
				param.StatusCode, // 状态码
				param.Latency, // 请求时间
				param.Request.UserAgent(), // 代理
				param.ErrorMessage, // 错误信息
		)
	}))
	router.Use(gin.Recovery())

	router.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	router.Run(":8080")
}
```

### 取消和开启日志颜色
```go
gin.ForceConsoleColor()
gin.DisableConsoleColor()
```


## Model绑定
```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)


type Login struct{
	// 字段  字段类型 数据表字段  json小写 xml小写 必要字段
	User string `from:"user" json:"user" xml:"user" binding:"required"`
	Password string `from:"password" json:"password" xml:"password" binding:"required"`
}


func main() {
	router := gin.Default()
	// 绑定json  JSON ({"user": "manu", "password": "123"})
	router.POST("/loginJSON", func(c *gin.Context) {
		var json Login
		// json格式检查
		if err := c.ShouldBindJSON(&json); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		// 如果账号密码不对
		if json.User != "manu" || json.Password != "123" {
			c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
			return
		}

		c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
	})

	// 绑定xml
	router.POST("/loginxml",func(c *gin.Context){
		var xml Login
		if err := c.ShouldBindXML(&xml);err != nil{
			c.JSON(http.StatusBadRequest,gin.H{"error":err.Error()})
			return
		}

		if xml.User != "manu" || xml.Password != "123"{
			c.JSON(http.StatusUnauthorized,gin.H{"status":"unauthorized"})
			return
		}
		c.JSON(http.StatusOK,gin.H{"status":"you are logged in"})
	})

	// 绑定表单
	router.POST("/loginForm", func(c *gin.Context) {
		var form Login
		// This will infer what binder to use depending on the content-type header.
		if err := c.ShouldBind(&form); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		if form.User != "manu" || form.Password != "123" {
			c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
			return
		}

		c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
	})
	
	

	router.Run("
```


```go
package main


// 启动文件
import (
	"fmt"
	"github.com/gin-gonic/gin"
	"log"
)

func main() {
	// 创建默认路由
	router := gin.Default()
	// 默认路由
	router.GET("/",rootPath)
	// ping server
	router.GET("/ping",pingServer)
	// name
	// get xx/v1/version/hzj
	router.GET("/v1/version/name/:name",GetName)
	// query name
	// get xx/v1/version/?name=hzj
	router.GET("/v1/version/",getQueryName)
	// post username form
	router.POST("/v1/version/login/",postFormByLogin)
	// get many
	router.GET("/v1/version/many/",getNameAndClassid)
	// post many
	router.POST("/v1/version/many",postNameAndClassid)

	// 返回byte类型数组
	router.GET("/v1/version/byte",getfullpathBybytes)
	// 返回json类型
	router.GET("/v1/version/json",getfullpathByJson)


	// 启动 默认路由为8080
	router.Run()
}


// 路由
func rootPath(context *gin.Context){
	context.String(200,"helloworld")
}
// 测试服务器
func pingServer(context *gin.Context)  {
	context.String(200,"pong")
}

// 带路由参数
func GetName(context *gin.Context){
	name := context.Param("name")
	fmt.Println("返回名字"+name)
	fmt.Println("返回全路径:"+context.FullPath())
	context.String(200,"返回名字"+name+"\n")
	context.String(200,"返回全路径"+context.FullPath())
}
// 查询参数
func getQueryName(context *gin.Context){
	name := context.Query("name")
	context.String(200,"返回查询参数中的name"+name)
}

// 获取表单
func postFormByLogin(context *gin.Context){
	fmt.Println(context.FullPath())
	username, exist := context.GetPostForm("username")
	if exist  {
		context.String(200,username)
	}
}


// 创建json实例
type Student struct {
	Name    string `form:"name"`
	Classes string `form:"classes"`
}
// get query绑定实例
//http://127.0.0.1:8080/v1/version/many/?name=hzj&classes=123
func getNameAndClassid(context *gin.Context){
	context.String(200,context.FullPath())
	var student Student
	err := context.ShouldBindQuery(&student)
	if err != nil{
		log.Fatal(err.Error())
		return
	}
	msg := "\nhello"+student.Name+student.Classes
	context.String(200,msg)
}

// post绑定
// post http://127.0.0.1:8080/v1/version/many/
func postNameAndClassid(context *gin.Context){
	context.String(200,context.FullPath())
	var student Student
	err := context.ShouldBind(&student)
	if err != nil{
		fmt.Println(err)
		return
	}
	fmt.Println(student)
	msg := "\nhello"+student.Name+student.Classes
	context.String(200,msg)
}

// 返回切片类型
// get http://127.0.0.1:8080/v1/version/byte
func  getfullpathBybytes(context *gin.Context){
	fullpath := context.FullPath()
	context.Writer.Write([]byte(fullpath))
}


// 返回json类型
//
type Fullp struct {
	Fullpath string `from:"fullpath"`
	Name string `from:name"`
}

func getfullpathByJson(context *gin.Context){
	f := context.FullPath()
	var fp Fullp
	fp.Fullpath = f
	fp.Name = "1"
	context.JSON(200,&fp)
	context.String(200,"xx")
}
```


```go
package main


// 启动文件
import (
	"github.com/gin-gonic/gin"
)

func main() {

	//  创建默认路由
	router := gin.Default()

	// 创建分组
	UserRouter := router.Group("/v1/version/user")
	UserRouter.GET("/:userinfo",getUserInfo)
	// 创建分组
	LoginRouter := router.Group("/v1/version/login")
	LoginRouter.POST("/",postLogin)
	// 创建货物分组
	//GoodsRouter := router.Group("/v1/vesion/goods")


	// 启动 默认路由为8080
	router.Run()
}

// ======== UserRouter

// 获取用户信息
func getUserInfo(context *gin.Context){
	userinfo := context.Param("userinfo")
	context.String(200,userinfo)
}

// 登录
func postLogin(context *gin.Context){
	username := context.PostForm("username")
	password := context.PostForm("password")

	if username == "hzj" && password == "123"{
		context.String(200,"success")
	}else {
		context.String(404,"faild")
	}
}

```

## 使用swagger文档
https://www.liwenzhou.com/posts/Go/gin_swagger/


## gin连接mysql
https://zhuanlan.zhihu.com/p/108317058

## 如何导入本地包

https://www.liwenzhou.com/posts/Go/import_local_package_in_go_module/

## mvc模板嵌套
![2021-01-22-04-43-08](http://noback.upyun.com/2021-01-22-04-43-08.png)
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div>
        <span>helloworld</span>
        {{ . }}
		<!-- 将flag映射过来 -->

    </div>
</body>
</html>
```
```go
package  main

import (
	"github.com/gin-gonic/gin"
)

func Pong(context *gin.Context) {
	context.String(200,"pong")
}

func Index(context *gin.Context)  {
	// 三个参数分别是返回码，静态文件名字 以及参数内容
	// 参数内容主要就是放到静态页面中去渲染的，
	// 渲染的使用方法和django类似
	flag := true
	context.HTML(200,"index.html",flag)
}
func main() {
	// 创建路由控制器
	r := gin.Default()
	r.GET("/ping",Pong)


	// 加载静态文件
	r.LoadHTMLGlob("./template/*")
	//r.LoadHTMLFiles("./template/index.html","./template/new.html")
	r.GET("/index",Index)
	// 启动
	r.Run()
}
```
- LoadHTMLFiles 是加载单个文件，可以同时加载多个
- LoadHTMLGlob 是加载整个文件夹下面的模板文件
- 一级目录加载 `LoadHTMLGlob("./template/*")`
- 多级目录加载 `LoadHTMLGlob("./template/**/*")`
- 多级目录加载 `LoadHTMLGlob("./template/**/**/*")`

使用多级目录的时候还需要在`index.html`的头部做声明
```go
context.HTML(200,"user/index.html",flag)
```

```html
{{ define "user/index.html" }}
	<!-- html -->	

{{ end }}
```


> 多级目录嵌套

![2021-01-22-12-51-52](http://noback.upyun.com/2021-01-22-12-51-52.png)
如上 我们定义了多个模板文件夹
```go
package  main

import (
	"github.com/gin-gonic/gin"
)

func Pong(context *gin.Context) {
	context.String(200,"pong")
}


func main() {
	// 创建路由控制器
	r := gin.Default()

	// 加载文件模板
	r.LoadHTMLGlob("template/**/*")

	// 根据
	r.GET("/post/index", func(c *gin.Context) {
		c.HTML(200,"user/index.html",nil)
	})
	r.GET("/user/index", func(c *gin.Context) {
		c.HTML(200,"post/index.html",nil)
	})

	r.GET("/ping",Pong)
	r.Run()
}

```
```html
<!-- post/index.html -->

<!DOCTYPE html>
{{ define "post/index.html"}}
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <span>post/index</span>
</body>
</html>
{{ end }}

<!-- user/index.html -->

<!DOCTYPE html>
{{ define "user/index.html" }}
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <span>user/index</span>
</body>
</html>
{{ end }}

```
这样做的原因就是因为这样比较好识别,如果不采用这种形式的话，所有模板文件夹中的index HTML文件都会被认作为`index.html`文件


### 申明静态文件
```go
// 申明静态文件
	//r.Static("/static","static")
	r.StaticFS("/static",http.Dir("static"))
```
```html
    <img src="/static/xx.png" alt="">
```
第一个参数是静态文件的地址`当前文件夹static则是/static`
第二个参数是html文件中的地址


### 返回json数组
```go
package  main

import (
	"github.com/gin-gonic/gin"/
	"net/http"
)

type msg struct {
	Name    string `json:"user"`
	Message string
	Age     int
}
func main() {

	r  := gin.Default()
	r.GET("/", func(c *gin.Context) {
		c.JSON(http.StatusOK,msg{
			Name: "hellow",
			Message: "world",
			Age: 20,
		})
	})
	r.Run()
}
```

### 返回xml页面
```go
package  main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

type msg struct {
	Name    string `json:"user"`
	Message string
	Age     int
}
func main() {

	r.GET("/xml", func(c *gin.Context) {
		c.XML(200,msg{
			Name:"hellow",
			Message: "world",
			Age:20,
		})
	})
	r.Run()
}

```

### 返回yaml
```go
	r.GET("/yaml", func(c *gin.Context) {
		c.YAML(200,msg{
			Name:"hellow",
			Message: "world",
			Age:20,
		})
	})
```
