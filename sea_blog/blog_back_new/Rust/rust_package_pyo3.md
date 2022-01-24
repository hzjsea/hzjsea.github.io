---
title: rust_package_pyo3
date: 2021-07-27 10:16:28
categories:  [rust]
tags: [rust,pyo3]
---


<!--more-->


# rust_package_pyo3

# openssl中的加解密

## openssl
openssl本身是一种加密库，由三大功能组合而成,这里主要介绍加解密
- 加解密
- SSL协议
- 证书操作


### 加解密
加解密的形式通常分为以下几种
- 对称加密算法
- 非对称加密算法
- 不可逆加密算法


> 对称算法: 信息的发送方和接收方使用同一份秘钥去加解密数据

优点是: 加解密的速度快，适合对于大数据量加密，但是秘钥管理困难因为只有一个秘钥，只要暴露了，就很容易破解加密后的信息。AES，DES等都是常用的对称加密算法

![](https://noback.upyun.com/2021-08-04-14-38-46.png!)

> 非对称算法: 信息的发送方和接收方分别持有一份秘钥，一份公开发布， 成为公钥; 一份私有， 称为秘钥, 从秘钥中可以导出对应的公钥

`一般情况下`，信息发送者用公开密钥去加密，而信息接收者则用私用密钥去解密。公钥机制灵活，但加密和解密速度却比对称密钥加密慢得多。`不同的使用场景下，也会衍生出其他的使用方法, 比如私钥加密，公钥解密`。RSA，DSA等是常用的非对称加密算法；

![](https://noback.upyun.com/2021-08-04-14-38-57.png!)

> 不可逆加密算法主要用于校验文件的一致性 摘要算法就是其中一种

摘要算法，用来把任何长度的明文以一定规则变成固定长度的一串字符。那么我们在对文件做签名的时候，通常都是先使用摘要算法，获得固定长度的一串字符，然后对这串字符进行签名，在做文件一致性校验的时候，接收者接收到文件后，同发送者一样，执行一遍摘要算法后再签名，前后数据一致，则表示文件在传输过程没有被串改。常用的摘要算法有MD5

![](https://noback.upyun.com/2021-08-04-14-39-19.png!)

> base64 不是加密算法，它是编码方式，方便传输过程ASCII码和二进制码之间的转换

类似于图片或者一些文本协议，在传输过程中通常可以使用base64转换成二进制码进程传输
http://blog.xiayf.cn/2016/01/24/base64-encoding/



### RSA加解密算法

RSA是一个流行的非对称加密算法, 生成公私钥内容如下
```bash
# 生成秘钥
openssl genrsa -out test.key 1024
# 从秘钥中导出公钥
openssl rsa -in test.key -pubout -out test_pub.key
# 公钥加密文件
echo "test" > hello
openssl rsautl -encrypt -in hello -inkey test_pub.key -pubin -out hello.en
# 私钥解密文件
openssl rsautl -decrypt -in hello.en -inkey test.key -out hello.de
```

https://www.jianshu.com/p/15b1d935a44b
http://abcdxyzk.github.io/blog/2018/02/09/tools-openssl/



### ssh加密流程
![](https://noback.upyun.com/2021-08-04-15-54-13.png!)

- 客户端发送自己的密钥ID给服务器端
- 服务器在自己的authorized_keys文件中查找是否存在此ID的公钥
- 如果有，则服务器生成一个随机数，用当前ID的公钥加密
- 服务器将加密后的随机数发给客户端
- 客户端用私钥解密该随机数，然后在本地为随机数做MD5加密
- 客户端将该MD5哈希发给服务器端
- 服务器端为一开始自己生成的随机数也做一个MD5哈希，然后用通讯通道“公共的密钥”将该哈希加密，再跟客户端发来的内容进行对比。如果双方内容一致，则通过验证，开放访问权限给客户端

https://blog.csdn.net/zstack_org/article/details/53100545



# 使用rust来为python编写模块

## 了解pyo3

pyo3主要用于创建原生Python的扩展模块,还支持从 Rust 二进制文件运行 Python 代码并与之交互
github地址: https://github.com/PyO3/pyo3
版本规定
- Python 3.6+ 
- Rust 1.41+

由一个小的demo了解一下从pyo3编译模块到python中正常使用的整个流程
```bash
cargo new --lib string-sum
```

> 创建项目

```toml
# lib.rs
[package]
name = "string-sum"
version = "0.1.0"
edition = "2018"

[lib]
name = "string_sum"
# "cdylib" is necessary to produce a shared library for Python to import from.
#
# Downstream Rust code (including code in `bin/`, `examples/`, and `tests/`) will not be able
# to `use string_sum;` unless the "rlib" or "lib" crate type is also included, e.g.:
# crate-type = ["cdylib", "rlib"]
crate-type = ["cdylib"]

[dependencies.pyo3]
version = "0.14.1"
features = ["extension-module"] // 扩展模块，像其他的还有auto-initialize
```

```rust
// src/lib.rs
use std::usize;

use  pyo3::prelude::*;

// like this
// def sum_as_string(a:str, b:str) -> str:
//      return a+b
#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String>{
    Ok((a+b).to_string())
}

// Mount method to module 
#[pymodule]
fn string_sum(py: Python, m: &PyModule) -> PyResult<()>{
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    Ok(())
}
```

> 编译与使用

```bash
pip3 install maturin
maturin build
```
编译完成之后，你会在target文件夹下面发现一个wheel文件，里面有(模块名+当前python版本+当前系统型号)组合命名的文件, 比如: `string_sum-0.1.0-cp39-cp39-macosx_10_7_x86_64.whl`
```bash
pip3 install ./target/wheel/string_sum-0.1.0-cp39-cp39-macosx_10_7_x86_64.whl
```

创建python文件
```python
# example.py
from string_sum import sum_as_string
print(sum_as_string(1,2))
# echo 3
```

## 实践----编译工具的选择
官方提供了两种编译工具的选择，一种是rust写的maturin , 另外一种是传统的`setup.py`的方式


> 使用maturin 编译

```bash
# 安装 
pip3 install maturin
# 编译
maturin build
# maturin publish 发布
# 虚拟环境中使用 会自动去寻找/target/wheel/ 下的 *.wheel文件然后安装
virtualenv venv
source ./venv/bin/activate
maturin develop
```

> 使用 setup.py 编译

```bash
# 安装
pip3 install setuptools-rust

```

编写setup.py文件
```bash
# setup.py


from setuptools import setup
from setuptools_rust import Binding, RustExtension

setup(
    # 包名称
    name="string_sum", 
    # 包版本 
    version="0.1",
    # rust扩展 其中"string_sum.string_sum"中
    # 第一个string_sum 指的是当前的包
    # 第二个指的是
    # #[pymodule]
    # fn string_sum(py: Python, m: &PyModule) -> PyResult<()>{
    #     m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    #     Ok(())
    # }
    # 中的string_sum
    rust_extensions=[
        RustExtension(
            "string_sum.string_sum", 
            binding=Binding.PyO3,
            debug=False
            )
    ],
    # 需要创建一个文件夹 string_sum
    packages=["string_sum"],
    # rust extensions are not zip safe, just like C-extensions.
    zip_safe=False,
    # 标注
    classifiers=[
        "License :: OSI Approved :: MIT License",
        "Development Status :: 3 - Alpha",
        "Intended Audience :: Developers",
        "Programming Language :: Python",
        "Programming Language :: Rust",
        "Operating System :: POSIX",
        "Operating System :: MacOS :: MacOS X",
    ],
    include_package_data=True
)

```

```bash
# 打包
mkdir string_sum
touch string_sum/__init__.py
virtualenv venv && source venv/bin/activate
pip setup.py build && pip setup.py install && pip setup.py develop
```
![](https://noback.upyun.com/2021-07-27-14-03-13.png!)
会引用本地的文件
![](https://noback.upyun.com/2021-07-27-14-05-40.png!)

### docker中的应用

如果模块需要在docker中使用的话，需要安装rust的环境，dockerfile具体如下

```bash
#!/bin/bash
curl https://sh.rustup.rs -sSf | bash -s -- -y
source $HOME/.cargo/env
rustc --version
python setup.py install
```

```docker
# ddockerfile 
FROM python:3.7
WORKDIR /app
ADD . /app
RUN pip install --upgrade pip \
    && pip install -r requirements.txt
RUN ./init.sh
CMD [python, xx.py]
```

```bash
# requirements.txt
semantic-version==2.8.5
setuptools-rust==0.12.1
toml==0.10.2
```

```bash
# rust国内镜像源 config
# /root/.cargo/config
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
[term]
verbose = true
color = 'auto'
```

具体目录如下
```bash
-rw-r--r-- Cargo.lock
-rw-r--r-- Cargo.toml
-rw-r--r-- config           # 配置文件
-rw-r--r-- Dockerfile
-rwxrwxrwx init.sh          # 初始化rust环境脚本
-rw-r--r-- requirements.txt
-rw-r--r-- setup.py         # 打包脚本
drwxr-xr-x src              # rust项目
drwxr-xr-x string_sum 
-rw-r--r-- xx.py            # 可行性测试文件
```

## 用pyo3写一个python的rsa加解密包
看过之前的文章的小伙伴，一定了解了rsa的整个加解密流程，一般情况下，公钥加密，私钥解密，但在一些其他的场景中，如果不想尝试复杂的加解密流程，可以使用 私钥加密，公钥解密。
> 具体场景如下， 客户端本地生成公私钥， 通过前期认证过程，将公钥发送给服务端保存， 后期通信过程中,客户端主动发送消息给服务端，客户端通过私钥对信息加密， 服务端通过对应的公钥进行解密。

需求已经定下来了，但是发现python竟然没有对应的库 qaq , python 里面只有公钥加密，私钥解密的库。 于是看了一些其他语言的案例，发现在go rust中都有对应的库和方法，于是想着用rust来编译一个加解密模块来提供给python使用

github地址: https://github.com/hzjsea/pyo3-crypto


```bash
# .cargo国内镜像源
# config
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
[term]
verbose = true
color = 'auto'
```

```bash
# 自动化脚本
#!/bin/bash

echo "init......"

# set python version 
# INSTALL_PYTHON_VERSION=python3.6

find_python() {
        set +e
        unset BEST_VERSION
        for V in 37 3.7 38 3.8 39 3.9 3; do
                if which python$V >/dev/null; then
                        if [ "$BEST_VERSION" = "" ]; then
                                BEST_VERSION=$V
                        fi
                fi
        done
        echo $BEST_VERSION
        set -e
}

if [ "$INSTALL_PYTHON_VERSION" = "" ]; then
        INSTALL_PYTHON_VERSION=$(find_python)
fi

# This fancy syntax sets INSTALL_PYTHON_PATH to "python3.7", unless
# INSTALL_PYTHON_VERSION is defined.
# If INSTALL_PYTHON_VERSION equals 3.8, then INSTALL_PYTHON_PATH becomes python3.8
# 找不到就python3.7
INSTALL_PYTHON_PATH=python${INSTALL_PYTHON_VERSION:-3.7}
echo $INSTALL_PYTHON_PATH

echo "Python version is $INSTALL_PYTHON_VERSION"
$INSTALL_PYTHON_PATH -m venv venv
if [ ! -f "activate" ]; then
        ln -s venv/bin/activate .
fi

. ./activate

python -m pip install --upgrade pip
python -m pip install wheel
python -m pip install -r ./requirements.txt
maturin build
maturin develop

echo "PASS:     "
current_shell=$(echo $SHELL)
if current_shell=/bin/bash; then
	echo "source /venv/bin/activate >> ~/.bashrc"
elif current_shell=/bin/zsh;then
	echo "source /venv/bin/activate >> ~/.zshrc"
fi
```


```rust
//  src/lib.rs 文件
use std::u32;
use pyo3::prelude::*;
use openssl::rsa::{Padding,Rsa};

const SECRET: &'static str = "CHFfxQA3tqEZgKusgwZjmI5lFsoZxXGXnQLA97oYga2M33sLwREZyy1mWCM8GIIA";

mod crypto_utils {
    use hmac::{Hmac, Mac, NewMac};
    use sha2::Sha256;
    use std::fmt::Write;

    type Hmacsha256 = Hmac<Sha256>;
    fn encode_hex(bytes: &[u8]) -> String {
        let mut s = String::with_capacity(bytes.len() * 2);
        for &b in bytes {
            match write!(&mut s, "{:02x}", b) {
                Ok(_) => {},
                Err(_) => {}
            };
        }
        s
    }
    
    pub fn hash_hmac(secret: &str, msg: &str) -> String {
        let mut mac = Hmacsha256::new_from_slice(secret.as_bytes()).expect("HMAC can take key of any size");
        mac.update(msg.as_bytes());
        let result = mac.finalize();
        let code_bytes = result.into_bytes();
        encode_hex(&code_bytes)
    }

}

// create public/private key  create_key(1024)
fn create_key(len:u32) -> (String,String){
    let rsa = openssl::rsa::Rsa::generate(len).unwrap();
    let pubkey = String::from_utf8(rsa.public_key_to_pem().unwrap()).unwrap();
    let prikey  = String::from_utf8(rsa.private_key_to_pem().unwrap()).unwrap();

    (pubkey, prikey)
}



#[pyclass]
struct Crypto {
    // #[pyo3(get, set)]
    // pubkey: String,
    // #[pyo3(get,set)]
    // prikey: String,
    pub_key: Rsa<openssl::pkey::Public>,
    pri_key: Rsa<openssl::pkey::Private>
}

#[pyfunction]
fn generate_key(len:u32) -> (String, String){
    create_key(len)
}

#[pymethods]
impl Crypto {
    #[new]
    pub fn __new__(pubkey: &str,prikey: &str) -> Self {
        Crypto {
            // pubkey: pubkey.to_owned(),
            // prikey: prikey.to_owned(),
            pub_key: Rsa::public_key_from_pem(pubkey.as_bytes()).unwrap(),
            pri_key: Rsa::private_key_from_pem(prikey.as_bytes()).unwrap(),
        }
    }

    // public decrypt 
    pub fn public_decrypt(&self, msg:&str) -> String {
        let mut out: [u8; 4096] = [0;4096];
        let decoded = openssl::base64::decode_block(msg).unwrap();
        if let Ok(size) = self.pub_key.public_decrypt(&decoded, &mut out, Padding::PKCS1) {
            let real_size = if size > 4096 {4096} else {size};
            // openssl::base64::encode_block(&out[..real_size])
            String::from_utf8(out[..real_size].to_vec()).unwrap()
        } else {
            String::default()
        }
    }
    
    // public encrypt 
    pub fn public_encrypt(&self, msg:&str) -> String {
        let mut out: [u8; 4096] = [0;4096];
        if let Ok(size) = self.pub_key.public_encrypt(msg.as_bytes(), &mut out, Padding::PKCS1) {
            let real_size = if size > 4096 {4096}else{size};
            openssl::base64::encode_block(&out[..real_size])
        } else {
            String::default()
        }
    }

    // private encrypt
    pub fn private_encrypt(&self, msg:&str) -> String{
        let mut out: [u8; 4096] = [0;4096];
        if let Ok(size) = self.pri_key.private_encrypt(msg.as_bytes(), &mut out, Padding::PKCS1) {
            let real_size = if size > 4096 {4096}else{size};
            openssl::base64::encode_block(&out[..real_size])
        } else {
            String::default()
        }
    }

    // private decrypt
    pub fn private_decrypt(&self, msg: &str) -> String{
        let mut out: [u8; 4096] = [0;4096];
        let decoded = openssl::base64::decode_block(msg).unwrap();
        if let Ok(size) = self.pri_key.private_decrypt(&decoded, &mut out, Padding::PKCS1) {
            let real_size = if size > 4096 {4096} else {size};
            // openssl::base64::encode_block(&out[..real_size])
            String::from_utf8(out[..real_size].to_vec()).unwrap()
        } else {
            String::default()
        } 
    }

    // sign
    pub fn sign(&self, msg: &str) ->String {
        crypto_utils::hash_hmac(SECRET, msg)
    }
}

#[pymodule]
fn yacrypto(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_class::<Crypto>()?;
    m.add_function(wrap_pyfunction!(generate_key, m)?).unwrap();
    Ok(())
}


#[cfg(test)]
mod tests {
    use base64;
    #[test]
    fn works(){

        // create rsa
        let rsa = openssl::rsa::Rsa::generate(1024).unwrap();
        // create public key 
        let public_key = rsa.public_key_to_pem().unwrap();
        println!("{:?}", String::from_utf8(public_key.clone()));
        let private_key = rsa.private_key_to_pem().unwrap();


        let data = "hellowo\n\t\rrld";
        // public encrypt 
        let mut buf:Vec<u8> = vec![0;rsa.size() as usize];
        let rsa_pub = openssl::rsa::Rsa::public_key_from_pem(&public_key).unwrap();
        let _ = rsa_pub.public_encrypt(data.as_bytes(), &mut buf , openssl::rsa::Padding::PKCS1);

        // private decrypt => 
        let data = buf;
        let mut buf:Vec<u8> = vec![0;rsa.size() as usize];
        let rsa_pri  = openssl::rsa::Rsa::private_key_from_pem(&private_key).unwrap();
        if let Ok(size) = rsa_pri.private_decrypt(&data, &mut buf, openssl::rsa::Padding::PKCS1){
            let real_size = if size > 1024 {1024} else {size};
            let buf = &buf[..real_size];
            let base64_string = openssl::base64::encode_block(&buf);
            let result = base64::decode(base64_string);
            println!("{:?}",result);
            // println!("buf => {:?}",openssl::base64::encode_block(&buf))

            let echo_str = String::from_utf8((&buf).to_vec()).unwrap();
            println!("{:?}",echo_str);
        
        }
    }
}

```