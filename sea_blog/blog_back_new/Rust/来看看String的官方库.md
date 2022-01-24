---
title: String的官方库
categories:
  - rust
tags:
  - rust
abbrlink: 8b7d4b87
date: 2020-08-13 18:26:35
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->


# String的官方库



## 字符串的容量和长度
```rust
fn main() {
    // 创建容量为10的空字符串
    let mut s = String::with_capacity(10);

    // assert_eq!(s.len(),10); //这里报错，容量不等于长度


    let cap = s.capacity(); // 获取s的容量 容量>=s.len
    for _ in 0..10{
        s.push('a');
    }

    assert_eq!(s.capacity(),cap);

    // s.push('a'); // 这里会报错

    // 创建容量为20的空字符串
    let mut ss:String = String::with_capacity(20);
    // 向其中填入10个字符
    for _ in 0..10{
        ss.push('a');
    }
    // 这个时候ss的容量为20 ，长度为10
    let length = ss.len();
    let cap = ss.capacity();
    assert!(length<cap);

}
```

## 返回字符串的几个基本属性
Returns the raw pointer to the underlying data, the length of the string (in bytes), and the allocated capacity of the data (in bytes).
```rust
pub fn into_raw_parts(self) -> (*mut u8, usize, usize)
```
返回字符串的基本属性 ，分别是存储地址(指针), 字符串长度，字符串容量
```rust
#![feature(vec_into_raw_parts)]
let s = String::from("hello");

let (ptr, len, cap) = s.into_raw_parts();

let rebuilt = unsafe { String::from_raw_parts(ptr, len, cap) };
assert_eq!(rebuilt, "hello");

```

## 定义多个基本属性创建字符串
```rust
use std::mem;

fn main() {
    unsafe {
        let s = String::from("hello");

        let mut s = mem::ManuallyDrop::new(s);
        
        let ptr = s.as_mut_ptr();
        let len = s.len();
        let capactity = s.capacity();

        let s = String::from_raw_parts(ptr, len, capactity);

        assert_eq!(String::from("hello"),s);
    }
}

```

## 提取包含整个的字符串切片&str
```rust
pub fn as_str(&self) -> &str
```
获取自身，返回切片类型
```rust
fn main() {
    let string_one  = String::from("helloworld");
    assert_eq!("helloworld",string_one.as_str());
}
```

## 提取包含整个的字符串切片&mut str
```rust
pub fn as_mut_str(&mut self) -> &mut str
```
获取自身，返回切片类型
```rust
fn main() {
    let mut s = String::from("hello");
    let sss = s.as_mut_str();
    assert_eq!("hello",sss);
}
```

## String添加&str内容，返回&str
```rust
fn main() {
    let mut s = String::from("hello");
    s.push_str("bar");
    assert_eq!("hellobar",s);
}
```
## String添加字符，返回&str
```rust
fn main() {
    let mut s = String::from("foo");
    s.push('b');
    assert_eq!("foob",s);
}
```

## 返回字符串容量
```rust
fn main() {
    let s = String::with_capacity(20);
    println!("{}",s.capacity());//20
}
```

## 对字符串进行扩容 reserve
字符串的本质也是一个集合，因此可以对字符串进行扩容，而保证原有的长度
```rust
fn main() {
    // let mut s = String::new();
    // s.reserve(10);
    // // let mut sss = String::from("helloworld");
    // //
    // // let mut ss = String::with_capacity(20);
    // // for i in 1..9{
    // //     ss.push('a');
    // // }

    // assert!(s.capacity() >= 10);
    // 创建一个容量只有10的集合
    let mut  v:Vec<i32> = Vec::new();
    for i in 1..9{
        v.push(1);
    } 
    println!("{}",v.len()); // 9
    println!("{}",v.capacity()); // 9

    // 对集合进行扩容
    v.reserve(20);
    println!("{}",v.len()); // 9 
    println!("{}",v.capacity()); // 29
}


fn main() {
    let mut  s = String::from("helloworld");
    println!("{}",s.len());  // 10
    println!("{}",s.capacity()); // 10 

    s.reserve(20);

    println!("{}",s.len()); // 10 
    println!("{}",s.capacity()); // 30
}
```

## 字符串插入单个字符  insert
```rust
fn main() {
    let mut  s = String::from("hellow");
    s.insert(1,'d');
    assert_eq!(s,"hdellow");
}
```


## 字符串插入字符串切片 insert_str
```rust
fn main() {
    let mut  s = String::from("hello");
    s.insert_str(5,"world");
    assert_eq!("helloworld",s)
}
```