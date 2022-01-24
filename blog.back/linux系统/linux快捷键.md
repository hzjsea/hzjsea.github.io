# vim笔记
## 大小写缓存插件
```bash
Plug ‘tpope/vim-capslock’ 

# 查看帮助
:help capslock  
# <C-L> 切换一个永久的大写模式，可以在底部的状态栏中看到
Ctrl+L

```
## vim基础使用方法

### 光标移动
A 移动到当前行最后并输入
gI 移动到当前行最前面并输入 

### 查看内置文档
:h vimtutor

### 查看内置的快捷键映射项
:h key-notation

在接下来的文章内容中，有几点需要注意
1. 指的是ctrl+p
2. g 指的是先按g 再按ctrl+p
3. 按回车键  也写作
4. 按shift+tab

### 常用快捷键
```bash
esc 退出编辑模式，不建议使用
<c-[>退出编辑模式，推荐使用

.  重复上一次修改操作
.. 重复上两次修改操作

0 光标跳转到最前面一个字符
$ 光标跳转到最后一个字符
a  在光标后插入
A

o 在当前行后插入一行
O 在当前行上插入一行

P 当前行粘贴
p 空一行粘贴

yy 拷贝当前行
ddp 剪切当前行

u 撤回上一步修改操作

n<command> 重复某个命令n次

```

### 行动
/pattern 搜索 enter确认 n跳转
:w 保存
:saveas <path/to/file> 另存为
 向下翻页
 向上翻页

zz 让光标所在的行居屏幕中央
zt 让光标所在的行居屏幕第一行
zb 让光标所在行居屏幕最后一行

### 移动光标
ngg nG :n 都可以快速移动到第n行
gg 第一行
GG 最后一行
w 到下一个单词
e 到下一个单词结尾
v 可视化选择
^ 到本行的第一个非blank字符
g_ 到本行最后一个非blan的字符
f{char} 到本行的下一个char字符
t{char}   到char前的第一个字符

### 常用组合键
gU 变大写
gu 变小写

### 快速缩进
```bash
# 20 -30 行 缩进
:20,30 > 
```
https://www.cnblogs.com/jiqingwu/archive/2012/06/14/vim_notes.html#id71

### vim改写操作
c[n]w 重写后面n个词
c[n]h 重写前面n个字母
c[n]l 重写后面n个字母
## linux命令行移动操作


c-a 跳转到本行行首
c-e 跳转到本行行尾
c-u 删除当前光标前面的文字
c-k 删除当前光标后面的文字

c-w c-w 移除光标前一个单词 ，由于我把tmux的前缀c-b变成了c-w 所以这里要按两次c-w
### 命令行快捷键
```bash
c-l # 清除屏幕
c-c # 中断
c-r # 查找历史命令

tab # 智能补全
c-a # 行首
c-e # 行尾
c-k # 删除光标后所有字符
c-u # 清空当前命令
c-w # 删除当前单词
c-y # 粘贴 c-k和c-u的删除的内容


Ctrl Z # 把当前进程放到后台（之后可用''fg''命令回到前台）
Ctrl PageUp # 屏幕输出向上翻页
Ctrl PageDown # 屏幕输出向下翻页
```

https://www.cnblogs.com/jiqingwu/archive/2012/06/14/vim_notes.html#id71