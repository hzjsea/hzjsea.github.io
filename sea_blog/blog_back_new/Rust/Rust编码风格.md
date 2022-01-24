---
title: Rust编码风格
date: 2021-03-10 15:12:38
categories:  [rust]
tags: [rust]
---


<!--more-->


# 编码风格


总而言之，Rust对面向对象编程风格的支持可以总结为以下几点。
- 封装。Rust提供了结构体（Struct）和枚举体（Enum）来封装数据，并可使用pub关键字定义其字段可见性；提供了impl关键字来实现数据的行为。
- 多态。通过trait和泛型以及枚举体（Enum）来允许程序操作不同类型的值。
- 代码复用。通过泛型单态化、trait对象、宏（macro）、语法扩展（procedural macro）、代码生成（code generation）来设计模式


## 建造者模式

建造者模式（Builder Pattern）是Rust中最常用的设计模式之一。建造者模式是指使用多个简单的对象一步步构建一个复杂对象的模式。该模式的主要思想就是将变和不变分离。对于一个复杂的对象，肯定会有不变的部分，也有变化的部分，将它们分离开，然后依次构建

```rust
// fn main(){
//     let s1:&str = "xx";
//     {
//         let s2:&str = "xxx";
//         let s6 = the_longest(s1, s2);
//         print!("{}",s6);
//     }
// }

// fn the_longest(s3: &str, s4: &str) -> &str{
//     if s3.len() > s4.len() {s3} else {s4}
// }


// trait Shape {
//     fn area(&self) -> f64;
//     fn width() -> f64;
// }

// fn main(){
    
// }


// struct Circle{
//     radius: f64
// }

// // 定义方法集
// trait ShapeAction{
//     fn area(&self) -> f64;
//     fn echo(){
//         print!("helloworld");
//     }
// }

// // 实现方法集
// impl ShapeAction for Circle{
//     fn area(&self) -> f64 {
//         std::f64::consts::PI *self.radius*self.radius
//     }
//     fn echo() {
//         print!("实现 Circle")
//     }

// }


// // 匿名trait
// impl Circle{
//     fn get_radius(&self) -> f64 { self.radius}
// }

// fn main(){
//     let c1 = Circle{
//         radius:1f64,
//     };
//     let c1_area = c1.area();
//     print!("{}",c1_area);

//     c1.get_radius(); // 默认实现

//     Circle::echo();
// }


use core::fmt;
use std::process::Command;

struct Circle{
    x: f64,
    y: f64,
    redius:f64,
}

struct CircleBuilder{
    x:f64,
    y:f64,
    redius:f64,
}

impl Circle{
    fn area(&self) -> f64{
        std::f64::consts::PI *(self.redius *self.redius)
    }
    fn new() -> CircleBuilder{
        CircleBuilder{
            x:0.0,
            y:0.0,
            redius:1.0,
        }
    }
}

impl CircleBuilder{
    fn gen_x(&mut self, coordinate: f64) -> &mut CircleBuilder{
        self.x = coordinate;
        self
    }
    fn gen_y(&mut self, coordinate: f64) -> &mut CircleBuilder{
        self.x = coordinate;
        self
    }
    fn gen_redius(&mut self, coordinate: f64) -> &mut CircleBuilder{
        self.x = coordinate;
        self
    }
    fn build(&self) -> Circle{
        Circle{
            x: self.x,
            y: self.y,
            redius: self.redius
        }
    }
}



fn main(){
    let c = Circle::new()
    .gen_x(20.0)
    .gen_y(20.0)
    .gen_redius(10.0)
    .build();

    Command::new("ls")
    .arg("-l")
    .arg("-a")
    .spawn()
    .expect("ls command failed to start");
    

    print!("{}",c.area());

}
```

### 源码中的建造者模式

源码来自于`std::fs::DirBuilder`
```rust
pub struct DirBuilder {
    inner: fs_imp::DirBuilder, //这个先不用去看他是什么类型的
    recursive: bool,
}

impl DirBuilder{
      
    pub fn new() -> DirBuilder {  // 返回一个自身变量类型
        DirBuilder { inner: fs_imp::DirBuilder::new(), recursive: false }
    }

    pub fn recursive(&mut self, recursive: bool) -> &mut Self {  // 接收一个自身的变量，其实就相当于不断地给struct这个结构体赋值属性
        self.recursive = recursive;
        self
    }

    pub fn create<P: AsRef<Path>>(&self, path: P) -> io::Result<()> {
        self._create(path.as_ref())
    }
}
```
使用后如下
```rust
fn main(){
    let file_builder = std::fs::DirBuilder::new()
                                .recursive(true) // 是否递归
                                .create("xx/yy/zz/")
                                .unwrap();
}
```