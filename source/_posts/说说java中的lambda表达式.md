---
title: 说说java中的lambda表达式
date: 2019-09-22 10:55:02
tags: Java
categories: Java
---
ambda表达式是java8推出的一个特性，使用lambda表达式可以很方便地传递代码块，使得代码更加的简洁，本文就主要说说java中的lambda表达式。

## lambda表达式的语法
lambda表达式就是一个代码块，以及必须传入代码的变量规范。lambda表达式格式如下：
```java
(参数列表)->{代码块}
```
语法示例：
```java
example 1:
()->{System.out.println("hello");}

exmaple 2:
(String m, String n)->{return m.length()-n.length();}
//如果参数能通过上下文确定类型，可以省略参数列表里的参数类型
(m,n)->{return m.length()-n.length();}

example 3:
//如果只有一个参数，小括号可以省略
x->{return x*x;}

example 4:
//如果代码块里只有一条语句，则return和大括号可以省略
x->x*x
```
## 函数式接口
lambda表达式是不能独立执行的，所以可以由指定目标类型的函数式接口对lambda表达式进行调用。
对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式，这种接口称为函数式接口(functional interface)。

lambda表达式可以转换为一个接口，让lambda表达式更有吸引力了。
```java
//定义一个函数式接口
package com;

public interface Compute
{
    int add(int a, int b);
}

//通过lambda表达式实现接口的抽象方法
package com;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class Main
{
    public static void main(String[] args)
    {
        Compute compute = (x, y) -> x + y;
        System.out.println(compute.add(1, 1));
    }
}
```
结果输出：
```java
2
```
又比如要对一个数组进行排序，调用Arrays.sort()方法，第二个参数是一个Comparator对象实例，Comparator就是只有一个方法的接口，所以可以提供一个lambda表达式作为sort方法的参数。
```java
package com;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Main
{
    public static void main(String[] args)
    {
        String[] arr =
        { "Champions", "we", "yeah", "are" };
        Arrays.sort(arr, (m, n) ->
        {
            if (m.length() > n.length()) {
                return -1;
            } else {
                return 0;
            }
        });

        // 打印排序后的数组
        List<String> names = Arrays.asList(arr);
        names.forEach(name -> System.out.println(name + ";"));
    }
}
```
结果输出：
```java
Champions;
yeah;
are;
we;
```
## 方法引用(method reference)

方法引用可以认为是lambda表达式的一个扩展，有时候lambda表达式只是引用了一个已经声明过的方法，为了增加可读性，可以使用方法引用。使用::操作符调用引用方法，常用的主要有4类方法引用：

> 1. objectInstance::instanceMethod引用实例变量的方法
> 2. Class::staticMethod引用类的静态方法
3. Class::instanceMethod特定类型的方法引用
4. Class::new引用构造方法

对于第一种情况，比如System.out::println，等价于x->System.out.println(x)；
对于第二种情况，比如Math::pow，等价于(x,y)->Math.pow(x,y);
对于第三种情况，比如String::compareToIngoreCase，等价于(x,y)->x.compareToIngoreCase(y);
对于第四种情况，比如String::new，等价于()->new String();

```java
//定时器事件：打印事件名称
//lambda表达式如下
Timer t = new Timer(1000, event->System.our.println(event));
//通过方法引用，改写为如下：
Timer t = new Timer(1000,System.out::println);

```

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
