+++
title = "神奇的i=i++"
date = 2020-03-27T20:35:47+08:00
draft = false
slug = "/i-plus-plus"
tags = ["Java"]
categories = ["技术"]
+++

先来看一段代码。

```java
		public static void main(String[] args) {
        int i = 5;
        int a = i++;
        System.out.println("a=" + a);
        System.out.println(i);
        int b = i--;
        System.out.println("b=" + b);
        System.out.println(i);
        int c = --i;
        System.out.println("c=" + c);】
        System.out.println(i);
        int d = ++i;
        System.out.println("d=" + d);
        System.out.println(i);
    }
```

这段代码的输出结果可能很多人都能看出来，如下所示

```java
a=5
6
b=6
5
c=4
4
d=5
5
k==5
```

其中的计算逻辑也很清晰，`i++`是先赋值后计算，而`++i`是先计算后赋值。

那么换一种写法呢，可能很多小伙伴都会猜错。

```java
		public static void main(String[] args) {
        int i = 5;
        i = i++;
        System.out.println(i);
        i = i--;
        System.out.println(i);
        i = --i;
        System.out.println(i);
        i = ++i;
        System.out.println(i);
    }
```

这段代码按照之前的逻辑来看，输出结果应该为6，5，4，5。但是执行程序后的输出结果为

```java
5
5
4
5
```

为了搞清楚这中间的逻辑，我们不妨使用`javap -c`编译一下源代码看看

```java
		public static void main(String[] args) {
        int i = 5;
        i = i++;
        System.out.println(i);
    }
```

编译后的代码如下：

```java
public static void main(java.lang.String[]);
    Code:
       0: iconst_5 
       1: istore_1
       2: iload_1
       3: iinc          1, 1
       6: istore_1
       7: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      10: iload_1
      11: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
      14: return
```

这其中用到的指令集的含义如下：

- **`*iconst*`** ：将单字节的int常量值(-128~127)推送至栈顶
- ***`istore`*** ：将栈顶int型数值存入指定本地变量
- ***`iload`*** ：将指定的int型本地变量推送至栈顶
- `***iinc***`：该指令用于对本地(局部)变量进行自增减操作。该指令第一参数为本地变量的编号，第二个参数为自增减的数量
- ***`getstatic`*** ：获取指定类的静态域，并将其值压入栈顶

所以这段代码的指令集的含义为：

1. `iconst_5` 表示将一个值为5的int值推送到栈顶
2. `istore_1` 把栈顶的int值5赋给第二个本地变量`i`（第1个本地变量应该是this）
3. `iload_1` 表示把第二个本地变量`i`的值推入栈顶，此时栈顶仍然为5
4. `iinc 1, 1` 表示将第二个本地变量的值加1，也就是执行了`i++`，此时的`i`值为6了已经
5. `istore_1` 再次将栈顶int值（仍然为5）赋给第二个本地变量`i`
6. `getstatic` 后面为输出的代码

因此最终打印出来的结果是5。