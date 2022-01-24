---
title: rust第三方库clap
date: 2021-04-13 09:20:27
categories:  [clap]
tags: [clap]
---


<!--more-->


# rust第三方库clap


## App
这个就是command的applacation
```rust
let m = App::new("My Program")
    .author("Me, me@mail.com")
    .version("1.0.2")
    .about("Explains in brief what the program does") // 配置注释
    .arg(
        Arg::with_name("in_file").index(1) // 配置参数
    )
    .after_help("Longer explanation to appear after the options when \
                 displaying the help information from --help or -h")  // 执行之后输出的内容
    .get_matches();
```

## Arg
参数配置

`.takes_value(true)`指定在运行时获取参数


## ArgMatches
匹配到的参数列表



## clap 代码形式配置command
```rust
use clap::*;
fn main() {
    let matches = App::new("My Super Program")
                          .version("1.0")
                          .author("Kevin K. <kbknapp@gmail.com>")
                          .about("Does awesome things")
                          .arg(Arg::with_name("config")
                               .short("c")
                               .long("config")
                               .value_name("FILE")
                               .help("Sets a custom config file")
                               .takes_value(true))
                          .arg(Arg::with_name("INPUT")
                               .help("Sets the input file to use")
                               .required(true)
                               .index(1))
                          .arg(Arg::with_name("v")
                               .short("v")
                               .multiple(true)
                               .help("Sets the level of verbosity"))
                          .subcommand(SubCommand::with_name("test")
                                      .about("controls testing features")
                                      .version("1.3")
                                      .author("Someone E. <someone_else@other.com>")
                                      .arg(Arg::with_name("debug")
                                          .short("d")
                                          .help("print debug information verbosely")))
                          .get_matches();

    // Gets a value for config if supplied by user, or defaults to "default.conf"
    let config = matches.value_of("config").unwrap_or("default.conf");
    println!("Value for config: {}", config);

    // Calling .unwrap() is safe here because "INPUT" is required (if "INPUT" wasn't
    // required we could have used an 'if let' to conditionally get the value)
    println!("Using input file: {}", matches.value_of("INPUT").unwrap());

    // Vary the output based on how many times the user used the "verbose" flag
    // (i.e. 'myprog -v -v -v' or 'myprog -vvv' vs 'myprog -v'
    match matches.occurrences_of("v") {
        0 => println!("No verbose info"),
        1 => println!("Some verbose info"),
        2 => println!("Tons of verbose info"),
        3 | _ => println!("Don't be crazy"),
    }

    // You can handle information about subcommands by requesting their matches by name
    // (as below), requesting just the name used, or both at the same time
    if let Some(matches) = matches.subcommand_matches("test") {
        if matches.is_present("debug") {
            println!("Printing debug info...");
        } else {
            println!("Printing normally...");
        }
    }

    // more program logic goes here...
}
```

## clap yml形式配置command
```bash
touch cli.yml

🏃:cidr_pro/ (master✗) $ tree . -L 2
.
├── Cargo.lock
├── Cargo.toml
├── src
│   ├── cli.yml
│   └── main.rs
└── target
    ├── CACHEDIR.TAG
    └── debug
```
配置`cli.yml`文件
```yml
name: cird
version: "1.0"
author: hzjsea
about: input ipnet info
args:
    - config:
        short: c
        long: config
        value_name: FILE
        help: Sets a custom config file
        takes_value: true
    - INPUT:
        help: Sets the input file to use
        required: true
        index: 1
    - verbose:
        short: v
        multiple: true
        help: Sets the level of verbosity
        args:
          - desc:
            short: d
            help: show description
subcommands:
    - ipv4:
        about: ipv4 
        version: "1.0"
        author: "hzjsea"
        args:
            - desc:
                short: d
                help: show description
    - test:
        about: controls testing features
        version: "1.3"
        author: Someone E. <someone_else@other.com>
        args:
            - debug:
                short: d
                help: print debug information
```
读取配置文件
```rust
// src/main.rs
use clap::*;

fn main(){
    let cli_yaml = load_yaml!("cli.yml");
    let matchers = App::from_yaml(cli_yaml).get_matches();

    // Gets a value for config if supplied by user, or defaults to "default.conf"
    let config = matchers.value_of("config").unwrap_or("default.conf");
    println!("Value for config: {}", config);

    // Calling .unwrap() is safe here because "INPUT" is required (if "INPUT" wasn't
    // required we could have used an 'if let' to conditionally get the value)
    println!("Using input file: {}", matchers.value_of("INPUT").unwrap());

    // Vary the output based on how many times the user used the "verbose" flag
    // (i.e. 'myprog -v -v -v' or 'myprog -vvv' vs 'myprog -v'
    match matchers.occurrences_of("v") {
        0 => println!("No verbose info"),
        1 => println!("Some verbose info"),
        2 => println!("Tons of verbose info"),
        3 | _ => println!("Don't be crazy"),
    }

    // You can handle information about subcommands by requesting their matches by name
    // (as below), requesting just the name used, or both at the same time
    if let Some(matches) = matchers.subcommand_matches("test") {
        if matches.is_present("debug") {
            println!("Printing debug info...");
        } else {
            println!("Printing normally...");
        }
    }


}
```