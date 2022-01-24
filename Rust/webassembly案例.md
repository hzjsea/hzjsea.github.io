---
title: webassembly
date: 2021-05-06 10:19:56
categories:  [rust]
tags: [rust]
---


<!--more-->


# Rust WebAssembly (一)

![](https://www.rust-lang.org/static/images/wasm-ferris.png)

## 什么是webassembly 

编写一个web页面，在之前一定逃避不了html,css,js这三个好兄弟，但就在2019年12月,W3C宣布 webassembly加入他们,成为"四驱兄弟",在现阶段,webassembly的并不能直接代替js的存在,而是一种互补合作的关系


### webassembly的优点

webassembly 有一套完整的语义，实际上wasm是体积小且加载快的二进制格式，其目的就是充分发挥硬件能力以达到原生的执行效率

编译和优化所需要的时间较少，因为在将文件推送到服务器之间已经进行了更多的优化，javascript需要动态类型多次编译代码

执行速度上，执行可以更快 wasm指令更接近机器码

目前wasm不直接支持垃圾回收，垃圾回收都是手动控制的，所以比自动垃圾回收效率更高，

### 如何互补

这就要聊一聊js的运行机制了,JavaScript是一种解释型语言,在代码运行之前不会进行编译工作，而是在执行的过程中实时编译，边编译边执行,JavaScript引擎就是专门来干这个事情的，在运行过程中，JavaScript解析JS源代码并将其转换成可执行的机器码并执行

![](https://noback.upyun.com/2021-05-24-15-21-15.png!)


但对于webassembly来说，这些短板上,它有了更大的提升，webassembly本身并不是一种编程语言，而是一种字节码的标准，他通过不同种类的高级编程语言，通过对应的编译器，比如rust当中的wasm_pack编译出.wasm文件并运行在webassembly虚拟机当中,目前主流的浏览器已经支持以webassembly规范为标准的虚拟机了，可以通过如下方法做个测试。

<div style='font-size:20px;color:red'>Chrome</div>

![](https://noback.upyun.com/2021-05-06-16-22-43.png!)

<div style='font-size:20px;color:red'>Safari</div>

![](https://noback.upyun.com/2021-05-10-13-47-37.png!)


> 有了 WebAssembly以后，针对不同的语言以及他们各自的编译器，将自身的语言转移成在.wasm二进制文件，跑在浏览器的虚拟机上，直接执行，不需要转译，执行效率自然能够提高

### webassembly中可靠的案例
infoQ上的一个演讲，关于如何将autocad的代码移植到wabassembly虚拟机上运行
[《AutoCAD & WebAssembly: Moving a 30 Year Code Base to the Web》](https://www.infoq.com/presentations/autocad-webassembly/?utm_source=presentations&utm_medium=ny&utm_campaign=qcon)




### 做一个比较

我们可以先用原生的js写一段fib代码测试时间，代码内容如下
```js
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <script>
        function fibonacci(n) {
            if (n == 0 || n == 1)
                return n;
            return fibonacci(n - 1) + fibonacci(n - 2);
        }
        function fibonacciTime() {
            console.time("js")
            fibonacci(40)
            console.timeEnd("js")
        }
        fibonacciTime()
    </script>
</body>

</html>
```
测试可以用live-server跑一下时间，每次时间大概在 1275毫秒到1329毫秒之间
![](https://noback.upyun.com/2021-05-24-14-39-13.png!)

然后我们在用rust 转义成wasm来跑一次看看，`注意这里用到的算法都是最普通的递归迭代，没有用其他的算法，不然可以试试动态规划来优化`

首先是下载wasm的编译器,将rust代码编译成js wasm ts等等代码
```bash
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```

配置Cargo.toml

```bash
[package]
name = "wasm"
version = "0.2.0"
authors = ["hzjsea"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[lib]
crate-type = ["cdylib"]
path = "src/main.rs"

[dependencies]
wasm-bindgen = "0.2.48"
chrono = "0.4.19"
```

`main.rs`
```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn fib(n: i32) -> i32 {
    match n {
        0 => 1,
        1 => 1,
        _ => fib(n - 1) + fib(n - 2),
    }
}

pub fn main(){
    let xx = fib(100); // 这里算法的问题，100是很难跑出结果的
    println!("{:?}",xx);
}
```

挂载wasm文件,类似于vue一样，用个index.html把这个wasm.js文件挂载起来
```js
<script type="module">

    // rust 
  main()
  async function main() {
    const wasm = await import('/pkg/wasm.js')
    await wasm.default('/pkg/wasm_bg.wasm')
    console.time("rust")
    console.log(wasm.fib(40))
    console.timeEnd("rust")
  }
  
  // js
  function fibonacci(n) {
    if(n==0 || n == 1)
        return n;
    return fibonacci(n-1) + fibonacci(n-2);
  }
  function fibonacciTime(){
    console.time("js")
    fibonacci(40)
    console.timeEnd("js")
  }
  fibonacciTime()
</script>

```

用wasm编译器编译文件

```bash
wasm-pack build --target web --no-typescript --mode normal
```

同样是用live-server起来看下时间相差一倍左右
![](https://noback.upyun.com/2021-05-24-14-36-34.png!)


## 编译究竟做了什么

![](https://noback.upyun.com/2021-05-24-15-38-21.png!)
此次编译指定不生成wasm的ts版本。所有只有上面的三个文件，其中
- package.json 
```bash
{
  "name": "wasm",
  "collaborators": [
    "hzjsea"
  ],
  "version": "0.2.0",
  "files": [
    "wasm_bg.wasm",
    "wasm.js"
  ],
  "module": "wasm.js",
  "sideEffects": false
}
```
指定打包的各类属性
- wasm_bg.wasm
wasm.js转义后的二进制文件
- wasm.js
由rust代码转移过来的wasm.js文件。
![](https://noback.upyun.com/2021-05-24-15-52-29.png!)
![](https://noback.upyun.com/2021-05-24-15-53-05.png!)



## rust wasm

rust中有个类似于react的框架，叫做yew.这是官方的描述 :sunglasses:
Yew is a modern Rust framework for creating multi-threaded front-end web apps with WebAssembly.

- Features a macro for declaring interactive HTML with Rust expressions. Developers who have experience using JSX in React should feel quite at home when using Yew.
- Achieves high performance by minimizing DOM API calls for each page render and by making it easy to offload processing to background web workers.
- Supports JavaScript interoperability, allowing developers to leverage NPM packages and integrate with existing JavaScript applications.


[官方地址](https://github.com/yewstack/yew)


### 计时器

官方提供了一个计时器的练手项目，大概能够了解到整个的运行流程，
项目地址 https://yew.rs/docs/zh-CN/getting-started/build-a-sample-app

主要来看下lib.rs这个文件
![](https://noback.upyun.com/2021-05-24-15-02-30.png!)

## 图源
部分图源来自于
https://www.rust-lang.org/zh-CN/what/wasm

<!-- ## 链接
[WebAssembly百度百科](https://zh.wikipedia.org/wiki/WebAssembly)
[页面上执行操作系统jslinus](https://bellard.org/jslinux/)
[wasm游戏](https://milek7.pl/openttd-wasm/)
[wasmtime](https://github.com/bytecodealliance/wasmtime)
[pywasm](https://www.v2ex.com/t/669767)
[llvm](https://www.google.com/search?q=llvm&oq=llvm&aqs=chrome..69i57j0l9.829j0j4&sourceid=chrome&ie=UTF-8)
[emoji_in_markdown](https://github.com/ikatyang/emoji-cheat-sheet/blob/master/README.md#table-of-contents)
[what_is_webassembly](https://gythialy.github.io/Introduction-WebAssembly/)
[wasm介绍](https://mogeko.me/2020/078/)
https://github.com/nbrr/yew_onselection_example/blob/0fe47d9fdb36204f9d8afb42ab6ef7d8c29c2f87/src/lib.rs#L54
[canvas画图](https://www.jianshu.com/p/4d5d6297ea5e)
https://mp.weixin.qq.com/s?__biz=MzkzNTIwMjYwMA==&mid=2247483720&idx=1&sn=6e015e90f0ada38e08076731d77286d5&chksm=c2b0db0ff5c7521919a7f285d56ed956e716d5b1dc84b68956091f05bb032b2b271a7e314613&mpshare=1&scene=23&srcid=0222Z5AchOoZ8gHYMwBgaPm5&sharer_sharetime=1614003174318&sharer_shareid=c84e4834a811b230f58a772dccc9da15%23rd
![](https://noback.upyun.com/2021-05-06-16-20-00.png!)
[w3c-webassembly](https://www.w3.org/2019/12/pressrelease-wasm-rec.html.en)
[wasm](https://juejin.cn/post/6931155704594038792)
[wasm](https://www.cnblogs.com/dojo-lzz/p/8053250.html)
https://hasaki.xyz/blog/2019-07-20-%E4%BD%BF%E7%94%A8-rust-%E7%BC%96%E5%86%99-webassembly-/ -->