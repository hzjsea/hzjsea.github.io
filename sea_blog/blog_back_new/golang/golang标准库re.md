---
title: golang标准库re
date: 2021-01-27 10:58:20
categories:  [golang]
tags: [golang,golibrary]
---


<!--more-->


# golang标准库




## golang正则
regexp是golang中自带的正则表达式的库，和python的正则库用法类似。
在golang中，会先使用生成一个正则的匹配规则，然后再用这个实例去调用各种匹配的方法
```golang
type Regexp struct {
	expr           string       // as passed to Compile
	prog           *syntax.Prog // compiled program
	onepass        *onePassProg // onepass program or nil
	numSubexp      int
	maxBitStateLen int
	subexpNames    []string
	prefix         string         // required prefix in unanchored matches
	prefixBytes    []byte         // prefix, as a []byte
	prefixRune     rune           // first rune in prefix
	prefixEnd      uint32         // pc for last rune in prefix
	mpool          int            // pool for machines
	matchcap       int            // size of recorded match lengths
	prefixComplete bool           // prefix is the entire regexp
	cond           syntax.EmptyOp // empty-width conditions required at start of match
	minInputLen    int            // minimum length of the input in bytes

	// This field can be modified by the Longest method,
	// but it is otherwise read-only.
	longest bool // whether regexp prefers leftmost-longest match
}
```

## 匹配方法

> golang中的方法去看库的时候发现其实很有规律，一般就是bytes一类方法，string一类方法，不同的对应对应一种类型的方法

返回该实例对象的代码如下,`Compile 与 MustCompile`两种方法，前者会返回err到结果当中，后者如果expr不规范会直接报panic
![2021-01-27-11-03-12](http://noback.upyun.com/2021-01-27-11-03-12.png)
```golang
	rule,err := regexp.Compile("\\w+") // 如果不适用``则需要转义
	rule  = regexp.MustCompile("\\w+")
	rule,err = regexp.Compile(`\w+`)
```

### 是否存在子序列
![2021-01-27-11-09-00](http://noback.upyun.com/2021-01-27-11-09-00.png)

检查是否存在子序列
```golang
	matched, err := regexp.MatchString("foo.*", "seafood")  
	fmt.Println(matched, err) // true nil
	matched, err = regexp.MatchString("bar.*", "seafood")
	fmt.Println(matched, err)// false nil
	matched, err = regexp.MatchString("a\\(b", "seafood")
    fmt.Println(matched, err)// false nil 
    matched,err = regexp.Match("foo.*",[]byte("seafood"))
	fmt.Println(matched,err) // true nil
	matched,err = regexp.MatchReader("中",strings.NewReader("中文"))
	fmt.Println(matched,err) // true nil
```

### 查找第一个子字符串
匹配字符串或者bytes数组，返回第一个匹配到的内容
![2021-01-27-11-26-54](http://noback.upyun.com/2021-01-27-11-26-54.png)
```go
	rule := regexp.MustCompile(`\w+`)
	str := "foox fooxf ooxofoxox"
	print(rule.FindString(str)) // foox 
```

### 查找第一个字符串的序列
![2021-01-27-11-37-01](http://noback.upyun.com/2021-01-27-11-37-01.png)
```go
    rule := regexp.MustCompile(`\w+`)
    str := "foox fooxf ooxofoxox"
	fmt.Println(rule.FindIndex(str,2)) //[0 4] 
```


### 查找所有的子字符串
匹配字符串或者bytes数组,返回一个存放所有匹配结果的数组,以及一个index 表示需要返回的序列
![2021-01-27-11-34-27](http://noback.upyun.com/2021-01-27-11-34-27.png)
```go
	rule := regexp.MustCompile(`\w+`)
	str := "foox fooxf ooxofoxox"
	fmt.Println(rule.FindAllString(str,-1)) // [foox fooxf ooxofoxox]
	fmt.Println(rule.FindAllString(str,2)) // [foox fooxf]
```

### 查找所有字符串的序列
匹配正则，并将匹配到的结果的index放到数组当中

![2021-01-27-11-37-19](http://noback.upyun.com/2021-01-27-11-37-19.png)
```go
	rule := regexp.MustCompile(`\w+`)
	str := "foox fooxf ooxofoxox"
	fmt.Println(rule.FindAllStringIndex(str,2)) //[[0 4] [5 10]]
```


### 查找匹配结果并返回正则中的分组
![2021-01-27-11-39-47](http://noback.upyun.com/2021-01-27-11-39-47.png)
![2021-01-27-11-40-31](http://noback.upyun.com/2021-01-27-11-40-31.png)

```go
	rule := regexp.MustCompile(`a(x*)b(y|z)c`)
	str := "axxxxxxbyc"
	substringlist := rule.FindAllStringSubmatch(str,-1)
	fmt.Println(substringlist) // [[axxxxxxbyc xxxxxx y]]
    fmt.Println(rule.NumSubexp()) // 2
}
```


### 匹配内容切割

```
n > 0 : 返回最多n个子字符串，最后一个子字符串是剩余未进行分割的部分。
n == 0: 返回nil (zero substrings)
n < 0 : 返回所有子字符串
```
```go
	res := regexp.MustCompile("a").Split("aaaadsddaasdasdasdas",3)
	for index,item :=range(res){
		fmt.Println(index,item)
    }
    // res
    // 0 
    // 1 
    // 2 aadsddaasdasdasdas
    res := regexp.MustCompile("a").Split("adsddaasdasdasdas",3)
	for index,item :=range(res){
		fmt.Println(index,item)
    }
    // 0 
    // 1 dsdd
    // 2 asdasdasdas
```

### 返回正则表达式转移后的字符串
```go
	res := regexp.QuoteMeta("\\s\\S")
    fmt.Println(res)  // \s\S
```


## 参考的内容如下
https://www.cnblogs.com/wanghui-garcia/p/10432389.html
https://www.cnblogs.com/golove/p/3270918.html