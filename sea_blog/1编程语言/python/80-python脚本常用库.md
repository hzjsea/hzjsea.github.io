# python

## argparse 命令行、参数、子命令解析器
库地址:https://docs.python.org/zh-cn/3/library/argparse.html#module-argparse

主要作用是用来编写脚本接口的


实例
```python
import  
```


## 命令行运行Python脚本时传入参数的方式
sys模块中的方法
## sys.argv

```python
import sys
try:
    t1 = sys.argv[1]
    t2 = sys.argv[2]
    print(t1,t2)
except:
    print("none")
```

sys.argv记录的是整个参数的列表，比如你在命令行执行了python3 load_remote 1 2 
则sys.argv中记录的列表是这样的
['load_remote.py', '1', '2']