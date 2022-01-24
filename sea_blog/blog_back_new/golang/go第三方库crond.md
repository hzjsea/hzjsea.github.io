---
title: go第三方库crond
date: 2021-03-11 12:40:01
categories:  [golang]
tags: [golang]
---


<!--more-->


# go第三方库crond


这个库是一个计时的库，类似于linux的crond服务

地址: https://github.com/robfig/cron
doc: https://pkg.go.dev/github.com/robfig/cron

另外还有个比较好的任务调度的库，这里如果只做定时的话 cron完全可以满足了


v3版本中创建需要如下
```golang
    // v3版本中创建需要如下 c := cron.New(cron.WithSeconds())
    // 老版本 c := cron.New()
```

```golang
import "github.com/robfig/cron/v3"
```

## 使用

```golang
c := cron.New()
c.AddFunc("0 30 * * * *", func() { fmt.Println("Every hour on the half hour") }) // 规则和crond服务一样
c.AddFunc("@hourly",      func() { fmt.Println("Every hour") })  // 每小时
c.AddFunc("@every 1h30m", func() { fmt.Println("Every hour thirty") }) // 每一个半小时
c.Start() // 开始
c.Stop() // 暂停

spec := "@every 5m"  // 每5分钟
spec := "@every 1h"  // 每1小时
spec := "@every 1s"  // 每秒钟
c.AddFunc(spec,func(){ fmt.Println("helloword")})
```

> 对应关系如下

```bash
Entry                  | Description                                | Equivalent To
-----                  | -----------                                | -------------
@yearly (or @annually) | Run once a year, midnight, Jan. 1st        | 0 0 0 1 1 *
@monthly               | Run once a month, midnight, first of month | 0 0 0 1 * *
@weekly                | Run once a week, midnight between Sat/Sun  | 0 0 0 * * 0
@daily (or @midnight)  | Run once a day, midnight                   | 0 0 0 * * *
@hourly                | Run once an hour, beginning of hour        | 0 0 * * * *
```




## 使用案例

每分钟输出一段内容
```golang
fn main(){
    i := 0
    // v3版本中创建需要如下 c := cron.New(cron.WithSeconds())
    // 老版本 c := cron.New()
    c := cron.New(cron.WithSeconds())
    spec := "0 */1 * * * *"
    c.AddFunc(spec, func() {
        i++
        log.Println("execute per second", i)
    })
    c.Start()
    select {}
}
```

每天上午9点到12点的第2和第10分钟执行

```golang
package main

import (
    "fmt"

    "github.com/robfig/cron"
)

func main() {
    spec := "2,10 9-12 * * *" // 每天上午9点到12点的第2和第10分钟执行
    c := cron.New()
    c.AddFunc(spec, myFunc)
    c.Start()
    defer c.Stop()
    select {}
}

func myFunc() {
    fmt.Println("execute")
}
```

每5分钟输出一段内容

```golang
func InitCrond(){
	spec := "@every 5m"
	c := cron.New()
	c.AddFunc(spec, func(){
		fmt.Println("xx")
	})
	//c := cron.New()
	//c.AddFunc("0 * * * * *", func() { fmt.Println("Every hour on the half hour")})
	//cron.Start()
	c.Start()
	defer c.Stop()
	select {}
}
```