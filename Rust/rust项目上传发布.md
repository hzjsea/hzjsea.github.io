## 打包发布

去除 target文件夹打包

```bash
tar -cvf bb.tar --exclude=./rocket_databases/target rocket_databases/*
# 上传文件
scp -r bb.tar root@10.0.6.43:/root/ && ssh root@10.0.6.43 tar -xvf /root/bb.tar -C /root/ 
# 编译 release
cargo build --release
# 运行
./target/release/rocket_databases
```
