---
title: Rust泛型
date: 2021-03-10 13:54:15
categories:  [rust]
tags: [rust]
---


<!--more-->


# 泛型


> turobfish

大部分时候当涉及到泛型时，编译器可以自动推断出泛型参数，但是对于需要自定义或者无法推断的参数类型就要使用turobfish来定义变量类型


> rust作用

泛型的存在主要就是为了适配类型，省力。泛型主要适配的是不同的变量类型，标准库里面的`option`就是一个泛型的体现，实现如下
```rust
enum Option<T> {
    Some(T),
    None
}
```
返回一个`Some<T>`和`None`
```rust
fn main(){
    let x:Option<i32> = Some(1);
    let x:Option<bool> = Some(true);
    let x:Option<&str> = Some("x");
    let x:Option<&str> = None;
    match x {
        Some(value) => println!("{:?}",value),
        None => println!("is None"),
    }
}
```
可以适配不同类型的值

> 结构体中的泛型和泛型的默认值

```rust
struct S<T>{
    data: T
}
fn main(){
    let d1:S<i32>  = S{
        data:2,
    };
    let d2:S<&str> = S::<&str>{
        data:"xx"
    };
    assert_eq!(2,d1.data);
    assert_eq!("xx",d2.data);
}
```
设定默认值，可以使用turobfish(::)来改变泛型的变量类型

```rust
// type parameters with a default must be trailing
// 带有默认值的参数必须尾随
struct S<T=i32>{
    data: T
}
fn main(){
    let d1:S<i32>  = S{ // 默认类型
        data:2,
    };
    let d2:S<&str> = S::<&str>{ //turobfish重新定义类型
        data:"xx"
    };
    assert_eq!(2,d1.data);
    assert_eq!("xx",d2.data);
}
```

> 函数中的泛型

在函数中使用泛型，需要先声明泛型
```rust
fn main(){
    let op1 = Some(1);
    let op2 = Some(2);
    let flag = ops(op1,op2);
    println!("{}",flag)
}

fn ops<T>(o1: Option<T>, o2:Option<T>) ->bool {
    let flag = match (o1,o2) {
        (Some(v1),Some(v2)) => true,
        (None,None) => true,
        _ => false,
    };
    flag
}       
```
但是函数中泛型并不支持默认类型，会报错
![](https://noback.upyun.com/2021-03-12-13-43-24.png!)


字符串里面的contains方法里面也有一个trait 里面也实现了泛型
```rust
fn main(){
    let x = "xx";
    println!("{}",x.contains("x"));
}
```
```rust
    pub fn contains<'a, P: Pattern<'a>>(&'a self, pat: P) -> bool {
        pat.is_contained_in(self)
    }

    pub trait Pattern<'a>: Sized {
        /// Associated searcher for this pattern
        type Searcher: Searcher<'a>;

        /// Constructs the associated searcher from
        /// `self` and the `haystack` to search in.
        fn into_searcher(self, haystack: &'a str) -> Self::Searcher;

        /// Checks whether the pattern matches anywhere in the haystack
        #[inline]
        fn is_contained_in(self, haystack: &'a str) -> bool {
            self.into_searcher(haystack).next_match().is_some()
        }

        /// Checks whether the pattern matches at the front of the haystack
        #[inline]
        fn is_prefix_of(self, haystack: &'a str) -> bool {
            matches!(self.into_searcher(haystack).next(), SearchStep::Match(0, _))
        }

        /// Checks whether the pattern matches at the back of the haystack
        #[inline]
        fn is_suffix_of(self, haystack: &'a str) -> bool
        where
            Self::Searcher: ReverseSearcher<'a>,
        {
            matches!(self.into_searcher(haystack).next_back(), SearchStep::Match(_, j) if haystack.len() == j)
        }
    }
```
第二个参数不是某个类型，而是一个泛型接口类型，而且这个类型必须满足`Pattern trait`的约束，因此只要实现了`Pattern trait`就可以被当做参数来使用
但同时也对这个trait实现了一定的约束，也就是实现这个trait的类型的大小在编译期必须知道大小 `Sized`


> impl块中的泛型
当我们希望为某一组类型统一impl某个trait的时候，泛型就非常有用了。有了这个功能，很多时候就没必要单独为每个类型去重复impl了

标准库中的Into和From是一对功能互逆的trait。如果`A：Into<B>`，意味着`B：From<A>`。因此，标准库中写了这样一段代码，意思是针对所有类型T，只要满足`U：From<T>`，那么就针对此类型`impl Into<U>`。有了这样一个impl块之后，我们如果想为自己的两个类型提供互相转换的功能，那么只需impl From这一个trait就可以了，因为反过来的Intotrait标准库已经帮忙实现好了。

```rust
impl<T, U> Into<U> for T where U: From<T>   {
    fn into(self) -> U {
        U::from(self)
    }       
}
```

## 泛型约束与泛型声明
泛型约束有两种方法，
一种是`<T: i32>` 
另外一种是`<T>(s:T) where T: i32`,对于复杂的泛型变量 `where`更适合
代码如下
```rust
fn main(){
    
}

fn max_nums<T>(n1:T,n2:T) -> T{
    if n1 > n2 {n1} else {n2} // error
}
```
这样做就会报错，rust认为没有实现`PartialOrd trait`的类型无法进行`>`
解决的办法是对`T`这个泛型变量类型做约束，保证`T`的这个类型必须是实现`PartialOrd trait`的
```rust
fn main(){
    let n1 = "xx";
    let n2 = "xx";
    max_nums(n1, n2);
}

fn max_nums<T:PartialOrd>(n1:T,n2:T) -> T{
    if n1 > n2 {n1} else {n2}
}

fn max_nums<T>(n1:T,n2:T) -> T  
    where T: PartialOrd{
    if n1 > n2 {n1} else {n2}
}

```

## 关联类型
关联类型是一种在trait内部定义的变量类型

> 使用关联类型的好处
使⽤关联类型能够使代码变得更加精简，同时也对⽅法的输⼊和输出进⾏了很好的隔离，使得代码的可读性⼤⼤增强。在语义层⾯上，使⽤关联类型也增强了trait表⽰⾏为的这种语义，因为它表⽰了和某个⾏为（trait）相关联的类型。在⼯程上，也体现出了⾼内聚的特点。

```rust
trait Add<RHS=Self>  { // 这里RHS=Self的意思就是获取自身变量的数据类型,保证了输入参数和调用参数的类型一致
    type OUTPUT; // 关联类型
    fn my_add(self, rhs:RHS) -> Self::OUTPUT;
}

trait AddExport<RHS>{
    type o;
    fn my_addd(self, rhs:RHS) -> Self::o;
}

trait AddExportS<RHS> {
    type o;
    fn my_adddd(self, rhs:RHS) -> Self::o;
}

impl Add for i32{
    type OUTPUT = i32;
    fn my_add(self, rhs:i32) -> Self::OUTPUT {
        self+rhs
    }
}

impl Add for u32{
    type OUTPUT = i32;
    fn my_add(self, rhs:u32) -> Self::OUTPUT {
        (self + rhs) as i32
    }
}



impl AddExport<i32> for i32{
    type o = i32;
    fn my_addd(self, rhs:i32) -> Self::o {
        self + rhs
    }
}



impl AddExportS<i32> for i32{
    type o = i32;
    fn my_adddd(self, rhs:i32) -> Self::o {
        self+ rhs
    }
}

impl AddExportS<String> for String{
    type o = String;
    fn my_adddd(self, rhs:String) -> Self::o {
        self + &rhs
    }
}

fn main(){
    let a:i32 = 20;
    let b = a.my_add(20);
    println!("{:?}",b);


    let c:i32 = 30;
    let cc = c.my_addd(30);
    println!("{:?}",cc);

    let d: i32 = 30;
    let dd = d.my_adddd(30);
    println!("{:?}",dd);

    let z: String = String::from("hello");
    let zz = z.my_adddd(String::from("world"));
    println!("{:?}",zz);
}
```
这里RHS=Self的意思就是获取自身变量的数据类型,保证了输入参数和调用参数的类型一致,比如做一个加法`1+2=3`是可以的，但是`1+"ss"`是不可以的，前后两个变量的类型不一致，写成方法就是`1.add("ss")`肯定是不可以的，

迭代器中也有用到这个关联类型的特性
```rust
pub trait Iterator {
    #[stable(feature ="rust1", since = "1.0.0")]
    type Item; //   关联类型

    /// Advances the iterator and returns the next value.
    ///
    /// Returns [`None`] when iteration is finished. Individual iterator
    /// implementations may choose to resume iteration, and so calling `next()`
    /// again may or may not eventually start returning [`Some(Item)`] again at some
    /// point.
    ///
    /// [`None`]: ../../std/option/enum.Option.html#variant.None
    /// [`Some(Item)`]: ../../std/option/enum.Option.html#variant.Some
    ///
    /// # Examples
    ///
    /// Basic usage:
    ///
    /// ```
    /// let a = [1, 2, 3];
    ///
    /// let mut iter = a.iter();
    ///
    /// // A call to next() returns the next value...
    /// assert_eq!(Some(&1), iter.next());
    /// assert_eq!(Some(&2), iter.next());
    /// assert_eq!(Some(&3), iter.next());
    ///
    /// // ... and then None once it's over.
    /// assert_eq!(None, iter.next());
    ///
    /// // More calls may or may not return `None`. Here, they always will.
    /// assert_eq!(None, iter.next());
    /// assert_eq!(None, iter.next());
    /// ```
    #[stable(feature = "rust1", since = "1.0.0")]
    fn next(&mut self) -> Option<Self::Item>;
    ...
    fn count(self) -> usize
    where
        Self: Sized,
    {
        #[inline]
        fn add1<T>(count: usize, _: T) -> usize {
            // Might overflow.
            Add::add(count, 1)
        }

        self.fold(0, add1)
    }

    ...
    ...
}
```
来看他的一个实现impl
```rust
#[stable(feature = "rust1", since = "1.0.0")]
impl<'a> Iterator for Chars<'a> {
    type Item = char;

    #[inline]
    fn next(&mut self) -> Option<char> {
        next_code_point(&mut self.iter).map(|ch| {
            // SAFETY: `str` invariant says `ch` is a valid Unicode Scalar Value.
            unsafe { char::from_u32_unchecked(ch) }
        })
    }

    #[inline]
    fn count(self) -> usize {
        // length in `char` is equal to the number of non-continuation bytes
        let bytes_len = self.iter.len();
        let mut cont_bytes = 0;
        for &byte in self.iter {
            cont_bytes += utf8_is_cont_byte(byte) as usize;
        }
        bytes_len - cont_bytes
    }

    #[inline]
    fn size_hint(&self) -> (usize, Option<usize>) {
        let len = self.iter.len();
        // `(len + 3)` can't overflow, because we know that the `slice::Iter`
        // belongs to a slice in memory which has a maximum length of
        // `isize::MAX` (that's well below `usize::MAX`).
        ((len + 3) / 4, Some(len))
    }

    #[inline]
    fn last(mut self) -> Option<char> {
        // No need to go through the entire string.
        self.next_back()
    }
}
```
我们调用`&str.charts`时候会生成一个u8的迭代器也就是Chars这个结构体
```rust
fn main(){
    let xx = "xx";
    let value = xx.chars().next();
    let count = xx.chars().count();
}

pub fn chars(&self) -> Chars<'_> {
    Chars { iter: self.as_bytes().iter() }
}

pub struct Chars<'a> {
    iter: slice::Iter<'a, u8>,
}

// 根据上面Iterator对Chars实现的方法，就可以调用count()以及next()
```
`next`里面返回了一个`Option<Self::Item>`的值,具体去迭代的方法不去研究，这样实现完了以后，只要针对不同类型的迭代器去定义不同类型的内容就可以实现泛型了
如上对于`Chars`类型的`Item`为`char`

对于u8的内容就如下实现
```rust
impl Iterator for Bytes<'_> {
    type Item = u8;
    ...
}
```
动手实现如下,
```rust
use std::iter::Iterator;
use std::fmt::Debug;

fn use_iter<ITEM,ITER>(mut iter: ITER)
    where ITER: Iterator<Item=ITEM>,
        ITEM: Debug,
{
    while let Some(i) = iter.next(){
        println!("{:?}",i)
    }
}

fn main(){
    let v = vec![1,2,3,4,5];
    use_iter(v.iter())
}
```
`use_iter`要求传入一个迭代器，需要满足迭代器中的所有选项都满足Debug trait

优化
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
    use_iter(v.iter())
}
```


## 泛型特化
pass



## 类型大小
类型的大小在 Rust 中很重要，Sized trait 是 std::marker 模块中的四大特殊 trait 之一。本文主要介绍 DST 和 ZST
DST 是 Dynamic Sized Type 的缩写，意思是动态大小类型，表示在编译阶段无法确定大小的类型。在讲这种类型之前，我们先从数组开始谈起。

https://l1nxy.me/2020/08/24/rust-sized-type/
https://laplacedemon.gitbooks.io/-rust/content/sized4e0e3f-sized.html
http://huonw.github.io/blog/2015/01/the-sized-trait/


对于动态类型，在编译期内才能确定大小，所以要加一个`?Sized`
```rust
use std::fmt::Debug;

fn dbg<T>(t: &T) where T: Debug { // T: Debug + Sized
    println!("{:?}", t);
}

fn main() {
    dbg("my str"); // &T = &str, T = str, str: Debug + !Sized ❌
}
```
`"my str" = &str,  &str = T  &T = str str: Debug + !Sized`
```rust
use std::fmt::Debug;

fn dbg<T>(t:T) where T: Debug { // T: Debug + Sized
    println!("{:?}", t);
}

fn main() {
    dbg("my str"); 
}
```
`"my str" = &str, T=&str &str :Debug`,正常运行

```rust
use std::fmt::Debug;

fn dbg<T>(t:&T) where T: Debug+?Sized { // T: Debug + Sized
    println!("{:?}", t);
}

fn main() {
    dbg("my str"); 
}
```
`"my str" = &str,  &str = T  &T = str str: Debug + ?Sized` 正常运行
 
