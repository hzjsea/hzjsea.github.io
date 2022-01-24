# shell脚本基础

## for循环

几种shell中从1到100的循环方法
```bash
# 类c语言
for ((i=1;i<=100;i++))
do 
    echo $i
done

# in使用
for i in {1..100}
do 
    echo $i
done

# seq使用
for  i in `seq 1 100`
do 
    echo $i
done
```
