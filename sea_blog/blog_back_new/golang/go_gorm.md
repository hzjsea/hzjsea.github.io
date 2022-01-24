---
title: go_gorm
date: 2021-06-16 11:05:55
categories:  [golang]
tags: [golang]
---


<!--more-->


# go_gorm

个人感觉go的orm没有python的好用，go还是用sqlx把

```golang
go get -u github.com/jinzhu/gorm


import _ "github.com/jinzhu/gorm/dialects/mysql"
// import _ "github.com/jinzhu/gorm/dialects/postgres"
// import _ "github.com/jinzhu/gorm/dialects/sqlite"
// import _ "github.com/jinzhu/gorm/dialects/mssql"
```


> 连接mysql

```golang
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mysql"
)

func main() {
  db, err := gorm.Open("mysql", "user:password@(localhost:port)/dbname?charset=utf8mb4&parseTime=True&loc=Local")
  defer db.Close()
}
```

> 连接Sqlite3

```golang
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/sqlite"
)

func main() {
  db, err := gorm.Open("sqlite3", "/tmp/gorm.db")
  defer db.Close()
}
```

> 连接mssql

```golang
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mssql"
)

func main() {
  db, err := gorm.Open("mssql", "sqlserver://username:password@localhost:1433?database=dbname")
  defer db.Close()
}
```

## 搭建数据库实例
如果只是测试用的话，可以用docker开一个容器跑mysql
```bash
docker run --name mysql8019 -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root1234 -d mysql:8.0.19
```
用本地网络的话，可以用 -h 指明本地
```bash
docker run -it --network host --rm mysql mysql -h127.0.0.1 -P13306 --default-character-set=utf8mb4 -uroot -p
```