---
title: Rust设计模式
date: 2021-04-09 15:12:54
categories:  [rust]
tags: [rust]
---


<!--more-->


# Rust设计模式

> 其他类型的设计模式
https://www.cnblogs.com/li-peng/p/12485666.html

## rust建造者模式(常用 好用)

```rust
use std::env::consts::DLL_SUFFIX;
// 胖指针
// Sized?
// Sized!
// 建造者模式

#[derive(Debug)]
struct FooBuilder{
    bar: String,
    download_url: String,
    keywards: String
}

#[derive(Default,Debug)]
struct Foo{
    bar: String,
    download_url: String,
    keywards: String,
}

impl Foo {
    pub fn new() -> FooBuilder {
        // set the minimally required fields of Foo.
        FooBuilder {
            bar: String::from("x"),
            download_url: String::from("http:://www.baidu.com"),
            keywards: String::from("foobar"),
        }
    }

    
}

impl FooBuilder {


    pub fn set_name(mut self, bar: String) -> FooBuilder {
        // set the name on the builder iteself,
        // and return the builder by value.
        self.bar = bar;
        self  // This is where the ownership moves
    }

    pub fn set_download_url(mut self, download_url: String) -> FooBuilder{
        self.download_url = download_url;
        self
    }

    pub fn set_keywards(mut self, keywards: String) -> FooBuilder{
        self.keywards = keywards;
        self
    }


    // if we can get away with not consuming the builder here, that is an 
    // advantage. It means we can use the FooBuilder as a template for constructing many Foo.
    pub fn build(self) -> Foo {
        // Create a Foo from Foo the FooBuilder, applying all settings in FooBuilder to Foo. 
        Foo{
            bar: self.bar,
            download_url: self.download_url,
            keywards: self.keywards,
        }
    }
}

#[derive(Debug)]
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle{
    fn new() -> Self { 
        Circle{
            x:0.0,
            y:0.0,
            radius:0.0
        }
    }

    fn set_x(mut self , x:f64) -> Circle{
        self.x = x;
        self
    }

    fn set_y(mut self, y:f64) -> Circle{
        self.y = y;
        self
    }

    fn set_radius(mut self, radius:f64) -> Circle{
        self.radius = radius;
        self
    }
    
    fn build(self) -> Self{
        Self {
            x: self.x,
            y: self.y,
            radius: self.radius
        }
    }
}

fn main(){
    // build mode
    let foo = Foo::new()
        .set_download_url("http://www.baidu.com".to_string())
        .set_keywards("foobar".to_string())
        .set_name("helloworld".to_string())
        .build();

    let c1 = Circle::new()
        .set_x(1.0)
        .set_y(2.0)
        .set_radius(10.0)
        .build();

    println!("{:?}{:?}",foo,c1);
    
}
```