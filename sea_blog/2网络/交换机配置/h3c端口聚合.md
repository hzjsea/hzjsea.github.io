端口聚合的作用,加量


## 端口做汇聚
```bash
interface Bridge-Aggretion 1
quit
interface Ten-GigabitEthernet 1/0/1
port link-aggregation group 1
interface Ten-GigabitEthernet 1/0/2
port link-aggregation group 1
quit
```

## python脚本
```bash
class Switch(object):
    
    # 初始化
    def __init__(self):
        print("创建成功,开始执行")
    

    # 选择要执行的功能，返回int类型
    def select_func(self):
        print("""    
                1. 端口聚合
                2. xxxx
                3. 暂定开发
                4. 返回
                """)
        func_type=input("请选择执行的功能:")
        
        return func_type

    def get_port(self):
        # 初始化方法,判断是否为连续端口
        init_func = int(input("端口是否连续: 1)是 2)否"))
        if init_func==1:
            port_start=input("请输入起始端口")
            port_end=input("请输入结束端口")
            # 操作方法
            string = "comware.CLI('system-view;interface g{0}')".format(port_start)
            print(string)
        else:
            print("还没写")

    # 根据select_func返回的值来选择相应的操作集
    def func_to_dolist(self):
        func_number = int(self.select_func())
        
        if func_number == 1:
            print("端口聚合")
            self.get_port()
            
        elif func_number == 2:
            print("xxx")
        elif func_number == 3:
            print("暂定开发")
        elif func_number == 4:
            print("返回")

if __name__ == "__main__":
    switch = Switch()
    switch.func_to_dolist()
```