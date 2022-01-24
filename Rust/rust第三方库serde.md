---
title: rust第三方库serde
date: 2021-04-20 14:10:57
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust第三方库serde
Serde is a framework for serializing and deserializing Rust data structures efficiently and generically.
https://github.com/serde-rs/serde
https://serde.rs/

<div style='font-size:30px;color:red'>json to rust struct ---->  https://transform.tools/json-to-rust-serde</div>



<div style='font-size:20px;color:red'>serde example</div>
https://dev.to/pintuch/rust-serde-json-by-example-2kkf

### 使用serde 序列化和反序列化

添加第三方包变量
```toml
# Cargo.toml
[package]
name = "my-crate"
version = "0.1.0"
authors = ["Me <user@rust-lang.org>"]

[dependencies]
serde = { version = "1.0", features = ["derive"] }

# serde_json is just for the example, not required in general
serde_json = "1.0" # 这里以json的序列化和反序列化为实例
```

使用
```rust
use serde::{Serialize,Deserialize};
#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let point = Point { x: 1, y: 2 };


    // struct to string 序列化
    let serialized = serde_json::to_string(&point).unwrap();
    println!("serialized = {}", serialized);
    // println!("test{}",serialized.x); // error
    

    // string to struct 反序列化
    let deserialized: Point = serde_json::from_str(&serialized).unwrap();
    println!("deserialized = {:?}", deserialized);
    println!("test {}",deserialized.x)

    let header = r#"{
        "authorization": ""
        "Content-Type" : "application/json; charset=utf-8"
    }"#;
    let header:Header = serde_json::from_str(header).unwrap();


 }
```

## serde_json
serde_json is a module in Serde 


添加第三方包变量
```toml
# Cargo.toml
[package]
name = "my-crate"
version = "0.1.0"
authors = ["Me <user@rust-lang.org>"]

[dependencies]
serde = { version = "1.0", features = ["derive"] }

# serde_json is just for the example, not required in general
serde_json = "1.0" # 这里以json的序列化和反序列化为实例
```

使用
```rust
use serde::{Serialize,Deserialize};
#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let point = Point { x: 1, y: 2 };


    // struct to string 序列化
    let serialized = serde_json::to_string(&point).unwrap();
    println!("serialized = {}", serialized);
    // println!("test{}",serialized.x); // error
    

    // string to json 反序列化
    let deserialized: Point = serde_json::from_str(&serialized).unwrap();
    println!("deserialized = {:?}", deserialized);
    println!("test {}",deserialized.x)


 }
```

<div style='font-size:20px;color:red'>另外针对不同的自定义配置，可以进行不一样的配置选择</div>
https://serde.rs/attributes.html

### serde_json Value

想象一个场景,request请求一个接口，返回一段json数据，这个时候有两种做法
1. 根据json创建对应struct 接收
2. 使用Value来接收，但是这样的坏处是后面要一直使用unwarp来保证数据类型

> 方法1 serde_json 

```rust
use reqwest::*；
use serde::{Serialize,Deserialize};

struct Template{
    name:String,
    age:i32
}


    let client = reqwest::Client::builder().build()?;
    let resp = client
        .request(reqwest::Method::GET, url)
        .header("x-eneru-apikey", "api-key")
        .send()
        .await?
        .json::<Template>() // 这里需要定义数据类型
        .await?;

    let name:String = resp.name;
    let age:i32 = resp.age;

```

> 方法2 使用serde中自带的Value来适配

```rust
use reqwest::*;
use serde::{Serialize,Deserialize};
use serde_json::{Value, json};

    let resp:Value = client
        .request(reqwest::Method::GET, url)
        .header("x-eneru-apikey", "api-key")
        .send()
        .await?
        .json()
        .await?;

    let name:&str =  resp.get("name").unwrap().as_str().unwrap();
    let name:Value = resp.get("name").unwrap();
```

### serde_json from_str from_value

转换数据


```rust
use serde::{Serialize,Deserialize};
use serde_json::{Value, json};

struct Phone{
    phone:String
}

struct Data{
    name:String,
    age:i32,
    phone:Vec<Phone>
}

let data = r#" { "name": "John Doe", "age": 43, "phones": ["+44 1234567"] } "#;
let v: Value = serde_json::from_str(data).ok().unwrap(); //Object({"age": Number(43), "name": String("John Doe"), "phones": Array([String("+44 1234567")])})
let v:Data = serde_json:;from_str(data).ok().unwrap();
```