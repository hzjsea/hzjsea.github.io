---
title: rust-String
date: 2021-04-16 14:12:21
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust-String

## 关于assert_eq!是否能比较生命周期

答案是不能

```rust
fn main(){
    // 无法对比生命周期的长度
    let xx = String::from("hello");
    {
        let yy = String::from("hello");
        assert_eq!(xx,yy);
    }
}
```


## 关于rust中 as_str 和 &str 的区别

> different between as_str and &str
Not quite, a simple "foo" is a &'static str, 
but if you borrow a &str from a String (e.g. s3.as_str() in your example), 
the &str will get a lifetime that is shorter than the liftetime of the String.

这边有一个很有意思的点，这两个在编译器上看来，返回都是一个`&str`的内容，但是有意思的是一个是`&static`字符串字面量，另一个是引用，且涉及到生命周期的内容
![](https://noback.upyun.com/2021-04-16-14-35-34.png!)
```rust
fn main(){
    let a_str = "foo"; // let a_str:&'static = "foo"
    // a_str's lifetime is &'static

    let b_String = String::from("foo");
    let b_str = b_String.as_str();
    // b_str's lifetime Current scope
}
```

<div style='font-size:20px;color:red'>Impot</div>

注意这里无法使用`String::from("xxx").as_str()`来创建字符串引用,因为`as_str`是要使用调用者本身的引用的，所有调用者本身的生命周期需要长于`as_str()`返回的值的生命周期

`as_str`源码
```rust
    #[inline]
    #[stable(feature = "string_as_str", since = "1.7.0")]
    pub fn as_str(&self) -> &str {
        self
    }
```

```rust
let bb_str = "xx";
let b_str = String::from("xx").as_str();
assert_eq!(bb_str,b_str)
// 会报错，报错的内容是生命周期不够
// temporary value dropped while borrowed
// consider using a `let` binding to create a longer lived value
// 上面的代码类似于
let b_str = {
    let __temp = String::from("xx")
    __temp.as_str()
} // drop __temp

// 解决办法是使用let绑定一个长的生命周期
let b_str = String::from("xx")
let bb_str = b_str.as_str();
```
![](https://noback.upyun.com/2021-04-16-14-41-19.png!)


## rust String join function

### concat,collect and join

<div style='font-size:25px;background:RGB(118,214,173)'>
    chars.iter( ).collect::<String>( )
</div>
将迭代器中的内容拼接在一起

```rust
    // vec Vec<char>
    let chars = vec!['a','b','c'];
    let string  = chars.iter().collect::<String>();
    // arrry [char;_]
    let chars = ['a', 'b', 'c'];
    let string = chars.iter().collect::<String>();
    // slice &[char]
    let chars = vec!['a', 'b', 'c'];
    let slice_chars = &chars[..];
    let string = slice_chars.iter().collect::<String>();
```

<div style='font-size:25px;background:RGB(118,214,173)'>
    strings.concat()
</div>
将数组中的内容拼接在一起

```rust
    // [&str;_]
    let chars = ["a","b","c"];
    let string = chars.concat();
    // Vec<&str>
    let chars = vec!["a", "b", "c"];
    let string = chars.concat();
    // slice
    let chars = ["a", "b", "c"];
    let string = &chars[..].concat();
```


<div style='font-size:25px;background:RGB(118,214,173)'>
    strings.join(separator)
</div>
合并并添加间隔符

```rust
    let names = vec!["firstName","lastName"];
    let string  = names.join(",");
    
    let names = ["firstName", "lastName"];
    let joined = names.join(", ");

    let names = &["firstName","lastName"];
    let string = names.join(",");
```


### bytes, chars
- bytes 转成`Bytes` 迭代器
- chars 转成`Chars` 迭代器
- 两个返回都是copy后的内容

```rust
    pub fn bytes(&self) -> Bytes<'_> {
        Bytes(self.as_bytes().iter().copied())
    }

        pub fn chars(&self) -> Chars<'_> {
        Chars { iter: self.as_bytes().iter() }
    }
fn main(){
    
    
    let s = String::from("helloworld");
    let s_vec:Vec<u8> = s.bytes().collect::<Vec<u8>>();

    let x:&[u8] = s.as_bytes();

    
    let c:String = String::from("helloworld");
    let c_ec:char = c.chars().next().unwrap();
    let c_ec:Vec<char> = c.chars().collect::<Vec<char>>();
}
```



### as_bytes and  from_utf8, from_utf8_lussy, into_bytes , as_bytes_mut

- `as_bytes` 将字符串转换成bytes数组 -> arrary`&[u8]`  拷贝了一份
- `as_bytes_mut` unsafe操作 转换成&mut [u8]
- `into_bytes` 将字符串转换成bytes数组 -> `Vec<u8>` 直接转换了一份
- `from_utf8` 将bytes转换成字符串
- `from_utf8_lussy` 将bytes转换为字符串，可能会丢失

```rust
    pub fn as_bytes(&self) -> &[u8] {
        &self.vec
    }
    pub fn into_bytes(self) -> Vec<u8> {
        self.vec
    }
    pub fn from_utf8(vec: Vec<u8>) -> Result<String, FromUtf8Error> {
        match str::from_utf8(&vec) {
            Ok(..) => Ok(String { vec }),
            Err(e) => Err(FromUtf8Error { bytes: vec, error: e }),
        }
    }
    

fn main(){

    let mut s = String::from("Hello");
    let bytes = unsafe { s.as_bytes_mut() };

    let s:String = String::from("helloworld");
    let bytes:&[u8] = s.as_bytes(); // arrary
    // use from_utf8 , must be vec
    // let vv = bytes.to_vec()
    let s = String::from_utf8(bytes.to_vec()).ok().unwrap();
    
    let s:String =  String::from("helloworld");
    let bytes = s.into_bytes(); // s move
}
```

### as_str and as_mut_str

- `as_str` 将字符串转变为字符串切片
- `as_mut_str` 将字符串转变为可变字符串

```rust
fn main(){
    pub fn as_str(&self) -> &str {
        self
    }
    pub fn as_mut_str(&mut self) -> &mut str {
        self
    }

    let ss = "String".to_owned();
    let s_str = ss.as_str();
    let mut ss = "String".to_owned();
    let mut s_str = ss.as_mut_str();
    //
    let mut ss  = "String".to_owned().as_mut_str(); // error lifetime not discontent
    let ss = "String".to_owned().as_str(); // error lifetime not discontent
}
```

### as_mut_vec

- `as_mut_vec` 将字符串转换成可变的集合

```rust
fn main(){
    let mut s = String::from("hello");
    unsafe {
        let vec = s.as_mut_vec();
        assert_eq!(&[104, 101, 108, 108, 111][..], &vec[..]);
    }
}
```

### len, is_empty, capacity, with_capacity, clear
```rust
fn main(){
    let a = String::from("foo");
    let len = a.len(); // 获取长度
    let is_empty = a.is_empty(); // 是否为空
    let capactity = a.capacity(); // 获取容量
    let aa = String::with_capacity(10); // 初始化设置容量
    
    let mut ss = String::from("foo");  
    ss.clear(); // 清空
    assert_eq!(ss.len(),0);
}
```
### drain pop
- pop 删除最后一个字符
- drain 删除字符串并返回

```rust
    pub fn pop(&mut self) -> Option<char> {
        let ch = self.chars().rev().next()?;
        let newlen = self.len() - ch.len_utf8();
        unsafe {
            self.vec.set_len(newlen);
        }
        Some(ch)
    }

    pub fn drain<R>(&mut self, range: R) -> Drain<'_>
    where
        R: RangeBounds<usize>,
    {
        // Memory safety
        //
        // The String version of Drain does not have the memory safety issues
        // of the vector version. The data is just plain bytes.
        // Because the range removal happens in Drop, if the Drain iterator is leaked,
        // the removal will not happen.
        let Range { start, end } = slice::range(range, ..self.len());
        assert!(self.is_char_boundary(start));
        assert!(self.is_char_boundary(end));

        // Take out two simultaneous borrows. The &mut String won't be accessed
        // until iteration is over, in Drop.
        let self_ptr = self as *mut _;
        // SAFETY: `slice::range` and `is_char_boundary` do the appropriate bounds checks.
        let chars_iter = unsafe { self.get_unchecked(start..end) }.chars();

        Drain { start, end, iter: chars_iter, string: self_ptr }
    }


fn main(){
    let mut ss = String::from("foobar");
    if let Some(value) = ss.pop(){
        assert_eq!('r',value)
    }

    let mut ss = String::from("facbar");
    let x = ss
                    .drain(1..2) // iter  [1,2) // a
                    .collect::<String>();
    assert_eq!(x,"a")
}
```

### insert, insert_str, remove
- insert
- insert_str
- remove

```rust
    let mut ss = String::from("xx");
    ss.insert(0, 'x');
    assert_eq!(ss,"xxx");
    
    let mut ss = String::from("xx");
    ss.insert_str(0,"xx");
    assert_eq!(ss,"xxxx")

    let mut s = String::from("foo");
    assert_eq!(s.remove(0), 'f');
```


### into_boxed_str
将String转换成`Box<str>` ,并去掉`capacity`
```rust


    let mut value = String::with_capacity(200);
    value.insert_str(0, "xx");
    assert_eq!(value, "xx");
    assert_eq!(value.capacity(),200);
    
    let value_2 = value.into_boxed_str(); // no capacity
```

### push,push_str
- push  从尾部添加一个字符
- push_str 从尾部添加一个字符串
```rust
    pub fn push(&mut self, ch: char) {
        match ch.len_utf8() {
            1 => self.vec.push(ch as u8),
            _ => self.vec.extend_from_slice(ch.encode_utf8(&mut [0; 4]).as_bytes()),
        }
    }
    pub fn push_str(&mut self, string: &str) {
        self.vec.extend_from_slice(string.as_bytes())
    }

fn main(){
    let mut ss = String::from("helloworld");
    ss.push('c');
    ss.push_str("cc");
    assert_eq!(ss,"helloworldccc");
}
```

### retain

- retain 保留过滤后的内容

```rust
let mut s = String::from("f_o_ob_ar");
s.retain(|c| c != '_');
assert_eq!(s, "foobar");
```

### replace,replace_range,replacen


- replace,  replace creates a new String, and copies the data from this string slice into it. 从调用者身上拷贝一份数据。并做替换操作
- replace_range 更换range范围内的数据
- replacen 和replace 类似， 无非是这个可以选择替换多次想要替换的内容

```rust
    pub fn replace<'a, P: Pattern<'a>>(&'a self, from: P, to: &str) -> String {
        let mut result = String::new(); // create a new String
        let mut last_end = 0; 
        for (start, part) in self.match_indices(from) {
            result.push_str(unsafe { self.get_unchecked(last_end..start) });
            result.push_str(to);
            last_end = start + part.len();
        }
        result.push_str(unsafe { self.get_unchecked(last_end..self.len()) });
        result
    }
    
    pub fn replace_range<R>(&mut self, range: R, replace_with: &str)
    where
        R: RangeBounds<usize>,
    {
        // Memory safety
        //
        // Replace_range does not have the memory safety issues of a vector Splice.
        // of the vector version. The data is just plain bytes.

        // WARNING: Inlining this variable would be unsound (#81138)
        let start = range.start_bound();
        match start {
            Included(&n) => assert!(self.is_char_boundary(n)),
            Excluded(&n) => assert!(self.is_char_boundary(n + 1)),
            Unbounded => {}
        };
        // WARNING: Inlining this variable would be unsound (#81138)
        let end = range.end_bound();
        match end {
            Included(&n) => assert!(self.is_char_boundary(n + 1)),
            Excluded(&n) => assert!(self.is_char_boundary(n)),
            Unbounded => {}
        };

        // Using `range` again would be unsound (#81138)
        // We assume the bounds reported by `range` remain the same, but
        // an adversarial implementation could change between calls
        unsafe { self.as_mut_vec() }.splice((start, end), replace_with.bytes());
    }

    pub fn replacen<'a, P: Pattern<'a>>(&'a self, pat: P, to: &str, count: usize) -> String {
        // Hope to reduce the times of re-allocation
        let mut result = String::with_capacity(32);
        let mut last_end = 0;
        for (start, part) in self.match_indices(pat).take(count) {
            result.push_str(unsafe { self.get_unchecked(last_end..start) });
            result.push_str(to);
            last_end = start + part.len();
        }
        result.push_str(unsafe { self.get_unchecked(last_end..self.len()) });
        result
    }

fn main(){
    let s = String::from("helloworld");
    let new_s  = s.replace("hello", "world");
    assert_eq!(new_s,"worldworld");

    let mut s = String::from("helloworld");
    s.replace_range(0..5, "world");
    assert_eq!("worldworld",s);

    let s = String::from("worldworldworld");
    let new_s = s.replacen("world", "hello", 3);
    assert_eq!(new_s,"hellohellohello");
}
```

### split, split_off,truncate

- split 切割成一个Split， 通过collect 转换成Vec
- split_off 将字符串切割成两份
- truncate 截断， 取前一部分

```rust
fn main(){
    let mut hello = String::from("Hello, World!");
    let world = hello.split_off(7);
    assert_eq!(hello, "Hello, ");
    assert_eq!(world, "World!");

    let mut s = String::from("hello");
    s.truncate(2);
    assert_eq!("he", s);

    let mut s = String::from("hello");
    s.truncate(2);
    assert_eq!("he", s);
    let new_s = &s[..2];
    assert_eq!("he",new_s);
}
```

### as_ptr

- as_ptr 获取指针

```rust
let s = "Hello";
let ptr = s.as_ptr();
```

### contain,start_with,end_with

```rust
fn main(){
    let bananas = "bananas";

    assert!(bananas.contains("nana"));
    assert!(!bananas.contains("apples"));

    let bananas = "bananas";

    assert!(bananas.starts_with("bana"));
    assert!(!bananas.starts_with("nana"));

    let bananas = "bananas";

    assert!(bananas.ends_with("anas"));
    assert!(!bananas.ends_with("nana"));
}

```

### find
- find find keywards return index 

```rust
fn main(){
    let s = "Löwe 老虎 Léopard Gepardi";
    assert_eq!(s.find('L'), Some(0));
    assert_eq!(s.find('é'), Some(14));
    assert_eq!(s.find("pard"), Some(17));

    let s = "Löwe 老虎 Léopard";
    assert_eq!(s.find(char::is_whitespace), Some(5));
    assert_eq!(s.find(char::is_lowercase), Some(1));
    assert_eq!(s.find(|c: char| c.is_whitespace() || c.is_lowercase()), Some(1));
    assert_eq!(s.find(|c: char| (c < 'o') && (c > 'a')), Some(4));

}

```

https://doc.rust-lang.org/std/string/struct.String.html#method.contains


### get
```rust
fn main(){
    let s = String::from("helloworld");
    let new_s = s.get(1..2).unwrap();
    assert_eq!("e",new_s);

    let mut s = String::from("helloworld");
    let new_s = s.get_mut(0..1).unwrap();
    assert_eq!("h",new_s);
    println!("{:?}",s);
}
```

### is_ascii
```rust
fn main(){
       // check the string is ASCII range
    let ascii = "hello!\n";
    let non_ascii = "Grüße, Jürgen ❤";

    assert!(ascii.is_ascii());
    assert!(!non_ascii.is_ascii()) 
}
```

### lines
```rust
fn main(){
    // lines
    let text = "foo\r\nbar\n\nbaz\n";
    let mut lines = text.lines();
    for line in lines{
        println!("{}",line)
    }
}
```

### match

```rust
fn main(){
    // match
    // 匹配内容 
    let v: Vec<&str> = "abcXXXabcYYYabc".matches("abc").collect();
    assert_eq!(v, ["abc", "abc", "abc"]);

    let v: Vec<&str> = "1abc2abc3".matches(char::is_numeric).collect();
    assert_eq!(v, ["1", "2", "3"]);

    // 匹配后并反转vector
    let v: Vec<&str> = "1abc2abc3".rmatches(char::is_numeric).collect();
    assert_eq!(v, ["3", "2", "1"]);

}
```


### parse
```rust
fn main(){
        // parse
    // Parses this string slice into another type.
    let v = "xx".parse::<i32>();
    let vv = "1".parse::<i32>();
    if let Ok(value) = vv{
        assert_eq!(1,value)
    }
}


```

### split,trim
https://doc.rust-lang.org/std/string/struct.String.html#method.replace