---
title: 说说java中的static关键字
date: 2019-09-22 11:01:15
tags: Java
categories: 
			- Java
			- 基础知识
---

java中的static关键字用的地方还是挺多的，静态变量、静态方法、静态代码块等等，感觉需要好好总结一下，本文就说说java中的static关键字。

## static关键字的六中用法
在java中static关键字表示“全局”或者“静态”的意思，被static修饰则不需要依赖于具体的对象实例，可以方便在没有创建对象的情况下进行调用（方法/变量等）。

我们知道静态方法、静态变量等，其实在java中static关键字一共有六种用法：

> 1. 静态变量；
2. 静态常量；
3. 静态方法；
4. 静态导入；
5. 静态代码块；
6. 静态内部类；

## 一. 静态变量
static变量和非static变量的区别是：static变量不依赖于具体的对象实例，是属于类的，static变量由所有的对象共享，在内存中只有一个副本。非static变量是依赖于对象实例的，每个对象拥有不同的变量副本。通过一个例子看的比较清楚：

```java
package com;

public class Student
{
  public static int totalStudentNum = 0;
  public int studentNum = 0;
}


package com;
public class Main
{
    public static void main(String[] args)
    {
        Student student1 = new Student();
        student1.studentNum++;
        student1.totalStudentNum++;

        Student student2 = new Student();
        student2.studentNum++;
        student2.totalStudentNum++;

        System.out.println("Student.totalStudentNum is: " + Student.totalStudentNum);
    }
}
```
结果输出：
```java
Student.totalStudentNum is: 2
```

## 二. 静态常量
被static修饰的常量即为静态常量`static final PI = 3.14`，同静态变量一样，静态常量是属于类的，所有对象共享，不依赖于具体的对象实例。

`System.out`其实就是一个静态常量：

```java
public class System
{
	//...
    public static final PrintStream out;
}
```

## 三. 静态方法
可以认为静态方法是没有this参数的方法，静态方法不依赖于对象实例，所以在静态方法中不能访问实例域，在静态方法中只能访问静态域；但是非静态方法可以访问静态域。

最常见的static方法即main方法，main方法之所以为static，是因为程序在执行main方法的时候没有创建任何对象，只能通过类名来访问。

### 构造函数是静态方法吗？

静态方法中是没有this参数的，而构造函数中可以使用this参数，所以认为构造函数不是static方法。

一般static方法适合如下场景：

> 一个方法不需要访问对象状态，所需要的参数都是通过显示提供的；
一个方法只需要访问类的静态域。

```java
public class Main
{
	public static void main(String[] args)
    {
    	System.out.println(add(10,11));
    }
    
    public static int add(int a, int b)
    {
    	return a+b;
    }
}
```

## 四.静态导入
静态导入是java5中的一个特性，import不仅可以导入类，还可以导入静态方法和静态域。

比如：
```java
import static java.lang.System.*;
```

这样就可以使用System的静态方法和静态域了，而不用在前面加类的前缀。

```java
import static java.lang.Math.*;
import static java.lang.System.*;

public class Main()
{
	public static void main(String[] args)
    {
    	out.println(pow(2, 3));
        out.println(max(1,24));
    }
}
```

## 五. 静态代码块
静态代码块，就是用static修饰的代码块，作用是对静态属性进行初始化。静态代码块可以有多个，JVM加载类时会按照顺序执行这些静态代码块。

关于代码块，主要分为四类：

### 普通代码块
即正常的比如一个方法{}之间的代码段；

### 静态代码块
用static修饰的代码块
```java
public class Student
{
	private static int totalNum;
    //静态代码块
    static
    {
    	totalNum = 1;
    }
}
```
### 初始化代码块
在一个类中，不带任何修饰{}包含的代码段，只要构造类的对象，这些块就会被执行。
```java
public class Student
{
	private String name;
    //初始化块
    {
    	name = "xiaoming";
    }
}
```
**初始化块和静态代码块的区别**：

> 1. 静态代码块只会执行一次，有多个静态代码块按顺序执行；
2. 初始化代码块，每次创建对象都会执行；
3. 执行顺序：静态代码块>初始化代码块>构造函数

### 同步代码块
同步代码块多用于线程互斥，用synchronized修饰的{}包含的代码，表示同一时间只能有一个线程进入同步代码块。

## 六. 静态内部类
用static修饰的内部类即为静态内部类，静态内部类不依赖于外部类的实例。

静态内部类不需要依赖外部类的对象；
静态内部类只能反问外部类的静态成员而不能访问其他域。

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
