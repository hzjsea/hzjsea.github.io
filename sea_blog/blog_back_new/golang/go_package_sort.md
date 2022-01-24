---
title: go_package_sort
date: 2021-06-16 15:51:07
categories:  [golang,go_package]
tags: [golang,go_package]
---


<!--more-->


# go_package_sort


go里面有个sort的排序包,用的都是优化后的排序算法,使用示例如下
```golang
package main

import (
	"fmt"
	"sort"
)

func main() {
	intList := [] int {2, 4, 3, 5, 7, 6, 9, 8, 1, 0}
	float8List := [] float64 {4.2, 5.9, 12.3, 10.0, 50.4, 99.9, 31.4, 27.81828, 3.14}
	stringList := [] string {"a", "c", "b", "d", "f", "i", "z", "x", "w", "y"}

	sort.Ints(intList)
	sort.Float64s(float8List)
	sort.Strings(stringList)

	fmt.Printf("%v\n%v\n%v\n", intList, float8List, stringList)

	//[0 1 2 3 4 5 6 7 8 9]
	//[3.14 4.2 5.9 10 12.3 27.81828 31.4 50.4 99.9]
	//[a b c d f i w x y z]

}

```

sort.Ints源码如下
```golang
func Ints(a []int) { Sort(IntSlice(a)) }
```
Sort接收一个Interface的接口类型
```golang
func Sort(data Interface) {
	n := data.Len()
	quickSort(data, 0, n, maxDepth(n))
}

type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

IntSlice 实现了这些接口，并返回一个`[]int`切片
```golang
type IntSlice []int

func (p IntSlice) Len() int           { return len(p) }
func (p IntSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p IntSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// Sort is a convenience method.
func (p IntSlice) Sort() { Sort(p) }
```


## 降序排序

```golang
func main() {
	intList := [] int {2, 4, 3, 5, 7, 6, 9, 8, 1, 0}
	float8List := [] float64 {4.2, 5.9, 12.3, 10.0, 50.4, 99.9, 31.4, 27.81828, 3.14}
	stringList := [] string {"a", "c", "b", "d", "f", "i", "z", "x", "w", "y"}

	//[0 1 2 3 4 5 6 7 8 9]
	//[3.14 4.2 5.9 10 12.3 27.81828 31.4 50.4 99.9]
	//[a b c d f i w x y z]

	sort.Sort(sort.Reverse(sort.IntSlice(intList)))
	fmt.Println(intList)
}
```



参考案例
https://itimetraveler.github.io/2016/09/07/%E3%80%90Go%E8%AF%AD%E8%A8%80%E3%80%91%E5%9F%BA%E6%9C%AC%E7%B1%BB%E5%9E%8B%E6%8E%92%E5%BA%8F%E5%92%8C%20slice%20%E6%8E%92%E5%BA%8F/