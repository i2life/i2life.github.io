---
title: 说说java中的断言
date: 2019-09-22 11:44:12
tags: Java
categories: 
			- Java
			- 基础知识
---
断言也是java中的一种异常处理手段，主要用于开发和测试阶段对程序的调试。相比于抛出异常和捕获异常，断言处理机制允许在测试期间在代码中插入一些断言检查语句，当代码发布的时候通过使能和去使能断言开关可以灵活的控制是否执行断言检查，这样不会因为有大量的异常判断而导致程序性能降低。

## 一. 断言的基本语法

```java
assert 条件
assert 条件 : 表达式
```

如果条件是false，则抛出AssertionError异常，在第二种形式中，表达式将被传入AssertionError构造器，并转换成一个消息字符串，对异常进行一个详细描述。

代码示例：
```java
package com;

import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

public class Main
{
    public static void main(String[] args)
    {
        Scanner input = new Scanner(System.in);

        int number = input.nextInt();

        // 断言判断输入的参数是否为正整数
        assert number > 0 : "input number is not positive!";
    }
}
```
**编辑器开启断言编译**
编辑器默认一般都是不开启断言的，所以我们要手动开启断言，以idea为例：
Run->Edit Configurations->在Configuration的VM optons里输入-ea(-enableAssertions的缩写)即可。

输入-1，结果输出：

```java
Exception in thread "main" java.lang.AssertionError: input number is not positive!
	at com.Main.main(Main.java:16)

Process finished with exit code 1
```

## 二. 断言使用场景
1. 在私有方法中使用assert作为输入参数的校验
公有方法一般不使用assert进行参数教研，公有方法调用是开放的，入参不可控，调用方获得AssertionError错误信息，可能导致程序终止。

2. 在流程控制中不可能达到的地方使用assert
assert此时表示的意义是：程序执行到这里就是错误的。

## 三. 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)