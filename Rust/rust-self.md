---
title: rust-self
date: 2021-04-16 11:05:42
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust-self

rust中的self


> <span style='font-size:20px;color:red'>&self</span>: We’ve chosen &self here for the same reason we used &Rectangle in the function version: we don’t want to take ownership, and we just want to read the data in the struct, not write to it.
> <span style='font-size:20px;color:red'>&mut self</span> `&mut self`: If we wanted to change the instance that we’ve called the method on as part of what the method does, we’d use &mut self as the first parameter.
> <span style='font-size:20px;color:red'>self</span> : Having a method that takes ownership of the instance by using just self as the first parameter is rare; this technique is usually used when the method transforms self into something else and you want to prevent the caller from using the original instance after the transformation.

```rust
fn main(){
    let pp = People{
        name: "xx".to_string(),
        age: 2,
        version: 3,
    }; 

    let add = AddVersion{
        add_version: 3,
    };
    
    pp.add_version(&add);
    
    println!("{}",add.add_version);

    pp.echo_version();
}

struct People{
    name: String,
    age: i32,
    version: i32,
}

struct AddVersion{
    add_version: i32,
}

impl  People {
    fn add_version(&self, cc: &AddVersion) -> i32{
        let c:&i32 = &cc.add_version; // import 
        let c:i32 = cc.add_version;  // import
        self.version + cc.add_version
    }

    fn echo_version(&self){
        let xx:i32 = self.version;  // import
        let xx:&i32 = &self.version; // import
        println!("{}",self.version)
    }
    
}
```


```rust
fn main(){

}
struct AddVersion{
    add_version: i32,
}

impl  People {
    fn add_version(&self, cc: &AddVersion) -> i32{
        let c:&i32 = &cc.add_version; // import 
        let c:i32 = cc.add_version;  // import
        colet c:&&i32 = &&cc.add_version;// import
        self.version + cc.add_version
    }

    fn echo_version(&self){
        let xx:i32 = self.version;  // import
        let xx:&i32 = &self.version; // import
        println!("{}",self.version)
    }
}    
```
还是要好好理解下引用这个寓意,
这里传递内容给形参的时候，用了引用。在函数内部要引用形参中的内容的话，还是需要再引用一下的


## 关于动态大小的类型该如何处理

![](https://noback.upyun.com/2021-04-16-11-49-52.png!)
