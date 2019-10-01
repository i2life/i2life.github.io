---
title: 说说java中的异常处理机制
date: 2019-09-22 11:08:23
tags: Java
categories: Java
---
“每天上班干嘛的？“”写bug的啊“。对于程序开发来说，bug太正常了，有的bug是开发人员的疏漏造成的，有的bug跟用户的操作以及程序运行的环境有关，但是不管怎么说，遇到bug总归不是一件愉悦的事情，那么如何优雅的处理程序中的bug呢，怎么把一件不怎么愉悦的事情变得稍微愉悦一点呢？java使用一种异常处理（exception handing）机制来处理程序中的异常。

## 一. 异常分类
java中的所有异常均派生自Throwable类，Throwable类派生出Error类和Exception类。

**Error类**：Error类及其子类的实例描述了java运行时系统的内部错误和资源耗尽错误，代表的是jvm本身的错误，应用程序不应该抛出这种类型的异常，因为对于这类异常，上层应用程序也无能为力，所以我们更应该关注Excepton类的异常。

**Exception类**：Exception类又派生出两个分支，一个是RuntimeException类；一个是其他异常。由程序错误导致的（比如访问空指针、数组越界、错误的类型转换等）异常为RuntimeException，而由于IO问题，用户输入类型不匹配等导致的异常属于其他异常。

**Error和Exception的区别**：Error通常是指灾难性的致命错误，程序无法控制和处理，当出现这种错误时，jvm一般会终止线程；Exception通常是可以被程序处理的，并且在程序中应该尽可能的去处理这些异常。

### 已检查异常与未检查异常
**未检查异常**：java语言规范将Error类以及RuntimeException类的所有异常称为未检查异常。对于这种异常我们应该修正代码，而不是通过异常处理器进行处理。

**已检查异常**：其他的异常称为已检查异常。对于这类异常，要么用try-catch语句进行捕获处理，或者通过throws语句抛出。

## 二. 异常处理机制
java中的异常处理机制本质上是:抛出异常+捕获异常。

### 抛出异常（throw和throws）
java中的异常抛出使用throw和throws关键字来实现。

throw：一般用于抛出异常，后面接具体的异常对象；throws可以单独使用，但是throw要么和try-catch-finally语句配套使用，要么与throws配套使用。

throws：在方法签名后，用于声明该方法可能抛出的异常。

对于已检查异常要么进行捕获，要么进行抛出：
```java
package com;

import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

public class Main
{
    public static void main(String[] args) throws FileNotFoundException
    {
        Scanner input = new Scanner(new File("D:\\test.txt"));

        int number = input.nextInt();
        if (number < 0) {
            throw new NumberFormatException();
        }
    }
}
```
### 捕获异常(try-catch-finally语句)
捕获异常的格式：

```java
try
{
	//code1
}
catch
{
	//code2
}
finally
{
	//code3
}
//code4
```
try:用于监听，将要被监听的代码放在try语句块之内，一旦发现程序异常，异常就会被抛出；
catch：用于捕获异常，匹配到异常后，对异常进行处理，多重catch语句时，注意子类在前，父类在后；
finally：始终会执行到，用于关闭文件和资源释放。

## 三. 异常链
在catch语句中可以抛出一个异常，这样相当于对捕获的异常进行了一个包装，改变异常的类型，比如子系统A供系统B调用，如果子系统A异常了，只是想知道子系统A是否故障，可以在子系统A的catch语句里再次抛出异常：throw new SystemAException()。其中SystemAException为自定义的表示系统A故障了的异常。

但是这样有一个问题，在系统B里不知道具体的原始异常是什么，只知道封装后的异常。可以采用异常链的方式，将原始异常设置为新异常的”诱饵”：
```java
try
{
	//code
}
catch(Exception e)
{
	Throwable se = new SystemAException();
    se.initcause(e);
    throw se;
}
```
这样，当捕获到异常后，通过Throwable e = se.getCause()就可以获得原始异常了。

## 四. 自定义异常
如果标准异常库不能充分描述异常问题，可以自定义异常，定义一个类继承Exception或者继承Excpetion的子类即可，定义的类应该包括两个构造器，一个默认构造器，一个带有详细描述信息的构造器。
```java
//自定义异常
public class MyExceptoin extends Exception
{
    public MyExceptoin()
    {
    }

    public MyExceptoin(String gripe)
    {
        super(gripe);
    }
}
//抛出自定义异常
package com;
import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

public class Main
{
    public static void main(String[] args) throws FileNotFoundException, MyExceptoin
    {
        Scanner input = new Scanner(new File("D:\\test.txt"));

        int number = input.nextInt();
        if (number < 0) {
            throw new MyExceptoin();
        }
    }
}
```

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)