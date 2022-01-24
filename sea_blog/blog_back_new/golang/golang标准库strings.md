---
title: golang标准库strings
categories:
  - golang
tags:
  - golang
  - golibrary
abbrlink: e7c4c27b
date: 2020-09-07 14:13:59
---

<!-- more -->


# golang标准库strings 


## 拼接字符串string
```go
package main

import (
	"bytes"
	"fmt"
	"strings"
)

// 拼接字符串

// 因此，这种拼接字符串的方式会导致大量的string创建、销毁和内存分配 这样的写法是错误的
func func1(ss []string) string{
	var str string
	for _,v :=range ss{
		str +=v
	}
	fmt.Println(str)
	return str
}

func func2(ss []string) string{
	var str bytes.Buffer
	for _,v := range ss {
		fmt.Fprint(&str,v)
	}
	fmt.Println(str.String())
	return str.String()
}


func func3(ss []string)string{
	var str strings.Builder
	for _,v := range ss {
		fmt.Fprint(&str,v) // 这个io stdin
	}
	fmt.Println(str.String())
	return str.String()
}

func main() {
	ss := []string{
		"A","B","C",
	}
	func1(ss)
	func2(ss)
	func3(ss)
}
```

最受用的是func3 速度快，占内存少


## Builder方法
```go
func (b *Builder) Write(p []byte) (int, error)
func (b *Builder) WriteByte(c byte) error
func (b *Builder) WriteRune(r rune) (int, error)
func (b *Builder) WriteString(s string) (int, error)
```
分别对应了四种内容
`byte数组`、`byte`、`rune`、`string`

`strings.Builder` 通过使用一个内部的 slice 来存储数据片段。当开发者调用写入方法的时候，数据实际上是被追加（append）到了其内部的 slice 上。
![2020-09-08-18-15-33](http://noback.upyun.com/2020-09-08-18-15-33.png)

根据上图可以知道strings通过不断的像buffer这个slice append内容来拼接字符串的，当你调用写入方法的时候，新的字节数据就被追加到 slice 上。如果达到了 slice 的容量（capacity）限制，一个新的 slice 就会被分配，然后老的 slice 上的内容会被拷贝到新的 slice 上。当 slice 长度很大时，这个操作就会很消耗资源甚至引起 内存问题。我们需要避免这一情况

在我们创建slice的时候，我们使用了`var ss = make([]string,10,10)`来规定初始的长度和容量，同样的`Grow`也可以初始化`Strings.Builder`的长度
```go
func (b *Builder) Grow(n int)
```

当调用 `Grow(n)` 时，我们必须定义要扩容的字节数n,`Grow(n)` 方法保证了其内部的 slice 一定能够写入 n 个字节。只有当 slice 空余空间不足以写入 n 个字节时，扩容才有可能发生
也就是说grow其实就是发生在扩容不足的时添加容量的那一个阶段
- builder len = 5
- builder cap = 10
- 当我们调用`grow(3)` 并不会扩容 因为够用
- 当我们调用`grow(10)` 会扩容 因为不够用
他会去和剩余长度进行比较

```go
if cap - len > n{
    true
} eles{
    false
}
```

最后得出的容量公式为 `原有容量*2+n`

当你预定义 strings.Builder 容量的时候还要注意一点。调用 `WriteRune()` 和 `WriteString()` 时，rune 和 string 的字符可能不止 1 个字节。因为，你懂的，`UTF-8` 的原因。




## golang strings包
```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"strings"
)

func main() {

	str_reader := strings.NewReader("helloworld"); // *Reader

	// 获取字符串长度
	fmt.Printf("字符串长度,%d\n",str_reader.Len()) // *Reader.Len()

	// 实例并初始化一个数组
	b := make([]byte,10)
	// 将字符串读取到数组b当中去
	n,err := str_reader.Read(b)  // *Reader.Read()

	if err!=nil{
		log.Fatal(err)
	}

	fmt.Printf("读取的字节数%d\n",n)
	fmt.Printf("读取后b数组中的内容%s\n",string(b[:]))
	fmt.Printf("被读取后的长度%d\n",str_reader.Len())


	//
	str_reader = strings.NewReader("helloworld")
	bb,err := str_reader.ReadByte()
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(string(bb))


	// 替换
	// 1. 替换规则
	rule := strings.NewReplacer("foo","bar") // *Replacer
	// 返回oldstrings替换后的拷贝内容
	newwords := rule.Replace("foofoo") // *Replacer.Replace
	fmt.Println(newwords)

	//
	buf := bytes.NewBufferString("hellowolrd") // 创建io.writer
	n,err = rule.WriteString(buf,"foofoo")
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(n)
	fmt.Println(buf)

	// 合并
	s := []string{"foo", "bar", "baz"}
	fmt.Println(strings.Join(s, ", "))

	// 判断两个编码字符是否相同
	fmt.Println(strings.EqualFold("go","GO"))  // true

	// 判断s是否有前缀字符串prefix
	fmt.Println(strings.HasPrefix("network","net")) // true

	// 判断s是否有后缀字符串suffix
	fmt.Println(strings.HasSuffix("network","work"))// true

	// 判断s中是否含有子字符串substr
	fmt.Println(strings.Contains("foobar","oo"))// true

	// 判断字符串s是否包含字符串chars中的任一字符。
	fmt.Println(strings.ContainsAny("foobar","o a")) // true

	// 返回子串sep在字符串s中第一次出现的文职
	fmt.Println(strings.Index("foobar","oo")) // 1


}

```


## 字符串操作

- 字符串长度；
- 求子串；
- 是否存在某个字符或子串；
- 子串出现的次数（字符串匹配）；
- 字符串分割（切分）为[]string；
- 字符串是否有某个前缀或后缀；
- 字符或子串在字符串中首次出现的位置或最后一次出现的位置；
- 通过某个字符串将[]string 连接起来；
- 字符串重复几次；
- 字符串中子串替换；
- 大小写转换；
- Trim 操作；


```go
package main

import (
	"fmt"
	"strings"
	"unicode"
)
// strings字符串操作

//字符串长度；
//求子串；
//是否存在某个字符或子串；
//子串出现的次数（字符串匹配）；
//字符串分割（切分）为[]string；
//字符串是否有某个前缀或后缀；
//字符或子串在字符串中首次出现的位置或最后一次出现的位置；
//通过某个字符串将[]string 连接起来；
//字符串重复几次；
//字符串中子串替换；
//大小写转换；
//Trim 操作；
//...

func main()  {
	//ss := "hello"
	//fmt.Println(getStringLen(ss))
	//返回一个空格
	fmt.Println(unicode.IsSpace)

	ss:="hello world"
	fmt.Println(strings.Fields(ss))

	ss="hello - world"
	fmt.Println(strings.FieldsFunc(ss,unicode.IsSpace))

	ss="foo,bar,baz"
	fmt.Println(SplitNString(ss,1,","))


}

// 获取字符串长度
func getStringLen(ss string)  int {
	return  len(ss)
}

// 字符串比较  不忽略大小写 返回int类型
func CompareString(ss string, ss2 string) int{
	return strings.Compare(ss,ss2)
}

// 字符串 忽略大小写 返回bool类型
func EqualFoldString(ss string, ss2 string) bool{
	return strings.EqualFold(ss,ss2)
}

// 子串在字符串中返回true
func ContainsSubstr(ss string , sub string) bool{
	return strings.Contains(ss,sub)
}
// 字符在字符串中返回true
func ContainsAnyChar(ss string, char string) bool{
	return strings.ContainsAny(ss,char)
}
// rune在字符串中返回true
func ContainsRuneChar(ss string,char rune) bool{
	return strings.ContainsRune(ss,char)
}

// 子串出现次数 ( 字符串匹配 )
//func Count(s, substr string) int {
//	// special case
//	if len(substr) == 0 {
//		return utf8.RuneCountInString(s) + 1
//	}
//	if len(substr) == 1 {
//		return bytealg.CountString(s, substr[0])
//	}
//	n := 0
//	for {
//		i := Index(s, substr)
//		if i == -1 {
//			return n
//		}
//		n++
//		s = s[i+len(substr):]
//	}
//}

// 计算某一个字符char 在字符串中出现的次数
func CountString(ss string,char string ) int{
	return strings.Count(ss,char)
}

// 字符串根据空格剪切
func FieldsString(ss string) []string  {
	return FieldsString(ss)
}

// 字符或子串在字符串中首次出现的位置
func IndexChar(ss string) int {
	return strings.Index(ss,"h")
}

// 将字符串数组（或 slice）连接起来可以通过 Join 实现，函数签名如下：
func JoinString(ss []string) string{
	return strings.Join(ss,"-")
}

// 字符串重复几次
func RepeatString(ss string) string{
	return strings.Repeat(ss,2)
}

// 分割字符串 返回字符串数组
func SplitString(ss string,seq string) []string{
	return strings.Split(ss,seq)
	//fmt.Printf("%q\n", strings.Split("foo,bar,baz", ","))
	//["foo" "bar" "baz"]
}

// 分割字符串 返回字符串数组保留分割符
func SplitAfterString(ss string,seq string ) []string{
	return strings.SplitAfter(ss,seq)
	//	fmt.Printf("%q\n", strings.SplitAfter("foo,bar,baz", ","))
	//	["foo," "bar," "baz"]
}

// 分割字符换，返回N个元素
// -1 全部分割
// 0 返回空字符串
// 1 返回原字符串数组
func SplitNString(ss string,index int,seq string) []string{
	return strings.SplitN(ss,seq,index)
}

// 字符串是否有某个前缀或后缀
func HasPrefixString(s,prefix string) bool{
	return strings.HasSuffix(s,prefix)
}

// 字符串是否存在某个后缀
func HasSuffixString(s,prefix string)bool{
	return strings.HasSuffix(s,prefix)
}

// 字符串替换
// -1 表示全部替换
// 0 表示不替换
// >0 表示替换前面n位
func ReplaceString(now_s,old_s,new_s string,n int) string{
	return strings.Replace(now_s,old_s,new_s,n)
}

// 字符串子串替换
func ReplaceAllString(now_s,old_s,new_s string) string{
	return strings.ReplaceAll(now_s,old_s,new_s)
}

// 大小写替换

func ToLowerString(s string) string{
	return strings.ToLower(s)
}

func ToUpper(s string) string{
	return strings.ToUpper(s)
}

// 标题大小写替换
func TitleString(s string) string{
	return strings.Title(s)
}

// 标题处理
func ToTitle(s string) string{
	return strings.ToTitle(s)
}


//https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter02/02.1.html
```

