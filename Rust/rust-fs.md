---
title: rust-fs
date: 2021-04-21 14:21:20
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust-fs

Why am I getting a “bad file descriptor” error when writing file in rust?

rust::fs::Open的时候需要指定权限

https://stackoverflow.com/questions/65862510/why-am-i-getting-a-bad-file-descriptor-error-when-writing-file-in-rust

```rust
    let mut f = OpenOptions::new()
        .append(true) // append can write more data 
        
        .write(true)
        .create(true)
        .truncate(true)
        .open("test.txt")
        .unwrap();
```
o


## read

https://dev.to/dandyvica/different-ways-of-reading-files-in-rust-2n30