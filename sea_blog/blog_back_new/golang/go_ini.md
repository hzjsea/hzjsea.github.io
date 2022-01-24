---
title: go_ini
date: 2021-06-15 14:09:11
categories:  [golang]
tags: [golang]
---


<!--more-->


# go_ini
ini文件解析库，官方地址 https://ini.unknwon.io/docs/howto/load_data_sources

文件解析样本
```golang
# possible values : production, development
app_mode = development

[paths]
# Path to where grafana can store temp files, sessions, and the sqlite3 db (if that is used)
data = /home/git/grafana

[server]
# Protocol (http or https)
protocol = http

# The http port  to use
http_port = 9999

# Redirect to correct domain if host header does not match domain
# Prevents DNS rebinding attacks
enforce_domain = true
```

> 读取文件

```golang
func main(){
    cfg,err := ini.load("my.ini")
    if err != nil{
        fmt.Printf("Fail to read file: %v", err)
        os.Exit(1)
    }
}
```

忽略不存在的文件
```golang
func main(){
    cfg, err := ini.LooseLoad("filename", "filename_404")
    if err != nil{
        fmt.Println("Fail to read file")
        os.Exit(1)
    }
}
```

> 无分区读写ini配置文件

```golang
func main(){
    app_mode := cfg.Section("").Key("app_mode").String()
}
```

> 预分配配置选择

可以在读取的时候设置默认值，以及一个配置参数数组，如果匹配到数组中的内容，则返回匹配结果，如果匹配不到，则返回默认值

```golang
func main(){
    protocol := cfg.Section("").key("protocol").In("http", []string{"http", "https"}))
}
```

> 自动类型转换

```golang
func main(){
    fmt.Printf("Enforce Domain: (%[1]T) %[1]v\n", cfg.Section("server").Key("enforce_domain").MustBool(false))
}
```

> 修改配置参数值，并保存

```golang
func main(){
    cfg.Section("").Key("app_mode").SetValue("production")
    cfg.SaveTo("my.ini.local")
}
```

> 分区

ini文件本身的设置有分区的概念，go-ini提供一下集中分区上的操作

获取分区，如果不存在则创建一个新的分区，源码如下
```golang
func (f *File) Section(name string) *Section {
	sec, err := f.GetSection(name)
	if err != nil {
		// Note: It's OK here because the only possible error is empty section name,
		// but if it's empty, this piece of code won't be executed.
		sec, _ = f.NewSection(name)
		return sec
	}
	return sec
}
```

> 获取分区

```golang
// 获取主分区
func main(){
    cfg,err := ini.load("my.ini")
    if err != nil{
        fmt.Printf("Fail to read file: %v", err)
        os.Exit(1)
    }
    
    cfg.Section("package").key("CLONE_URL").String()
}
// 获取子分区
func main(){
    cfg.Section("package.sub").Key("CLONE_URL").String()  
}
```

## 结构体与分区的双向绑定

配置如下
```golang
Name = Unknwon
age = 21
Male = true
Born = 1993-01-01T20:17:05Z

[Note]
Content = Hi is a good man!
Cities = HangZhou, Boston
```

```golang
type Note struct {
    Content string
    Cities  []string
}

type Person struct {
    Name string
    Age  int `ini:"age"`
    Male bool
    Born time.Time
    Note
    Created time.Time `ini:"-"`
}

func main() {
    cfg, err := ini.Load("my.ini")
    if err != nil{
        panic("file not read")
    }

    p := new(Person)
    err = cfg.MapTo(p)
    if err != nil{
        panic("not transfer")
    }

    // 一切竟可以如此的简单。
    err = ini.MapTo(p, "path/to/ini")
    // ...

    n := new(Note)
    err = cfg.Section("Note").MapTo(n)
    // ...
}
```