---
title: rust-slice
date: 2021-04-20 11:35:10
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust-slice
slice 就是一个动态数组，在rust中动态数组的数据类型为vector `Vec<i32>`
静态数组的类型为 arrary 也就是`[u8;len]`



在类型表示上`[u8]`表示一个静态数组的类型，这就涉及到了`胖指针`和`裸指针`的问题了,其中胖指针除了指针地址以外还有其他的信息，比如变量的长度信息 `[u8]和str`一样不包含长度信息，但是`&[u8]`和`&str`才有length信息，这就决定了他们是否能在编译


在函数中表示形参时
- 动态数组 `fn test(s:&vec<i32>) -> Vec<i32>`
- 静态数组 `fn test(s:&[u8]) -> [u8]`


## 静态数组array

一个 array 是一组相同类型的数据集合，这些数据位于连续的内存块中
```rust
fn main(){
    let array: [i32; 4] = [42, 10, 5, 2];
}
```

## 动态数组vector
Array 最大的一个限制是它的固定大小。与之相对，vector 可以在运行时扩容

```rust
fn main(){
        //There are three elements in the vector initially
        let mut v: Vec<i32> = vec![1, 2, 3];
        //prints 3
        println!("v has {} elements", v.len());
        //but you can add more at runtime
        v.push(4);
        v.push(5);
        //prints 5
        println!("v has {} elements", v.len());
}
```

<div style='font-size:20px;color:red'>自动扩容方式
Vector 是如何做到在运行时扩容的呢？在其内部，vector 把所有的元素放在一个分配在堆（heap）上的 array 上。当一个新元素被 push 进来时，vector 检查 array 是否有足够的剩余空间。如果空间不足，vector 就分配一个更大的 array，将所有的元素都拷贝到这个新的 array 中，然后释放旧的 array。这可以在下面的代码中验证：
</div>

![](https://noback.upyun.com/2021-04-20-11-47-16.png!)

## 切片slice
切片可以由上面两种类型产生
动态数组的切片
![](https://noback.upyun.com/2021-04-20-13-38-14.png!)
静态数组的切片，但是 `buffer pointer` 不是指向堆（heap）上的 buffer，而是指向栈（stack）上的 array。

可变引用的唯一性，下面这段程序就跑不了，因为引用不一致，出现在了两个地方
当 slice 被创建时，它指向 vector 内部 buffer 的第一个元素并且当一个新元素被 push 进 vector 时，它（指 vector）会分配一个新的 buffer 并且旧的 buffer 会被释放。这就导致 slice 指向了一个无效的内存地址，如果访问这个无效地址则会导致未定义行为。Rust 再一次从灾难中拯救了你。
```rust
fn main() {
    let mut v: Vec<i32> = vec![1, 2, 3, 4];
    let s = &v[..];
    v.push(5);
    println!("First element in slice: {:}", s[0]);
}
```


## 表现手法

https://stackoverflow.com/questions/57754901/what-is-a-fat-pointer

```rust
fn test(s:&str) -> String{
    s.to_string()
}

fn test_vec(s:&[u8]) -> Vec<u8>{
    s.to_vec()
}
```

编译期间的大小处理问题，rust在编译期间必须知道数据类型的大小，
![](https://noback.upyun.com/2021-04-20-14-01-06.png!)
如上图，在编译期间无法判断数据类型的大小，因此要使用胖指针引用来解决这个问题
`[u8] -> &[u8]`
`String -> &str`