---
title: golang陷阱
date: 2021-02-05 14:08:15
categories:  [golang]
tags: [golang]
top: 1
---


<!--more-->


# golang陷阱


## 循环输出时变量的地址相同
这是因为输出的时候是在同一个变量地址删操作的，输出完上一个 当变得变量地址上的值重新赋值下一个
```golang
package main

import (
	"fmt"
)

func main() {
	listTemp := []int{1,2,3,4,4,5}
	for _, value := range listTemp{
		fmt.Println(&value)
	}

	var mapTemp = make(map[string]int)
	mapTemp["1"] = 1
	mapTemp["2"] = 2
	mapTemp["3"] = 3
	mapTemp["4"] = 4
	mapTemp["5"] = 5
	for _, value := range mapTemp{
		fmt.Println(&value)
	}
}
// 0xc000016098
// 0xc000016098
// 0xc000016098
// 0xc000016098
// 0xc000016098
// 0xc000016098
// 0xc0000160b0
// 0xc0000160b0
// 0xc0000160b0
// 0xc0000160b0
// 0xc0000160b0

```
解决的办法是使用临时变量做过渡

```golang
func main() {
	listTemp := []int{1,2,3,4,4,5}
	for _, value := range listTemp{
		temp := value
		fmt.Println(&temp)
		//fmt.Println(&value)
	}

	var mapTemp = make(map[string]int)
	mapTemp["1"] = 1
	mapTemp["2"] = 2
	mapTemp["3"] = 3
	mapTemp["4"] = 4
	mapTemp["5"] = 5
	for _, value := range mapTemp{
		temp := value
		fmt.Println(&temp)
		//fmt.Println(&value)
	}
}

// 0xc000016098
// 0xc0000160b0
// 0xc0000160b8
// 0xc0000160c0
// 0xc0000160c8
// 0xc0000160d0
// 0xc0000160d8
// 0xc0000160e0
// 0xc0000160e8
// 0xc0000160f0
// 0xc0000160f8
```
