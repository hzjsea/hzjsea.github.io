---
title: golang标准库json
categories:
  - golang
tags:
  - golang
abbrlink: e1ec54ee
date: 2020-08-23 21:31:27
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->

# go库json

go里面处理json数据类型主要使用的是 `"encoding/json"` 这个库，可以自动引用。
```go
func Marshal(v interface{}) ([]byte, error )  // 序列化
func Unmarshal(data []byte, v interface{}) error // 反序列化
```

## 构建json数据
- struct、slice、array、map都可以转换成json
- struct转换成json的时候，只有字段首字母大写的才会被转换
- map转换的时候，key必须为string
- 封装的时候，如果是指针，会追踪指针指向的对象进行封装

## golang StructToJson工具
https://www.sojson.com/json/json2go.html
过于复杂的json 转struct 我们可以用工具来解决

## json序列化
```go
package  main

import (
	"encoding/json"
	"fmt"
)

type Student struct {
	Name    string
	Age     int
	Guake   bool
	Classes []string
	Price   float32
}


func main() {
	st := &Student {
		"Xiao Ming",
		16,
		true,
		[]string{"Math", "English", "Chinese"},
		9.99,
	}
	
	b,err := json.Marshal(st) // 序列化
	if err == nil{
		fmt.Println(string(b))
    }
}

```

`Marshal()`返回的是一个`[]byte`类型,转成字符串类型string(json.Marshal(st))

对输出的格式进行美化，可以使用`MarshalIndent`对输出的格式进行美化
```go
fn main(){
	b,err := json.MarshalIndent(st,"","\t")
	if err == nil{
		fmt.Println(string(b))
	}
}
```


## structTojson
如果想要使用小写字母来标记，可以用tag来标记在转换成json时候的字段
```go
// type Post struct {
// 	Id      int      `json:"ID"`
// 	Content string   `json:"content"`
// 	Author  string   `json:"author"`  // 这里做了标记
// 	Label   []string `json:"label"`
// }

package  main

import (
	"encoding/json"
	"fmt"
)

type Post struct {
	Id      int      `json:"ID"`
	Content string   `json:"content"`
	Author  string   `json:"author"`  // 这里做了标记，也就是转换之后再json里面的字段是怎样的
	Label   []string `json:"label"`
}

func main() {
	pp := Post{1,"hellow","hzj",[]string{"1","2"}}
	p,err := json.Marshal(pp)
	if err == nil{
		fmt.Println(string(p))
	}
}

// {"ID":1,"content":"hellow","author":"hzj","label":["1","2"]}

```

tag的用处
- tag可以设置为`json:"-"`来表示本字段不转换为json数据，即使这个字段名首字母大写
- 如果想要json key的名称为字符"-"，则可以特殊处理`json:"-,"`，也就是加上一个逗号
- 如果tag中带有,omitempty选项，那么如果这个字段的值为0值，即false、0、""、nil等，这个字段将不会转换到json中
- 如果字段的类型为bool、string、int类、float类，而tag中又带有,string选项，那么这个字段的值将转换成json字符串

```go
type Post struct {
	Id      int      `json:"ID,string"`  // 转化成json时候 类型为string
	Content string   `json:"content"`
	Author  string   `json:"author"`
	Label   []string `json:"label,omitempty"` // 0值，不会出现在json中
}
``` 



## jsonToMap
```go

package  main

import (
"encoding/json"
"fmt"
)

type Student struct {
	Name    string
	Age     int
	Guake   bool
	Classes []string
	Price   float32
}


func main() {
	st := &Student {
		"Xiao Ming",
		16,
		true,
		[]string{"Math", "English", "Chinese"},
		9.99,
	}
	b,err := json.Marshal(st) // 序列化
	if err == nil{
		fmt.Println(string(b))
	}

//	将 json转换成map

	var m map[string]interface{}
	_ = json.Unmarshal(b,&m)
	fmt.Println(m)

}
```