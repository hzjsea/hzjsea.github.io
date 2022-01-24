---
title: rust各种宏
date: 2021-03-15 17:11:46
categories:  [rust]
tags: [rust]
top: 1
---


<!--more-->


# rust常用宏

宏(macro)是Rust的一个重要特性。Rust的“宏”（macro）是一种编译器扩展，它的调用方式为`some_macro!(...)`，比如`println!(...)`。宏调用与普通函数调用的区别可以一眼区分开来，凡是宏调用后面都跟着一个感叹号。宏也可以通过`some_macro![...]`和`some_macro！{...}`两种语法调用，只要括号能正确匹配即可。

## 宏的作用

使用宏，我们可以在编译阶段分析这个字符串常量和对应参数，确保它符合约定。另外一个常见的场景是，利用宏来检查正则表达式的正确性
```rust
fn main(){
    println!("file {} line {}",file!(),line!())
}
```
可以输出当前工作文件的名字和当前函数的行


## vec的宏源码
```rust
macro_rules! vec {
    ($elem:expr; $n:expr) => (
        $crate::vec::from_elem($elem, $n)
    );
    ($($x:expr),*) => (
        <[_]>::into_vec(box [$($x),*])
    );
    ($($x:expr,)*) => ($crate::vec![$($x),*])
}


fn main(){
    let v = vec![1,2,3,4,5];
}
```

## 自定义hashmap创建宏
```rust
macro_rules! hashmap {
    ($($key:expr => $val: expr),*) => {{
        let mut map = ::std::collections::HashMap::new();
        $(map.insert($key,$val);)*
        map;
    }};
}
fn main(){
    let counts = hashmap!['A' => 0, 'C' => 0, 'G' => 0, 'T' => 0];
    // 实际创建
    let counts =
        {
            let mut map = ::std::collections::HashMap::new();
            map.insert('A', 0);
            map.insert('C', 0);
            map.insert('G', 0);
            map.insert('T', 0);
            map
        };
}
```

支持重复，可以支持重复多个这样的语法元素。我们可以使用`+`模式和`*`模式来完成。类似正则表达式的概念，`+`代表一个或者多个重复，`*`代表零个或者多个重复。

展开宏
```rust
rustc -Z unstable-options --pretty=expanded temp.rs
```

## 自定义宏
```rust
macro_rules! pat {
    ($i:ident) => (Some($i))
}

fn main(){
    if let pat!(x) = Some(20){
        println!("{}",x)
    }
}
```

## println!
输出格式化，rust 提供了`std::fmt`标准库来完成格式化输出的工作

<font color='red'>几种格式化输出</font>

- `format!` 格式化输出成字符串 string
- `print!` 格式化输出并打印
- `println!` 格式化输出并打印完毕后空一行
- `eprint!` 格式化输出打印到标准错误`io::stderr`
- `eprintln!` 格式化输出打印到标准错误`io::stderr` 并空出一行

```rust

fn main() {
    //格式化成字符串
    format!("hello"); // => println!("hello")
    let s_string = format("helloworld")
    println!("{}",s_string) // 结果是 helloworld

    // 占位符模式
    println!("hello{}!","world");
    println!("{}{}","hello","world"); // => halloworld
    println!("{value}",value=4); // => 4
    println!("{:04}",42); //=> 0042

    // Debug模式
    format!("{:?}",(3.4)); // => res: (3,4)

    // 宽度  1和0就是识别后面传入参数的位置  没必要那么复杂的去写
    println!("Hello {:5}!", "x");  // => "Hello x    !"  默认左对齐
    println!("Hello {:1$}!", "x", 5); // => "Hello x    !"
    println!("Hello {1:0$}!", 5, "x"); // => "Hello x    !"
    println!("Hello {:width$}!", "x", width = 5);  // => "Hello x    !"

    // 格式
    assert_eq!(format!("Hello {:<5}!", "x"),  "Hello x    !"); // 默认左对齐
    assert_eq!(format!("Hello {:-<5}!", "x"), "Hello x----!"); // 输出-
    assert_eq!(format!("Hello {:^5}!", "x"),  "Hello   x  !"); // 居中
    assert_eq!(format!("Hello {:>5}!", "x"),  "Hello     x!"); // 右对齐

    // sign
    assert_eq!(format!("{:#x}!", 27), "0x1b!");  // 输出转换成16进制

     //输出指针地址
    println!("{:p}",&x);

}

```
![](http://noback.upyun.com/2021-02-14-21-06-13.png!)
rust中还有一系列的宏，如`format! write! writeln!` 可以看`std::fmt`中的文档  
https://doc.rust-lang.org/std/fmt/#traits


