---
title: Rust中的各种String
date: 2021-03-10 11:48:30
categories:  [rust]
tags: [rust]
---


<!--more-->


# Rust中的各种String

Rust中提供了三种不同的字符串变量类型
- String 堆上的
- &str  
- &String
String是一个可变的、堆上分配的UTF-8的字节缓冲区。
str是一个不可变的固定长度的字符串，如果是从String解引用而来的，则指向堆上，如果是字面值，则指向静态内存。
因此 &str 有两种形态 
- 一种是直接表示字面量 
- 另外一种是由堆上的String 解引用而来

Rust 中有一个表示字符串的原始（primitive）类型 str。str 是字符串切片（slice），每个字符串切片具有固定的大小并且是不可变的。通常不能直接访问 str ，因为切片属于动态大小类型（DST）
所以，只能通过引用（&str）间接访问字符串切片


通过字面量创建&str
```rust
let a: &str  = "foo";
let b: &str = "bar";
let aa: &str = &a[..3];
```
通过String类型解引用而来
```rust
let s: String = String::from("Hello");
let ss: &str = &s;
```

创建String类型，String 是一个分配在堆上的可增长的字符串类型
```rust
let c: String =  String::new();
```
通过字面量构造String
```rust
let c:String = String::from("foo");
```


拼接字符串
String 可以和&str 拼接
```rust
let mut a = "foo";
let mut b = "bar";
let mut d = String::from("bar");
// let ad = a+d;
let ad = d+a;
// 或者
let ad = d.push_str(a)
```
整个的过程中d的所有权发生转换，aa的内容被复制到了新的字符串当中

从 String 到 str 的转换是廉价的，反之，从 str 转为 String 需要分配新的内存。
```rust
fn main(){
    let a: &str = "hello world";
    let b: &str = "OK";
    let mut s: String = String::from("Hello Rust");
    println!("{}", s.capacity());    // prints 12
    s.push_str("Here I come!");
    println!("{}", s.len()); // prints 24
    let s: &str = "Hello, Rust!";
    println!("{}", s.capacity()); // compile error: no method named `capacity` found for type `&str`
    println!("{}", s.len()); // prints 12
}
```
如上 a b都表示 &str字面量  s 表示String  在堆上 动态长度
因此s会有一个容量 但是ab 只有一个len 因为他们是固定长度的
&str 是 str的一个的borrowed 类型，可以称为一个字符串切片，一个不可变的string

对String的引用类型为&String 也可以看成是&str


相互转换
```rust
let a: &str = "zz";
let c: String = a.to_string();
let d: String = String::from(b);
let d: String = a.to_owned();
```

```rust
let e = &String::from("Hello Rust");
// 或使用as_str()
let e_tmp = String::from("Hello Rust");
let e = e_tmp.as_str();
// 不能直接这样使用 
// let e = String::from("Hello Rust").as_str()
```
如果只想要一个字符串的只读视图，或者&str作为一个函数的参数，那就首选&str。如果想拥有所有权，想修改字符串那就用String吧。