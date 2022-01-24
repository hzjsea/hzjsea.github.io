---
title: vec的官方库
categories:
  - rust
tags:
  - rust
abbrlink: 950c0458
date: 2020-08-13 15:31:41
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->


# 来看看vec的官方库


官方库 https://doc.rust-lang.org/std/vec/index.html
集合的官方库 https://doc.rust-lang.org/std/vec/struct.Vec.html

![2020-08-13-15-34-29](http://noback.upyun.com/2020-08-13-15-34-29.png)


## 基本类型

```rust
fn main(){
    let mut vec = Vec::new();
    vec.push(1);
    vec.push(2);

    assert_eq!(vec.len(), 2);
    assert_eq!(vec[0], 1);

    assert_eq!(vec.pop(), Some(2));
    assert_eq!(vec.len(), 1);

    vec[0] = 7;
    assert_eq!(vec[0], 7);

    vec.extend([1, 2, 3].iter().copied());

    for x in &vec {
        println!("{}", x);
    }
    assert_eq!(vec, [7, 1, 2, 3]);
}
```

## append集合合并
```rust
pub fn append(&mut self, other: &mut Vec<T>)
```
接收一个可变集合的自身引用，以及另外一个可变集合的引用
```rust
fn main(){
    let mut v1=vec![1,2,3,45];
    let mut v2= vec![1,2,3,4,5];
    v1.append(&mut v2);

    println!("{:?}",v1)
}
```

## 拖动集合中的元素drain
```rust
fn main(){
    let mut  v = vec![1,2,3,4];
    let v2: Vec<_>  = v.drain(1..).collect();

    // 大致的意思就是 集合v中的元素拖动到v2中去
    

    assert_eq!(v,[1]);
    // assert_eq!(v,&[1]);
    assert_eq!(v2,[2,3,4]);
    // assert_eq!(v2,&[2,3,4]);

    // 全部拖动
    let mut v3 = vec![1,2,3,4];
    v3.drain(..);

    assert_eq!(v3,[]);
    // assert_eq!(v3,&[]);

}
```

## 清空集合clear

```rust
pub fn clear(&mut self)
```
接收一个可变参数自身

```rust
fn main(){
    let mut  v = vec![1,2,3,4];
    // 判断是否为空集合
    v.drain(..);
    println!("{}",v.is_empty());

    // 清空集合  被清空的集合必须是可变集合
    let mut v3 = vec![1,2,34];
    v3.clear();
    println!("{}",v3.is_empty());
}
```

## 集合长度len

```rust
pub fn len(&self) -> usize
```
获取一个自身参数
```rust
fn main(){
    let v = vec![1,2,3,4];
    // 获取一个集合的长度
    println!("{}",v.len());
}
```

## 集合是否为空is_empty
```rust
pub fn is_empty(&self) -> bool
```
获取一个自身参数
```rust
fn main(){
    let v = vec![1,2,3,4];
    assert_eq!(v.is_empty(),false);
}
```


## 截取集合成为一个新的集合 split_off
```rust
pub fn split_off(&mut self, at: usize) -> Vec<T>
```
获取一个自身可变参数和一个截取序号，返回一个集合

当at > vec.len()的时候就会出现panic报错

```rust
fn main(){
    let mut v = vec![1,2,3,4];
    let _new_v:Vec<_> = v.split_off(2);
    assert_eq!(_new_v,[3,4]);
}
```


## 截断或者补充  resize_with
```rust
pub fn resize_with<F>(&mut self, new_len: usize, f: F)
```
接收一个可变自身，一个新的长度，以及填充物
当new_len > vec.len()的时候，会填充后面的f内容
当new_len < vec.len()的时候，会进行截断形成新的集合

```rust
fn main(){
    let mut v = vec![1,2,3,4];

    // 输出集合长度
    println!("{}",v.len());

    // 截断
    v.resize_with(2,Default::default);
    assert_eq!(v,[1,2]);
    println!("{:?}",v);

    // 填充
    v.resize_with(8, Default::default);
    assert_eq!(v,[1,2,0,0,0,0,0,0]);
    println!("{:?}",v);

}
```

## 截断或者补充 resize
```rust
pub fn resize(&mut self, new_len: usize, value: T)
```
这个和resize_with差不多
```rust
fn main(){
    let mut v = vec![1,2,3,4];

    // 输出集合长度
    println!("{}",v.len());

    // 截断
    v.resize(2,0);
    assert_eq!(v,[1,2]);

    println!("{:?}",v);
    // 填充
    v.resize(8, 0);
    assert_eq!(v,[1,2,0,0,0,0,0,0]);
    println!("{:?}",v);  
}
```

## 对连续的内容去重 dedup
```rust
pub fn dedup(&mut self)
```
调用可变自身
```rust
let mut vec = vec![1, 2, 2, 3, 2];
vec.dedup();
assert_eq!(vec, [1, 2, 3, 2]);
```

<!-- ## 删除存在的元素 remove_item
删除第一个存在的元素
```rust
pub fn remove_item<V>(&mut self, item: &V) -> Option<T>
```
调用可变自身，以及元素的引用， -->



## 集合返回一个迭代
```rust
pub fn iter(&self) -> Iter<T>
```
返回一个迭代器
```rust
let x = &[1, 2, 4];
let mut iterator = x.iter();

assert_eq!(iterator.next(), Some(&1));
assert_eq!(iterator.next(), Some(&2));
assert_eq!(iterator.next(), Some(&4));
assert_eq!(iterator.next(), None);
```

## 将两个集合合并成一个字典
```rust
use  std::collections::HashMap;

fn main() {
    let mut v = vec![1,2,3,4,5,6];
    let mut v2 = vec![1,2,3];
    println!("{:?}",v.iter());
    println!("{:?}",v.iter().zip(v2.iter()));

    let score: HashMap<_,_> = v.iter().zip(v2.iter()).collect();
    println!("{:?}",score);
}

//res :
// Iter([1, 2, 3, 4, 5, 6])
// Zip { a: Iter([1, 2, 3, 4, 5, 6]), b: Iter([1, 2, 3]), index: 0, len: 3 }
// {1: 1, 2: 2, 3: 3}
```

