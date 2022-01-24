---
title:  rust所有权_old
date: 2021-03-19 12:25:10
categories:  [rust]
tags: [rust]
---


<!--more-->


#  rust所有权


常量基本数据结构存放在栈上，
其他的数据存放在堆上 
相应的指针也会存放在栈和堆上面， 一旦离开作用域 指针被释放，
比如这样
```rust
fn main(){
​    {
​        let a:i32  = 1;
​        let aa :String = "xx".to_string();
​        let aaa :&str = "xx";
​    }
​    // println!("{}",a);
​    // println!("{}",aa);
​    println!("{}",aaa);
}
```
a的作用域在第一个`{}`当中 离开作用域后 无论是指针还是存放的数据都会被自动释放掉


## 所有权

所有权
所有权规则
规则1：Rust 中的每一个值都有一个被称为其 所有者（owner）的变量。
规则2：值在任一时刻有且只有一个所有者。
规则3：当所有者（变量）离开作用域，这个值将被丢

### 作用域
想要知道所有权的死亡周期 就要知道作用域的边界 最简单的作用域就是{} 
```rust
fn main(){
​    let a = "xx";
​    println!("a = {}",a);
​    println!("a = {}",a);
}
```
以上 作用域就是main{} a在这个作用域内的生命周期是整个程序的运行过程
```rust
fn main(){
​    let a = "xx";
​    {                        // ----'b 
​        let b  = "xx";       //      | 
​    }                        // ----'b
​    println!("a = {}",a);
​    println!("a= {}",a);
}
```
如上 b的作用域就是'b 这个区间
move语义
move主要作用是交接所有权
```rust
fn main(){
​    let a = String::from("xx");
​    let b = a;
​    println!("a = {}",a);
}
error[E0382]: borrow of moved value: `a`
   --> src/main.rs:118:23
​    |
116 |     let a = String::from("xx");
​    |         - move occurs because `a` has type `std::string::String`, which does not implement the `Copy` trait
117 |     let _b = a;
​    |              - value moved here
118 |     println!("a = {}",a);
​    |                       ^ value borrowed here after move
```

交接所有权之后 a 不会在存在所有者， 离开作用域后被释放

这里错误提示中说道 他是一个没有实现`Copy trait`的变量，所以这里只能发生的move

Copy trait 浅拷贝

浅拷贝只会拷贝值 ，也就是拷贝一份副本

基础变量都会实现Copy trait 这些变量存放在栈上
```rust

fn main(){

​    let a:i32 = 2;

​    let b = a; // copy  trait

​    println!("a = {}", a);

}
```

考虑到使用上的便利性，在有些场景下并不希望应用默认的move语义。Rust的基础数据类型都实现了 std::marker::Copy trait，所以在进行let操作时，实际上发生了copy。

哪些类型是满足Copy特性的？类似整型这样的存储在栈上的类型、不需要分配内存或某种形式资源的类型。常见的Copy 类型如下

- 所有整数类型，比如 u32。
- 布尔类型，bool，它的值是 true 和 false。
- 所有浮点数类型，比如 f64。
- 字符类型，char。
- 元组，当且仅当其包含的类型也都是 Copy 的时候。比如，(i32, i32) 是 Copy 的，但 (i32, String) 就不是

Clone trait 深拷贝

深拷贝会在拷贝值的同时 拷贝指针 长度 容量等等为副本
```rust

fn main(){

​    let a :String = "xx".to_string();

​    let b :String = a.clone(); // clone trait

​    println!("a = {}", a);

}
```

对于一些高级的变量，也可以使用类似上面的Trait
```rust

\#[derive(Debug,Clone,Copy)]

struct Peorson {

​    name: String

}

// 会报错，因为String 默认不能使用Copy trait

\#[derive(Debug,Clone,Copy)]

struct Peorson {

​    name: i32,

​    age: i32

}

fn main(){

​    let p1 = Peorson{

​        name: 1,

​        age: 32,

​    };

​    let p2 = p1;

​    println!("p1 = {:?}",p1);

}
```

所有权与函数

函数同样有作用域， 函数的调用过程中涉及 形参与实参的, 因此会涉及到move copy clone等操作 move的时候所有权发生变化
```rust
fn echo(msg_moved:String){
​    println!("msg = {}",msg)
}
fn main(){
​    let msg:String = "xx".to_string();
​    echo(msg);
​    println!("{}",msg);
}
// 这里 msg 是String类型的变量 实现了Clone ，echo() 的时候调用msg 将所有权交接给 msg_moved
// fn echo(msg_moved:String){
//     println!("msg = {}",msg)
// }
fn echo_nums(nums_copys: i32){
​    println!("msg = {}", nums);
}

fn main(){
​    // let msg:String = "xx".to_string();
​    // echo(msg);
​    // println!("{}",msg);
​    let nums:i32 = 20;
​    echo_nums(nums);
​    println!("nums = {}",nums);
}
```
这里使用的i32类型的变量 实现了 Copy， echo_nums的时候调用msg 的Copy trait 复制了一个副本 nums_copys;
```rust
fn main() {
​    let s1 = String::from("hello");
​    let (s2, len) = calculate_length(s1);
​    println!("The length of '{}' is {}.", s2, len);
}
fn calculate_length(s: String) -> (String, usize) {
​    let length = s.len(); // len() 返回字符串的长度
​    (s, length)
}
```
既然String类型在做函数操作，赋值给形参的时候会出现move操作 如果在之后还要使用这个String变量 有如下几种方法
1. 使用clone
```rust
fn main() {
​    let s1:String = String::from("hello");
​    let len = calculate_length(s1.clone());
​    println!("The length of '{}' is {}.", s1, len);
}
fn calculate_length(s: String) -> usize {
​    s.len()
}
```
1. 传入再传出
```rust
fn main() {
​    let s1 = String::from("hello");
​    println!("The length of '{}' is {}.", s2, len);
}
fn calculate_length(s: String) -> (String, usize) {
​    let length = s.len(); // len() 返回字符串的长度
​    (s, length)
}
```
1. 使用引用
Rust提供了引用，**&就是引用。它允许你使用值但不获取其所有权 我们把采用引用作为函数参数称为借用（borrowing）**借用的使用所有权会跟过去？
```rust
fn main() {
​    let s1 = String::from("hello");
​    let len = calculate_length(&s1);
​    println!("The length of '{}' is {}.", s1, len);
}
// fn calculate_length(s: &String) -> usize {
//     s.len()
// }
fn calculate_length(s: &str) -> usize {
​    s.len()
}
其中&str = &String
离开作用域，释放的时候调用的Drop
// fn return_str<'a>() -> &'a str{
//     let mut  s = String::from("xx");
//     for i in 0..32{
//         s.push_str("Good");
//     }
//     &s[..]
// }
// fn main(){
//     let x = return_str();
// }
```
上面这段代码无法被编译通过的原因




生命周期

标注生命周期参数并不能改变任何引用的生命周期长短，它只用于编译器的借用检查， 来防止悬吊指针

输出（借用方）的生命周期长度必须不长于输入（出借方）的生命周期长度（此条件依然遵循借用规则一）

只要是实现了Copy trait的生命周期不考虑

发生move的操作  生命周期需要考虑 因为要考虑到堆上的数据和指针释放的问题 ，没用实现Copy trait的类型只有
clone trait 只能发生move

```rust
fn the_max(a1:i32, a2:i32) -> i32{

​    if a1 > a2 {a1} else {a2}

}

fn main(){

​    let a1 = 1;

​    {

​        let a2 = 2;

​        let a3 = the_max(a1, a2);

​        print!("a3 = {}", a3);

​    }

}
```


```rust
fn the_max(a1:i32, a2:i32) -> i32{
    if a1 > a2 {a1} else {a2}
}

fn main(){
    let a1 = 1;
    {
        let a2 = 2;
        let a3 = the_max(a1, a2);
        print!("a3 = {}", a3);
    }
}
```

比如这串代码中，i32类型实现了Copy trait 在实参和形参过程中发生的是copy行为，

```rust
fn the_longest(s1:&str, s2:&str) -> &str{
    if s1.len() > s2.len() { s1 } else {s2}
}


fn main(){
    let s1:&str = "yyy";     // lifetime 'a 
    {
        let s2:&str = "aa";  // lifetime 'b
        let s3 :&str= the_longest(s1, s2);
        print!("s3 = {}",s3)
    }
}


```

但这一串代码中， &str类型实现同样是copy，只是copy了一份指针堆上的栈上的内容不变，这时候the_longest中的 s1 s2 的生命周期长短不一 并且 `'a` > `'b` 这时候根据程序逻辑走向，理论上被返回的是s1, `s1.len() =3 ; s2.len() = 2`但是s3 的生命周期是`'b` 结果 `'a`存在在`'b`的作用域内 就会出现报错

```rust

fn main() {
    let s = String::from("hello,world");
    
    let hello = &s[0 .. 5]; //let hello = &s[ .. 5];
    let world = &s[6 .. 11]; //let hello = &s[6.. ];
    //let whole = &s[ .. ];
    /* 
        "&str" is slice
        it hasn't ownership and cannot change, but it can contain String type and slice type with pinot and len
    */

    let my_string = hello.to_owned() + "," + world;
    let word_index = first_world(&my_string);
 
    /* 
       my_string.clear();
        cannot borrow `my_string` as mutable, as it is not declared as mutable,cannot borrow as mutable
        (E0596)
        cannot borrow `my_string` as mutable because it is also borrowed as immutable,mutable borrow occurs here
        (E0502)
    */

    println!("my_string is '{}', and its word_index is {}",my_string, word_index);
}

fn first_world(s: &str) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() { //deconstruct string
        //iter() can crate a iterator which return every  immutable elements in array
        //enumerate()  can box the  elements into tuple
        if item == b' ' {
            return  &s[ .. i];
        }
    }
    &s[ .. ]
}
```



```rust
fn the_longest(s1:&str, s2:&str) -> String{
    println!("s1_s1_ptr = {:p}",s1); // s1_s1_ptr = 0x10e9eceb3
    println!("s2_s2_ptr = {:p}",s2); // s2_s2_ptr = 0x10e9ecebd
    if s1.len() > s2.len() { s1.to_string() } else {s2.to_string()}
}


// fn the_longest<'a>(s1:&'a str, s2:&'a str) -> &'a str{
//     if s1.len() > s2.len() { s1 } else {s2}
// }


// fn the_longest(s1:&str, s2:&str) -> &str{
    // if s1.len() > s2.len() { s1 } else {s2}
// }

fn the_max(n1:i32, n2:i32) -> i32{
    println!("n2_n2 = {:p}", &n2);
    println!("n1_n1 = {:p}", &n1);
    if n1 > n2 {n1} else {n2}
}

fn the_longest2<'a>(s4: &'a String, s5:&'a String) -> &'a String{
    println!("s4 = {:?}",s4.as_ptr()); // s4 = 0x7fefc9c02e70
    println!("s5 = {:?}",s5.as_ptr()); //  s5 = 0x7fefc9c02e80
    if s4.len() > s5.len() {s4} else {s5} 
}

fn the_longest3<'a>(s4: &'a String, s5:&'a String) -> String{
    println!("s4 = {:?}",s4.as_ptr()); // s4 = 0x7f9b13402e80
    println!("s5 = {:?}",s5.as_ptr()); //  s5 = 0x7f9b13402e90
    if s4.len() > s5.len() {s4.clone()} else {s5.clone()} 
}






fn main(){
    println!("&str------------");
    let s1:&str = "xxx";
    println!("s1_ptr {:?}",s1.as_ptr());  // s1_ptr 0x10e9eceb3
    {
        let s2:&str = "yyy";
        println!("s2_ptr {:?}",s2.as_ptr());   // s2_ptr 0x10e9ecebd
        let s3:String= the_longest(s1, s2); // Copy trait
        println!("s3 = {}",s3)
    }
    println!("s1 = {}",s1); 


    println!("i32 ---------------");

    let n1 = 2;
    println!("n1 = {:p}",&n1);
    {
        let n2 = 3;
        println!("n2 = {:p}",&n2);
        let n3 = the_max(n1, n2); // Copy trait
        println!("n3 =  {:p}",&n3);
        println!("n3 = {}",n3);
    }

    println!("String ---------------");

    let s4 = "xx".to_string();
    println!("s4_ptr {:?}",s4.as_ptr());  // s4_ptr 0x7fc40ec02e70
    {
        let s5 = "yyy".to_string();
        println!("s5_ptr {:?}",s5.as_ptr());  // s5_ptr 0x7fc40ec02e80
        let s6 = the_longest2(&s4, &s5);
        println!("s6_ptr {:?}",s6.as_ptr());  // s6_ptr 0x7fc40ec02e80
    }
  
   println!("String ---------------");

    let s4 = "xx".to_string();
    println!("s4_ptr {:?}",s4.as_ptr());  // s4_ptr s4_ptr 0x7f9b13402e80
    {
        let s5 = "yyy".to_string();
        println!("s5_ptr {:?}",s5.as_ptr());  // s5_ptr 0x7f9b13402e90
        let s6 = the_longest3(&s4, &s5);
        println!("s6_ptr {:?}",s6.as_ptr());  // s6_ptr 0x7f9b13402ea0
    }
}

```

s1 s2 在函数内外的地址不变 所以是发生了指针上的复制，但是数据存放的地址不变

而基础变量像 i32 类型的 是直接复制了一个副本过去

再看String类型的 发生了借用，使用值(包括变量值 指针地址等等)进入到函数内做比较 比较变量值，最后返回整个使用值赋值到s6上面  因此s6的地址肯定会和s4 s5 中其中一个相同

再之后传入的依旧是借用后的使用值， 但是返回的是一个新的值 `.clone`重新又设置了一个变量 



综上 不管是Copy 还是 借用 生命周期都会跟着走

```rust
fn the_longest(a1:&str, a2:&str ) ->&str{ // error 
    if a1.len() > a2.len() {a1} else {a2}
}
fn main(){
    let a1 :&str = "xx";
    {
        let a2:&str = "yyy";
        let max:&str =the_longest(a1, a2);
        println!("max = {:?}",max);
    }
}
```

再来看这段代码，会报错， 首先是copy了一个值传到函数里面去，但是他们具有不同的生命周期，返回的结果无法判断生命周期，所以要统一化生命周期的长短，使用`'a`标签  统一之后 默认以最长的生命周期为`'a`的生命周期

```rust
fn the_longest<'a>(a1:&'a str, a2:&'a str ) ->&'a str{ // error 
    if a1.len() > a2.len() {a1} else {a2}
}
fn main(){
    let a1 :&str = "xx";                     // lifetime 'x
    {
        let a2:&str = "yyy";                 // lifetime 'y
        let max:&str =the_longest(a1, a2);   // lifetime 'a == 'x
        println!("max = {:?}",max);
    }
}
```


```rust

fn main(){
    let a = "Hello".to_string();
}
```

let的意义是这样的：标识符 a和String类型字符串＂hello＂通过let绑定在了一起，a拥有字符串＂hello＂的所有权。更深层次的意义是，let绑定了标识符a和存储字符串＂hello＂的那块内存，从而a对那块内存拥有了所有权。所以此处a被称为绑定。本书中的“变量”“变量绑定”或“绑定”，均是指“绑定”的

```rust
fn main(){
    let a = "Helloworld".to_string();
    let b = a ; // move
}
```

此时使用let声明另一个绑定b，并将a赋值给b，如let b=a，则此时必然会将a对字符串的所有权转移给b。因为a是String类型，不能实现Copy，所以这种行为其实也可以理解为对a进行解绑，然后重新绑定给b

> 生命周期何时绑定
>
> 变量绑定具有“时空”双重属性。
>
> ·空间属性是指标识符与内存空间进行了绑定。
>
> ·时间属性是指绑定的时效性，也就是指它的生存周期。

所有权借用

```rust
fn foo(mut v:[i32;3]) -> [i32;3]{
    v[0] = 3;
    assert_eq!([3,2,3],v);
    v
}

fn main(){
    let v = [1,2,3];
    foo(v);
    assert_eq!([1,2,3],v);
}
```

第8行代码声明的数组v是不可变的，但是传入了foo函数中就变成了可变数组，主要原因是函数参数签名也支持`模式匹配，相当于使用let将v重新声明成了可变绑定`

