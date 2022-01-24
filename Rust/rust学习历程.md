---
title: rust学习历程
date: 2021-04-09 18:06:23
categories:  [rust]
tags: [rust]
---


<!--more-->

## rust 学习历程


## Some books
https://stevedonovan.github.io/rust-gentle-intro/readme.html
https://www.bilibili.com/video/BV15N411o7e4/?spm_id_from=333.788.recommend_more_video.1

## 又拍rust练习目录

约瑟夫生死者
30 个人在一条船上，超载，需要 15 人下船。于是人们排成一队，排队的位置即为他们的编号。报数，从 1 开始，数到 9 的人下船。如此循环，直到船上仅剩 15 人为止，问都有哪些编号的人下船了呢？
```rust
    // let v1 = (1..30).map(|value| value).collect::<Vec<u32>>();
    // println!("{:?}",v1);

    // let v1 = (1..30).map(|value| value);
    // let v1 = (1..30).filter(|value| value % 2 == 0);

    // let v1 = (1..30).filter_map(|value | Some(value % 2 == 0) );

    // let mut v1:Vec<u8> = (1..=30).collect();
    // println!("{:?}", v1);
    // while v1.len() > 15 {
    //   println!("{}号下船了。", v1[8]);
    //   let v2 = &v1[..8].to_vec(); // Copies `self` into a new `Vec`.
    // //   println!("v1 address {:#p}",&v1); 0x00007ffee998b070
    // //   println!("v2 address {:#p}",&v2); 0x00007ffee998b120
    //   v1 = v1[9..].to_vec();
    // //   println!("v1 address {:#p}",&v1); 0x00007ffee998b070
    //   v1.extend(v2);
    // }

    // fn main(){
    //     let mut a:Vec<u8> = (1..31).collect();
    //     while a.len() > 15 {
    //         let mut b = a.split_off(8);
    //         println!("{} was down",b.remove(0));
    //         b.extend(a);
    //         a = b;
    //     }
    // }
```


## 水仙花数

> 153 是阿姆斯特朗数. 
370 是阿姆斯特朗数. 
371 是阿姆斯特朗数. 
407 是阿姆斯特朗数. 
1634 是阿姆斯特朗数. 
8208 是阿姆斯特朗数. 
9474 是阿姆斯特朗数. 
54748 是阿姆斯特朗数. 
92727 是阿姆斯特朗数. 
93084 是阿姆斯特朗数.

```rust

let mut s = (100..=100000).collect::<Vec<u64>>();
    s.retain(|v| {
        let (x,x_len) = (v.to_string(),v.to_string.as_bytes().len());
        let c = x.as_bytes().iter().fold::<u64, _>(0, |a,b| {
            a as u64 +(*b as u64 - 0x30).pow(x_len as u32)
        });
        c == *v
    });
    println!("s = {:?}",s);
```


## rust二分排序
```rust
fn main(){
    let mut vv = vec![10,7,8,9,1,5];
    let vv_len = vv.len() as isize - 1;
    let end_index = vv.len() as isize - 1;
    quickSort(&mut vv,0,vv_len as i32);
    println!("{:?}",vv);
}

fn quickSort(vv:&mut Vec<u8>,start_index: i32, end_index: i32){
    if start_index <= end_index {                               // 1
        let pivot = subregion(vv, start_index, end_index);     // 2
        quickSort(vv, start_index, pivot - 1);   // 3
        quickSort(vv, pivot + 1, end_index);   // 4
    }   
}

// [ [h1,h2,h3] middle [e1,e2,e3] ]
// head < middle 
// end > middle

fn subregion(vv: &mut Vec<u8>, start_index: i32, end_index: i32) -> i32{  
    let small_values = vv[start_index as usize];
    let mut i = start_index;
    for cursor in start_index..end_index{
        if vv[cursor as usize] < vv[end_index as usize]{
            vv.swap(i as usize, cursor as usize);
            i +=1;
        }
    }
    vv.swap(i as usize, end_index as usize );
    i
} 
```


## 猴子吃桃

今天的算法题： 猴子吃桃
猴子第一天摘下若干个桃子，当即吃了一半，还不瘾，又多吃了一个；第二天早上又将剩下的桃子吃掉一半，又多吃了一个；以后每天早上都吃了前一天剩下的一半零一个。到第10天早上想再吃时，见只剩下一个桃子了。求第一天共摘了多少

```rust
fn main(){
    let mut total = 1;
    loop {
        let (n,flag) = monkey_eat(total,true);
        if flag && n==1{
            println!("{}",total);
            break;
        } else{
            total +=1
        }
    }
}

fn monkey_eat(total:i32,flag:bool) -> (i32,bool){
    let mut n = total;
    for _i in 0..9{
        if n % 2 == 0 && n > 0{
            n = n - n/2 -1
        } else {
            return (n,false)
        }
    }
    (n,flag)
}
```

逆向写法
```rust
fn main() {
    let mut m: u32 = 1;
    for _ in (0..9).rev() {
        let n: u32 = (m + 1) * 2;
        m = n;
    }
    println!("{}", m);
}
```

## 国王发金币

国王发金币

国王将金币作为工资，发放给忠诚的骑士。第1天，骑士收到一枚金币；之后两天(第2天和第3天)里，每天收到两枚金币；之后三天(第4、5、6天)里，每天收到三枚金币；之后四天(第7、8、9、10天)里，每天收到四枚金币……这种工资发放模式会一直这样延续下去：求第n天，骑士一共获得了多少金币？

```rust
fn main(){
    let mut inputs ="".to_string();
    std::io::stdin().read_line(&mut inputs).unwrap();
    let all_days = inputs.trim().parse::<i32>().unwrap();
    
    let (mut nums, mut day) = (0,0);

    for i in 1..=all_days{
        for j in 0..i{
            nums += i;
            day += 1;
            if day == all_days{
                println!("{}",nums)
            }
        }
    }
}

```


## trait

```rust
trait AppendBar {
    fn append_bar(self) -> Self;

    fn append_bar2(&mut self) -> Self;

    fn append_bar3(self) -> Self;
}
impl AppendBar for String {
    //此处添加代码

    fn append_bar(self) -> Self {
        let mut s = self.as_bytes().to_vec();
        let extend_str = String::from("Bar");
        s.extend_from_slice(
            &extend_str.as_bytes().to_vec()[..]
        );
        String::from_utf8(s).ok().unwrap()
    }
    

    fn append_bar2(&mut self) -> Self {
        // let extend_str = String::from("Bar");
        self.push_str("Bar");
        self.to_string()
    }

    fn append_bar3(self) -> Self {
        let mut s = self;
        s.push_str("Bar") ;
        s.to_string()
    }
}
fn main() {
    let s = String::from("Foo");
    let s = s.append_bar();
    println!("s: {}", s);

    let mut s =  String::from("Foo");
    let s = s.append_bar2();
    println!("s: {}", s);


    let mut s =  String::from("Foo");
    let s = s.append_bar3();
    println!("s: {}", s);
}
#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn is_FooBar() {
        assert_eq!(String::from("Foo").append_bar(), String::from("FooBar"));
    }
    #[test]
    fn is_BarBar() {
        assert_eq!(
            String::from("").append_bar().append_bar(),
            String::from("BarBar")
        );
    }

 
    #[test]
    fn is_FooBar2() {
        assert_eq!(String::from("Foo").append_bar2(), String::from("FooBar"));
    }
    #[test]
    fn is_BarBar2() {
        assert_eq!(
            String::from("").append_bar2().append_bar2(),
            String::from("BarBar")
        );
    }


    #[test]
    fn is_FooBar3() {
        assert_eq!(String::from("Foo").append_bar3(), String::from("FooBar"));
    }
    #[test]
    fn is_BarBar3() {
        assert_eq!(
            String::from("").append_bar3().append_bar3(),
            String::from("BarBar")
        );
    }
}
```