# ipv6

1. 为什么使用ipv6？
2. ipv6和ipv4的区别？
3. 使用ipv6



**ipv6和ipv4的区别**
总体结构上，IPv6数据报格式与IPv4数据报格式是一样的，也是由IP报头和数据（在IPv6中称为有效载荷）这两个部分组成的，但在IPv6数据报数据部分还可以包括0个或者多个IPv6扩展报头,看一下ipv6的报头
![](http://img.noback.top/blog/imgip.png)

- version（版本）
- traffic class (流量类别)
- Flow label （流量标签）
- Payload length（有效载荷长度）
- Next header（下一报头）


## 使用ipv6
以前的v4是由32位组成的，分成4组，每组8个二进制表示
现在的v6是由128位组成，分成8组，每组4位16进制表示，同样以:来分隔
比如   2001:cdba:0000:0000:0000:0000:3257:9652
为了简写方便，规定可以省略前导0,上述的ip地址可以表示为如下所示
2001:cdba:0:0:0:0:3257:9652
为了简写方便，规定可以省略一系列的0，上诉的ip地址可以表示为如下所示
2001:cdba::3257:9652

**ipv6分类**
在v4中，根据使用的内容，一般被分为5大类（abcde）
在v6中，主要被划分为3种地址类型: 单播地址、组播地址和任意播地址





https://juejin.im/post/5acc8221f265da239236accf#heading-1
https://juejin.im/post/5c7e2d55e51d45604e59f76a
http://www.h3c.com/cn/d_201311/804102_30005_0.htm#_Toc366671008
https://blog.csdn.net/firefly_2002/article/details/8034897
