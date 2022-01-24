---
title: Rust第三方库json
date: 2021-03-18 09:52:23
categories:  [rust]
tags: [rust]
---


<!--more-->


# Rust第三方库json
Serde数据模型是与数据结构和数据格式进行交互的API。您可以将其视为Serde的类型系统。RUST中常用这个库来交互各种数据结构
其中`serder_json`是一个分支，另外还有例如
- `serder_yaml` https://github.com/dtolnay/serde-yaml
- `serder_toml` https://github.com/alexcrichton/toml-rs
- `serder-picker` https://github.com/birkenfeld/serde-pickle
等等工具

所有需要序列化和反序列化的数据都需要先声明marco,这是一种派生宏，s

## serde
https://serde.rs/
序列化与反序列化，引用过程
```rust
// ./Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }

# serde_json is just for the example, not required in general
serde_json = "1.0"
```
```rust
// ./main.rs
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let point = Point { x: 1, y: 2 };

    // Convert the Point to a JSON string. // 序列化
    let serialized = serde_json::to_string(&point).unwrap();

    // Prints serialized = {"x":1,"y":2}
    println!("serialized = {}", serialized);

    // Convert the JSON string back to a Point. // 反序列化
    let deserialized: Point = serde_json::from_str(&serialized).unwrap();

    // Prints deserialized = Point { x: 1, y: 2 }
    println!("deserialized = {:?}", deserialized);
}
```
## 对结构体增加属性
serde支持对结构体绑定属性

### 更名属性
类似于
```
#[serde(rename(serialize = "ser_name"))]
#[serde(rename(deserialize = "de_name"))]
#[serde(rename(serialize = "ser_name", deserialize = "de_name"))] #默认情况下是这个
```

```rust
use serde::{Serialize, Deserialize};
use serde_json::*;

#[derive(Serialize, Deserialize, Debug)]
#[serde(rename = "e")] 
struct Point {
    #[serde(rename = "a")]
    x: i32,
    #[serde(rename = "b")]
    y: i32,
}
fn main() {
    let point = Point{
        x:1,
        y:2,
    };
    let ser_point = serde_json::to_string(&point).unwrap();
    println!("{}",ser_point); //  {"a":1,"b":2}
}
```
```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
    #[serde(rename(serialize = "ser_name", deserialize = "de_name"))]     // 反序列化后key值为x 序列化后key值为a
    x: i32,
    y: i32,
}
fn main() {
    let point = Point{
        x:1,
        y:2,
    };
    let ser_point = serde_json::to_string(&point).unwrap();
    println!("{}",ser_point); //  {"a":1,"y":2}
    let ser_point = r#"
    {
        "de_name":1,
        "y":2
    }
    "#;
    let deser_point:Point = serde_json::from_str(ser_point).unwrap();
    println!("{:?}",deser_point);
}

```
```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
#[serde(rename_all = "UPPERCASE")]
struct Point {
    // #[serde(rename(serialize = "ser_name", deserialize = "de_name"))]     // 反序列化后key值为x 序列化后key值为a
    x: i32,
    y: i32,
}
fn main() {
    let point = Point{
        x:1,
        y:2,
    };
    let ser_point = serde_json::to_string(&point).unwrap();
    println!("{}",ser_point); //  {"X":1,"Y":2}
}
```


## serde_json

