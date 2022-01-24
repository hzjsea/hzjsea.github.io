---
title: java基础
categories:
  - java
tags:
  - java
abbrlink: b8b0eacd
date: 2020-10-09 08:24:53
---

![2020-10-27-11-17-26](http://noback.upyun.com/2020-10-27-11-17-26.png)
<!-- more -->


# java基础


- 程序结构
  - 程序不同之间的访问级别
- 数值计算or数学函数
- 注释
- 运行流程
- 数据类型
  - 常量
  - 变量
  - 运算符
  - 数据类型转换
  - 三元操作符
  - 


## java程序结构
```java
package com.company;

public class Main {

    public static void main(String[] args) {
        System.out.println("helloworld");
    }
}
```
逐一解释java程序语言的结构，关键词public 是访问修饰符，这个修饰符用于控制程序的其他部分对这段代码的访问级别，
类是构建所有java应用程序和applet的构建块。java应用程序中的全部内容都必须放置在类中。
JAVA中常用驼峰命名，不适用下划线命名


## java数据类型
java是一种强类型语言(每一个变量声明一种类型)
- 4种整型
  - int   4字节
  - short 2字节
  - long 8字节
  - byte 1字节
- 2种浮点型
  - float 4字节
  - double 8字节
- 1种用于表示Unicode编码的字符单元char
  - 表示单个字符，需要使用单引号括起来。
- 1种用于表示真值的boolean类型



### Unicode和char类型
要想弄清 char 类型,就必须了解 Unicode 编码机制。Unicode 打破了传统字符编码机制 的限制。 在 Unicode 出现之前， 已经有许多种不同的标准: 美国的 ASCII、 西欧语言中的 ISO 8859-1 俄罗斯的 KOI-8、 中国的 GB 18030 和 BIG-5 等。这样就产生了下面两个问题: 一个是对于任意给定的代码值， 在不同的编码方案下有可能对应不同的字母; 二是采用大字符集的语言其编码长度有可能不同。 例如,有些常用的字符采用单字节编码,而另一些字符 则需要两个或更多个字节。
设置Unicode编码的目的就是解决这些问题，我们强烈建议不要在程序中使用 char 类型， 除非确实需要处理 UTF-16 代码单元。最好 将字符串作为抽象数据类型处理 

### 常量final
使用关键字指示常量，常量使用全大写表示

```java
final double CM_PER_INCH=2.54;
```

定义在类外面的常量，叫做`类常量`，可以在全局使用该变量
```java
package com.company;

public class Main {

    public  static final double PI = 3.14;

    public static void main(String[] args) {
        double sum = 20.0;
        double sums = 20.1;
        System.out.println(PI);
    }
}
```

### 数学函数
```java
package com.company;

import java.lang.management.PlatformLoggingMXBean;
import java.net.SocketOption;

public class Main {

    public static void main(String[] args) {
        int i = 20;
        // 计算平方根
        System.out.println(Math.sqrt(i));
        // 计算幂指数 // 接收double类型 输出double类型
        System.out.println(Math.pow(i,2));

    }
}
```
所有的数学函数都在Math这个包里面，其他的数学函数还包括
三角函数
- math.sin
- math.cos
- math.tan
π值 E值
- math.PI
- math.E

![2020-10-09-08-35-18](http://noback.upyun.com/2020-10-09-08-35-18.png)


### 类型转换
java中使用强制类型转换，也就是在类型前面加上需要转换的类型，但同时也会丢失一些数据，比如将double类型转换成int类型，转换的过程中会截断小数部分的内容进行转换
```java
double num = 20.9999;
int new_num = (int)num;
```

### 三元操作符
```java
package com.company;

public class Main {

    public static void main(String[] args) {

        int nums = 20;
        int nums2 = 10;
        String msg = nums < nums2 ? "20大于10" : "None";
        System.out.println(msg);
    }
}
```


### 位运算符
位运算计算

### 字符串
```java
public class Main {

    public static void main(String[] args) {
        String ss = "helloworld";
        String subs = ss.substring(0,2);
        System.out.println(subs);
    }
}
```

### 检测字符串是否相等
```java
package com.company;

public class Main {

    public static void main(String[] args) {
        String ss = "helloworld";
        String ss2 = "helloworld";
        boolean Flag = ss.equals(ss2);
        System.out.println(Flag);
        // 忽略大小写
        boolean FlagIgnore = ss.equalsIgnoreCase("HelloWorld");
        System.out.println(FlagIgnore)
    }
}
```
连等"=="运算符检测两个字符串是否放置在同一个位置上

### 空串和NULL字符串
空串 "" 是长度为0 的字符串
null 表示目前没有任何对象与该变量关联

```java
package com.company;

public class Main {

    public static void main(String[] args) {
        String emptyString = "";
        if (emptyString.length() == 0) {
            System.out.println("len is 0");
        }
        if (emptyString == "") {
            System.out.println("len is 0");
        }
        String NULLString  = null;
        if (NULLString == null) {
            System.out.println(NULLString);
        }
    }
}
```



### 构建字符串
可变长度字符串，对于多个子字符串拼接长字符串比较适用StringBuilder
```java
package com.company;

public class Main {

    public static void main(String[] args) {
        StringBuilder SD = new StringBuilder();
        SD.append(20);
        SD.append("hellworld");
        System.out.println(SD);
        // 转换成string对象
        String completeString = SD.toString();
    }
}
```

### 标准输入
标准输入输出流
```java
package com.company;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner input = new Scanner(System.in); // 创建输入管道流
        System.out.println("input your name");
        String name = input.nextLine(); // 输入内容
        System.out.println("your name is :"+name);
    }
}
```

### 格式化输出
pass



## 控制流程

### 块作用域
作用域的主要用来定义局部变量，变量离开当前作用域后就会被jvm释放掉
```java
package com.company;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        int n = 0;
        {
            int j = 10;
        } // j 在这里被释放
    } // n 在这里被释放
}
```

### if 控制流程
```java
package com.company;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        int n = 20;
        int j = 30;
        if (n > j ){
            System.out.println("is n");
        }else {
            System.out.println("is j");
        }

    }
}
```

### while 控制流程
```java
package com.company;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        int n = 20;
        while (n>10){
            System.out.println(n);
            n --;
        }
    }
}
```
先执行后进入while循环
```java
package com.company;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        int n = 20;
//        while (n>10){
//            System.out.println(n);
//            n --;
//        }

        do {
            System.out.println(n);
            n--;
        } while (n >10);
    }
}
```

### for确定循环
```java
package com.company;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        for (int n=0;n<20;n++){
            System.out.println(n);
        }
    }
}

```

### switch多重选择
```java
package com.company;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        int n = 20 ;
        switch (n){
            case 1:
                break;
            case 2:
                break;
            case 20:
                System.out.println("this number value is 20");
            default:
                break;
        }
    }
}

```


### 中断流程控制语句
使用break退出当前循环
```java
package com.company;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        int n = 20;
        while (n > 10){
            if (n==15){
                System.out.println(n);
                break;
            }
            n--;
        }
    }
}

```


## 静态数组 与动态数组
这里提个数组类型 `int[]` 这样的表示int类型的数组
```java
package com.company;

import java.lang.reflect.Array;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        // 创建数组 int数组 数组变量名 100个元素
        int[] a = new int[100];
        String[] ss = new  String[1];

        // 获取数组长度
        System.out.println(a.length);
        // 内容填充
        for (int i=0;i<100;i++) a[i] = 2;
        System.out.println(Arrays.toString(a));
        for (int element : a) System.out.println(element);
    }
}
```

### 数组初始化以及匿名函数
```java
package com.company;

import java.lang.reflect.Array;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        // 创建数组
        int[]  numslists = new int[100];
        // 数组创建和初始化
        int[] numslists2 = {1,2,3,4,5,6};

        // 匿名数组
        // new int[]{1,2,3,5};
    }
}
```

### 数组拷贝

```java
package com.company;

import java.lang.reflect.Array;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        int[] smallNumbers = {1,2,3,4,5,6,7,8};
        int[] luckNumbers = smallNumbers;
        luckNumbers[5] = 12;

        System.out.println(Arrays.toString(smallNumbers));
        System.out.println(Arrays.toString(luckNumbers));


        int[] numbers = {1,2,3,4,5,6,7};
        int[] nn = Arrays.copyOf(numbers,numbers.length);  // 创建新的数组
        int[] nnn = Arrays.copyOf(numbers,numbers.length*2); // 创建新的数组，扩容两倍
        nn[5] = 12;
        nnn[5] = 12;
        System.out.println(Arrays.toString(nn));
        System.out.println(Arrays.toString(numbers));
        System.out.println(Arrays.toString(nnn));
    }
}

// [1, 2, 3, 4, 5, 12, 7, 8]
// [1, 2, 3, 4, 5, 12, 7, 8]
// [1, 2, 3, 4, 5, 12, 7]
// [1, 2, 3, 4, 5, 6, 7]
// [1, 2, 3, 4, 5, 12, 7, 0, 0, 0, 0, 0, 0, 0]
```


### 动态数组
```java
package com.company;

import java.sql.Array;
import java.util.ArrayList;

public class Main {

    public static void main(String[] args) {
        // 静态数组
        int[]  a = new  int[10];
        for (int i=0;i<10;i++){
            a[0] = i;
        }

        // 动态数组
        ArrayList<String> aa = new ArrayList<String>();
        // 创建初始容量
        // ArrayList<int> aa = new ArrayList<int>(100);
        for(int i=0;i<10;i++){
            aa.add(i+"");
        }

        System.out.println(aa.toString());
    }
}
```
没有泛型类时，原始的ArraryList类提供的get方法别无选择只能返回Object,因此，get方法的调用者必须对返回值进行类型转换。


## 对象和类
面向对象程序设计(OOP),类(class)是构造对象的模板和蓝图，由类构造(construct) 对象的过程称为创建类的实例(instance)
`封装(encapsulation, 有时称为数据隐藏)`是与对象有关的一个重要概念
- 对象中的数据叫做实例域
- 曹总数据的过程叫做方法
- 对于每个特定的 类实例(对象)都有一组特定的实例域值，这些值得集合就是这个对象的当前状态

### 对象的特性
- 对象的行为(behavior) 也就是对象的方法
- 对象的状态（state) 对象中的字段
- 对象标识 (identity) 辨别具有相同行为与状态的不同对象


### 类构建
```java
package com.company;

import java.lang.reflect.Array;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.Date;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        System.out.println("helloworld");

        // 实例化类
        LocalDate date = LocalDate.now();
        Employee epl = new Employee("hzj",22,date);
        epl.GetAttrFunc();
        String name = epl.GetName();
        System.out.println(name);
    }

}

// 对象
class Employee{
    // 对象中的属性字段
    private String name;
    private int age;
    private LocalDate hirDay;

    // 对象中的行为方法
    // 初始化
    // 初始化方法中不需要返回的值类型 但是方法需要和类名字相同
    public Employee(String n,int a,LocalDate h){
        name = n;
        age = a;
        hirDay = h;
    }

    // 对象中的行为方法
    // 返回对象属性
    // 返回空
    public void GetAttrFunc(){
        System.out.println(name+","+age+","+hirDay);
    }

    public String GetName(){
        return name;
    }
}

```


### 类构造器
类的初始化需要一个构造器来解决，类构造器可以有多个，这样的形式可以保证对象的多态性，多态性的意义就是可以通过多个类的构造器创建多个具有不同属性的类，如下,我们把一个类具有多个类构造器的行为 叫做那个`重载`，但重载不仅仅适用于类构造器，`也适用于同类同名方法的时候使用`
```java
package com.company;

import java.lang.reflect.Array;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.Date;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        System.out.println("helloworld");

        // 实例化类
        LocalDate date = LocalDate.now();
        Employee epl = new Employee("hzj",22,date);
        epl.GetAttrFunc();
        String name = epl.GetName();
        System.out.println(name);


        // 类2
        Employee epl2 = new Employee("hzj2",22);
        String name2 = epl2.GetName();
        System.out.println(name2);
    }

}

// 对象
class Employee{
    // 对象中的属性字段
    private String name;
    private int age;
    private LocalDate hirDay;

    // 对象中的行为方法
    // 初始化
    // 初始化方法中不需要返回的值类型 但是方法需要和类名字相同
    public Employee(String n,int a,LocalDate h){
        name = n;
        age = a;
        hirDay = h;
    }
    public Employee(String n ,int a){
        name = n;
        age = a;
        hirDay = null;
    }

    // 对象中的行为方法
    // 返回对象属性
    // 返回空
    public void GetAttrFunc(){
        System.out.println(name+","+age+","+hirDay);
    }

    public String GetName(){
        return name;
    }
}

```

类构造器的部分如下
```java
    // 对象中的行为方法
    // 初始化
    // 初始化方法中不需要返回的值类型 但是方法需要和类名字相同
    public Employee(String n,int a,LocalDate h){
        name = n;
        age = a;
        hirDay = h;
    }
    public Employee(String n ,int a){
        name = n;
        age = a;
        hirDay = null;
    }
```
类构造器需要注意的地方
- 构造器与类同名 
- 每个类可以有一个以上的构造器 
- 构造器可以有 0 个、1 个或多个参数 
- 构造器没有返回值 
- 构造器总是伴随着 new 操作一起调用


::: tip 
无参构造器 
:::
如果在编写一个类时没有编写构造器， 那么系统就会提供一个无参数构造器。 这个构造 器将所有的实例域设置为默认值。 于是， 实例域中的数值型数据设置为 0、 布尔型数据设置 为 false、所有对象变量将设置为 nul。
```java
public Employee(){
    name = "";
    int = 0;
    hirDay = lcoalDate.now();
}
```

### 显式参数和隐式参数
```java
package com.company;

import java.lang.reflect.Array;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.Date;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        System.out.println("helloworld");

        // 实例化类
        LocalDate date = LocalDate.now();
        Employee epl = new Employee("hzj",22,date);
        epl.GetAttrFunc();
        String name = epl.GetName();
        System.out.println(name);


        // 类2
        Employee epl2 = new Employee("hzj2",22);
        String name2 = epl2.GetName();
        System.out.println(name2);
    }

}

// 对象
class Employee{
    // 对象中的属性字段
    private String name;
    private int age;
    private LocalDate hirDay;

    // 对象中的行为方法
    // 初始化
    // 初始化方法中不需要返回的值类型 但是方法需要和类名字相同
    public Employee(String n,int a,LocalDate h){
        name = n;
        age = a;
        hirDay = h;
    }
    public Employee(String n ,int a){
        name = n;
        age = a;
        hirDay = null;
    }

    // 对象中的行为方法
    // 返回对象属性
    // 返回空
    public void GetAttrFunc(){
        System.out.println(name+","+age+","+hirDay);
    }

    public String GetName(){
        return name;
    }

    public String GetHob(String hob){
        return hob;
    }
}

```
在这当中hob由外界写入的是隐式参数，类内部的是显式参数


### 类的设计 只读性和访问权限

1. 一定要保证数据私有
这是最重要的; 绝对不要破坏封装性。有时候， 需要编写一个访问器方法或更改器方法， 但是最好还是保持实例域的私有性。很多惨痛的经验告诉我们， 数据的表示形式很可能会改 变， 但它们的使用方式却不会经常发生变化。当数据保持私有时， 它们的表示形式的变化不 会对类的使用者产生影响， 即使出现 bug 也易于检测。
2. 一定要对数据初始化
Java 不对局部变量进行初始化， 但是会对对象的实例域进行初始化。 最好不要依赖于系 统的默认值， 而是应该显式地初始化所有的数据， 具体的初始化方式可以是提供默认值， 也 可以是在所有构造器中设置默认值
3. 不要在类中使用过多的基本类型
4. 不是所有的域都需要独立的域访问器和域更改器
5. 将职责过多的类进行分解
6. 类名和方法名要能够体现它们的职责
7. 优先使用不可变的类
为了保证类内部的私有字段不被破坏，java对方法和类属性都做了访问权限的设置，其中对类内部不可开放的
```java
package com.company;

import java.lang.reflect.Array;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.Date;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        // 访问类内部公开字段
        Employee epl = new Employee("hzj",22,LocalDate.now());
        System.out.println(epl.PI); // 正常访问
        //System.out.println(epl.name); // 访问失败

        // 访问类内部公开方法
        String name =  epl.GetName();
        System.out.println(name);
    }


}

// 对象
class Employee{
    // 对象中的属性字段
    private String name;
    private int age;
    private LocalDate hirDay;
    // 类内部固定值
    public final double PI =  3.14;

    // 对象中的行为方法
    // 初始化
    // 初始化方法中不需要返回的值类型 但是方法需要和类名字相同
    public Employee(String n,int a,LocalDate h){
        name = n;
        age = a;
        hirDay = h;
    }

    public String GetName(){
        return name;
    }

}

```
于是，要保证一个类的完整性和不可破坏性，则需要通过方法来设置和修改类内部的值
```java
package com.company;

import java.lang.reflect.Array;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.Date;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        // 访问类内部公开字段
        Employee epl = new Employee("hzj",22,LocalDate.now());
        System.out.println(epl.PI); // 正常访问
        //System.out.println(epl.name); // 访问失败

        // 访问类内部公开方法
        String name =  epl.GetName(); // hzj
        System.out.println(name);
        epl.SetName("hzj2");
        String name2 = epl.GetName();
        System.out.println(name2);
    }


}

// 对象
class Employee{
    // 对象中的属性字段
    private String name;
    private int age;
    private LocalDate hirDay;
    // 类内部固定值
    public final double PI =  3.14;

    // 对象中的行为方法
    // 初始化
    // 初始化方法中不需要返回的值类型 但是方法需要和类名字相同
    public Employee(String n,int a,LocalDate h){
        name = n;
        age = a;
        hirDay = h;
    }

    // 获取私有值
    public String GetName(){
        return name;
    }
    // 设置私有值
    public void SetName(String n){
        name = n;
    }
}

```

### 静态域与静态方法
关键字 `static` 用来设置静态域和静态方法，之前所设计的内容都是针对于对象实例的，不同的实例，具有不同的属性值。而静态域和静态方法则是针对于对象来设计的。 对象的实例具有多态性，不同的对象属性不同，但是他们的静态域和静态方法都都会统一，另外静态方法可以直接从对象中调用，而不需要先实例化对象后再调用方法,类似于幂指数乘法的时候使用的Math包 `Math.pow(2,5)`
```java
package com.company;

import java.lang.reflect.Array;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.Date;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        // 访问类内部公开字段
        Employee epl = new Employee("hzj",22,LocalDate.now());
        Employee epl2 = new Employee("hzj2",23,LocalDate.now());
        Employee epl3 = new Employee("hzj3",23,LocalDate.now());
        Employee epl4 = new Employee("hzj4",23,LocalDate.now());
        Employee epl5 = new Employee("hzj5",23,LocalDate.now());
        Employee epl6 = new Employee("hzj6",23,LocalDate.now());

        // 改变了 epl2的id 会发现所有实例对象的id都修改了
        epl2.id++;
        System.out.println(
                epl.id+","+
                epl2.id
        );
        // 可以直接调用
        Employee.SayHi();
    }


}

// 对象
class Employee{
    // 对象中的属性字段
    private String name;
    private int age;
    private LocalDate hirDay;
    // 类内部固定值
    public final double PI =  3.14;
    public static int id = 1;

    // 对象中的行为方法
    // 初始化
    // 初始化方法中不需要返回的值类型 但是方法需要和类名字相同
    public Employee(String n,int a,LocalDate h){
        name = n;
        age = a;
        hirDay = h;
    }

    public static void  SayHi(){
        System.out.println("hello java");
    }
}

```

### 对象析构
有些面向对象的程序设计语言， 特别是 C++, 有显式的析构器方法， 其中放置一些当对 象不再使用时需要执行的清理代码。在析构器中， 最常见的操作是回收分配给对象的存储空 间。由于 Java 有自动的垃圾回收器， 不需要人工回收内存， 所以 Java 不支持析构器。
当然， 某些对象使用了内存之外的其他资源， 例如， 文件或使用了系统资源的另一个对 象的句柄。 在这种情况下， 当资源不再需要时， 将其回收和再利用将显得十分重要。
可以为任何一个类添加 finalize 方法。finalize 方法将在垃圾回收器清除对象之前调用。 在实际应用中， 不要依赖于使用 finalize 方法回收任何短缺的资源， 这是因为很难知道这个 方法什么时候才能够调用。通常情况下 在调用外部资源的时候，可以使用close的方法来关闭资源池


## 包管理
Java 允许使用包 ( package > 将类组织起来。 借助于包可以方便地组织自己的代码， 并将 自己的代码与别人提供的代码库分开管理。为了保证包名的绝对 唯一性， Sun 公司建议将公司的因特网域名 (这显然是独一无二的) 以逆序的形式作为包 名， 并且对于不同的项目使用不同的子包


## 继承
子类与父类之间的存在着联系，我们把这之间的联系叫做继承关系
关键字 extends 表明正在构造的新类派生于一个已存在的类。 
- 已存在的类称为超类 ( superclass)、 基类(base class) 或父类(parent class); 
- 新类称为子类(subclass、) 派生类 (derivedclass) 或孩子类(childclass)。

在子类中使用super调用父类方法，包括构造函数


```java
package com.company;

import javax.naming.Name;
import java.time.LocalDate;


public class Main {

    public static void main(String[] args) {
        Employee epl = new Employee("hzj",22,LocalDate.now());
        Manager mm = new Manager("hzj",22,LocalDate.now());
        mm.setBonus(20);

        epl.InfoMessage();
        mm.InfoMessage();
//        i am father,name is hzj
//        i am children,name is hzj

    }


}

// 对象
class Employee{
    private String name;
    private int age;
    private LocalDate hirDay;

    public Employee (String n,int a,LocalDate l){
        name = n;
        age = a;
        hirDay = LocalDate.now();
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void InfoMessage(){
        System.out.printf("i am father,name is %s\n",name);
    }
}


class  Manager extends Employee{
    private double bonus;

    public Manager(String n, int a, LocalDate l) {
        super(n, a, l); // 子类构造函数调用父类构造函数
        bonus = 0;
    }

    public void setBonus(double b){
        bonus = b;
    }
    // 重写方法
    public  void  InfoMessage(){
        String  name = super.getName();

        System.out.printf("i am children,name is %s\n",name);
    }
}
```

### 禁止继承
使用final来做禁止继承
```java
final  class Employee{
    private String name;
    private int age;
    private LocalDate hirDay;

    public Employee (String n,int a,LocalDate l){
        name = n;
        age = a;
        hirDay = LocalDate.now();
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void InfoMessage(){
        System.out.printf("i am father,name is %s\n",name);
    }
}

```

### 所有类的超类
Object 类是 Java 中所有类的始祖， 在 Java 中每个类都是由它扩展而来的。 但是并不需 要这样写:
`public class Employee extends Object`


### 子类的重写
```java
package com.company;

import java.awt.*;
import java.time.LocalDate;


public class Main {

    public static void main(String[] args) {

    }

}

class Father{
    private String  fathername;


    public Father(String n){
        fathername = n;
    }

    public String getFathername() {
        return fathername;
    }

    public void setFathername(String fathername) {
        this.fathername = fathername;
    }

    public void sayhello(){
        System.out.println("hello");
    }
}

class Children extends Father{
    private  String childrenname;

    public Children(String n,String c){
        super(n);
        childrenname = c;
    }

    @Override
    public void sayhello(){
        System.out.println("hello"+childrenname);
    }

}
```


- Java 中的覆盖@Override注解 写与不写的一点点理解 
- 一般来说，写与不写没什么区别，JVM可以自识别 
- 写的情况下：即说明子类要覆盖基类的方法，基类必须存在方法 控制类型public,protected，返回值，参数列表类型）与子类方法完成一致的方法，否则会报错（找不到被Override的方法）。 
- 在不写@Override注解的情况下，当基类存在与子类各种条件都符合的方法是即实现覆盖； 
- 如果条件不符合时，则是当成新定义的方法使用。 
- 所以如果想覆盖基类方法时，最好还是写上@Override注解，这样有利于编译器帮助检查错误 

### 枚举类
`public enuni Size { SMALL, MEDIUM, LARGE, EXTRAJARGE };`


### 抽象类
抽象类被用作成为一个模板
```java
package com.company;

// 抽象类只能作为模板，不能被申明
public abstract class Person{
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public abstract String getDesc();

    public String getName(){
        return name;
    }
}
```
```java
package com.company;

public class Student extends Person{
    private  String major;

    public Student(String name, String major) {
        super(name);
        this.major = major;
    }

    @Override
    public String getDesc() {
        return "a student majoring in " + major;
    }
}

```
```java
package com.company;

public class Main {

    public static void main(String[] args) {
        Person p = new Person("hzj") {
            @Override
            public String getDesc() {
                return null;
            }
        };
    }
}
```
如上 Student继承抽象类Person，需要实现他的模板方法`getDesc()`
你可以把抽象类看成一个模板，当子类继承这个模板的时候，就要实现这个模板中的所有抽象方法字段。



## 反射
反射库(reflection library) 提供了一个非常丰富且精心设计的工具集， 以便编写能够动 态操纵 Java 代码的程序。
- 在运行时分析类的能力 • 。
- 在运行时查看对象， 例如， 编写一个 toString 方法供所有类使用。 
- 实现通用的数组操作代码。
- 利用 Method 对象， 这个对象很像中的函数指针

pass


## 接口interface 
```java
package com.company;

import java.time.LocalDate;


public class Main {

    public static void main(String[] args) {
        Employee epl = new Employee("hzj",22,LocalDate.now());
        epl.GenDate(LocalDate.now());

    }

}

// 这就是说， 任何实现 Comparable 接口的类都需要包含 compareTo 方法， 并且这个方法的参 数必须是一个 Object 对象， 返回一个整型数值。
// 接口中的所有方法自动地属于 public。 因此， 在接口中声明方法时， 不必提供关键字 public o
interface Comparable{
    int compareTo(Object object);
    LocalDate GenDate(LocalDate ll);
}

class Employee implements Comparable{
    private  String name;
    private  int age ;
    private LocalDate HriDay;
    public final int PI = (int)3.14;

    public  Employee(String n, int a ,LocalDate l){
        name = n;
        age = a;
        HriDay = l;
    }

    public String getName() {
        return name;
    }

    // 实现接口中对应的方法时， 必须是Public
    public int compareTo(Object object){
       Employee em = (Employee) object;
       return em.PI;
    }
    public LocalDate GenDate(LocalDate ll){
        return ll;
    }
}
```

在接口声明中，没有将 compareTo 方法声明为 public, 这是因为在接口中的所有 方法都自动地是public。不过，在实现接口时，必须把方法声明为pubHc; 否则，编译器 将认为这个方法的访问属性是包可见性， 即类的默认访问属性， 之后编译器就会给出试 图提供更严格的访问权限的警告信息

## 泛型程序设计
泛型程序设计的意义在于相同的行为可以兼容多种数据类型以及对象的实现


### 简单泛型类
```java
package pair1;

public class Pair<T>{
    private T first;
    private T second;

    public Pair(){ first = null ; second = null;}
    public Pair(T first,T second) { this.first = first;this.second = second;}

    public T getFirst() {
        return first;
    }

    public T getSecond() {
        return second;
    }

    public void setSecond(T second) {
        this.second = second;
    }

    public void setFirst(T first) {
        this.first = first;
    }


}
```
```java
package pair1;

public class PairTest1 {
    public static void main(String[] args) {
        String[] words = {"Mary","had","a","little"};
        ArraryAlg ag = new ArraryAlg();
        Pair<String> mm = ag.minmax(words);
        System.out.println("min"+mm.getFirst());
        System.out.println("max"+mm.getSecond());
    }
}


class  ArraryAlg{

    public  Pair<String> minmax(String[] a){
        if (a == null || a.length == 0 ) return null;
        String min = a[0];
        String max  = a[0];
        for (int i = 0; i<a.length;i++){
            if (min.compareTo(a[i])>0) min=a[i];
            if (max.compareTo(a[i])<0) max=a[i];

        }
        return new Pair<>(min,max);
    }

    public static <T extends Comparable> T min(T[] a) {
        if (a == null || a.length == 0 ) return null;
        T smallest = a[0];
        for (int i = 0; i<a.length;i++){
           if(smallest.compareTo(a[i])>0){
               smallest = a[i];
           }
        }
        return smallest;
    }
}
```

```java
package pair1;

import java.time.LocalDate;

public class PairTest2 {
    public static void main(String[] args) {
        LocalDate[] birthdays = {
                LocalDate.of(1906,12,9)
                LocalDate.of(1907,12,9)
                LocalDate.of(1908,12,9)
                LocalDate.of(1909,12,9)
                LocalDate.of(1910,12,9)
        };
        Pair<LocalDate> mm = ArraryAlg.minmax(birthdays);
        System.out.println(mm.getFirst());
        System.out.println(mm.getSecond());
    }
}


class  ArraryAlg{
    public static  <T extends Comparable> Pair<T> minmax(T[] a){
        if (a == null || a.length == 0) {
            return null;
        }
        T min = a[0];
        T max = a[0];
        for (int i=0;i<a.length;i++){
            if (min.compareTo(a[i])>0) min=a[i];
            if (max.compareTo(a[i])<0) max = a[i];
        }
        return new Pair<>(min,max);

    }
}
```

## 集合
![2020-10-16-16-02-34](http://noback.upyun.com/2020-10-16-16-02-34.png)


### map映射
```java
package com.company;

import java.util.HashMap;
import java.util.Map;

public class Main {

    public static void main(String[] args) {
	// write your code here
        Map<String,String> hs = new HashMap<>();
        // 向map中添加内容
        hs.put("foo","bar");
        // 根据key来获取value,没有对应的value 则返回null;
        String result = hs.get("foo");
        // 获取value ，没有则返回default
        String result1 = hs.getOrDefault("foo","bar copy");


        Func ff = new Func();
        ff.InputTohs(hs);

        // 对这个映射中的所有值做这个动作
//        hs.forEach((k,v)->System.out.println(k+"-"+v));
        // 根据对应的Key删除对应的元素
        hs.remove("foo");
        // 返回键值对个数
        hs.size();

        Map<String,String> hsChildren = new HashMap<>();
        // 合并字典
        hsChildren.putAll(hs);
//        hsChildren.forEach((k,v)->System.out.println(k+"-"+v));
        // 是否存在key
        hsChildren.containsKey("foo");
        // 是否存在vlaue
        hsChildren.containsValue("bar");

    }
}


class Func{
    public Func() {
    }
    public void InputTohs(Map<String,String> hs){
        for(int i =0 ;i<20;i++){
            hs.put("foo"+i,"bar");
        }
    }
}



```