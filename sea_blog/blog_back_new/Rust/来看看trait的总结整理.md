---
title: 来看看trait的总结整理
categories:
  - rust
tags:
  - rust
abbrlink: 63c8ad26
date: 2020-08-17 10:32:46
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->

# 来看看trait的总结整理

trait是服务于变量类型的，他规定了类型必须提供哪些功能语言特性

比如在之前要计算一个圆的面积，可以单独使用一个方法来解决，在得到圆的各个参数后，调用此方法就可以得出最后的结论
```rust

struct circle{
    x: f64,
    y: f64,
    radius: f64,
}

impl circle {
    fn area(&self) -> f64{
        std::f64::consts::PI * (self.radius * self.radius)
    }
}


fn main() {
   let c1 = circle{x:32.1,y:32.1, radius: 0.0 };
    println!("{}",c1.area());
}

```
但是同样的我们也可以使用trait来实现这个原理，原理就是对circle来类型来创建一个trait
```rust

struct circle{
    x: f64,
    y: f64,
    radius: f64,
}

trait HasArea {
    fn area(&self) -> f64;
}

impl HasArea for circle {
    fn area(&self) -> f64{
        std::f64::consts::PI * (self.radius * self.radius)
    }
}

fn main() {
   let c1 = circle{x:32.1,y:32.1, radius: 1.0 };
    println!("{}",c1.area());
}

```

乍一看感觉都差不多，并且trait的代码还多一点。但其实trait并不止这些

```rust

trait Animal {
   fn talk(&self);
}
struct Cat {}
struct Dog {}
impl Animal for Cat {
  fn talk(&self) {
    println!(“meow”);
  }
}
impl Animal for Dog {
  fn talk(&self) {
    println!(“bark”);
  }
}
fn animal_talk(a: &dyn Animal) {
  a.talk();
}
fn main() {
  let d = Dog {};
  let c = Cat {};
  animal_talk(&d);
  animal_talk(&c);
}
```
这里面用dyn来告诉rust不要去确定Animal的类型



```rust

#![allow(unused)]

use std::fmt::Display;
use std::path::Iter;

fn main() {
    trait Summary {
        fn summarize(&self) -> String;
    }

    struct NewsArticle {
        pub headline: String,
        pub location: String,
        pub author: String,
        pub content: String,
    }

    impl Summary for NewsArticle {
        fn summarize(&self) -> String {
            format!("{}, by {} ({})", self.headline, self.author, self.location)
        }
    }

    struct Tweet {
        pub username: String,
        pub content: String,
        pub reply: bool,
        pub retweet: bool,
    }

    impl Summary for Tweet {
        fn summarize(&self) -> String {
            format!("{}: {}", self.username, self.content)
        }
    }

    // 实现了Summary trait的变量
    fn notify(item: impl Summary) {
        println!("Breaking news! {}", item.summarize());
    }
    // trait bound
    fn notify2<T:Summary>(item: T){
        println!("Breaking news! {}", item.summarize());
    }
    // 绑定多个trait Display是官方已经实现的一个trait
    fn notify3<T: Summary+Display>(item: T){
        println!("Breaking news! {}", item.summarize());
    }
    // 简化trait bound
    fn some_function<T, U>(t: T, u: U) -> i32
        where T: Display + Clone,
              U: Clone + Debug
    {

    }

    // 在返回值中使用trait
    fn returns_summarizable() -> impl Summary {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from("of course, as you probably already know, people"),
            reply: false,
            retweet: false,
        }
    }
}

```

```rust
// 定义通用接口

trait Show{
    // 定义接口中的方法
    fn  show(&self) -> String;
}

impl Show for i32{
    fn show(&self) -> String{
        format!("creat i32 type datestuct {}",self)
    }
}

impl Show for f64{
    fn show(&self) -> String{
        format!("create f64 type datestruct{}",self)
    }
}



fn main() {
    let num1 = 32;
    let num2 = 32.9;
    let v1:Vec<&Show> = vec![&num1,&num2];  // 这里引用的是实现了  Show tarit的类型
    for d in v1.iter(){
        println!("{}",d.show());
    }
}

```

```rust
// 定义通用接口

trait Show{
    // 定义接口中的方法
    fn  show(&self) -> String;
}

impl Show for i32{
    fn show(&self) -> String{
        format!("creat i32 type datestuct {}",self)
    }
}

impl Show for f64{
    fn show(&self) -> String{
        format!("create f64 type datestruct{}",self)
    }
}



fn main() {
    let num1 = 32;
    let num2 = 32.9;
    let v1:Vec<&Show> = vec![&num1,&num2];  // 这里引用的是实现了  Show tarit的类型
    for d in v1.iter(){
        println!("{}",d.show());
    }


    let bo1 = Box::new(1); //创建堆上内容
    let bo2 = Box::new(2);
    let v2:Vec<Box<Show>> = vec![bo2,bo1];
    for d in v2.iter(){
        println!("{}",d.show());
    }
}

```

```rust

#![allow(unused_variables)]
fn main() {
    trait Quack {
        fn quack(&self);
    }

    struct Duck ();

    impl Quack for Duck {
        fn quack(&self) {
            println!("quack!");
        }
    }

    struct RandomBird {
        is_a_parrot: bool
    }

    impl Quack for RandomBird {
        fn quack(&self) {
            if ! self.is_a_parrot {
                println!("quack!");
            } else {
                println!("squawk!");
            }
        }
    }

    let duck1 = Duck();
    let duck2 = RandomBird{is_a_parrot: false};
    let parrot = RandomBird{is_a_parrot: true};

    let ducks: Vec<&Quack> = vec![&duck1,&duck2,&parrot];

    for d in &ducks {
        d.quack();
    }
// quack!
// quack!
// squawk!
}
```

