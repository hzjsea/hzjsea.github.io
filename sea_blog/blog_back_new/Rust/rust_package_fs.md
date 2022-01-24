---
title: rust_package_fs
date: 2021-03-19 16:30:35
categories:  [rust]
tags: [rust]
---


<!--more-->

# rust_package_fs

## 源码

本来以为file的源码会是一个结构体，定义了一些必要的字段，类似于go一样的内容，但事实上并不是这样，其实定义的各个操作系统的c接口，编译的时候会调度对应系统的fs代码

![](https://noback.upyun.com/2021-06-18-10-50-20.png!)
![](https://noback.upyun.com/2021-06-18-10-51-00.png!)

所以我们这里就不纠结这个file的结构体了，就直接来学习他的功能和使用方法就行了
![](https://noback.upyun.com/2021-06-18-10-51-33.png!)

<div style='font-size:20px;color:red'>另外插嘴说一句，rust的源码注释，真的比golang的好太多太多了</div>


## 目录
```rust
/// A builder used to create directories in various manners.
///
/// This builder also supports platform-specific options.
#[stable(feature = "dir_builder", since = "1.6.0")]
#[derive(Debug)]
pub struct DirBuilder {
    inner: fs_imp::DirBuilder,
    recursive: bool,
}

/// Entries returned by the [`ReadDir`] iterator.
///
/// An instance of `DirEntry` represents an entry inside of a directory on the
/// filesystem. Each entry can be inspected via methods to learn about the full
/// path or possibly other metadata through per-platform extension traits.
#[stable(feature = "rust1", since = "1.0.0")]
pub struct DirEntry(fs_imp::DirEntry);
```



> 创建文件


```rust
fn main(){
    // 获取当前文件夹权限
    let dir_builder = std::fs::DirBuilder::new();
    // DirBuilder { inner: DirBuilder { mode: 511 }, recursive: false }
    println!("{:?}", dir_builder);

    // 获取当前文件夹权限
    let mut builder = std::fs::DirBuilder::new();
    let recursive_builder = builder.recursive(true);
    // DirBuilder { inner: DirBuilder { mode: 511 }, recursive: true }
    println!("{:?}",recursive_builder);

    // 创建迭代文件
    let path = "./helloworld";
    std::fs::DirBuilder::new().recursive(true).create(path).unwrap()
}
```


> 获取迭代目录

<!-- 
## 构造内容

> 创建一个新的文件，并写入数据

```rust
use std::fs::File;
use std::io::prelude::*;
///
fn main() -> std::io::Result<()> {
    let mut file = File::create("foo.txt")?;
    file.write_all(b"Hello, world!")?;
    Ok(())
}
```

> 读取一个文件中的内容

```rust
use std::fs::File;
use std::io::prelude::*;
///
fn main() -> std::io::Result<()> {
    let mut file = File::open("foo.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    assert_eq!(contents, "Hello, world!");
    Ok(())
}
```

> 先读取到缓存到中，再慢慢吐出来

```rust
use std::fs::File;
use std::io::BufReader;
use std::io::prelude::*;
///
fn main() -> std::io::Result<()> {
    let file = File::open("foo.txt")?;
    let mut buf_reader = BufReader::new(file);
    let mut contents = String::new();
    buf_reader.read_to_string(&mut contents)?;
    assert_eq!(contents, "Hello, world!");
    Ok(())
}
```


```rust
fn main() -> std::io::Result<()>{

    // writer info to file
    std::fs::write("xx.txt", "xxx")?;
    std::fs::write("yy.txt", "yyy")?;


    // copy file_a  content to file_b content
    std::fs::copy("yy.txt", "xx.txt")?;

    Ok(())
}
```
```rust
use std::io::{Read, Write};

fn main() -> std::io::Result<()> {

    //  std::fs::DirBuilder
    //  创建文件或者文件夹
    let file_builder = std::fs::DirBuilder::new()
                                .recursive(true) // 是否递归
                                .create("xx/yy/zz/")
                                .unwrap();
    
    //  std::fs::DirEntry
    //  
    for entry in std::fs::read_dir(".")?{
        let dir = entry?;
        println!("{:?}",dir.path())
    }

    // std::fs::File
    let mut file  = std::fs::File::create("./y.txt")?;
    file.write_all(b"buf")?;
    

    let mut file = std::fs::File::open("y.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    assert_eq!(contents, "buf");


    // std::fs::openOption
    use std::fs::OpenOptions;

    let file = OpenOptions::new()
            .read(true)
            .write(true)
            .create(true)
            .open("foo.txt");

    // std::fs::FileType
    let  file = std::fs::File::open("y.txt")?;
    let flag = file.metadata()?.is_file();
    assert_eq!(flag,true);

    // std::fs::metadata
    let file_type = std::fs::File::open("y.txt")?.metadata()?;
    println!("{:?}",file_type);

    // normal
    
    // 获取文件或者文件夹信息
    let file_metadata = std::fs::metadata("xx/yy/zz").unwrap();
    println!("{:?}",file_metadata.is_dir());

    // 创建文件夹
    // std::fs::create_dir("xx")?;
    // 递归创建文件夹
    // std::fs::create_dir_all("xx/xx/xx")?;
    // println!("Hello, world!");
    Ok(())
}

``` -->