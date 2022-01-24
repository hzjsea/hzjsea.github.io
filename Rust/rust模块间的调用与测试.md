---
title: rust模块间的调用与测试
date: 2021-04-19 14:14:23
categories:  [rust]
tags: [rust]
---


<!--more-->

# rust模块间的调用与测试


https://lelouchhe.github.io/rust_3_2

一般测试都是写偏底层的库时候需要用到的,在创建一个库的时候，我们一般会这样创建
```rust
cargo new adder --lib
```

## 函数测试
对于一些简单的函数测试类场景，如果我们需要写测试，可以直接在当前文件下面编写一个test 就可以做一个test来看看

比如像这样,运行方法为`cargo test`;
```rust
struct Wrapper<T> {
    value: T,
}
impl<T> Wrapper<T> {
    pub fn new(value: T) -> Self {
        Wrapper { value }
    }
}
#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn store_u32_in_wrapper() {
        assert_eq!(Wrapper::new(42).value, 42);
    }
    #[test]
    fn store_str_in_wrapper() {
        assert_eq!(Wrapper::new("Foo").value, "Foo");
    }
}
```

> 将Result<T,E> 用于测试

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

## 集成测试
在 Rust 中，集成测试对于你需要测试的库来说完全是外部的。同其他使用库的代码一样使用库文件，也就是说它们只能调用一部分库中的公有 API 。集成测试的目的是测试库的多个部分能否一起正常工作。一些单独能正确运行的代码单元集成在一起也可能会出现问题，所以集成测试的覆盖率也是很重要的。为了创建集成测试，你需要先创建一个 tests 目录

```rust
🏃:rust-sdk/ (master✗) $ tree . -L 2
.
├── Cargo.lock
├── Cargo.toml
├── README.md
├── src
│   ├── auth.rs
│   ├── lib.rs
│   ├── upyun.rs
│   └── utils.rs
└── tests
    ├── auth_test.rs
    ├── upyun_test.rs
    └── utils_test.rs
```
如上src是我们写好的库，tests中是我们测试的库，他们之间的调用需要做一些动作
1. 需要在pub开放权限
2. 在lib中统一调出
3. 在tests中调用

```rust

// src/auth.rs
impl UpYun {
    pub fn make_rest_auth(&self,config:&RESTAuthConfig) -> String{ // 开放权限
        let sign = vec![
            &config.method,
            &config.uri,
            &config.date_str,
            &config.length_str,
            &self.uyc.password
        ]
        .as_slice()
        .iter()
        .map(|value|value.to_string())
        .collect::<Vec<String>>();

        format!(
            "UpYun{}:{}",
            &self.uyc.operator,
            utils::md5_str(
                &sign.join("&")
            )
        )
    }
}

// src/lib.rs
pub mod utils;
pub mod auth;
pub mod upyun;

// tests/auth_test.rs 
use rust_sdk::utils::{ // 引用并调用
    make_user_agent,
    md5_str,
    base64_to_str,
    hmac_sha1,
    is_hex,
    un_hex,
};


#[test]
fn make_user_agent_test(){
    assert_eq!(
        "UPYUN Go SDK V2/v1.0",
        make_user_agent("v1.0")
    )
}
```


## 如何在测试的时候输出内容

```bash
# 查看test选项
cargo test -- --help


# 显示输出
cargo test -- --show-output
cargo test -- --nocapture
```

## 链接
https://kaisery.github.io/trpl-zh-cn/ch11-03-test-organization.html
http://llever.com/cli-wg-zh/tutorial/testing.zh.html
https://rustwiki.org/zh-CN/rust-by-example/testing/integration_testing.html
example use mod
https://github.com/rust-crdt/rust-crdt