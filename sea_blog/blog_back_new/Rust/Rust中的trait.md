---
title: Rust中的trait
date: 2021-03-10 11:50:05
categories:  [rust]
tags: [rust]
---


<!--more-->



# Rust中的trait

Rust中的trait作为一个方法集，用来作为接口类型来规范泛型的使用范围
```rust
trait Shape {
    fn area(&self) -> f64; // 关联函数 //方法 .调用
    fn width() -> f64;  // 静态函数  ::调用
}

fn main(){
    
}
```
所有的trait中都有一个隐藏的类型Self（大写S），代表当前实现了此trait的具体类型。trait中定义的函数，也可以称作关联函数（associated function）。函数的第一个参数如果是Self相关的类型，且命名为self（小写s），这个参数可以被称为“receiver”（接收者）。具有receiver参数的函数，我们称为“方法”（method），可以通过变量实例使用小数点来调用。没有receiver参数的函数，我们称为“静态函数”（static function），可以通过类型加双冒号：：的方式来调用。在Rust中，函数和方法没有本质区别

`大写的Self`是类型名，`小写的self`是变量名，为了方便rust中有映射关系，关系如下
```rust
trait T {
    fn method1(self: Self);
    fn method2(self: &Self);
    fn method3(self: &mut Self);
}


trait T {
    fn method1(self);
    fn method2(&self);
    fn method3(&mut self);
}
```

为一个结构体实现一个trait
```rust
struct Circle{
    radius: f64
}

// 定义接口
trait ShapeAction{
    fn area(&self) -> f64; // 方法
    fn echo() ; // 静态函数
}

// 实现接口
impl ShapeAction for Circle{
    fn area(&self) -> f64 {
        std::f64::consts::PI *self.radius*self.radius
    }
    fn echo(){
        println!("None");
    }
}

fn main(){
    let c1 = Circle{
        radius:1f64,
    };
    let c1_area = c1.area();
    assert_eq!(1f64,c1.radius);
    Circle::echo(); // None
}
```
trait可以看成一个接口，接口当中实现了很多的方法, 通过impl对某一个变量来实现接口中的所有方法


> 匿名trait

```rust

struct Circle{
    radius: f64
}


// 匿名trait
impl Circle{
    fn get_radius(&self) -> f64 { self.radius}
}

fn main(){
    let c1 = Circle{
        radius:1f64,
    };
    let c1_area = c1.area();
    print!("{}",c1_area);

    c1.get_radius(); // 匿名trait
}
```
如上可以看做Circle实现了一个匿名trait


> 默认实现

```rust
struct Circle{
    radius: f64
}

// 定义方法集
trait ShapeAction{
    fn area(&self) -> f64;
    fn echo(){
        print!("helloworld");
    }
}

// 实现方法集
impl ShapeAction for Circle{
    fn area(&self) -> f64 {
        std::f64::consts::PI *self.radius*self.radius
    }
    fn echo() {
        print!("实现 Circle")
    }

}


// 匿名trait
impl Circle{
    fn get_radius(&self) -> f64 { self.radius}
}

fn main(){
    let c1 = Circle{
        radius:1f64,
    };
    let c1_area = c1.area();
    print!("{}",c1_area);

    c1.get_radius(); // 默认实现

    Circle::echo(); // Circle
}
```

trait支持默认实现，可以在impl中重写



> 泛型trait

```rust
trait Add<RHS=Self>  { // 这里RHS=Self的意思就是获取自身变量的数据类型,保证了输入参数和调用参数的类型一致
    type OUTPUT;
    fn my_add(self, rhs:RHS) -> Self::OUTPUT;
}

trait AddExport<RHS>{
    type o;
    fn my_addd(self, rhs:RHS) -> Self::o;
}

trait AddExportS<RHS> {
    type o;
    fn my_adddd(self, rhs:RHS) -> Self::o;
}

impl Add for i32{
    type OUTPUT = i32;
    fn my_add(self, rhs:i32) -> Self::OUTPUT {
        self+rhs
    }
}

impl Add for u32{
    type OUTPUT = i32;
    fn my_add(self, rhs:u32) -> Self::OUTPUT {
        (self + rhs) as i32
    }
}



impl AddExport<i32> for i32{
    type o = i32;
    fn my_addd(self, rhs:i32) -> Self::o {
        self + rhs
    }
}



impl AddExportS<i32> for i32{
    type o = i32;
    fn my_adddd(self, rhs:i32) -> Self::o {
        self+ rhs
    }
}

impl AddExportS<String> for String{
    type o = String;
    fn my_adddd(self, rhs:String) -> Self::o {
        self + &rhs
    }
}

fn main(){
    let a:i32 = 20;
    let b = a.my_add(20);
    println!("{:?}",b);


    let c:i32 = 30;
    let cc = c.my_addd(30);
    println!("{:?}",cc);

    let d: i32 = 30;
    let dd = d.my_adddd(30);
    println!("{:?}",dd);

    let z: String = String::from("hello");
    let zz = z.my_adddd(String::from("world"));
    println!("{:?}",zz);
}
```
这里RHS=Self的意思就是获取自身变量的数据类型,保证了输入参数和调用参数的类型一致,比如做一个加法`1+2=3`是可以的，但是`1+"ss"`是不可以的，前后两个变量的类型不一致，写成方法就是`1.add("ss")`肯定是不可以的，


---
rust 孤儿规则
rust规定如果要实现某个trait ，那么给trait和要实现该trait的那个类型至少有一个要在当前create当中定义
因此如果我们要改写标准库当中的add的话，会出现错误

正常使用如下
```rust

use std::ops::Add;


fn main(){
    let a = 3;
    let aa = a.add(3);
    println!("{:?}",aa);
}
```
重写
```rust
use std::ops::Add;

#[derive(Debug)]
struct Point{
    x: i32,
    y: i32,
}


impl Add<u64> for u32{
    type Output = u32;
    fn add(self, i:u32) -> u32{
        (self as u32) + i
    }
}
```
这样写会报错，因为Add并没有被定义在当前create当中，而是被定义在标准库的create当中
重写
```rust
use std::ops::Add;

#[derive(Debug)]
struct Point{
    x: i32,
    y: i32,
}

impl Add for Point{
    type Output = Point;
    fn add(self, other: Point) -> Point{
        Point{
            x: self.x + other.x,
            y: self.y + other.y
        }
    }
}

fn main(){
    println!("{:?}",Point{x:1,y:2}.add(Point{x:1,y:2}));
    println!("{:?}",Point{x:1,y:2}+Point{x:1,y:2});
}
```

调用者自身的变量类型
```rust
trait Add<RHS=Self> {
    type O;
    fn add(self, value: RHS) -> Self::O;
}

impl Add for u32{
    type O = u64;
    fn add(self, value: u32) -> Self::O {
        (self as u64) + (value as u64)
    }
}

fn main(){
    let a = 1u32;
    let b = 2u32;
    assert_eq!(a.add(b),3);
}
```
一直在想这个`=Self`加了和没加有什么区别， 首先要这个这个RHS就是一个泛型变量类型 用来局限泛型的范围的，现在RHS=Self 也就是调用者自身的变量类型 rust中的Add源码就是上面这个， 现在可以兼容各种类型的变量类型 比如一个坐标
```rust
trait Add<RHS=Self> {
    type O;
    fn add(self, value: RHS) -> Self::O;
}

impl Add for u32{
    type O = u64;
    fn add(self, value: u32) -> Self::O {
        (self as u64) + (value as u64)
    }
}

#[derive(Debug,PartialEq, PartialOrd)]
struct Point{
    x: i32,
    y: i32,
}

impl Add for Point{
    type O = Point;
    fn add(self, value: Point) -> Self::O {
       Point {
           x: self.x + value.x,
           y: self.y + value.y
       }
    }
}

fn main(){
    let a = 1u32;
    let b = 2u32;
    assert_eq!(a.add(b),3);

    let p1 =  Point{x:1,y:2};
    let p2 = Point{x:2,y:3};

    let p3 =  p1.add(p2);
    assert_eq!(p3, Point { x: 3, y: 5 });
}
```

---
trait默认实现
trait 支持默认实现,默认实现的方法中需要传入&self 不能使用self 这个涉及到动态大小的问题
```rust
trait Page{
    fn set_page(&self,nums:i32){
        println!("{}",nums);
    }
}

trait PerPage{
    fn set_perpage(&self,nums:i32){
        println!("{}",nums);
    }

}


struct MyPaginate{page: i32}
impl Page for MyPaginate{}
impl PerPage for MyPaginate{}


fn main(){
    let mp = MyPaginate{page:1};
    mp.set_page(1)
}
```

动态类型
https://doc.rust-lang.org/book/ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait

---
trait继承
```rust
trait Page{
    fn set_page(&self,nums:i32){
        println!("{}",nums);
    }
}

trait PerPage{
    fn set_perpage(&self,nums:i32){
        println!("{}",nums);
    }

}


struct MyPaginate{page: i32}
impl Page for MyPaginate{}
impl PerPage for MyPaginate{}

trait Paginate: Page+ PerPage{
    fn set_skip_page(&self, num: i32){
        println!("skip poage :{:?}",num)
    }
}

impl <T: Page + PerPage > Paginate for T{}

fn main(){
    let mp = MyPaginate{page:1};
    mp.set_page(1);
    mp.set_skip_page(20);
}
```


trait限定 也是为了来限制泛型的范围
```rust
use std::ops::Add;


fn sum<T: Add<T, Output=T>>(a:T, b:T) -> T{
    a+b
}

fn main(){
    assert_eq!(sum(2,3),5);
}
```
add的源码
```rust
pub trait Add<Rhs = Self> {
    type Output;
    fn add(self, rhs: Rhs) -> Self::Output;
}
```
这里限制了Output必须为T 输入的两个参数也必须为T


trait 所有对泛型的范围的约束 统称为 Trait限定  trait Bound

---
trait 继承优化
利用where 提升代码可读性
```rust
trait Page{
    fn set_page(&self,nums:i32){
        println!("{}",nums);
    }
}

trait PerPage{
    fn set_perpage(&self,nums:i32){
        println!("{}",nums);
    }

}


struct MyPaginate{page: i32}
impl Page for MyPaginate{}
impl PerPage for MyPaginate{}

trait Paginate: Page+ PerPage{
    fn set_skip_page(&self, num: i32){
        println!("skip poage :{:?}",num)
    }
}

impl <T: Page + PerPage > Paginate for T{}

fn once<T: Page + PerPage >(s: T, nums:i32) {
    s.set_skip_page(nums);
} 

fn once2<T>(s:T, nums: i32) where T: PerPage+ Page {
    s.set_skip_page(nums);
}

fn main(){
    let mp = MyPaginate{page:1};
    mp.set_page(1);
    mp.set_skip_page(20);
    once(mp, 2);
    // once2(mp, 30);

}
```

---
标签
Rust一共提供了5个重要的标签trait，都被定义在标准库std：：marker模块中。它们分别是：
- Sized trait，用来标识编译期可确定大小的类型。
- Unsize trait，目前该trait为实验特性，用于标识动态大小类型（DST）。
- Copy trait，用来标识可以按位复制其值的类型。
- Send trait，用来标识可以跨线程安全通信的类型。
- Sync trait，用来标识可以在线程间安全共享引用的类型。





## rust中常见的trait

### From and Into
两个是互逆的关系，官方库里面trait内容是这样实现的,源码库位置`std::convert::From` 以及`std::convert::Into`
```rust
pub trait From<T> {
#[lang = "from"]    pub fn from(T) -> Self;
}

pub trait Into<T> {
    pub fn into(self) -> T;
}

impl<T, U> Into<U> for T 
    where U: From<T>
{
    pub fn into(self) -> U
}
   
```
上面的两段内容意思分别是 
- From 传入类型T 返回自身类型Self
- Into 将自身的类型转换为T
- 实现From trait的类型T 自动实现Into trait


看一个String字符串中的实现，其实我们只要实现From trait就ok了
```rust
impl<'_> From<&'_ String> for String{
    pub fn from(s: &String) -> String
}
impl<'_> From<&'_ mut str> for String{
    pub fn from(s: &mut str) -> String
}
impl<'_> From<&'_ str> for String{
    pub fn from(s: &str) -> String
}
```
这里源码里面分别对 String类型, 可变的字符串引用类型，以及字面量类型都实现了这个From trait,相应的他们也会自动实现Into trait
```rust
fn main(){
    
    let xx = String::from("xx");
    let xxx = String::from(xx); // String
    let xx = String::from("xx");
    let xxx_into:String = xx.into();
    assert_eq!(xxx,xxx_into);

    let xx = String::from("xx"); // &str
    let xx_into:String = "xx".into();
    assert_eq!(xx,xx_into);
    
    let mut x = "xx";
    let xx  = String::from(x); // &mut str
    let mut x = "xx";
    let xx_into:String  = x.into();

}
```

因此对于一些变量类型我们可以手动实现From trait就可以实现Into trait

```rust
use std::convert::From;

struct Number{
    value :i32
}

impl From<i32> for Number {
    fn from(item:i32) -> Self {
        Number{
            value: item
        }
    }
}

fn main(){
    let x = "xx";
    let xx = String::from(x);
    let yy:String = x.into();

    let n1 = Number::from(20);
    let n2:Number = 20.into();
    let n3:Number = 30.into();
}

```

https://rustcc.gitbooks.io/rustprimer/content/intoborrow/into.html
https://rustwiki.org/zh-CN/rust-by-example/conversion/from_into.html
https://ricardomartins.cc/2016/08/03/convenient_and_idiomatic_conversions_in_rust
https://islishude.github.io/blog/2020/09/22/rust/Rust-%E4%B8%AD%E7%9A%84-From-%E5%92%8C-Into-trait/

### AsRef 与 AsMut
也在标准库中的convert下面`std::convert::AsRef` 以及 `std::convert::AsMut`
```rust
pub trait AsMut<T> where T: ?Sized, {
    pub fn as_mut(&mut self) -> &mut T;
}
pub trait AsRef<T> where T: ?Sized, {
    pub fn as_ref(&self) -> &T;
}
```
上面的是源码库中的代码
- AsMut 获取一个自身的可变变量的引用
- AsRef 获取一个自身的引用
- ?Sized 因为返回的是一个引用类型，在编译器内无法获取类型大小，所以要实现？Sized这个trait

在String上的实现
```rust
impl AsMut<str> for String{
    pub fn as_mut(&mut self) -> &mut str
}

impl AsRef<[u8]> for String{
    pub fn as_ref(&self) -> &[u8]
}
```
这两个的好处就是方便和统一化，比如我们我们要写一个方法分别接受String和str，需要写两个方法
```rust
fn main(){
    let xx = "xx";
    is_hello_str(xx);

    let yy = "xx".to_owned();
    is_hello_String(yy);
}


fn is_hello_str(s:&str)
{
    assert_eq!("xx",s)
}

fn is_hello_String(s:String)
{
    assert_eq!("xx",s.as_str());
}
```
但是你用了AsRef这个trait 后就可以直接用一个泛型就写完了
```rust
fn main(){
    let xx = "xx";
    is_hello(xx);

    let yy = "xx".to_owned();
    is_hello(yy);
}


fn is_hello<T>(s:T)
    where T: AsRef<str>
{
    assert_eq!("xx",s.as_ref())
}
```

### Drop trait
当对象离开作用域时会自动调用该 方法。Drop trait 的主要作用是释放实现者的实例拥有的资源。

```rust
enum Class {
    People {name:String,age:i32},

}

impl Drop for Class {
    fn drop(&mut self) {
        println!("i will drop")
    }
}

fn main(){
    let c1 = Class::People{
        name:"xx".to_owned(),
        age:32,
    };
}
```

### 其他
https://rustcc.gitbooks.io/rustprimer/content/intoborrow/deref.html