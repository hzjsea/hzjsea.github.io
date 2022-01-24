---
title: rust_package_clap
date: 2021-03-18 13:00:04
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust_package_clap

这个库就是rust中的一个command库,现在就来看看这个库里面的源码，正好可以学习一下rust这个语言的编写模式
其中可能会掺杂很多基础的只是内容,pass


## App
app是一个结构体，结构体中包含多种属性，这些属性都服务于command,比如
- about help中的key arguments的描述
- version 版本描述
- aliases 别名
等等

结构体内容如下
```rust
#[derive(Default, Debug, Clone)]
pub struct App<'help> {
    pub(crate) id: Id,
    pub(crate) name: String,
    pub(crate) long_flag: Option<&'help str>,
    pub(crate) short_flag: Option<char>,
    pub(crate) bin_name: Option<String>,
    pub(crate) author: Option<&'help str>,
    pub(crate) version: Option<&'help str>,
    pub(crate) long_version: Option<&'help str>,
    pub(crate) about: Option<&'help str>,
    pub(crate) long_about: Option<&'help str>,
    pub(crate) before_help: Option<&'help str>,
    pub(crate) before_long_help: Option<&'help str>,
    pub(crate) after_help: Option<&'help str>,
    pub(crate) after_long_help: Option<&'help str>,
    pub(crate) aliases: Vec<(&'help str, bool)>, // (name, visible)
    pub(crate) short_flag_aliases: Vec<(char, bool)>, // (name, visible)
    pub(crate) long_flag_aliases: Vec<(&'help str, bool)>, // (name, visible)
    pub(crate) usage_str: Option<&'help str>,
    pub(crate) usage: Option<String>,
    pub(crate) help_str: Option<&'help str>,
    pub(crate) disp_ord: usize,
    pub(crate) term_w: Option<usize>,
    pub(crate) max_w: Option<usize>,
    pub(crate) template: Option<&'help str>,
    pub(crate) settings: AppFlags,
    pub(crate) g_settings: AppFlags,
    pub(crate) args: MKeyMap<'help>,
    pub(crate) subcommands: Vec<App<'help>>,
    pub(crate) replacers: HashMap<&'help str, &'help [&'help str]>,
    pub(crate) groups: Vec<ArgGroup<'help>>,
    pub(crate) current_help_heading: Option<&'help str>,
    pub(crate) subcommand_placeholder: Option<&'help str>,
    pub(crate) subcommand_header: Option<&'help str>,
}
```
其中的配置属性，具体用到了可以再去看，主要看这个结构体
- 声明了一个内部的`'help`的生命周期
- pub(crate) 表示变量只在当前的create中可见



### impl块分成多块写
多块写的目的是项目结构更加整洁完整
```rust
struct People{
    name: String,
    age: i32,
}

impl People {
    fn todo(&self){
        println!("{}",self.age)
    }
}

impl People{
    fn todo2(&self){
        println!("{}",self.name)
    }
}

fn main(){
    let p1 = People{ name:"hzj".to_owned(), age:12};
    p1.todo();
    p1.todo2();
}
```

### inline
inline是一个内联函数的标记
https://www.thinbug.com/q/37639276

![](https://noback.upyun.com/2021-03-18-14-38-15.png!)

### new

```rust
    pub fn new<S: Into<String>>(name: S) -> Self {
        let name = name.into();
        App {
            id: Id::from(&*name),
            name,
            disp_ord: 999,
            ..Default::default()
        }
    }
```
new方法是初始化一个App结构体，必须接受一个 实现`Into<String> trait`的参数，在方法内再转移成`String`类型，这样做就是统一化接受的类型


### Default
实现了Default的结构体，可以调用`.default`初始化结构体，如下
```rust
#[derive(Default)]
struct Person{
    name: String,
    age: i32,
    
}

impl Person{
    fn todo2(&self){
        println!("{}",self.name)
    }
}

fn main(){
    let p2 = Person::default(); // 初始化全部
    let p3  = Person{
        name:"hzj".to_owned(), 
        ..Default::default()  // 初始化部分
    };
    p2.todo2();
}
```

### author bin_name等基本属性
```rust
    pub fn author<S: Into<&'help str>>(mut self, author: S) -> Self {
        self.author = Some(author.into());
        self
    }

    pub fn bin_name<S: Into<String>>(mut self, name: S) -> Self {
        self.bin_name = Some(name.into());
        self
    }
    
    pub fn about<S: Into<&'help str>>(mut self, about: S) -> Self {
        self.about = Some(about.into());
        self
    }

    pub fn long_about<S: Into<&'help str>>(mut self, about: S) -> Self {
        self.long_about = Some(about.into());
        self
    }

    pub fn name<S: Into<String>>(mut self, name: S) -> Self {
        self.name = name.into();
        self
    }
```


把自己传进来，设置了一个author的值就返回出去了

### arg 设置key 
```rust
    pub fn arg<A: Into<Arg<'help>>>(mut self, a: A) -> Self {
        let mut arg = a.into();
        if let Some(help_heading) = self.current_help_heading {
            arg = arg.help_heading(Some(help_heading));
        }
        self.args.push(arg);
        self
    }
```
同理，App结构体中有一个args属性，内容如下
```rust
#[derive(Default, PartialEq, Debug, Clone)]
pub(crate) struct MKeyMap<'help> {
    pub(crate) keys: Vec<Key>,
    pub(crate) args: Vec<Arg<'help>>,

    // FIXME (@CreepySkeleton): this seems useless
    built: bool, // mutation isn't possible after being built
}

```
```rust
pub(crate) struct Key {
    pub(crate) key: KeyType,
    pub(crate) index: usize,
}
```

```rust
#[allow(missing_debug_implementations)]
#[derive(Default, Clone)]
pub struct Arg<'help> {
    pub(crate) id: Id,
    pub(crate) name: &'help str,
    pub(crate) about: Option<&'help str>,
    pub(crate) long_about: Option<&'help str>,
    pub(crate) blacklist: Vec<Id>,
    pub(crate) settings: ArgFlags,
    pub(crate) overrides: Vec<Id>,
    pub(crate) groups: Vec<Id>,
    pub(crate) requires: Vec<(Option<&'help str>, Id)>,
    pub(crate) r_ifs: Vec<(Id, &'help str)>,
    pub(crate) r_unless: Vec<Id>,
    pub(crate) short: Option<char>,
    pub(crate) long: Option<&'help str>,
    pub(crate) aliases: Vec<(&'help str, bool)>, // (name, visible)
    pub(crate) short_aliases: Vec<(char, bool)>, // (name, visible)
    pub(crate) disp_ord: usize,
    pub(crate) unified_ord: usize,
    pub(crate) possible_vals: Vec<&'help str>,
    pub(crate) val_names: VecMap<&'help str>,
    pub(crate) num_vals: Option<u64>,
    pub(crate) max_vals: Option<u64>,
    pub(crate) min_vals: Option<u64>,
    pub(crate) validator: Option<Arc<Mutex<Validator<'help>>>>,
    pub(crate) validator_os: Option<Arc<Mutex<ValidatorOs<'help>>>>,
    pub(crate) val_delim: Option<char>,
    pub(crate) default_vals: Vec<&'help OsStr>,
    pub(crate) default_vals_ifs: VecMap<(Id, Option<&'help OsStr>, &'help OsStr)>,
    pub(crate) default_missing_vals: Vec<&'help OsStr>,
    pub(crate) env: Option<(&'help OsStr, Option<OsString>)>,
    pub(crate) terminator: Option<&'help str>,
    pub(crate) index: Option<u64>,
    pub(crate) help_heading: Option<&'help str>,
    pub(crate) global: bool,
    pub(crate) exclusive: bool,
    pub(crate) value_hint: ValueHint,
}

```

## Arg
```rust
#[allow(missing_debug_implementations)]
#[derive(Default, Clone)]
pub struct Arg<'help> {
    pub(crate) id: Id,
    pub(crate) name: &'help str,
    pub(crate) about: Option<&'help str>,
    pub(crate) long_about: Option<&'help str>,
    pub(crate) blacklist: Vec<Id>,
    pub(crate) settings: ArgFlags,
    pub(crate) overrides: Vec<Id>,
    pub(crate) groups: Vec<Id>,
    pub(crate) requires: Vec<(Option<&'help str>, Id)>,
    pub(crate) r_ifs: Vec<(Id, &'help str)>,
    pub(crate) r_unless: Vec<Id>,
    pub(crate) short: Option<char>,
    pub(crate) long: Option<&'help str>,
    pub(crate) aliases: Vec<(&'help str, bool)>, // (name, visible)
    pub(crate) short_aliases: Vec<(char, bool)>, // (name, visible)
    pub(crate) disp_ord: usize,
    pub(crate) unified_ord: usize,
    pub(crate) possible_vals: Vec<&'help str>,
    pub(crate) val_names: VecMap<&'help str>,
    pub(crate) num_vals: Option<u64>,
    pub(crate) max_vals: Option<u64>,
    pub(crate) min_vals: Option<u64>,
    pub(crate) validator: Option<Arc<Mutex<Validator<'help>>>>,
    pub(crate) validator_os: Option<Arc<Mutex<ValidatorOs<'help>>>>,
    pub(crate) val_delim: Option<char>,
    pub(crate) default_vals: Vec<&'help OsStr>,
    pub(crate) default_vals_ifs: VecMap<(Id, Option<&'help OsStr>, &'help OsStr)>,
    pub(crate) default_missing_vals: Vec<&'help OsStr>,
    pub(crate) env: Option<(&'help OsStr, Option<OsString>)>,
    pub(crate) terminator: Option<&'help str>,
    pub(crate) index: Option<u64>,
    pub(crate) help_heading: Option<&'help str>,
    pub(crate) global: bool,
    pub(crate) exclusive: bool,
    pub(crate) value_hint: ValueHint,
}

```

### new
```rust
    pub fn new<S: Into<&'help str>>(n: S) -> Self {
        let name = n.into();
        Arg {
            id: Id::from(&*name),
            name,
            disp_ord: 999,
            unified_ord: 999,
            ..Default::default()
        }
    }
```

### short long 等属性
```rust
    pub fn short(mut self, s: char) -> Self {
        if s == '-' {
            panic!("short option name cannot be `-`");
        }

        self.short = Some(s);
        self
    }

    pub fn long(mut self, l: &'help str) -> Self {
        self.long = Some(l.trim_start_matches(|c| c == '-'));
        self
    }
```

### take_value
这个参数表示是否会携带参数
```rust
/*
use clap::App;
use clap::Arg;

fn main(){
    let command = App::new("helloworld")
            .author("author")
            // option -c
            .arg(Arg::new("config")
                .short('c')
                .long("config")
                .takes_value(false))
            
            .get_matches();
}


USAGE:
    xx [FLAGS]

FLAGS:
    -c, --config     
    -h, --help       Prints help information
    -V, --version    Prints version information
    
*/


use clap::App;
use clap::Arg;

fn main(){
    let command = App::new("helloworld")
            .author("author")
            // option -c
            .arg(Arg::new("config")
                .short('c')
                .long("config")
                .takes_value(true))
            
            .get_matches();
}

/*
USAGE:
    xx [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -c, --config <config>    
    
*/
```

### required
是否为必须的参数



## subcommand
subcommand是一个app结构体的合集，方法参考App

```rust
 match matches.subcommand() {
        ("server", Some(sub_cmd)) => {
            let rt = tokio::runtime::Builder::new_multi_thread()
                .thread_stack_size(256 * 1024 * 1024)
                .enable_io()
                .enable_time()
                .build()
                .unwrap();
        }
 }

println!("{:?}",matches.subcommand()) // ("", None)
```

subcommand() 将会返回一个subcommand 以及多个参数(agrsMatches)

```rust
use clap::*;
use tokio::time::{self, Duration};

fn main(){
    let version = "0.1";
    let app = App::new("CMPP Server/Client")
    .version(version)
    .author("// Upai <gongxun.xia@upai.com>")
    .about("CMPP Server and Client Utilities")
    .subcommand(
        App::new("server")
            .about("Run CMPP Server")
            .arg(
                Arg::with_name("simple")
                    .long("simple")
                    .value_name("SIMPLE")
                    .help("run as simple mode, just test submit command")
                    .takes_value(false),
            )
            .arg(
                Arg::with_name("submit_rate")
                    .long("submit-rate")
                    .value_name("SUBMIT_RATE")
                    .help("limit the input submit rate")
                    .takes_value(true),
            )
            .arg(
                Arg::with_name("delivery_rate")
                    .long("delivery-rate")
                    .value_name("DELIVERY_RATE")
                    .help("limit the output delivery rate")
                    .takes_value(true),
            )
            .arg(
                Arg::with_name("nsq_server")
                    .long("nsq")
                    .value_name("NSQ")
                    .help("specifiy the host of NSQ server")
                    .takes_value(true),
            ),
    );
    let matches = app.get_matches();

    
    // println!("{:?}",matches.subcommand)
    println!("{:?}",matches.subcommand());

    match matches.subcommand(){
        // test run command 
        // cargo run server --simple --submite-rate 10
        (sub_cmd,Some(args)) => {
            println!("{:?}",sub_cmd);
            println!("{:?}",args);
        }
        _ => {
             println!("usage error, please provide subcommand!");
             println!("{:?}",matches.usage());
             return ;
        }
    }
}
```



### get_matches
获取输入的参数
```rust
    pub struct ArgMatches {
        pub(crate) args: IndexMap<Id, MatchedArg>,
        pub(crate) subcommand: Option<Box<SubCommand>>,
    }
    
    pub fn get_matches(self) -> ArgMatches {
        self.get_matches_from(&mut env::args_os())
    }

    pub fn get_matches_from<I, T>(mut self, itr: I) -> ArgMatches
    where
        I: IntoIterator<Item = T>,
        T: Into<OsString> + Clone,
    {
        self.try_get_matches_from_mut(itr).unwrap_or_else(|e| {
            // Otherwise, write to stderr and exit
            if e.use_stderr() {
                e.message.print().expect("Error writing Error to stderr");

                if self.settings.is_set(AppSettings::WaitOnError) {
                    wlnerr!("\nPress [ENTER] / [RETURN] to continue...");
                    let mut s = String::new();
                    let i = io::stdin();
                    i.lock().read_line(&mut s).unwrap();
                }

                drop(self);
                drop(e);
                safe_exit(2);
            }

            drop(self);
            e.exit()
        })
    }
```



## env
系统自带的环境变量的库

获取当前的文件夹
```rust
fn main(){
    
    let current_dir = env::current_dir().unwrap();
    println!("{:?}",current_dir)
}
```

获取command参数
```rust
fn main(){
    let args = env::args_os();
    println!("{:?}",args)
}
```