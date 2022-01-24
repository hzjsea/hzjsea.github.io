---
title: rust-hashmap
date: 2021-04-19 14:08:23
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust-hashmap

```rust
use std::collections::HashMap;

fn main() {
    //1,new 创建一个空的 HashMap
    let mut map = HashMap::new(); //默认长度是0
    map.insert("blue", 10);
    let yellow = "yellow";
    let mut num = 50;
    map.insert(yellow, num);
    println!(
        " map : {:?} ,len : {} ,capacity : {}",
        map,
        map.len(),
        map.capacity()
    );

    //2,声明一个有初始长度的HashMap
    let mut map2: HashMap<i32, i32> = HashMap::with_capacity(10);
    map2.insert(num, num);
    println!(
        " map2 : {:?} ,len : {} ,capacity : {}",
        map2,
        map2.len(),
        map2.capacity()
    );

    //3,用队伍列表和分数列表创建哈希 map
    let keys = vec![String::from("blue"), String::from("yellow")];
    let values = vec![10, 50];
    let map3: HashMap<_, _> = keys.iter().zip(values.iter()).collect();
    println!(
        " map3 : {:?} ,len : {} ,capacity : {}",
        map3,
        map3.len(),
        map3.capacity()
    );

    //HashMap的所有权
    //对于像 i32 这样的实现了 Copy trait 的类型，其值可以拷贝进哈希 map。
    // 对于像 String 这样拥有所有权的值，其值将被移动而哈希 map 会成为这些值的所有者
    //println!("{}" ,yellow)//value borrowed here after move
    println!(
        " num : {} , keys :  {:?} , values : {:?}  ",
        num, keys, values
    ); //显然是可以的
    num = 1;
    println!(" num {}", num);

    //访问哈希 map 中的值
    for (key, value) in &map {
        println!(" 循环输出 map {}: {}", key, value);
    }

    if let Some(x) = map.get(&"yellow") {
        println!(" get {}", *x)
    }

    if let Some(x) = map.get_mut(&"yellow") {
        *x += 1;
        println!(" get_mut {}", *x)
    }

    //修改key:"yellow"的value
    println!(" 修改 yellow的value  {:?}", map);
    if let Some(x) = map.get_mut(&"yellow") {
        *x = 21;
    }
    println!(" 修改 yellow的value 21  {:?}", map);

    //修改
    map.insert("yellow", 22);
    println!(" 修改 yellow的value 22  {:?}", map);

    //只在键没有对应值时插入
    map.entry("yellow").or_insert(60);
    map.entry("green").or_insert(50);
    println!(" 添加 yellow的value 60  {:?}", map);
    println!(" 添加 green 的value 50  {:?}", map);

    //删除
    map.remove("yellow");
    println!(" 删除 yellow 的value   {:?}", map);

    //统计字符出现的次数
    let mut map3 = HashMap::new();
    for ch in "a short treatise on fungi".chars() {
        let counter = map3.entry(ch).or_insert(0);
        *counter += 1;
    }
    println!(" 统计字符出现的次数 : {:?}", map3);
}
```