---
title: 来看看hashmap的官方库
categories:
  - rust
tags:
  - rust
abbrlink: 25d4ec10
date: 2020-08-14 14:58:27
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->


# 来看看hashmap的官方库


## 哈希 map 和所有权
对于像 i32 这样的实现了`Copy trait` 的类型，其值可以拷贝进哈希 map。对于像 String 这样拥有所有权的值，其值将被移动而哈希 map 会成为这些值的所有者。
```rust
use  std::collections::HashMap;

fn main() {
    let s1 = String::from("foo");
    let s2 = String::from("bar");

    let mut  map1 = HashMap::new();
    map1.insert(s1,s2);
    println!("{:?}",map1);
}
```

## 访问hashmap中的值
```rust
use std::collections::HashMap;
fn main(){
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score:Option<&i32> = scores.get(&team_name);
    match score {
        Some(T) => println!("{}",T),
        None => println!("None"),
    }

}
```

## 循环遍历hashmap中的值
```rust
use std::collections::HashMap;
use std::path::Prefix::Verbatim;

fn main() {
    let v1 = vec![1,2,3,4,5];
    let v2 = vec![1,2,3,4,5,6];

    let has1:HashMap<_,_> = v1.iter().zip(v2.iter()).collect();

    println!("{:?}",has1);

    for (i,j) in has1{
        println!("{},{}",i,j);
    }

}
```

## 更新hashmap中的值
```rust
use std::collections::HashMap;

fn main() {
    // let v1 = vec![String::from("1"),String::from("2")];
    // let v2 = vec![String::from("3"),String::from("4")];
    //
    // let mut hash1:HashMap<&String,&String> = v1.iter().zip(v2.iter()).collect();
    // println!("{:?}",hash1); //{"1": "3", "2": "4"}
    // //
    // // // 覆盖更新
    // hash1.insert(&String::from("1").borrow(),&String::from("2").borrow());
    // println!("{:?}",hash1);
    // 上面这个不行

    // 覆盖更新
    let mut hash2 = HashMap::new();
    hash2.insert("1","2");
    hash2.insert("3","4");
    println!("{:?}",hash2);  //{"1": "2", "3": "4"}
    hash2.insert("1","5");
    println!("{:?}",hash2);   //{"1": "5", "3": "4"}

    //在没有对应值得时候插入
    let res = hash2.entry("1");
    println!("{:?}",res); // Entry(OccupiedEntry { key: "1", value: "5" })
    let res = hash2.entry("1").or_insert("2"); // "5"  会返回value
    println!("{:?}",res);
}
```