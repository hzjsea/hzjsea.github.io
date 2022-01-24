---
title: rust第三方库ipnet
date: 2021-04-12 11:54:33
categories:  [rust]
tags: [rust]
---


<!--more-->


# rust第三方库ipnet


![](https://noback.upyun.com/2021-04-12-13-47-47.png!)


标准库里面内置了一个net库`use std::net::ipv4addr`,但是这对平常的使用来说 并不够用，有一个第三方库 ipnet 支持子网 掩码等操作

地址
https://docs.rs/ipnet/2.3.0/ipnet/struct.Ipv4AddrRange.html

使用如下
```rust
[dependencies]
ipnet = "2.3.0"
```


## 创建带掩码的Ipnet地址
```rust
fn main(){
    // build from string
    let _net5 = Ipv4Net::from_str("10.1.1.0/24").unwrap();
    let _net5_ipv6 = Ipv6Net::from_str("fd00::/24");
 
    // build from string ipv4 or ipv6
    // from_str
    let _net = IpNet::from_str("10.0.6.55/30");
    // parse
    let _net  = "10.0.6.55/30".parse::<IpNet>();
    let _net:IpNet = "10.0.6.55/30".parse().unwrap();
}
```

## 返回各类信息
```rust
fn main(){
    // 返回主机地址
    let net4_addr = Ipv4Net::from_str("10.0.6.55/16").unwrap();
    assert_eq!(net4_addr.addr().to_string() ,"10.0.6.55");
    // 
    println!("{:?}",net4_addr.prefix_len()); // 16
    println!("{:?}",net4_addr.max_prefix_len()); // 32

    // 返回掩码
    assert_eq!(net4_addr.netmask().to_string(),"255.255.0.0");
    // 反掩码
    assert_eq!(net4_addr.hostmask().to_string(),"0.0.255.255");

    // 网络位
    assert_eq!(net4_addr.network().to_string(),"10.0.0.0");

    // 广播地址
    assert_eq!(net4_addr.broadcast().to_string(),"10.0.255.255");

    // 判断是否为子网
    let n1: Ipv4Net = "10.1.0.0/24".parse().unwrap();
    let n2: Ipv4Net = "10.1.1.0/24".parse().unwrap();
    let n3: Ipv4Net = "10.1.2.0/24".parse().unwrap();

    assert!(n1.is_sibling(&n2));
    assert!(!n2.is_sibling(&n3));
}
```

## 获取主机地址值

```rust
fn main(){
    // ipnet addr range
    let hosts = "10.0.6.55/30".parse::<Ipv4Net>().unwrap().hosts().collect::<Vec<Ipv4Addr>>();
    for i in hosts{
        println!("{:?}",&i)
    }
}
```


## 全局代码

```rust
use ipnet::{IpNet, Ipv4AddrRange, Ipv4Net, Ipv4Subnets, Ipv6AddrRange, Ipv6Net};
use core::num;
use std::{borrow::Borrow, char::from_digit, collections::VecDeque, net::{IpAddr,Ipv4Addr,Ipv6Addr}, str::FromStr, u64, vec};

// ipnet  std:net:ipaddr+prefix
// ipsubnets
// ipaddrrange


fn main(){
    // build 
    let _net4 = Ipv4Net::new(Ipv4Addr::new(10, 1, 1, 0), 24);
    let net_net4 = Ipv4Addr::new(10,0,6,55);

    // build from string
    let _net5 = Ipv4Net::from_str("10.1.1.0/24").unwrap();
    let _net5_ipv6 = Ipv6Net::from_str("fd00::/24");

    // build from string ipv4 or ipv6
    // from_str
    let _net = IpNet::from_str("10.0.6.55/30");
    // parse
    let _net  = "10.0.6.55/30".parse::<IpNet>();
    let _net:IpNet = "10.0.6.55/30".parse().unwrap();

    // Returns a copy of the network with the address truncated to the prefix length.
    
    assert_eq!(
        "192.168.12.34/16".parse::<Ipv4Net>().unwrap().trunc(),
        "192.168.0.0/16".parse().unwrap()
    );

    // 返回主机地址
    let net4_addr = Ipv4Net::from_str("10.0.6.55/16").unwrap();
    assert_eq!(net4_addr.addr().to_string() ,"10.0.6.55");
    // 
    println!("{:?}",net4_addr.prefix_len()); // 16
    println!("{:?}",net4_addr.max_prefix_len()); // 32

    // 返回掩码
    assert_eq!(net4_addr.netmask().to_string(),"255.255.0.0");
    // 反掩码
    assert_eq!(net4_addr.hostmask().to_string(),"0.0.255.255");

    // 网络位
    assert_eq!(net4_addr.network().to_string(),"10.0.0.0");

    // 广播地址
    assert_eq!(net4_addr.broadcast().to_string(),"10.0.255.255");

    // 判断是否为子网
    let n1: Ipv4Net = "10.1.0.0/24".parse().unwrap();
    let n2: Ipv4Net = "10.1.1.0/24".parse().unwrap();
    let n3: Ipv4Net = "10.1.2.0/24".parse().unwrap();

    assert!(n1.is_sibling(&n2));
    assert!(!n2.is_sibling(&n3));


    // 返回主机地址集
    // ipv4addr range
    let new_net4 = Ipv4Net::from_str("10.0.6.35/30").unwrap();
    let hosts = new_net4.hosts();
    
    let hosts_vec = hosts.collect::<Vec<Ipv4Addr>>();

    // ipnet is range 
    assert_eq!(
        hosts_vec,
        vec!["10.0.6.33".parse::<Ipv4Addr>().unwrap(), "10.0.6.34".parse::<Ipv4Addr>().unwrap()]
    );

    println!("{:?}",new_net4.subnets(32).unwrap().collect::<Vec<Ipv4Net>>());
    println!("{:?}",new_net4.subnets(32).unwrap().collect::<Vec<Ipv4Net>>()
            .iter()
            .map(|value| value.to_string())
            .collect::<Vec<_>>());
    

    // 是否包含子网段
    let net = "10.0.6.55/24".parse::<Ipv4Net>().unwrap();

    // ipnet type is contain netmask
    // ipaddr type is not contain netmask
    let net_sub_yes = "10.0.6.56/24".parse::<Ipv4Net>().unwrap();
    let net_sub_no = "10.0.5.66/24".parse::<Ipv4Net>().unwrap();

    let ip_sub_yes = "10.0.6.55".parse::<Ipv4Addr>().unwrap();
    let ip_sub_no = "10.0.5.55".parse::<Ipv4Addr>().unwrap();
    
    assert!(net.contains(&net_sub_yes));
    assert!(!net.contains(&net_sub_no));
    assert!(net.contains(&ip_sub_yes));
    assert!(!net.contains(&ip_sub_no));


    // ipnet subnets ====================================================
    // 生成 起始地址 到 结束地址 之间的ip地址，并且不小于所给到的掩码地址
    let subnets = Ipv4Subnets::new(
        "10.0.0.0".parse().unwrap(),
        "10.0.0.239".parse().unwrap(),
        26
    );
    println!("{:?}",subnets.collect::<Vec<Ipv4Net>>()); // [10.0.0.0/26, 10.0.0.64/26, 10.0.0.128/26, 10.0.0.192/27, 10.0.0.224/28]
    
    // ipnet addr range
    let hosts = "10.0.6.55/30".parse::<Ipv4Net>().unwrap().hosts().collect::<Vec<Ipv4Addr>>();
    for i in hosts{
        println!("{:?}",&i)
    }
}
```