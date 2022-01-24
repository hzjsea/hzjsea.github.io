---
title: go_工程项目的测试与测试框架
date: 2021-05-18 16:43:11
categories:  [golang]
tags: [golang]
---


<!--more-->


# go_工程项目的测试与测试框架

go中有自带的test框架，但是用多了rust中的assert之后，go的test框架就有点不爽了。
当然golang中也有解释为什么不适用assert

https://golang.org/doc/faq#assertions

这里说到不推荐assert是因为在错误上不想让编写他的人偷懒。所以我们反向思考一下，如果要偷懒，我们就用下assert吧 qaq, 首先是 testify 这个框架 以及 gomock 这个框架，后面这个是用来生成假数据的

使用实例如下

```golang
package yours

import (
  "testing"
  "github.com/stretchr/testify/assert"
)

func TestSomething(t *testing.T) {

  // assert equality
  assert.Equal(t, 123, 123, "they should be equal")

  // assert inequality
  assert.NotEqual(t, 123, 456, "they should not be equal")

  // assert for nil (good for errors)
  assert.Nil(t, object)

  // assert for not nil (good when you expect something)
  if assert.NotNil(t, object) {

    // now we know that object isn't nil, we are safe to make
    // further assertions without causing any errors
    assert.Equal(t, "Something", object.Value)

  }

}

```


## gomock

https://github.com/golang/mock


## golang中的数组和切片

每个语言都要区分这两个，这两个在一定意义上是有区别的，静态数组与切片的区分 静态数组经常以arrary vector 来作为命名， 而切片就是 slice <div style='font-size:20px;color:red'>切片仅仅是列表上的一个视图</div>

在golang当中静态数组是一种值类型， 比方说`[10]string`与`[5]string`他们是两种不同的类型，分别是长度为10的string数组 以及 长度为5的string类型 
而切片就是静态数组上划了一个口，衍生出一个指针,切片是只有在运行时才会确定内容的结构,因此他常常被定义为`[]string` 只能定义切片中的类型，没有固定长度
切片在列表之上增加了一个动态层。从列表创建新的切片既不会分配新的内存也不会复制任何数据，切片仅仅是开在列表某一部分的窗口。从技术角度解释，切片可被视作一个struct，内含一个指针指向列表中处在切片开始位置的元素，以及两个整数分别描述切片的长度和容量。

```golang
type slice struct{
    Data uintptr
    Length int
    Capacity int
}
```
如下是切片的结构视图
![](https://noback.upyun.com/2021-05-18-17-14-06.png!)

> 初始化切片

1. 通过下标的方式获得数组或者切片的一部分； `arrary[3:10] or slice[3:10]`
2. 使用字面量初始化新的切片； `slice := []int{1, 2, 3}`
3. 使用关键字 make 创建切片：`slice := make([]int, 10)`

这里要讲一讲字面量初始化

```golang
func main(){
    slice := []int{1,2,3} // 切片
    arrary := [3]int{1,2,3} // 静态数组
    arrary1 := [...]int{1,2,3} // 静态数组
}
```