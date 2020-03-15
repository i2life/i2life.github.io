---
title: 说说java中的反射机制
date: 2019-08-10 16:22:31
tags: Java
categories: 
			- Java
			- 基础知识
---
## 反射的基本概念
首先区分两个概念：编译期和运行期。编译期指编译器将源代码编译生成机器码的过程，比如将java文件编译生成.class字节码文件；运行期是对编译后的可执行文件进行执行的过程。反射机制正是工作在运行期。

**在程序运行期间，动态获取对象信息以及动态调用类的属性和方法的能力即为java的反射机制。**

## 反射机制
java在运行时，始终为所有的对象维护一个运行时的类型标识，保存对象的基本信息。该类型标识也是一个类，即Class类。
我们知道根类Object有一个方法getClass，该方法即返回一个Class对象，也就是返回对象的类型标识信息。

### 获取Class对象的三种方式
#### 方法一：类名.Class
```java
//Student是一个类
Class aClass = Student.class;
```
#### 方法二：对象实例.getClass()
```java
Student xiaoming = new Student();
Class bClass = xiaoming.getClass();
```
#### 方法三：Class.forName()
```java
//参数为带包路径的字符串
Class cClass = Class.forName("com.Student");
```
### Class类的API方法
类的三大属性：Field、Method、Constructor分别描述类的域、方法以及构造器。

对应的API(更多API参照API文档):
```java
getFields();返回类的public域(包含父类的public成员)
getDeclaredFields();返回类的全部域（不包含父类的成员）

getMethods()；返回类的public方法(包含父类的public方法)
getDeclaredMethods()；返回类的全部方法（不包含父类的方法）

getConstructors()；返回类的public构造器(包含父类的public构造器)
getDeclaredConstructors()；返回类的全部构造器（不包含父类的构造器）

Modifier.isPublic(field.getModifiers())判断一个域是否为public，private以及protected类似判断。
```

### 访问类的私有成员
```java
//Student类有一个私有成员idNumber
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
//通过反射访问打印出来idNumber
package com;
import java.lang.reflect.Field;
public class Main
{
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        Student xiaoming = new Student("2008001");
        Class aClass = xiaoming.getClass();
        //获取idNumber私有成员
        Field field = aClass.getDeclaredField("idNumber");
        //设置访问权限为可见
        field.setAccessible(true);
        //获取该成员的当前状态
        Object object = field.get(xiaoming);
        System.out.println("print studnet's idNumber by reflect: "+object);
    }
}
```
结果输出：
```java
print studnet's idNumber by reflect: 2008001
```
### 访问类的私有方法
```java
//Student类添加一个私有方法
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

    // 定义一个私有方法，更新学号，学号前面统一加上111
    private void updateIdNumber(String idNumber)
    {
        System.out.println("The new idNumber is: " + "111" + idNumber);
    }
}
//通过反射访问该私有方法
package com;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main
{
    public static void main(String[] args)
            throws NoSuchFieldException, IllegalAccessException, NoSuchMethodException, InvocationTargetException
    {
        Student xiaoming = new Student("2008001");
        Class aClass = xiaoming.getClass();
        //获取该私有方法，参数为方法名，和参数类型
        Method method = aClass.getDeclaredMethod("updateIdNumber",String.class);
        //设置访问权限
        method.setAccessible(true);
        //invoke访问
        method.invoke(xiaoming, xiaoming.getIdNumber());
    }
}
```
结果输出：
```java
The new idNumber is: 1112008001
```
## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
[http://blog.51cto.com/4247649/2109128](http://blog.51cto.com/4247649/2109128)
