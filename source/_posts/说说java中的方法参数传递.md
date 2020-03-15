---
title: 说说java中的方法参数传递
date: 2019-08-10 15:47:22
tags: Java
categories: 
			- Java
			- 基础知识
---



## indroduction
因为java中参数传递涉及到基本数据类型和对象引用类型，比较容易混淆，所以小结一下说说java中的方法参数传递。

参数传递主要有两种方式（当然还有一种call by name，比较老的参数传递方式，已经out了）：

1. call by value，即按值传递，表示方法接收的是调用者传递的值，形参是实参的一份拷贝。
2. call by reference，即按引用传递，表示方法接收的时调用者提供的变量的地址，形参和实参具有相同的地址。

根据参数传递方式形参和实参的关系，不难理解一个方法可以修改传递引用所对应的变量的值，因为形参和实参具有相同的地址；而不能修改值传递所对应的变量值，因为形参只是实参的拷贝，修改副本不会改变原件。

## conclusion
那java是call by value还是call by reference呢？明确一下结论：

> java的参数传递采用的是call by value，也就是说方法得到的是实参变量的一份拷贝，方法是修改不了实参变量的。但是呢，如果传递的实参是一个可变对象类型呢？我们知道java中任何对象变量的值都是对存储在另外一个地方的对象的引用，这个时候形参是实参的一份拷贝，换句话说形参对象也是实参对象的一份引用，call by value没法通过修改形参对象的引用关系来改变实参对象的引用关系，但是可以通过修改形参对象的状态来修改实参对象的状态的。举例说，传递一个参数Student类对象指向学生a给方法，方法不能让该对象指向另一个学生b，但是可以改变学生a的学好、姓名等属性。

进一步明确结论：

> java的参数传递采用的是call by value方式；
一个方法不能修改一个基本数据类型的参数；
一个方法可以改变一个对象参数的状态；
一个方法不能实现让一个对象参数引用另外一个对象。

## example
### example 1:不能修改基本数据类型的参数
```java
public class Main
{
    public static void main(String[] args)
    {
        int money = 100;
        changeMoney(money);
        System.out.println("After changeMoeny is called, money is: " + money);
    }

    private static void changeMoney(int money)
    {
        money *= 1000;
    }
}
```
结果输出：
```java
After changeMoeny is called, money is: 100
```
### example 2:可以修改一个对象参数的状态
```java
//定义一个Student类
package com;

public class Student
{
    private String idNumber;

    public Student(String idNumber)
    {
        this.idNumber = idNumber;
    }

    public String getIdNumber()
    {
        return idNumber;
    }

    public void setIdNumber(String idNumber)
    {
        this.idNumber = idNumber;
    }

}

//main函数调用
package com;

public class Main
{
    public static void main(String[] args)
    {
        // 初始化詹姆斯是03一代的状元秀
        Student james = new Student("2003001");
        // 初始化科比是96一代的状元秀
        Student kobe = new Student("1996001");

        // 然而我们知道，科比是96黄金一代的13顺位，所以尝试对其ID进行修改
        changeId(kobe, "1996013");

        System.out.println("After changId, kobe's idNumber is: " + kobe.getIdNumber());
    }

    private static void changeId(Student student, String idNumber)
    {
        student.setIdNumber(idNumber);
    }
}
```
结果输出：

```java
//发现科比的顺位ID被成功修改过来了
After changId, kobe's idNumber is: 1996013
```

### example 3：不能改变对象参数的引用关系
```java
package com;

public class Main
{
    public static void main(String[] args)
    {
        // 初始化詹姆斯是03一代的状元秀
        Student james = new Student("2003001");
        // 初始化科比是96一代的状元秀
        Student kobe = new Student("1996001");

        // 然而我们知道，科比是96黄金一代的13顺位，所以尝试对其ID进行修改
        //让kobe指向一个新的Student对象
        changeStudent(kobe, new Student("1996013"));

        System.out.println("After changeStudent, kobe's idNumber is: " + kobe.getIdNumber());
    }

    private static void changeStudent(Student student, Student newStudent)
    {
        student = newStudent;
    }
}
```
结果输出：
```java
//引用关系并未发生改变，仍然输出之前对象的信息
After changeStudent, kobe's idNumber is: 1996001
```
## reference
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
