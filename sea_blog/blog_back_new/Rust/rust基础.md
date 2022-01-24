---
title: rust基础
categories:
  - rust
tags:
  - rust
abbrlink: e13282a0
date: 2021-03-03 10:04:35
top: 1 
---


<!--more -->


# rust 基础


## 常见集合



## 生命周期以及引用有效性

```rust
Rust 中的每一个引用都有其 生命周期（lifetime），也就是引用保持有效的作用域。大部分时候生命周期是隐含并可以推断的，正如大部分时候类型也是可以推断的一样。类似于当因为有多种可能类型的时候必须注明类型，也会出现引用的生命周期以一些不同方式相关联的情况，所以 Rust 需要我们使用泛型生命周期参数来注明他们的关系，这样就能确保运行时实际使用的引用绝对是有效的。
```

生命周期的主要目标是避免悬垂引用，它会导致程序引用了非预期引用的数据
垂直引用的主要原因就是指针指向的内存地址为空

```rust
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

如上，x 离开作用域后，被 rust 调用的 drop 释放掉，r 所指向的内存地址为空

### 生命周期介绍

首先我们要明确一点，生命周期的出现是为了避免出现悬垂引用，因此对于不是引用的参数没有影响，比如

```rust
fn main() {
    let a:i32 = 1;
    let b:i32 = 2;
    println!("{}",func1(1,2));
}
// 不会报错 生命周期为全局
fn func1(x:i32,y:i32) -> i32{
    if x > y{
        x
    }else {
        y
    }
}
```

为什么没有影响，后面会介绍

```rust
fn main() {
    let a:i32 = 1;
    let b:i32 = 2;
    println!("{}",func1(1,2));
}
// 不会报错 生命周期为全局
fn func1(x:&i32,y:&i32) -> i32{
    if x > y{
        *x
    }else {
        *y
    }
}
```

没有影响

接下来是有影响的部分，也就是 String 与&str 两个值，如下

```rust
fn main(){
    let a:&str = "hello";
    let b = String::from("helloworld");
    println!("{}",func(a,b.as_str())); //报错
}
// 报错 无法判断x和y两个参数的生命周期
fn func(x:&str,y:&str) -> &str{
    if x.len() > y.len(){
        x
    }else {
        y
    }
}
```

上面的内容就会出现错误，rust 无法判断 x 和 y 的生命周期，因此要定义一个生命周期 来保证 x y 以及返回的内容再同一个生命周期当中

```rust
fn func<'a>(x:&'a str,y:&'a str) -> &'a str{
    if x.len() > y.len(){
        x
    }  else {
        y
    }
}
// 现在函数签名表明对于某些生命周期 'a，函数会获取两个参数，他们都是与生命周期 'a 存在的一样长的字符串 slice。函数会返回一个同样也与生命周期 'a 存在的一样长的字符串 slice。它的实际含义是 longest 函数返回的引用的生命周期与传入该函数的引用的生命周期的较小者一致。这就是我们告诉 Rust 需要其保证的约束条件。记住通过在函数签名中指定生命周期参数时，我们并没有改变任何传入值或返回值的生命周期，而是指出任何不满足这个约束条件的值都将被借用检查器拒绝。注意 longest 函数并不需要知道 x 和 y 具体会存在多久，而只需要知道有某个可以被 'a 替代的作用域将会满足这个签名

// 当在函数中使用生命周期注解时，这些注解出现在函数签名中，而不存在于函数体中的任何代码中。这是因为 Rust 能够分析函数中代码而不需要任何协助，不过当函数引用或被函数之外的代码引用时，让 Rust 自身分析出参数或返回值的生命周期几乎是不可能的。这些生命周期在每次函数被调用时都可能不同。这也就是为什么我们需要手动标记生命周期。

// 当具体的引用被传递给 longest 时，被 'a 所替代的具体生命周期是 x 的作用域与 y 的作用域相重叠的那一部分。换一种说法就是泛型生命周期 'a 的具体生命周期等同于 x 和 y 的生命周期中较小的那一个。因为我们用相同的生命周期参数 'a 标注了返回的引用值，所以返回的引用值就能保证在 x 和 y 中较短的那个生命周期结束之前保持有效。
```

<font color='red'>生命周期注解并不改变任何引用的生命周期的长短。与当函数签名中指定了泛型类型参数后就可以接受任何类型一样，当指定了泛型生命周期后函数也能接受任何生命周期的引用。生命周期注解描述了多个引用生命周期相互的关系，而不影响其生命周期。生命周期注解有着一个不太常见的语法：生命周期参数名称必须以撇号（'）开头，其名称通常全是小写，类似于泛型其名称非常短。'a 是大多数人默认使用的名称。生命周期参数注解位于引用的 & 之后，并有一个空格来将引用类型与生命周期注解分隔开。</font>

```rust
&i32 //引用
&'a i32  // 显示生命周期为'a 的i32类型引用
&'a mut i32 // 显示生命周期为'a 的可变的i32类型引用
```

### 生命周期的实例

我们已经知道了生命周期会统一参数中最小的生命周期为当前生命周期

```rust
fn main() {
    let xx = "helloword";
    {
        let yy = String::from("hello");
        let res = func(xx,yy.as_str());
        println!("{}",res);
    }
}

fn func<'a>(x:&'a str,y:&'a str) -> &'a str{
    if x.len() > y.len(){
        x
    }  else {
        y
    }
}
```

如上，生命周期最小的是 y,于是'a 会统一所有带有显式生命周期`'a`缩短到和 y 生命周期同样长度，在 y 结束生命周期后，恢复到原来的生命周期

### 深入生命周期

```rust
fn main() {
    let xx = "helloword";
    {
        let yy = String::from("hello");
        let res = func1(xx,yy.as_str());
        println!("{}",res);
    }
}

fn func1<'a>(x:&'a str,y:&str) -> &'a str{
    if x.len() > y.len(){
        x
    } else {
        y  // 报错，因为y的生命周期并不是'a  与返回值不同
            // 生命周期的长度不同，这里'a的声明周期长度应该是xx

    }
```

### 生命周期省略规则

函数或方法的参数的生命周期被称为 输入生命周期（input lifetimes），而返回值的生命周期被称为 输出生命周期（output lifetimes）。

- 每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
- 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`
- 如果方法有多个输入生命周期参数并且其中一个参数是`&self`或 `&mut self`，说明是个对象的方法(method) 那么所有输出生命周期参数被赋予 `self` 的生命周期。第三条规则使得方法更容易读写，因为只需更少的符号

```rust
fn main(){
    //  - 每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
    //  - 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`
    //  - 如果方法有多个输入生命周期参数并且其中一个参数是` &self `或 `&mut self`，说明是个对象的方法(method) 那么所有输出生命周期参数被赋予 `self` 的生命周期。第三条规则使得方法更容易读写，因为只需更少的符号
    println!("lifrtime for function");
}


// 第一个规则所有引用参数都有它的生命周期
fn func1_1(xx:&str){}

fn func1_1_1<'a>(xx:&'a str) {}

//
fn func1_2(xx:&str,yy:&str) { }
fn func1_2_1<'a,'b>(xx:&'a str,yy:&'b str) {}

// 如果只有一个生命周期，那么他被赋予所有输出生命周期参数
fn func2(xx:&str) -> &str{}
fn func2<'a>(xx: &'a str) -> &'a str{}

//
```

### 静态生命周期

`'static`，其生命周期能够存活于整个程序期间。所有的字符串字面值都拥有 `'static` 生命周期，我们也可以选择像下面这样标注出来：

```rust
let s: &'static str = "I have a static lifetime.";
```

### 生命周期+泛型+trait bound 实例

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```




## 引用与解引用

```rust
fn by_ref(x: &i32) -> i32{
    *x + 1
}

fn main() {
    let i = 10;
    let res1 = by_ref(&i);
    let res2 = by_ref(&41);
    println!("{} {}", res1,res2);
}
```


## 切片
切片(Slice)是一种没有所有权的数据类型。 切片引用连续的内存分配而不是整个集合。 它允许安全，高效地访问数组而无需复制。 切片不是直接创建的，而是从现有变量创建的。 切片由长度组成，并且可以是可变的或不可变的。 切片的行为与数组相同。//原文出自【易百教程】，商业转载请联系作者获得授权，非商业请保留原文链接：https://www.yiibai.com/rust/rust-slices.html

使用切片

```rust
fn main() {
    let words = "hello wolrd";

    // 获取字符串长度
    let words_numbers = words.len();
    // 获取所有字符串
    let words1 = &words[0..];
    // 字符串切片
    let words2 = &words[0..2];

    println!("words is '{}'\nwords_numbers is '{}' \nwords1 is '{}'\n",words,words_numbers, words1);
    println!("words2 is '{}'",words2);
}

```

```rust
// 累加和
fn main() {
    let arr = [10,20,30,40];
    let res = sum_numbers(&arr[0..]);

    println!("{}",res)
}

// 获取一个数组切片 类型为i32，返回一个i32
fn sum_numbers(arr: &[i32]) -> i32{
    let mut res = 0;
    for i in 0..arr.len(){
        res += arr[i];
    }
    res

}
```

<font color='red'>切片的 get 方法</font>

get 方法返回一个 option 值

- 如果给定位置，则返回对该位置处元素的引用或 None 超出范围。
- 如果给定范围，则返回对应于该范围的子切片，或者 None 超出范围。

```rust
fn main() {
    let ints = [1, 2, 3, 4, 5];
    let slice = &ints;
    let first = slice.get(0);
    let last = slice.get(5);

    println!("first {:?}", first);
    println!("last {:?}", last);
}
// first Some(1)
// last None
```



## 迭代器

```rust
fn main() {
    let mut iter = 0..3;
    assert_eq!(iter.next(), Some(0));
    assert_eq!(iter.next(), Some(1));
    assert_eq!(iter.next(), Some(2));
    assert_eq!(iter.next(), None);
}
```

迭代器很容易定义。 下面是一个”对象”，它使用 next 方法返回一个 Option。只要这个值不是 None，我们就一直 next 下去:
使用迭代进行循环

```rust
fn main() {
    let arr = [10, 20, 30];
    for i in arr.iter() {
        println!("{}", i);
    }

    // 切片将隐式转换为迭代器...
    let slice = &arr;
    for i in slice {
        println!("{}", i);
    }
}
```

使用迭代进行数字累加

```rust
fn main() {
    let sum: i32  = (0..5).sum();
    println!("sum was {}", sum);

    let sum: i64 = [10, 20, 30].iter().sum();
    println!("sum was {}", sum);
}
```

[更多切片方法](https://doc.rust-lang.org/std/primitive.slice.html)



## 定义匹配范围

```rust
 let text = match n {
        0...3 => "small",
        4...6 => "medium",
        _ => "large",
     };
```


## never类型

```rust
fn main() {
    // let vec: Vec<()> = vec![();10];
    // let vec: () = vec![();10];
    let i = if false{
        foo();
    } else {
        100
    };

    foo();

    let x = "1";
    println!("{:?}",)
}



fn foo() -> ! {
    loop { 
        println!("jh");
    }
}

```


## 多态
静态多态与动态多态

## rust泛型
泛型是一种参数化多态,使用泛型可以编写更加抽象的代码，减少工作量。泛型就是把一个泛化的类型作为参数，单个类型就可以抽象化为一簇类型

- 泛型类型 指定泛型所支持类型的范围，`<T>` 所有类型
- 泛型参数

```rust
fn foo<T>(s:T) -> T{
    s
}

fn main(){
    print!("{:?\n}",foo(3));
    assert_eq!(foo(3),3);
    assert_eq!(foo("xx"),"xx");
}
```
这里`foo<T>` 叫做泛型的声明， 泛型只有在被声明之后才能使用，为泛型结构体实现具体方法的时候，也需要声明泛型类型，如果
```rust
struct Point<T>{
    x: T,
    y: T,
}

impl<T> Point<T>{
    fn new(x:T, y:T) -> Self{
        return Point{
            x:x,
            y:y
        }
    }
}

fn main(){
    let point = Point{
        x:1,
        y:2,
    };

    let point1 = Point::new(1,2);
    assert_eq!(point1.y,2); 
    assert_eq!(point1.x,1);
}
```
这里的`::`双冒号的用法叫做Turbofish,Turbofish通常用于在表达式中为泛型类型、函数或方法指定参数，比如下面这个例子
```rust
fn main(){
    let mut v = Vec::new(); // 报错 vec的类型unknow
    let mut v = Vec::<i32>::new();
}
```

> Rust中的泛型属于静多态，它是一种编译器多态。在编译器，不管是泛型枚举，还是泛型函数和泛型结构体，都会被单态化(Monomorphization). 单态化是编译器进行静态分发的一种策略，单态化意味着编译器要将一个泛型函数生成两个具体类型对应的函数。单态化静态分发的好处是性能好，没有运行时开销；缺点是容易造成编译后生成的二进制文件膨胀


## trait
从类型系统的角度来说，trait是Rust对Ad-hoc多态的支持。从语义上来说，trait是在行为上对类型的约束，trait有如下4种用法:
- 接口抽象，对类型行为的统一约束
- 泛型约束，trait可以作为接口变量 将泛型的类型限定在更有限的范围内
- 抽象类型。在运行时作为一种间接的抽象类型去使用，动态地分发给具体的类型 
- 标签trait。对类型的约束，可以直接作为一种“标签”使用。

同一个trait，在不同的上下文中实现的行为不同。为不同的类型实现trait，属于一种函数重载，也可以说函数重载就是一种Ad-hoc多态

接口抽象
```rust
struct Foo{
    name: String
}

struct Bar{
    name: String
}
trait  Ins {
    fn echo(&self,msg: &str);
    fn echo_message(&self, msg: &str);
}

impl Ins for Foo{
    fn echo(&self, msg: &str) {
        println!("{:?}",msg)
    }
    fn echo_message(&self, msg: &str) {
        self.echo(msg)
    }
}

impl Ins for Bar{
    fn echo(&self, msg: &str) {
        println!("{:?}",msg)
    }
    fn echo_message(&self, msg: &str) {
        self.echo(msg)
    }
}
```

类型约束
```rust
fn echo_static<T: Ins>(s: T){
    s.echo("helloworld");
}
// 调用
fn main(){
    let foo = Foo{
        name: "hz".to_string()
    };
    echo_static(foo);
}
// 或者
fn main(){
    let foo = Foo{
        name: "hz".to_string()
    };
    echo_static::<Foo>(foo)
}
```

抽象类型 动态分发
```rust

fn echo_dyn(s: &dyn Ins){
    s.echo("helloworld");
}


fn main(){
    let foo = Foo{
        name: "hz".to_string()
    };
    echo_dyn(&foo)
}
```

总的代码
```rust
struct Foo{
    name: String
}

struct Bar{
    name: String
}
trait  Ins {
    fn echo(&self,msg: &str);
    fn echo_message(&self, msg: &str);
}

impl Ins for Foo{
    fn echo(&self, msg: &str) {
        println!("{:?}",msg)
    }
    fn echo_message(&self, msg: &str) {
        self.echo(msg)
    }
}

impl Ins for Bar{
    fn echo(&self, msg: &str) {
        println!("{:?}",msg)
    }
    fn echo_message(&self, msg: &str) {
        self.echo(msg);
        self.echo(&self.name)
    }
}


fn echo_static<T: Ins>(s: T){
    s.echo("helloworld");
}


fn echo_dyn(s: &dyn Ins){
    s.echo("helloworld");
}


fn main(){
    let foo = Foo{
        name: "hz".to_string()
    };
    echo_dyn(&foo)
}
```


### trait使用方法



> 关联类型
```rust
trait Add<RHS,OUTPUT>  {
    fn my_add(self, rhs:RHS) -> OUTPUT;
}

impl Add<i32,i32> for i32{
    fn my_add(self, rhs:i32) -> i32 {
        self+rhs
    }
}

impl Add<u32,i32> for u32{
    fn my_add(self, rhs:u32) -> i32 {
        (self + rhs) as i32
    }
}

fn main(){
    let a:i32 = 20;
    let b = a.my_add(20);
    println!("{:?}",b);
}
```
如上trait定义了两个参数类型，一个是RHS 作为加数的变量，另一个OUTPUT作为返回结果的变量

## 划分线


```rust
trait Add<RHS=Self>  {
    type OUTPUT;
    fn my_add(self, rhs:RHS) -> Self::OUTPUT;
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

trait AddExport<RHS>{
    type o;
    fn my_addd(self, rhs:RHS) -> Self::o;
}

impl AddExport<i32> for i32{
    type o = i32;
    fn my_addd(self, rhs:i32) -> Self::o {
        self + rhs
    }
}

trait AddExportS<RHS> {
    type o;
    fn my_adddd(self, rhs:RHS) -> Self::o;
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

> rust 孤儿规则
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


trait 所有对泛型的范围的约束 统称为 Trait限定  trait Bound

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