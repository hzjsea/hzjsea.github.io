---
title: rustv2
categories:
  - rust
tags:
  - rust
abbrlink: de13282a0
date: 2020-07-08 22:57:05
top: 1
---

# rustv2


## Rust工具链使用
```rust
rustup self update // 更新rustup本身
rustup self uninstall // 卸载rust所有程序
rustup update // 更新工具链
rustup install nightly  // 安装nightly版本
rustup default nightly // 安装nightly版本
```

### Rust更换下载源
```rust
cd 
cd ./cargo
touch config

[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```



## Rust 包管理

- package
通过`cargo new --bin `创建；项目包
- crate 
通过`cargo new --lib`创建。有根包和子包。即一个根包下可以包含多个子包
`crate`是二进制或者库项目。rust约定在`Cargo.toml`的同级目录下包含`src`目录并且包含`main.rs`文件，就是与包同名的二进制crate，如果包目录中包含`src/lib.rs`，就是与包同名的库crate
- mod
通过关键字mod加模块定义
- function


### 同文件引用
```rust
fn main() {
    // 相对路径
    say::hello();
    // 绝对路径调用 // src即为根目录
    crate::say::hello();

    // 无法调用
    // say::hello_2(); // error , because it's not public

    say::hi::hi_1();
    say::hi::hi_2();
    // 使用 use 后就可以这么调用
    use crate::say::hi;
    hi::hi_1();

    // 重导出名称
    people::hi::hi_1();
    people::hello();
    // 但是不能 
    // people::say::hello();

    people_2::people::hello();
    people_2::info::name();
}

mod say {
    pub fn hello() {
        println!("Hello, world!");
    }
    fn hello_2() {
        println!("hello")
    }
    pub mod hi {
        pub fn hi_1() {
            super::hello_2();
        }
        pub fn hi_2() {
            println!("hi there");
        }
    }
}

pub mod people {
    // 调用pub的就用pub声明 
    pub use crate::say::hi;
    // 调用不是pub的就不能用pub声明
    use crate::say;
    pub fn hello() {
        say::hello();
    }
    pub mod info {
        pub fn name() {
            println!("hzjsea");
        }
    }
}


mod people_2 {
    // 调用局部 self 代指 people mod下面的所有方法，而非指所有方法
    pub use crate::people::{self, info};
    pub fn hello() {
        info::name();
    }
}
```



## rust宏
rust语言中有`宏`的概念定义,每个宏都会以`!`结尾,宏的存在主要是为了更好地进行错误检查,如果在宏中出现参数个数 格式等各种原因不匹配会直接导致编译错误,而函数则不具备字符串格式化的静态检查功能,如果出现参数不匹配等情况,只会包运行期错误
类似的宏有 `println!` `format!` `write!` `writeln!` 等等

## Rust变量

Rust变量分为如下几类
- 基础数据类型
- 高级数据类型
- 变量遮蔽shadowing
- 下划线命名

### 基础数据类型

```rust
fn main(){
    // 整型
    let a:i32 = 30;
    // 浮点型
    let b:f32 = 30.1;
    // 布尔型
    let c:bool = true;
    // 字符串字面量
    let d:&'static str = "xx";
    // 字符串
    let e:String =  String::from("xx");
    // 元祖
    let tup:(i32,i32) = (2,30);
    assert_eq!(tup.0 , 2);
    assert_eq!(tup.1 , 30);
    
    // 静态数组
    let f:[i32;3] = [1,2,3];
    assert_eq!(f[0] , 1);
    assert_eq!(f[0] , 2);
}
```

### 高级数据类型

#### Struct
结构体通常用来定义对象,rust中常常用结构体来表示某一对象，在之后对这个对象进行一系列的扩展，最终完成一个对象的所有行为
如下为结构体初始化过程

```rust
#[derive(Debug,PartialEq, PartialOrd)]
struct Peoson{
    name:String,
    age:i32
}

impl Peoson {
    fn new() -> Self{
        Peoson{
            name:"xx".to_owned(),
            age:20
        }
    }
}

fn main(){
    let p1 = Peoson::new();
    assert_eq!(p1.age,20);
    let p2 = Peoson{
        name:"xx".to_owned(),
        age:20
    };
    assert_eq!(p1,p2);
}
```

#### Struct-结构体初始化
初始化代码如下，同时如果Peoson实现了Default这个trait 则可以直接调用`..Default::default()` 以默认值初始化

> 以默认值初始化

```rust
#[derive(Default)]
struct Peoson{
    name:String,
    age:i32
}

fn main(){
    let p1 = Peoson{
        age:32,
        ..Default::default() // 设置为默认值
    };
    println!("{}",p1.name)
    
}
```

> 以同类型初始化

struct可以从同类型中初始化实例,产生move语义，即p2 根据 p1 创建实例后， p1 会被移除

```rust
#[derive(Debug)]
struct Peoson{
    name:String,
    age:i32
}



fn main(){
    let p2 = Peoson{
        age:32,
        name:"xx".to_owned(),
    };
    let p3 = Peoson{
        ..p2 // p2 moved here
    };
    println!("{:?}",p2);
}
```


#### Struct-元祖结构体

元组结构体有着结构体名称提供的含义，但没有具体的字段名，只有字段的类型

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
fn main(){

    // 声明实例
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
    // 输出属性
    println!("{}{}",black.0,black.1);

}
```

#### Struct-关联函数
```rust
#[derive(Debug)]

// 创建一个结构体

struct  Rectangle{
    width: i32,
    height: i32,
}


// 为结构体编写方法
impl Rectangle {
    fn area(&self) -> i32 {
        self.width * self.height
    }

    // 关联函数
    fn init(size:i32) -> Rectangle{
        Rectangle {
            width: size,
            height: size,
        }
    }
}

fn main(){

    // 初始化函数
    let rec3 = Rectangle::init(3);
    println!("{:?}",rec3);// 打印需要加上 #[derive(Debug)]
    // res: Rectangle { width: 3, height: 3 }
    // 创建实例
    let rec1 = Rectangle{ width:20,height:30 };
    println!("{}",rec1.area());
}
```

#### Struct-多impl块

对于同一个结构体，rust 支持多个 impl 块

```rust
#[derive(Debug)]

// 创建一个结构体

struct  Rectangle{
    width: i32,
    height: i32,
}


// 为结构体编写方法
impl Rectangle {
    fn area(&self) -> i32 {
        self.width * self.height
    }

    // 关联函数
    fn init(size:i32) -> Rectangle{
        Rectangle {
            width: size,
            height: size,
        }
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main(){

    // 初始化函数
    let rec3 = Rectangle::init(3);
    println!("{:?}",rec3);// 打印需要加上 #[derive(Debug)]
    // res: Rectangle { width: 3, height: 3 }
    // 创建实例
    let rec1 = Rectangle{ width:20,height:30 };
    println!("{}",rec1.area());

    println!("{}",rec1.can_hold(&rec3));
}

```

#### 枚举
枚举(Enum)  常用来定义一个对象的固有属性组或者方法组

#### 枚举-固有属性
固有属性表示存储固定的值，比如一周的7天
```rust
#[derive(Debug)]
enum Week {
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday,
}

fn main() {
    let today = Week::Saturday;  // 使用枚举
    let tomorrow = Week::Sunday;
}
```
```rust
enum MathMethods {
    Add,
    Sub
}

impl MathMethods {
    fn do_action(&self,x:i32,y:i32) -> i32{
        match self {
            MathMethods::Add =>  x + y,
            MathMethods::Sub => x - y
        }
    }
}

struct People{
    name:String,
    age:i32
}

impl People {
    fn run(&self, action: MathMethods){
        action.do_action(20, 30);
    }
}

fn main(){
    let p1 = People{
        name:"xx".to_owned(),
        age:30,
    };
    if p1.age > 30{
        p1.run(MathMethods::Add)
    } else {
        p1.run(MathMethods::Sub)
    }
    
}
```

#### 枚举-存储元组

```rust
enum MathMethods {
    Add(i32,i32),
    Sub(i32,i32)
}

impl MathMethods {
    fn do_action(&self) -> i32{
        match self {
            MathMethods::Add(x,y) =>  x + y,
            MathMethods::Sub(x,y) => x - y
        }
    }
}

struct People{
    name:String,
    age:i32
}

impl People {
    fn run(&self, action: MathMethods){
        action.do_action();
    }
}

fn main(){
    let p1 = People{
        name:"xx".to_owned(),
        age:30,
    };
    if p1.age > 30{
        p1.run(MathMethods::Add(2,20))
    } else {
        p1.run(MathMethods::Sub(2,30))
    }
    
}
```

```rust
enum SpreadsheetCall {
    IntType(i32),
    FloatType(f64),
    Text(String),
}

fn main() {
    let row = vec![
        SpreadsheetCall::IntType(63),
        SpreadsheetCall::FloatType(10.12),
        SpreadsheetCall::Text(String::from("blue")),
    ];
    println!("Hello, world!");

}

```

#### 枚举-存储结构体
```rust
enum Class {
    People {name:String,age:i32},

}

fn main(){
    let c1 = Class::People{
        name:"xx".to_owned(),
        age:32,
    };
}
```

#### 枚举-使用use
使用use可以直接调用枚举中的内容,后续调用无序添加前缀
```rust
enum Status {
    Rich,
    Poor,
}

enum Work {
    Civilian,
    Soldier,
}

fn main() {
    // 显式地 `use` 各个名称使他们直接可用，而不需要指定它们来自 `Status`。
    use Status::{Poor, Rich};
    // 自动地 `use` `Work` 内部的各个名称。
    use Work::*;

    // `Poor` 等价于 `Status::Poor`。
    let status = Poor;
    // `Civilian` 等价于 `Work::Civilian`。
    let work = Civilian;

    match status {
        // 注意这里没有用完整路径，因为上面显式地使用了 `use`。
        Rich => println!("The rich have lots of money!"),
        Poor => println!("The poor have no money..."),
    }

    match work {
        // 再次注意到没有用完整路径。
        Civilian => println!("Civilians work!"),
        Soldier  => println!("Soldiers fight!"),
    }
}

```


#### 向量(动态数组Vector)

vector 就是是一个可以自动扩展容量的动态数组。它重载了Index运算符，可以通过中括号取下标的形式访问内部成员。它还重载了Deref/DerefMut运算符，因此可以自动被解引用组切片,创建方式如下
```rust
fn main(){
    let v = vec![1,2,3,4,5];
    use_iter(v.iter());
    let v1:Vec<i32> = Vec::<i32>::new();
    let v2:Vec<String> = Vec::with_capacity(200);
    let v3:Vec<i32> = vec![1,2,3];
}

```
因为`new()`本身就是一个泛型的方法，可以使用turobfish约束泛型类型

#### Vec-使用方法
```rust
fn main(){
    let v1:Vec<i32> = Vec::<i32>::new();
    let v2:Vec<String> = Vec::with_capacity(200);
    let v3:Vec<i32> = vec![1,2,3];
    
    // 插入数据
    let mut v4:Vec<i32> = Vec::new();
    println!("{}",v4.capacity()); // 0
    v4.push(1);
    assert_eq!(v4.capacity(),1);
    v4.extend_from_slice(&[1,2,3,4,5]);
    println!("{}",v4.capacity()); // 6
    v4.insert(2,100);
    println!("{}",v4.capacity()); // 12

    // 访问数据
    v4[5] = 5;
    let i = v4[5];
    assert_eq!(i,5);
    // Index 运算符直接访问,如果越界则会造成 panic,而 get 方法不会,因为它返回一个 Option<T>
    if let Some(i) = v4.get(6){
        println!("{}",i)
    }
    let slice = &v4[4..];
    println!("{:?}",slice);
}
```
Vec里面存在一个指向堆上的指针，它永远是非空的状态，编译器可以据此做优化

向量（Vector）是一个封装了动态大小数组的顺序容器（Sequence Container）。跟任意其它类型容器一样，它能够存放各种类型的对象。可以简单的认为，向量是一个能够存放任意类型的动态数组
第一个类型是 `Vec<T>`，也被称为 vector。vector 允许我们在一个单独的数据结构中储存多于一个的值，它在内存中彼此相邻地排列所有的值。vector 只能储存相同类型的值，类型由`T`来决定
```rust
fn main() {

    let v:Vec<i32> = Vec::new();
    let v = vec![1,2,3,4,5];

    // 向可变集合中添加参数
    let mut v1:Vec<i32> = Vec::new();
    v1.push(2);
    v1.push(3);

    println!("{:?}",v1); // [2,3]
    // 获取集合长度
    assert_eq!(v1.len(),2);
    // 根据序号获取集合中的元素
    assert_eq!(v1[0],2);
    // 删除集合最后一个元素，并且返回一个 Option<T> 的值
    println!("{:?}",v1.pop()); // Some(3)
    println!("{:?}",v1); // [2]

    // 修改集合中指定序号的值
    v1[0]=20;
    println!("[2] -> {:?}",v1); // [2] -> [20]


    // 集合合并
    v1.extend([1,2,3].iter().copied());
    for x in &v1 {
        println!("{}",x)
    }
    assert_eq!(v1,[20,1,2,3]);

    // 初始化数据
    let vec  = vec![0;5];
    assert_eq!(vec,[0,0,0,0,0]);

    // 和上面的类似，但是效率更加的低
    // 1. 创建5个长度的集合
    // 2. 重新设置长度，将值变为5
    let mut vec1 = Vec::with_capacity(5);
    vec1.resize(5,0);
    assert_eq!(vec1,[0,0,0,0,0]);

    // 这样写就没啥用  他只会重新设置长度
    let mut ve1 = Vec::new();
    ve1.push(1);
    ve1.push(1);
    ve1.push(1);
    ve1.push(1);
    ve1.push(1);

    ve1.resize(4,0);
    println!("{:?}",ve1);


    // 测试集合弹出的内容
    let mut stack = Vec::new();
    stack.push(1);
    stack.push(1);
    stack.push(1);
    stack.push(1);

    while let Some(top)  = stack.pop(){
        println!("{}",top);
    }

    // 切片是只读的，要获取切片的内容，可以使用&
    let v = vec![0,1];
    println!("{:?}",&v);


    // 使用切片获取集合中的元素
    let vv = vec![0,1,2,3,4];
    let third: &i32 = &vv[2];
    println!("{:?}",third);
    println!("{:?}",vv.get(2));

    // 使用get来获取集合中的元素，返回一个Option<&T>
    match vv.get(2){
        Some(third) => println!("The third element is {}",third),
        None => println!("There is no third element."),
    }

    

    // 
    let v = vec![1,2,3,4,5];
    
    // let does_not_exist = &v[100];   // 这里直接出现panic的报错内容，这个方法更适合当程序认为尝试访问超过 vector 结尾的元素是一个严重错误的情况，这时应该使程序崩溃。
    let does_not_exist1 = v.get(100); // 不会返回panic而是返回None，当偶尔出现超过 vector 范围的访问属于正常情况的时候可以考虑使用它。接着你的代码可以有处理 Some(&element) 或 None 的逻辑，
   

    match does_not_exist1 {
        Some(res) => println!("{:?}",res),
        None => println!("这个结果是None"),
    }

    // 遍历集合中的元素

    let v = vec![1,2,3,4,56];
    for i in &v{
        println!("{}",i);
    }


    // 遍历可变元素
    let mut v = vec![1,2,3,4,5];
    // 为了修改可变引用所指向的值，在使用 += 运算符之前必须使用解引用运算符（*）获取 i 中的值。第十五章的 “通过解引用运算符追踪指针的值” 部分会详细介绍解引用运算符
    for i in &mut v {
        *i +=50;
    }   
}
```

#### Vec-双向队列

```rust
use std::collections::VecDeque;
fn main(){
    let mut queue = VecDeque::with_capacity(64);
    for i in 1..10{
        queue.push_back(i)
    }
    while let Some(i) = queue.pop_front(){
        println!("{}",i)
    }
}
```

#### HashMap

#### Json

### 下划线命名
Rust建议对不使用的变量使用`_`来标明，不然会在编译的过程中报wrong

```rust
fn main() {
    let num = "hello";
    let _nums = "hellos";
}
//    Compiling helloworld v0.1.0 (/Users/alpaca/hzj/yl/test/helloworld)
// warning: unused variable: `num`
//   --> src/main.rs:18:9
//    |
// 18 |     let num = "hello";
//    |         ^^^ help: consider prefixing with an underscore: `_num`
//    |
//    = note: `#[warn(unused_variables)]` on by default

//     Finished dev [unoptimized + debuginfo] target(s) in 0.19s
//      Running `target/debug/helloworld src/main.rs`

// 像上面这块内容，加了下划线的变量是不会报错的
```

### 变量遮蔽
```rust
fn main(){
    let x= "1";
    let x = x; // move
    let x = "2"; // 变量遮蔽
}
```

## 所有权

篇幅过大 另起一篇

## 泛型

篇幅过大 另起一篇

## Trait

篇幅过大，另起一篇

## 生命周期

篇幅过大，另起一篇

## 断言

## 模式匹配

### 匹配方法

#### match匹配
```rust
#[derive(Debug)]

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn main() {
    let coin = Coin::Dime;
    let res = value_in_cents(coin);
    // println!("{:?}",coin); // 报错，因为已经移动了
    println!("{}",res);

    let res_extend = value_in_cents_extend(Coin::Penny);
    println!("{}",res_extend)

}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
// 如果coin=penny 则返回1 
// 如果coin=nickel则返回5 
// 等等
// 类似于switch
// 所做的比较 coin 是否等于 Coin::xx


fn value_in_cents_extend(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```


#### if let匹配

对于只需要匹配一个内容的地方可以使用`if let`

```rust


if let Some(3) = some_u8_value {
    println!("three");
}


let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

#### while let匹配


### let

### 特殊匹配符_

对于不需要匹配的内容，我们可以用`_`来处理

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```


### 常用匹配类型

## 特殊枚举
Rust下分别有两个特殊枚举 一个是Option 另一个是Result
同时对应了两类隐式处理的方式，以及Result额外还对应一个问号表达式的处理方式

### Option

Option 是一个enum类型，枚举类型如下
```rust
pub enum Option<T> {
    None,
    Some(T),
}
```
option常常和模式匹配使用在一起
```rust
fn main(){
    let xx = Some(32);
    if let xx = Some(32){
        println!("true");
    };
    while let xx = Some(32) {
        println!("true")
    };
    match xx {
        Some(value) => println!("{}",value),
        None => println!("None") 
    }
}
```

### Result

Result也是一个枚举类型
```rust
#[must_use = "this `Result` may be an `Err` variant, which should be handled"]
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
类似于Option 只不过Result包含一个OK的值(类似于some) 以及一个Err的值(捕获错误)
```rust
let x: Result<i32, &str> = Ok(-3);
assert_eq!(x.is_ok(), true);

let x: Result<i32, &str> = Err("Some error message");
assert_eq!(x.is_ok(), false);

fn main(){
    // let xx = Some(32);
    // let yy = xx.unwrap();
    let x: Result<i32, &str> = Ok(-3);
    match x {
        Ok(value) => print!("{}",value),
        Err(err) => print!("{}",err),
    }
}

```

### unwrap
https://learning-rust.github.io/docs/e4.unwrap_and_expect.html
遇到错误，则直接报错`panic!`,没报错则返回`Some()` 或者`OK()`
源码如下
```rust
 pub fn expect(self, msg: &str) -> T {
        match self {
            Some(val) => val,
            None => expect_failed(msg),
        }
    }
```

### expect
遇到错误，则直接报错,并可以自定义报错内容,没报错则返回`Some()`或者`OK()`
源码如下
```rust
pub fn unwrap(self) -> T {
    match self {
        Ok(t) => t,
        Err(e) => unwrap_failed("called `Result::unwrap()` on an `Err` value", &e),
    }
}
```

### 问号表达式
https://www.jianshu.com/p/46872e6bffce
1. 问号表达式只能使用在返回Result的方法
2. 碰到panic直接上抛
```rust
fn main(){
    let res = get_res(100);
    print!("{:?}",res) // Err(TooBig)
    
}

fn get_res(index:i32) -> Result<i32, MathError>{
    let res = is_ok(index)?;
    Ok(res)
}

#[derive(Debug)]
enum MathError {
    _DivisionByZero,
    _NegativeLogarithm,
    TooBig

}

fn is_ok(n1:i32) -> Result<i32,MathError>
{
    if n1 > 20{
        Err(MathError::TooBig)
    }  else {
        Ok(32)
    }
}
```

## 逻辑判断


### if 表达式

```rust
fn main() {
    let number = 3;

    // if-else
    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
    // if-elseif-else
    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}

}
```

<font color='red'>另外 rust 并不会将大于 0 的数字转换成 bool 类型的数据类型</font>因此，如果你写出如下面的代码，那么编译肯定是不会通过的

```rust

fn main() {
    println!("hello world");
    let number:i32 = 20;
    if number {
        println!("{}",number);
    }
}

// 会报如下错误
//  ^^^^^^ expected `bool`, found `i32`
```

### if 表达式的扩展

之前有讲过表达式和语句的关系，表达式会返回值，而语句则是一句话的结束，并不会返回任何内，因此在语句中可以包含表达式
<font color='red'>在 let 中使用 if 表达式赋值</font>

```rust
fn main() {
    let number:i32 = 20;
    let flag:bool = if number < 10{
        // or return true
        true
    } else {
        false
    };
    println!("this flag is {}",flag);
}
```

这里 if 表达式是返回了一个值给 let 赋值语句

### 循环表达式 loop

```rust
fn main() {
    // loop的常规用法 无线循环
    let mut normal_number:i32 = 1;
    loop_func_normal(normal_number);


    // loop 跳出循环
    let mut number:i32 = 20;
    let flag:bool = loop {
        number -=1;
        if number < 10  {
            break false;
        }
    };
}

fn loop_func_normal( mut number:i32){
    loop {
        number +=1;
        println!("normal_number is {}",number);
    }
}
```

首先 loop 相当于 while true 可以无限循环，另外 loop 也是一个表达式，有返回值
跳出循环的方法有两个 一个是`ctrl+c` 跳出循环 ，另外一个是设置条件语句，使用`break` 跳出循环

<font color='red'>break 的用法， break 可以跳出循环，写法是 break + return_value; 需要返回的内容要跟在 break 后面</font>

### 循环表达式 while

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```

### 循环表达式 for

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    println!("{:?}",a.iter());
    for i in a.iter(){
        println!("{}",i)
    }
}


// result
// Iter([10, 20, 30, 40, 50])
// 10
// 20
// 30
// 40
// 50

```

使用`range+rev`完成倒计时

```rust
fn main(){
    for i in (1..4).rev(){
        println!("{}",i);
    }

    for i in (1..4){
        println!("{}",i);
    }

    // for i in 1..4 {
    //     println!("{}",i);
    // }
}
// result
// 3
// 2
// 1
// 1
// 2
// 3
```


![2020-08-17-09-52-17](http://noback.upyun.com/2020-08-17-09-52-17.png)

和 python 一样 前闭后开

其中这里只包括 0-4 不包括 5


```rust
fn main() {
    let v1 = vec![1,2,3,4,5];
    forfunc(&v1);
    forfunc2(&v1);
    println!("{:?}",getMax(&v1));
}


fn getMax(list:&[i32]) -> i32{
    let mut max = list[0];
    for item in 1..list.len(){
       if max < list[item]{
            max = list[item]
       }
    }

    max
}


fn getMax2(list:&[i32]) -> i32{
    let mut max = list[0];
    for &item in list{
        if item > max{
            max = item
        }
    }
    max
}

fn forfunc(list:&[i32]){
    for &item in list.iter(){
        println!("{:?}",item);
    }
}

fn forfunc2(list:&[i32]){
    for item in list.iter(){
        println!("{:?}",item);
    }
}
```



## Rust函数
Rust中函数和方法没有太大的区别

### Rust 函数指针与匿名函数

> 在rust中匿名函数以`函数指针`的形式出现
函数指针的另外的一个作用就是以函数类型的形式出现，他可以作为变量的类型

```rust
fn main(){
    let res = return_value(change_value, 20);
    println!("{:?}",res) // 40
}

fn change_value(y:i32) -> i32{
    return y + 20
}

fn return_value(f: fn(y:i32) -> i32, y:i32) -> i32 {
    return f(y)
}
```
如上`return_value`接收一个函数类型的f 和一个变量y, 在`main` 当中，传入了一个对应函数类型的函数，这里函数被当成了变量在使用


### rust闭包

使用闭包将上面的代码替换
```rust
fn main(){
    let res2 = return_value(|y:i32| -> i32 {y}, 20);
    println!("{:?}",res2); // 40
}
fn return_value(f: fn(y:i32) -> i32, y:i32) -> i32 {
    return f(y)
}
```

闭包的概念，闭包类似于其他语言中的 lambda 表达式或者 lambda, 是一类能够捕获周围 作用域中变量的函数

Rust的闭包的完整语法格式为：`|param: type| -> return type { func body }`
`闭包代码优化，如果你调用了该闭包，得益于Rust的类型推导，参数类型和返回类型可以省略，否则不可省略， 当函数主体只有一句时，可以省略那对花括号`
如下是一个简单地闭包
```rust
fn main(){
    let print_info = |z|z;
    println!("{:?}",print_info(20)); // 20
    println!("{:?}",print_info("message")); // error 
}
```
第二句报错是因为，rust编译器在读到第二行的时候，就推出了闭包的变量类型`let print_info: |...| -> i32 = |z:i32| -> i32 { z }`，后面接收其他的变量类型时，会报错  
只要闭包的类型确定(不管是在类型推导后确定的还是最初就定义好的)，就不能改变。


> python当中的垃圾回收处理方式

```python
def cal_avg():
    useless_value = "test"
    a = 0
    b = 0

    def average(new_value):
        nonlocal a, b  # 使用外部的变量
        a += 1
        b += 2
        new_value = a + b + new_value
        return new_value

    return average


def main():
    avg = cal_avg()
    print(avg(10))  # 13
    print(avg(10))  # 16
    print(avg(10))  # 19
    print(avg(10))  # 22


if __name__ == '__main__':
    main()
```
在Python中，我们使用`nonlocal`来声明捕获外部的变量，否则就当作是局部变量使用。在这个例子中，average函数捕获了变量`a`和`b`，此外，在函数`cal_avg`内部，还定义了一个局部变量`useless_value`，这是为了说明普通的局部变量和被闭包引用的变量有什么不同，我们知道，一旦一个对象的引用计数为0时，它就不能再被获取(弱引用除外)，就会被当作垃圾而回收掉以释放资源，很显然，在一个函数被调用完成后，除了返回值，其他所有局部变量都不可获取，但是闭包引用的值a和b仍然没有被回收 `useless_value`被回收，这是因为，变量avg引用了函数值cal_avg()即内部的average函数，所以变量a和b一直都在被引用，这才没有被当作垃圾回收。

> rust当中的垃圾处理方式

```rust
fn main(){
    let res2 = return_value(|y| y, 20);
    println!("{:?}",res2); // 40
}
fn return_value(f: fn(y:i32) -> i32, y:i32) -> i32 {
    return f(y)
}
```
这里在执行`f(y)`的时候会去获取y的引用，拿到的所有权


> 闭包语法

```rust
fn main() {
    // 通过闭包和函数分别实现自增。
    // 译注：下面这行是使用函数的实现
    fn  function            (i: i32) -> i32 { i + 1 }

    // 闭包是匿名的，这里我们将它们绑定到引用。
    // 类型标注和函数的一样，不过类型标注和使用 `{}` 来围住函数体都是可选的。
    // 这些匿名函数（nameless function）被赋值给合适地命名的变量。
    let closure_annotated = |i: i32| -> i32 { i + 1 };
    let closure_inferred  = |i     |          i + 1  ;
    
    // 译注：将闭包绑定到引用的说法可能不准。
    // 据[语言参考](https://doc.rust-lang.org/beta/reference/types.html#closure-types)
    // 闭包表达式产生的类型就是 “闭包类型”，不属于引用类型，而且确实无法对上面两个
    // `closure_xxx` 变量解引用。
    
    let i = 1;
    // 调用函数和闭包。
    println!("function: {}", function(i));
    println!("closure_annotated: {}", closure_annotated(i));
    println!("closure_inferred: {}", closure_inferred(i));

    // 没有参数的闭包，返回一个 `i32` 类型。
    // 返回类型是自动推导的。
    let one = || 1;
    println!("closure returning one: {}", one());
}
```

> 闭包trait

Rust 语言的闭包设计非常有趣，一方面，它看起来非常复杂，为了支持闭包设计了三种不同的 trait，Fn、FnMut 和 FnOnce；一方面其设计又透露出了语言设计中闭包的本质
> 对于函数基于栈的且没有垃圾回收（Garbage Collection）的语言，往往无法实现完全的闭包。这是因为，闭包从语义上应当能够延长其捕获的变量的生存期（lifetime）到长于或等于闭包的生存期。对于广泛利用栈进行函数局部变量分配和流程控制的语言，函数的局部变量的生存期严格与函数调用栈绑定，即从函数调用到函数返回（严格来说是局部变量内存的生存期，显然局部变量的生存期必然小于等于其内存的生存期）
> 举例来说，有上述特征的 C++ 的闭包就易于引发未定义行为（Undefined Behavior）。因为其引用捕获的局部变量的生存期无法自动延长。而例如 Java，JavaScript 和 Go 的闭包就不会，因为其编译器（对于 JavaScript 来说往往是 JIT 编译器）将对局部变量做逃逸分析（Escape Analysis）。将可能“逃逸”的变量生存期延长，由垃圾回收器而不是函数调用栈维护其生存期。又或者将所有局部变量分配在堆上由垃圾回收器维护也是一样

闭包的通常形式 闭包通常被实现为其捕获的词法环境和一个函数的组合

https://www.linyinfeng.com/posts/how-do-rust-closures-work/
https://juejin.cn/post/6844903917420019719
https://www.twle.cn/c/yufei/rust/rust-basic-closure.html
https://www.jianshu.com/p/fe61d8833fdc
https://skyao.io/learning-rust/grammar/pointer/fn.html
https://cloud.tencent.com/developer/article/1700466
http://127.0.0.1:4000/2021/02/22/blog_back_new/Rust/rust%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0%E4%B8%8E%E9%97%AD%E5%8C%85/
https://kaisery.github.io/trpl-zh-cn/ch13-01-closures.html
<!-- 

Fn, FnOnce, FnMut
当一个闭包中只有对捕获变量的不可变引用时，我们说它实现了Fn这个trait。
当一个闭包中发生了变量所有权的移动或者是某些值被消耗掉，drop, 我们说它实现的是FnOnce这个trait，即这个闭包只能使用一次。所有的函数和闭包都默认实现了这一trait。
当一个闭包中出现了对变量的可变引用时，我们说它实现了FnMut这个trait。
它们三者之间的关系可以用集合论的方法来认识，FnOnce包含FnMut包含Fn。 -->



## 数据类型转换

### as

### Into from


## 测试

1. 设置任何所需的数据或状态
2. 运行需要测试的代码
3. 断言其结果是我们所期望的

### 添加属性

为了将一个函数变成测试函数，需要在 fn 行之前加上 `#[test]`。当使用 `cargo test` 命令运行测试时，Rust 会构建一个测试执行程序用来调用标记了 test 属性的函数，并报告每一个测试是通过还是失败

### 编写测试

```rust
// 创建一个新的库
cargo new --lib adder
// 内容如下
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}

// 开始测试
cargo test
```


