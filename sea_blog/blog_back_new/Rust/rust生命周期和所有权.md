---
title: rust生命周期和所有权
date: 2021-03-10 10:25:48
categories:  [rust]
tags: [rust]
---


<!--more-->


# 生命周期和所有权


所有权（系统）是 Rust 最为与众不同的特性，它让 Rust 无需垃圾回收（garbage collector）即可保障内存安全。因此，理解 Rust 中所有权如何工作是十分重要的
所有权的作用: 所有运行的程序都必须管理其使用计算机内存的方式。

对于一些放置在堆上的内容，不同的语言有不同的处理方式
比如 python 自带一个 gc(垃圾回收)来处理程序运行中不在使用的内容， java 中的 jvm 也是同样的道理
再比如 C 中的 free malloc 这些则需要程序员亲自分配和释放内存
但是在 Rust 中，则选择了第三种方式，通过所有权系统管理内存，编译器在编译时，会根据一系列的规则进行检查，在运行时，所有权系统的任何功能都不会减慢程序的运行

> 所有权规则

```rust
1. Rust 中的每一个值都有一个被称为其 所有者（owner）的变量。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者（变量）离开作用域，这个值将被丢弃。
```

rust 是根据作用域来判断是否需要释放内存的，在作用域内声明的所有变量，在离开作用域之前都是有效的，但是一旦离开作用域，rust 就会默认当前的变量已经失去了作用，就会被调用一个 drop 函数来释放这些变


### rust所有权与生命周期
rust中不同的数据类型 所有权以及生命周期处理的方式不同，一一道来

类型
 - 值类型  i32 bool u32 u64 struct Person{ age: i32 } 字符串字面量  栈上保存的局部变量在退出当前作用域的时候会自动释放  栈有一个确定的最大长度，超过了这个长度会产生“栈溢出”（stack overflow）
 - 引用类型  &str String &vec 、 &mut str  &mut vec  struct Person{ name: String }  堆上分配的空间没有作用域，需要手动释放  堆的空间一般要更大一些，堆上的内存耗尽了，就会产生“内存分配不足”（out of memory

处理方法
 - 值类型 存储在栈上,使用完之后会立即被收回，在rust当中就是离开当前作用域{}，值类型在右值（在值表达式中）执行赋值操作时，会自动复制一个新的值副本，也就是实现copy trait，在其他语言中，这个叫做浅拷贝
 - 引用类型，将数值存储在堆上，但是指针 长度 容量等存储在栈上，指针指的是 指向堆中数据的地址，因此对引用类型的操作效率一般比较低，使用完交给GC回收，没有GC的语言则需要靠手工来回收

Rust的主要设计目标之一，是在不用自动垃圾回收机制的前提下避免产生segfault。从这个意义上来说，它是独一无二的。至于它是如何做到的，本书将会详细剖析


> rust中的几种处理数据的方式

- 复制语义 copy trait  or clone
复制语义分为两种，一种是实现了`copy trait`的类型，一些基础变量 ，另一种则是在堆上的数据类型，如果要执行复制，则发生的是`clone`
发生`Copy trait`的时候会在栈上复制一份新的副本，新的地址  看`example0`
发生`Clone`的时候会复制一份栈上的地址和堆上的数据 看 `example1`


- 移动语义 move 
Rust中默认情况下都是move语义，在执行`let`赋值语句的时候发生的就是move语义
但是也有特殊的情况，比如上面介绍的一些基础数据类型以及字符串常量发生的是复制语义


```rust
fn main(){

    // -- example0
    let xx: i32 = 1;
    let xxx = xx;
    print!("{:p}\n",&xx); // 0x7ffee83b9518
    print!("{:p}\n",&xxx); // 0x7ffee83b951c

    // -- example1 
    let xx = String::from("xx"); // move语义
    print!("{:p}\n",xx.as_ptr()); // 格式化  0x7fbc3d402e40  print!("{:?}\n",xx.as_ptr()); // 0x7fbc3d402e40
    print!("{:p}\n",&xx); // 0x7ffeebc71430 栈上
    let xxx = xx.clone();
    print!("{:?}\n",xxx.as_ptr()); // 0x7fbc3d402f40
    print!("{:p}\n",&xxx);// 0x7ffeebc714f8 栈上

    print!("\n");

    let m = String::from("m");
    println!("{:p}",m.as_ptr());  // 0x7fafa7402f50
    println!("{:p}",&m); // 0x7ffee627a438
    let mm = return_str(&m);
    println!("{:p}",mm.as_ptr()); // 0x7fafa7402f50
    println!("{:p}",&mm); // 0x7ffee627a500
    // 只复制了栈上的一份指针


    print!("\n");

    let z = "xx";
    println!("{:p}",z.as_ptr());  // 0x104fd0e00 堆上的地址
    println!("{:p}",&z); // 0x7ffeeac53440 栈上的地址
    let zz = return_str(z); 
    println!("{:p}",zz.as_ptr());  // 0x104fd0e00 堆上的地址
    println!("{:p}",&zz); // 0x7ffeeac53500 栈上的地址
    // 只复制了栈上的一份指针

    print!("\n");

}


fn return_str(m:&str) -> &str{
    m
}

```
![](http://noback.upyun.com/2021-03-10-10-46-48.png!)



### 所有权操作

rust中的变量有两种属性 分别是`空间属性 也就是内存的管理(所有权)` 以及 `时间属性 也就是生命周期`
对于上面的两种语义
- 发生复制语义的时候 因为产生了新的副本，所以所有权也产生了新的
- 发生move语义的时候，所有权交接
- 对于只需要是有值，而不需要所有权的时候，可以使用引用(或者叫借用)

代码如下

> 复制语义-所有权

```rust
fn main(){
    let n1 = 20;
    {
        let n2 = 30;
        let n5 = the_max(n1, n2);
        print!("{}",n5);
    }
}

fn the_max(n3:i32, n4: i32) -> i32{
    if n3 > n4 {n3} else {n4}
}

```
如上 n3 n4 分别为n1 和 n2的新的副本, 从`the_max`返回后,产生了新的副本n5，整个过程发生的都是复制语义，所有权不断的在创建新的，生命周期也没有什么影响


> 移动语义-所有权

```rust
fn main(){
    let s1 = "xx".to_string();
    {
        let s2= "xxx".to_string();
        let s6 = the_longest(s1, s2);
        print!("{}",s6);
        // print!("{}",s2);//error  value borrowed here after move
    }
}

fn the_longest(s3: String, s4: String) -> String{
    if s3.len() > s4.len() {s3.to_string()} else {s4.to_string()}
}
```
如上 s3 s4 分别为s1 和 s2 的移动后的内容(move语义), 从`the_longest`返回后, s3 s4其中一个变量会move给s6

> 借用-所有权

```rust
fn main(){
    let s1 = "xx".to_string();
    {
        let s2 = "xxx".to_string();
        let s6 = the_longest(&s1, &s2);
        print!("{}",s6);
    }
}

fn the_longest<'a>(s3: &'a str, s4: &'a str) -> &'a str{
    if s3.len() > s4.len() {s3} else {s4}
}
```
如上 s3 s4 分别为s1 和 s2 的新的副本(发生借用 s3 s4 临时拥有s1 s2 的所有权 值和生命周期) 
但在返回的时候要考虑生命周期的问题， 因为 s1 的生命周期为a1 s2 的生命周期为a2 此时s3, s4 的生命周期为a1 a2   函数返回后的结构s6生命周期是a6 借用方s6的声明周期不能长于出借方s1,s2的生命周期 所以需要声明生命周期`'a`

```rust
fn main(){
    let xx  = "xx"; // 字面量
    let xxx = xx;
    println!("{:p}",&xx); // 0x7ffeec009460
    println!("{:p}",&xxx); // 0x7ffeec009470  在栈上复制了一份
    assert_eq!(xx,xxx);

    let yy  = "xx";
    let yyy = return_str(yy); // 发生借用 临时拥有所有权 值和生命周期 yyy = yy_copy
    println!("{:p}",&yy); // 0x7ffeeb70b2f0
    println!("{:p}",&yyy); // 0x7ffeeb70b300 
    assert_eq!(yy,yyy);

    let  gg = "xx".to_string();
    println!("{:?}",&gg); // "xx"
    let  ggg = return_str(&gg); // 借用 临时拥有所有权 值和生命周 在函数内部 m = "xx"
    println!("{:p}",&gg); 
    println!("{:p}",&ggg);
}


fn return_str(m:&str) -> &str{
    m
}
```

```rust

fn main(){
    let vv = [1,2];
    let vvv = change_value2(vv);
    println!("{:?}",vvv); // [10,2]

    let mut vvv = [1,2];
    change_value(&mut vvv);
    println!("{:?}",vvv); // [10,2]
}


fn change_value(v:&mut [i32;2]){
    v[0] = 10;
}

fn change_value2(mut v: [i32;2]) -> [i32;2]{
    v[0] = 10;
    v
}

```

- 规则一：借用的生命周期不能长于出借方（拥有所有权的对象）的生命周期。
- 规则二：可变借用（引用）不能有别名（Alias），因为可变借用具有独占性。
- 规则三：不可变借用（引用）不能再次出借为可变借用

### 借用

借用就要涉及到生命周期，不涉及函数时，借用就是引用

```rust
fn main(){
    let r:&i32;
    {
        let xx:i32 = 20;
        r = &xx; // 发生引用，出现指针  ^^^ borrowed value does not live long enough
    } // 出作用域 xx 被drop, 如果正常编译， 就会出现一个悬吊指针
    println!("{}",r);
}
```

如上，r的生命周期为`'a`, xx的生命周期为`'b` ，根据规则 借用方r的声明周期不能长于出借方xx的生命周期，所以这里会报错

标注生命周期参数并不能改变任何引用的生命周期长短，它只用于编译器的借用检查，来防止悬垂指针

> 禁止在没有任何输入参数的情况下返回引用

```rust
fn main(){
  let x = return_str();
}

fn return_str<'a>() -> &'a str{
    let s = "Rust".to_string();
    for i in 0..3{
        s.push_str("Good");
    }
    &s[..]
}
```

 报错 生命周期`'a`太短，解决办法把`'a str`改成`String` 调用函数的时候发生的是move



> 滥用生命周期

```rust
fn main(){
  let x = return_str();
}


fn return_str<'a>(s1:&'a str,s2:&'a str) -> &'a str{
    let result = String::from("xx");
    &result
}
```

整个操作和生命周期无关 不需要声明， 从函数中返回（输出）一个引用，其生命周期参数必须与函数的参数（输入）相匹配，否则，标注生命周期参数也毫无意义

```rust
fn main(){
    let one = String::from("xx");
    let x = return_str( &one,"yy");
    println!("{}",x);
}


fn return_str<'a>(s1:&'a str,s2:&'a str) -> String{
    let mut  s1 = s1.to_string();
    s1.push_str(s2);
    s1
}

```

> 拼接字符串问题

```rust
fn main(){
    let mut xx = "xx";
    xx.to_string().push_str("x"); // 无法执行并且不会报错 这是因为前面的to_string返回的是String  push_str 需要的是&mut self 也就是 &mut String
  
  	let mut xx = "xx".to_string();
    xx.push_str("x"); // 正常执行
}
```

> 需要标注生命周期的实例

```rust
fn main(){
    let s1 = "xx";
    {
        let s2 = "xx";
        let s3 = the_longest(s1, s2);    
        print!("{}",s3)    
    }
}


fn the_longest<'a>(s1: &'a str ,s2: &'a str) -> &'a str{
    if s1.len() > s2.len() {s1} else {s2}
}
```

![](https://noback.upyun.com/2021-03-10-10-47-35.png!)

> 在函数中扩展字符串的问题

```rust
fn main(){
    let s1 = "xx";
    {
        let s2 = "xx";
        let s3 = export_str(s1, s2);
        print!("{}",s3);
    }
}

fn export_str<'a>(s1: &'a str ,s2: &'a str) -> &'a str{
    let mut s3 = s1.to_string();
    s3.push_str(s2);
    &s3[..] // s3 is borrowed here
}
```

这里会报错，是因为拼接之后的s3是在函数内部产生的，离开函数的作用域后就会被drop掉，所以如果要成功编译，只能用move语义，代码如下
```rust
fn main(){
    let s1 = "xx";
    {
        let s2 = "xx";
        let s3 = export_str(s1, s2);
        print!("{}",s3);
    }
}

fn export_str(s1: &str ,s2: &str) -> String{
    let mut s3 = s1.to_string();
    s3.push_str(s2);
    s3
}

```

> 结构体中的声明周期

```rust
// struct Foo{
//     part: &str 
//                 /*
//                 --> src/main.rs:1:5
//             |
//             1 | use core::fmt;
//             |     ^^^^^^^^^
//             |
//             = note: `#[warn(unused_imports)]` on by default

//             error[E0106]: missing lifetime specifier
//             --> src/main.rs:31:11
//             |
//             31 |     part: &str
//             |           ^ expected named lifetime parameter
//             |
//             help: consider introducing a named lifetime parameter
//             |
//             30 | struct Foo<'lifetime>{
//             31 |     part: &'lifetime str
//             |
//                 */
// }

struct Foo<'a>{
    part: &'a str
}

fn main(){
    let words = String::from("main函数中声明了一个String类型的字符串words，然后使用split方法按逗号规则将words进行分割，再通过next方法和expect方法返回一个字符串切片first。其中next方法为迭代器相关的内容，第6章会讲到");
    let first  = words.split(',').next().expect("None");
    let f = Foo{
        part: first 
    };
    println!("{}",f.part);  
}
```

如果结构体中出现引用的内容，也需要声明生命周期，这里的生命周期参数标记，实际上是和编译器约定了一个规则：结构体实例的生命周期应短于或等于任意一个成员的生命周期
```rust
struct Foo<'a>{
    part: &'a str
}

impl<'a> Foo<'a>{
    fn split_first(s: &'a str ) -> &'a str{
        s.split(',').next().expect("None");
        s
    }

    fn new(s:&'a str) -> Self{
        Foo{
            part: Foo::split_first(s)
        }
    }
}


fn main(){
    let words = String::from("main函数中声明了一个String类型的字符串words，然后使用split方法按逗号规则将words进行分割，再通过next方法和expect方法返回一个字符串切片first。其中next方法为迭代器相关的内容，第6章会讲到");
    let f= Foo::new(&words);
    println!("{}",f.part);  
}
```
为带生命周期的结构体实现方法

![](https://noback.upyun.com/2021-03-10-10-47-49.png!)
静态字符串按位复制的仅仅是存储于栈上的地址




### 生命周期限定

生命周期参数可以像trait那样作为泛型的限定，有以下两种形式。·

-  T：＇a，表示T类型中的任何引用都要“活得”和＇a一样长。· 
-  T：Trait+＇a，表示T类型必须实现Trait这个trait，并且T类型中任何引用都要“活得”和＇a一样长 



## 智能指针
rust默认数据放在栈上面， 栈上面数据读取快
智能指针能把存在栈上的数据放到堆上面，栈上面只存放指针和堆上数据的长度， 指针指向堆上， 堆上存放元数据
```rust
struct Person{
  name: String
}

fn main(){
   let x = retrun_str();
   let xx  = Box::new(32);
   let p1 = Person{
       name : "xx".to_string()
   };
   let xxx = Box::<Person>::new(p1);
}
```