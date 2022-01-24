---
title: Rust迭代器
date: 2021-03-10 15:04:51
categories:  [rust]
tags: [rust]
top: 1
---


<!--more-->


# Rust迭代器

实现一个迭代器
```rust
use std::iter::Iterator;
use std::fmt::Debug;

fn use_iter<ITER>(mut iter: ITER)
    where ITER: Iterator,
        ITER::Item: Debug,
{
    while let Some(i) = iter.next(){
        println!("{:?}",i)
    }
}

fn main(){
    let v = vec![1,2,3,4,5];
    use_iter(v.iter());

}

```


```rust
use std::iter::Iterator;

struct Name{
    first: String,
    second: String
}

impl Name {
    fn new() -> Self {
        Name{
            first: "Default".to_owned(),
            second: "default".to_owned(),
        }
    }
}

impl Iterator for Name{
    type Item = String;
    fn next(&mut self) -> Option<Self::Item> {
        let mut value = self.first.to_owned();
        if value != ""{
            return  Some(value+ &self.second);
        }else{
            return None
        }
    }
}

fn main(){
    let mut n1 = Name::new();
    println!("{:?}",n1.next());
}
```

> 从容器中创建迭代器的几种方法

- iter() 创造一个Item是&T类型的迭代器；
- iter_mut() 创造一个Item是&mut T类型的迭代器；
- into_iter()创造一个Item是T类型的迭代器。


```rust
fn main(){
    let mut v = vec![1,2,3,4,5];
    let mut v_iter = v.iter_mut();
    let mut v = vec![1,2,3,4,5];
    let v_iter2 = v.iter();
    let mut v = vec![1,2,3,4,5];
    let v_iter3 = v.into_iter();
    println!("{:?}\n{:?}\n{:?}\n",v_iter,v_iter2,v_iter3);
    //IterMut([1, 2, 3, 4, 5])
// Iter([1, 2, 3, 4, 5])
// IntoIter([1, 2, 3, 4, 5])
}
```


## rust迭代器的扩展
rust中迭代器不仅仅只是上面的这些功能，这里要说一个`迭代器适配器`,它们有个共性，返回的是一个具体类型，而这个类型本身也实现了Iterator trait。这意味着，我们调用这些方法可以从一个迭代器创造出一个新的迭代器

> take(n) 返回一组n个数量的迭代器

```rust
fn main(){
    let v = vec![1,2,3,4,5,6];
    let mut vector = v.iter().take(2);
    println!("{:?}",vector); // Take { iter: Iter([1, 2, 3, 4, 5, 6]), n: 2 }
}

fn main(){
    let v = vec![1,2,3,4,5,6];
    let mut vector = v.iter().take(2);
    println!("{:?}",vector); // Take { iter: Iter([1, 2, 3, 4, 5, 6]), n: 2 }
    
    let v = vec![1,2,3,4,5,6];
    let mut vector = v.iter()
        .take(2) // Take { iter: Iter([1, 2, 3, 4, 5, 6]), n: 2 }
        .filter(|&x| x %2 == 0) // Filter { iter: Take { iter: Iter([1, 2, 3, 4, 5, 6]), n: 2 } }
        .map(|&x|x*x) // Map { iter: Filter { iter: Take { iter: Iter([1, 2, 3, 4, 5, 6]), n: 2 } } }
        .enumerate(); // Enumerate { iter: Enumerate { iter: Map { iter: Filter { iter: Take { iter: Iter([1, 2, 3, 4, 5, 6]), n: 2 } } }, count: 0 }, count: 0 }
    if let Some((i,v)) = vector.next() {
        println!("{}{}",i,v);
    }
}
```


