---
title: golang字符串与集合类型
date: 2021-03-11 16:33:19
categories:  [golang]
tags: [golang]
---


<!--more-->




# golang字符串与集合类型

> UTF-8编码规则⼤致如下：
- 当⼀个字符在ASCII码的范围（兼容ASCII码）内时，就⽤1字节
表⽰，因为ASCII码中的字符最多使⽤7个⽐特位，所以前⾯需要补0。
- 当⼀个字符占⽤了n字节时，第⼀字节的前n位设置为1，第n+1
位设置为0，后⾯字节的前两位设置为10。
那道来说，它的码位是 U+9053，相应的⼆
进制表⽰为1001_0000_0101_0011，按上述 UTF-8 编码规则进⾏编
码，则变为字节序列1110_1001_10_000001_10_010011，⽤⼗六进制表
⽰的话，就是0xE90x810x93。
```bash
1110_1001_10_000001_10_010011
三字节组成 1001_0000_01_01_0011 => 1001_0000_0101_0011
```
像这种将Unicode码位转换为字节序列的过程，就叫作编码
（Encode）；反过来，将编码字节序列转变为字符集中码位的过程，
就叫作解码（Decode）。


- 静态存储区。有代表性的是字符串字⾯量，&＇static str类型的
字符串被直接存储到已编译的可执⾏⽂件中，随着程序⼀起加载启
动

- 堆分配。如果&str类型的字符串是通过堆String类型的字符串取
切⽚⽣成的，则存储在堆上。因为String类型的字符串是堆分配的，
&str只不过是其在堆上的切⽚。

- 栈分配。⽐如使⽤`str::from_utf8`⽅法，就可以将栈分配的
[u8；N]数组转换为⼀个&str字符串

与&str类型相对应的是String类型的字符串。&str是⼀个引⽤类
型，⽽String类型的字符串拥有所有权,String是由标准库提供的可变
字符串，可以在创建后为其追加内容或更改其内容。String类型本质为
⼀个成员变量是 Vec＜u8＞类型的结构体，所以它是直接将字符内容
存放于堆中的。

String类型由三部分组成：指向堆中字节序列的指针
（as_ptr⽅法）、记录堆中字节序列的字节长度（len⽅法）和堆分配
的容量（capacity⽅法)

```rust
fn main(){
    let mut Str_xx = String::from("helloworld"); // 0x7fbfb1402e40
    // 获取堆中字节序列的指针地址
    print!("{:p}\n",Str_xx.as_ptr()); // 0x7ffee2cf74f8
    // 获取栈中字节序列的指针地址
    print!("{:p}\n",&Str_xx);
    // 获取堆中字节序列的字节长度
    print!("{}",Str_xx.len()); // 10
    // 获取堆中容量的长度
    print!("{}",Str_xx.capacity()); // 10
    // 添加分配容量
    Str_xx.reserve(20); // 20 + 10 = 30
    assert_eq!(Str_xx.capacity(),30);
}

```

>  use to_owned or to_string
https://medium.com/@ericdreichert/converting-str-to-string-vs-to-owned-with-two-benchmarks-a66fd5a081ce
建议使用to_owned()

```bash
`to_string` 135 just calls `String::from`, and `String::from` 111 just calls `to_owned`. They’re all inlined, so there’s no performance cost to any of them. Format might be more expensive since it goes through all of the formatting abstractions.
```
https://users.rust-lang.org/t/what-is-the-idiomatic-way-to-convert-str-to-string/12160/6




## 字符串操作

- 存在性判断。相关⽅法包括contains、starts_with、ends_with。
- 位置匹配。相关⽅法包括find、rfind。
- 分割字符串。相关⽅法包括split、rsplit、split_terminator、rsplit_terminator、splitn、rsplitn。
- 捕获匹配。相关⽅法包括matches、rmatches、match_indices、rmatch_indices。
- 删除匹配。相关⽅法包括trim_matches、trim_left_matches、trim_right_matches。
- 替代匹配。相关⽅法包括replace、replacen。

```rust
fn main() {
    let mut Str_xx = String::from("helloworld"); // 0x7fbfb1402e40
                                                 // 获取堆中字节序列的指针地址
    print!("{:p}\n", Str_xx.as_ptr()); // 0x7ffee2cf74f8
                                       // 获取栈中字节序列的指针地址
    print!("{:p}\n", &Str_xx);
    // 获取堆中字节序列的字节长度
    print!("{}", Str_xx.len()); // 10
                                // 获取堆中容量的长度
    print!("{}", Str_xx.capacity()); // 10
                                     // 添加分配容量
    Str_xx.reserve(20); // 20 + 10 = 30
    assert_eq!(Str_xx.capacity(), 30);

    // -----------

    let string: String = String::new(); // 创建空字符串 容量为0
    println!("{}", string.capacity());
    assert_eq!("", string);
    let string: String = String::from("hello"); // 创建字符串并赋值
    let string: String = String::with_capacity(20); // 创建容量为20的空字符串
    let str: &'static str = "x  x"; // 创建字符串常量
    let string: String = str 
                    .chars() // 转换成utf8的迭代器
                    .filter(|c| !c.is_whitespace())// 判断每一个元素如果不是空格就返回
                    .collect(); // 迭代器的collect⽅法来⽣成String类型的字符串
    println!("{:?}",string);
    let string: String = str.to_owned(); // 比 to_string 快 to_owned⽅法利⽤&str 切⽚字节序列⽣成新的String字符串
    let string: String = str.to_string(); // 对 String::from()的包装 String::from 去调用 to_owned
    let str :&str = &string[0..string.len()];
    assert_eq!("x  x",str);


    // 返回charts 迭代器
    let string = "helloworld";
    let mut chars = string.chars();
    print!("{:?}",chars); // Chars(['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd'])
    for i in string.chars(){
        println!("{}",i);
    }
    for i in string.bytes(){
        println!("{}",i)
    }
    let string  = String::from("xxxxx");
    println!("{:?}",string.bytes()); // 迭代器
    println!("{:?}",string.into_bytes()); // 数组


    // 获取字符串切片
    let string = "helloworld";
    let string_slice =  string.get(0..4);
    // 判断值为some还是none
    // let flag_bool = string.get(1..4).is_none();
    // let flag_bool = string.get(1..4).is_some();
    // 获取可变
    let mut xx = string.to_owned();
    let xx_mut_slice = xx.get_mut(0..4);
    match xx_mut_slice {
        Some(value) => println!("{}",value), // hell
        None => println!("{}","None"),
    }

    // 根据索引位切割
    let string  = String::from("foobar");
    let (foo, bar) = string.split_at(3); //  会在第三位后分割
    assert_eq!(foo,"foo");
    assert_eq!(bar,"bar");


    let mut string: String = String::from("foobar");
    let (mut foo, mut bar ) = string.split_at_mut(3);
    foo.to_uppercase();
    bar.to_uppercase();
    assert_eq!(foo,"foo");
    assert_eq!(bar,"bar");

    // 检验某index是否为合法边界
    let flag_bool = string.is_char_boundary(20);
    assert_eq!(flag_bool,false);

    // 按字符串切割
    let string  = String::from("foo|bar");
    let mut res = string.split("|");
    println!("{:?}",res.next().expect("None")); // foo
    println!("{:?}",res.next().unwrap());// bars

    // 字符串拼接
    let mut ss = String::from("xx");
    ss.push_str("x"); // xxx
    assert_eq!(ss,"xxx");
    ss.push('x'); // xxxx
    assert_eq!(ss,"xxxx");

    // 通过迭代器添加
    let mut string = String::from("hell");
    string.extend(['o','!'].iter());
    assert_eq!(string,"hello!");
    let string_extend = String::from("rust");
    string.extend(string_extend.chars());
    assert_eq!("hello!rust",string);
    
    // 指定位置插入字符串
    let mut string = String::with_capacity(20);
    string.insert(0, '1');
    string.insert_str(1, "xx");
    print!("{}",string);

    
    /*
        pub fn insert_str(&mut self, idx: usize, string: &str) {
        assert!(self.is_char_boundary(idx));

        unsafe {
            self.insert_bytes(idx, string.as_bytes());
        }
    }
    */
    // 内部调用self.is_char_boundary(index) 看是否超出界限

    // + += 拼接字符串
    let mut left:String =  String::from("left-");
    let right:&str = "right";
    left += "!";
    let left_right = left+right; // move
    println!("{}",left_right);
}
```

## 字符串查找
这个有点复杂，单独独立出来讲
