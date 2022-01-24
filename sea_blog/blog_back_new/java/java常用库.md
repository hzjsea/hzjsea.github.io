---
title: Java常用库
categories:
  - java
tags:
  - java
abbrlink: ad66360
date: 2020-10-19 14:23:59
---


![2020-10-27-11-17-26](http://noback.upyun.com/2020-10-27-11-17-26.png)
<!-- more -->

# 库的使用

## math库的使用
```java
package com.company;


public class Main {

    public static void main(String[] args) {
        // Math 库 总结


        // 记录圆周率
        System.out.println(Math.PI);

        // 记录e的常量
        System.out.println(Math.E);

        // 求绝对值
        int nums = -1;
        System.out.println(Math.abs(nums));

        // 正弦函数 sin  反正弦函数 asin
        // 余弦函数 cos acos
        // 正弦函数 tan atan

        // 得到不小于某数的最大整数 ，进位
        float num = (float) 0.2;
        System.out.println(Math.ceil(num)); // 1.0

        // 得到不大于某数的最小整数
        System.out.println(Math.floor(2.1)); // 2.0

        // 求余
        System.out.println(Math.IEEEremainder(7,3));  // 1.0
        System.out.println(7 % 3); // 1

        // 去除余数,地板除
        System.out.println(7/3);// 2

        // 求两数中最大的数
        System.out.println(Math.max(20,30));

        // 求两数中最小的数
        System.out.println(Math.min(20,30));

        // 求开方
        System.out.println(Math.sqrt(20));

        // 求某个数的任意次方
        System.out.println(Math.pow(2,3));

        // e的任意次方
        System.out.println(Math.exp(2));

        // 求距离某数最近的整数  // 返回double类型
        System.out.println(Math.rint(2.1)); // 2.0

        // 返回距离某数最近的一个int类型或者long类型的
        System.out.println(Math.round(2.0));

    }
}


```


## hashset的使用
hashset是一个没有重复元素的集合
它是由HashMap实现的，不保证元素的顺序，而且HashSet允许使用 null 元素。
HashSet是非同步的。如果多个线程同时访问一个哈希 set，而其中至少一个线程修改了该 set，那么它必须 保持外部同步。这通常是通过对自然封装该 set 的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 Collections.synchronizedSet 方法来“包装” set。最好在创建时完成这一操作，以防止对该 set 进行意外的不同步访问：
```java
Set s = Collections.synchronizedSet(new HashSet(...));
```
申明一个hashset
```java
Set hash2 = new HashSet();
```



## String的使用
```java
package com.company;


import java.util.HashSet;
import java.util.Set;

public class Main {

    public static void main(String[] args) {
        // 定义字符串
        /*
        * 使用第一种方式创建字符串对象时，JVM 首先会检查该对象是否在字符串常量池中，内存。
        * 如果在，就返回该对象引用，否则新的字符串将在常量池中被创建。
        * 这种方式可以减少同一个值的字符串对象的重复创建，节约内存
        */
        String str = "hello java";
        String str2;
        str2 = "hello world";

    

        // 使用java的构造方法创建        
        /*
        * 使用第二种方式创建字符串对象时，
        * 首先在编译类文件时，"abc" 常量字符串将会放入到常量结构中，在类加载时，"abc" 将会在常量池中创建；
        * 其次，在调用 new 时，JVM 命令将会调用 String 的构造函数，同时引用常量池中的 "abc" 字符串，在堆内存中创建一个 String 对象；最后，str 将引用 String 对象
        */
        String str3 = new String("hello world");

        // 获取字符串长度
        int length = str.length();

        // 得到索引为7的字符
        char ch = str.charAt(7);

        // 字符串比较是否相等
        /*
        * 不忽略大小写 equals
        * 忽略大小写 compareToIgnoreCase()*/
        System.out.println(str.equals(str2)); // 这个是来比较字符串值是否相同
        System.out.println(str == str2); // 这个是来比较他们是否引用相同的实例
        System.out.println(str.equalsIgnoreCase(str2));

        // 字符串大小
        System.out.println(str.compareTo(str2)); // 按字典顺序比较两个字符串的大小
        // 忽略字符串的大小写情况下比较字符串的大小
        System.out.println(str.compareToIgnoreCase(str2));

        // String 转换为int
        String ss = "123";
        int n = Integer.parseInt(ss);
         // 或者
        n = Integer.valueOf(ss).intValue();

        // int转换String
        int i = 20;
        String s =  String.valueOf(i);
        String szs  = Integer.toString(i);
        String sss = "" + i;

        System.out.println("Stringbuffer的使用方法============");
        // StringBuffer的使用方法
        // 创建空对象
        StringBuffer sb = new StringBuffer();
        // 分配512字节的字符缓冲区
        StringBuffer sb1 = new StringBuffer();
        // 创建带有内容的StringBuffer对象，在字符缓冲区中存放字符串
        StringBuffer sb2 = new StringBuffer("how are you?");
        // 追加内容,拼接字符串
        sb2.append(true);
        // 删除指定位置的字符
        System.out.println(sb2.toString());
        sb2.deleteCharAt(2);
        System.out.println(sb2.toString());
        // 删除指定区间的字符
        sb2.delete(1,3);
        System.out.println(sb2.toString());
        // 插入数据
        sb2.insert(3,false);
        // 反转字符串
        sb2.reverse();
        // 在字符串的摸个位置设置字符
        sb2.setCharAt(1,'D');
        // 获取字符串长度
        System.out.println(sb2.length());
        // 设置字符缓冲区大小
        // 如果小于当前字符串长度的值调用setlength()方法，则新长度后面的字符串会消失
        sb2.setLength(100);
        sb2.setLength(200);
        // 获取字符串的容量
        System.out.println(sb2.capacity());
        // 重新设置字符串容量大小
        sb2.ensureCapacity(32);
        System.out.println(sb2.capacity());

        StringBuffer sb4  = new StringBuffer();
        sb4.ensureCapacity(32);
        System.out.println(sb4.capacity()); // 34
    }
}

```

### String StringBuffer StringBuilder的区别
StringBuffer、StringBuilder和String一样，也用来代表字符串。
- String类是不可变类，任何对String的改变都 会引发新的String对象的生成；
- StringBuffer则是可变类，任何对它所指代的字符串的改变都不会产生新的对象

String 是 Java 语言非常基础和重要的类，提供了构造和管理字符串的各种基本逻辑。它是典型的 Immutable 类，被声明成为 final class，所有属性也都是 final 的。也由于它的不可变性，类似拼接、裁剪字符串等动作，都会产生新的 String 对象。由于字符串操作的普遍性，所以相关操作的效率往往对应用性能有明显影响。

StringBuffer 是为解决上面提到拼接产生太多中间对象的问题而提供的一个类，我们可以用 append 或者 add 方法，把字符串添加到已有序列的末尾或者指定位置。StringBuffer 是一个线程安全的可修改字符序列。StringBuffer 的线程安全是通过在各种修改数据的方法上用 synchronized 关键字修饰实现的。

StringBuilder 是 Java 1.5 中新增的，在能力上和 StringBuffer 没有本质区别，但是它去掉了线程安全的部分，有效减小了开销，是绝大部分情况下进行字符串拼接的首选。

StringBuffer 和 StringBuilder 底层都是利用可修改的（char，JDK 9 以后是 byte）数组，二者都继承了 AbstractStringBuilder，里面包含了基本操作，区别仅在于最终的方法是否加了 synchronized。构建时初始字符串长度加 16（这意味着，如果没有构建对象时输入最初的字符串，那么初始值就是 16）。我们如果确定拼接会发生非常多次，而且大概是可预计的，那么就可以指定合适的大小，避免很多次扩容的开销。扩容会产生多重开销，因为要抛弃原有数组，创建新的（可以简单认为是倍数）数组，还要进行 arraycopy。

除非有线程安全的需要，不然一般都使用 StringBuilder。

#


先来看一下String类的定义
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```
String类被final关键字修饰，表示不可继承String类。
String类的数据存储于char[]数组，这个数组被 final 关键字修饰，表示 String 对象不可被更改。

- 保证 String 对象安全性。避免 String 被篡改。
- 保证hash值不会频繁变更，避免String被篡改
- 可以实现字符串常量池

### 字符串拼接
如果需要使用字符串拼接，应该优先考虑 `StringBuilder` 或 `StringBuffer(线程安全)` 的append方法替代使用+号
虽然如果用的是+号进行拼接的话，也会转换成为StringBuilder 但是 连续相加则会不断的生成新的StringBuilder 所以最好是直接使用StringBuilder来拼接
在字符串常量中，默认会将对象放入常量池；在字符串变量中


### 使用String.itern节省内存
在每次赋值的时候使用 String 的 intern 方法，如果常量池中有相同值，就会重复使用该对象，返回对象引用，这样一开始的对象就可以被回收掉
在字符串常量中，默认会将对象放入常量池；在字符串变量中，对象是会创建在堆内存中，同时也会在常量池中创建一个字符串对象，复制到堆内存对象中，并返回堆内存对象引用。
如果调用 intern 方法，会去查看字符串常量池中是否有等于该对象的字符串，如果没有，就在常量池中新增该对象，并返回该对象引用；如果有，就返回常量池中的字符串引用。堆内存中原有的对象由于没有引用指向它，将会通过垃圾回收器回收。
```java
@Data
public class SharedLocation {

	private String city;
	private String region;
	private String countryCode;
}

SharedLocation sharedLocation = new SharedLocation();
sharedLocation.setCity(messageInfo.getCity().intern());		
sharedLocation.setCountryCode(messageInfo.getRegion().intern());
sharedLocation.setRegion(messageInfo.getCountryCode().intern());
```
虽然使用 new 声明的字符串调用 intern 方法，也可以让字符串进行驻留，但在业务代码中滥用 intern，可能会产生性能问题。
