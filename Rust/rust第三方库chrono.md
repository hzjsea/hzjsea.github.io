> The article is from https://docs.rs/chrono/0.4.19/chrono/#date-and-time

## translation

Chrono currently uses its own Duration type to represent the magnitude of a time span.

chrono 目前 使用 它自己的 Duration 类型来 代表时间跨度的幅度

## UTC time and Local time

UTC 是世界协调时间，世界协调时间(Universal Time Coordinated,UTC),

LT  Local time 是本地时间，也就是当前机器所在时区的时间

> 两者的区别就是时区不同，以UTC为标准，所以UTC的时区就是0时区的时间，LT地方时为本地时间，如北京为早上八点（东八区），UTC时间就为零点，时间比北京时晚八小时，以此计算即可.

比如,UTC相当于本初子午线(即经度0度)上的平均太阳时，过去曾用格林威治平均时(GMT)来表示.北京时间比UTC时间早8小时，以1999年1月1日0000UTC为例，UTC时间是零点，北京时间为1999年1月1日早上8点整,有时在邮件中会看见UTC(7:00)+8:00之类的，就是说这封邮件是从东经120度(东八区)发来的，也就是从中国发来的

## ISO日期时间表示法

> **国际标准ISO 8601**，是[国际标准化组织](https://zh.wikipedia.org/wiki/%E5%9C%8B%E9%9A%9B%E6%A8%99%E6%BA%96%E5%8C%96%E7%B5%84%E7%B9%94 "国际标准化组织")的日期和时间的表示方法，全称为《数据存储和交换形式·信息交换·日期和时间的表示方法》。目前是2004年12月1日发行的第三版“**ISO8601:2004**”以替代1998年的第一版“**ISO8601:1998**”与2000年的第二版“**ISO8601:2000**”。

https://zh.wikipedia.org/wiki/ISO_8601

周表示

```rust
pub enum Weekday {
    /// Monday.
    Mon = 0,
    /// Tuesday.
    Tue = 1,
    /// Wednesday.
    Wed = 2,
    /// Thursday.
    Thu = 3,
    /// Friday.
    Fri = 4,
    /// Saturday.
    Sat = 5,
    /// Sunday.
    Sun = 6,
}
```

## 时间戳(Unix timestamp)

时间戳是以秒来计算的，基准值为 UTC时间`1970-01-01T00:00:00`为起点，

## chrono

这个库是在`rust std::time`这个库上做的扩展

rust 代码如下

```rust
fn main(){
    // 当地时间
    let local  = Local.ymd(1970, 1,1).and_hms(0, 0, 0);
    assert_eq!(-28800,local.timestamp());
    // UTC时间 时间戳为0
    let utc  = Utc.ymd(1970, 1,1).and_hms(0, 0, 0);
    assert_eq!(0,utc.timestamp());
  
    let Local_timestamp = Local.timestamp(-28800, 0);
    assert_eq!(Local.ymd(1970,1,1).and_hms(0,0,0),Local_timestamp);
}
```

### chrono使用

```rust
use core::time;
use std::{alloc::System, ffi::IntoStringError, fmt::DebugSet, ops::{Add, Sub}, thread::sleep, time::{Duration, Instant, SystemTime}};

use chrono::{DateTime, Local, LocalResult, TimeZone, Utc, Weekday};



fn main() {
    // // 秒 + 纳秒
    // let timmer  = Duration::new(0,0);
    // assert_eq!(timmer.as_secs(), 0);
    // assert_eq!(timmer.as_nanos(),0);


    // // 毫秒
    // let timmer_millis = Duration::from_millis(10);
    // assert_eq!(timmer_millis.as_millis(),10);


  
    // // =============
    // // 测量过去的时间
    // // =============
    // use std::thread::sleep;
    // use std::time::Instant;
    // let now = Instant::now();

    // // we sleep for 2 seconds
    // sleep(Duration::new(2, 0));
    // // it prints '2'
    // println!("{}", now.elapsed().as_secs());

    // // =============

    // let now = Instant::now();
    // sleep(Duration::new(2,10));
    // let new_now = Instant::now();
    // println!("{:?}",new_now.duration_since(now));


    // // 
    // let now = std::time::SystemTime::now();
    // println!("{:?}",now);

    use chrono::prelude::*;

    let utc:DateTime<Utc> = Utc::now();
    println!("{:?}",utc); // 2021-04-08T02:32:41.839558Z
    let local_time:DateTime<Local> = Local::now(); // 2021-04-08T10:33:47.740477+08:00 东八区
    println!("{:?}",local_time);  

    let dt = Utc.ymd(2010, 10,30).and_hms(9, 10, 11); //年月日 时分秒
    println!("{:?}",dt); // 2010-10-30T09:10:11Z
    println!("{:?}",dt.time()); // 09:10:11
    println!("{:?}",dt.timestamp()); // 1288429811

    let db_local = Local.ymd(2010, 10, 10);
    println!("{:?}",db_local); //2010-10-10+08:00 
    // println!("{:?}",db_local.timestamp); // error no .time()  -> no timestamp()
    println!("{:?}",db_local.timezone()); // Local time  timezone is Local
    let dt_local = Local.ymd(2010, 10, 30).and_hms(9, 10, 11);
    println!("{:?}", dt_local);  // 2010-10-30T09:10:11+08:00
    println!("{:?}",dt_local.time()); // 09:10:11
    println!("{:?}", dt_local.timestamp()); //  1288401011

    // humanized time set
    let dt_local = Local.isoywd(2010, 10, Weekday::Tue); // 表示2010年10月的周二

    // 添加时间到秒
    let dt_local = Local.isoywd(2010, 10, Weekday::Tue).and_hms_milli(10,9, 8,7);

    // 动态验证
    let dt_local_res = LocalResult::Single(Utc.ymd(2014, 7, 8).and_hms(21, 15, 33));
    let dt_local_opt = Utc.ymd_opt(2014, 7, 8).and_hms_opt(21, 15, 33);

    if let LocalResult::None = dt_local_opt{
        println!("Invalid format",)
    }

    // 毫秒
    // let dt_local_res_micro = Utc.ymd(2014, 7, 8).and_hms_micro(9, 10, 11, 12_000);
    // 纳秒
    // let dt_local_res_nanos = Utc.ymd(2014, 1, 10).and_hms_nano(91, 19, 11, 12_000_000);


    // FixedOffset 根据其他时区来定义当前时区


    // formating and parsing
    let utc = Utc.ymd(2014, 11, 28).and_hms(12, 0, 9);
    let utc_format_string = utc.format("%Y-%m-%d %H:%M:%S");
    assert_eq!(utc_format_string.to_string(),"2014-11-28 12:00:09");
    assert_eq!(utc.to_string(), "2014-11-28 12:00:09 UTC");
    assert_eq!(utc.to_rfc2822(), "Fri, 28 Nov 2014 12:00:09 +0000");
    assert_eq!(utc.to_rfc3339(), "2014-11-28T12:00:09+00:00");


    // timestamp or unix time
    let local  = Local.ymd(1970, 1,1).and_hms(0, 0, 0);
    assert_eq!(-28800,local.timestamp());
    let utc  = Utc.ymd(1970, 1,1).and_hms(0, 0, 0);
    assert_eq!(0,utc.timestamp());

    let Local_timestamp = Local.timestamp(-28800, 0);
    assert_eq!(Local.ymd(1970,1,1).and_hms(0,0,0),Local_timestamp);


    sleep(Duration::new(20,0));
}


```

### 时间戳与Local 以及UTC 时间之间的转换

```rust
fn main(){
        // 时间戳与日期之间的转换
    let utc_time = Utc.timestamp(331155752, 0);
    // 转换成日期
    let utc_date_string = utc_time.format("%Y-%m-%d %H:%M:%S").to_string();
    println!("{:?}",utc_date_string); // "1980-06-29 19:42:32"
    // 日期转换成时间戳
    let local_time = Local.timestamp(331155752, 0);
    let local_date_string = local_time.format("%Y-%m-%d %H:%M:%S").to_string();
    println!("{:?}",local_date_string); // "1980-06-30 03:42:32"

    // 日期转换成时间戳
    let utc_time = Utc.ymd(2010, 10, 10).and_hms(10, 10, 10);
    let utc_time_unix = utc_time.timestamp();
    println!("{:?}",utc_time_unix); // 1286705410
}
```

### 定义具体的时间

```rust
fn main(){
    // 单秒
    let one_second = Duration::new(1,0);
    println!("{:?}",one_second); // 1s
    let two_second = Duration::add(one_second,Duration::new(1,0));
    println!("{:?}",two_second); // 2s
}
```

### 日期格式化

[chrono::format::strftime - Rust (docs.rs)](https://docs.rs/chrono/0.4.19/chrono/format/strftime/index.html#specifiers)
