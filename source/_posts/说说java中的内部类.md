---
title: 说说java中的内部类
date: 2019-08-10 16:43:01
tags: Java
categories: Java
---
## 内部类基本概念
内部类即定义在另一个类里面的类。内部类主要可以分为四类：普通内部类(成员内部类)、局部内部类、匿名内部类以及静态内部类。

### 1.普通内部类
普通内部类即一个类定义在另一个类里面，下面通过代码看一下普通内部类的定义以及基本使用方法：
```java
package com;
//外部类
public class School
{
    //外部类成员
    private String name;
    //外部类构造函数
    public School(String name)
    {
        this.name = name;
    }
    //外部类成员方法
    public void studentInfo(Student student)
    {
        System.out.println("the student name is: " + student.name);
    }
    //内部类
    class Student
    {
        //内部类成员
        private String name;
        //内部类构造函数
        public Student(String name)
        {
            this.name = name;
        }
        //内部内成员方法
        public void selfIntroduction()
        {
            System.out.println("My name is " + name + ", I am from " + School.this.name);
        }
    }
}

package com;

public class Main
{
    public static void main(String[] args)
    {
        //创建一个外部类对象
        School school = new School("Nanjing foreign language middle school");
        //new 一个内部类对象的方法为: 外部类实例.new 内部类名
        School.Student student = school.new Student("xiaoming");
        //调用外部类的成员方法
        school.studentInfo(student);
        //调用内部类的成员方法
        student.selfIntroduction();
    }
}
```
结果输出：
```java
the student name is: xiaoming
My name is xiaoming, I am from Nanjing foreign language middle school
```
**成员内部类规则总结：**

1. 内部类可以无条件访问外部类的所有成员属性和成员方法，包括外部类的private、static等成员；如果内部类的成员和外部类的成员同名，优先访问内部类的成员，此时通过如下语法访问外部类的成员`外部类.this.成员`;

2. 外部类访问内部类的成员，需要先创建一个内部类对象，然后通过该内部类对象访问内部类成员；

3. 创建内部类对象，需要先创建一个外部类对象，通过外部类对象创建内部类对象：

    ```java
    //创建一个外部类对象
    School school = new School("Nanjing foreign language middle school");
    //new 一个内部类对象的方法为: 外部类实例.new 内部类名
    School.Student student = school.new Student("xiaoming");
    ```

4. 外部类只能用public和default修饰（default表示包可见，public表示所有可见）。顺便说一下如果用private修饰外部类，其他类都无法访问外部类没有意义，如果用protected修饰外部类，则同包和子孙类可见，如果子孙类和该类同包，则default的权限能满足，如果子孙类与该类不同包，则public权限能覆盖，所以用protected修饰外部类就有点多余了，记住外部类只有两种修饰权限：public和default，包可见和全部可见。而内部类可以为public、protected、default以及private。

### 2.局部内部类
局部内部类即定义在外部类的一个局部作用域的类，比如定义在一个方法里面的类。
```java
package com;

//外部类
public class School
{
    // 外部类成员
    private String name;

    // 外部类构造函数
    public School(String name)
    {
        this.name = name;
    }

    // 外部类成员方法
    public void studentInfo()
    {
        // 内部类
        class Student
        {
            // 内部类成员
            private String name;

            // 内部类构造函数
            public Student(String name)
            {
                this.name = name;
            }

            // 内部内成员方法
            public void selfIntroduction()
            {
                System.out.println("My name is " + name + ", I am from " + School.this.name);
            }
        }
        Student student = new Student("xiaoming");
        student.selfIntroduction();
        System.out.println("the student name is: " + student.name);
    }
}

package com;

public class Main
{
    public static void main(String[] args)
    {
        //创建一个外部类对象
        School school = new School("Nanjing foreign language middle school");
        school.studentInfo();
    }
}
```
结果输出：
```java
My name is xiaoming, I am from Nanjing foreign language middle school
the student name is: xiaoming
```

**局部内部类规则总结：**

1. 局部内部类的作用域限定在局部方法里面，不能用public、protected、default、private修饰局部内部类；
2. 局部内部类不只可以访问外部类的变量，还能访问局部变量，但是只能访问final的局部变量；
3. 局部内部类也没法在作用域外进行实例化，其它同成员内部类规则。

### 3.匿名内部类
将局部内部类再简省以下，只创建一个类的对象，而不对该类进行命名，即为匿名内部类。
```java
//定义一个接口
package com;

public interface Introduction
{
  void selfIntroduction();
}
//外部类
package com;

//外部类
public class School
{
    // 外部类成员
    private String name;

    // 外部类构造函数
    public School(String name)
    {
        this.name = name;
    }

    // 外部类成员方法
    public void studentInfo()
    {
        // 匿名内部类
        Introduction introduction = new Introduction()
        {
            @Override
            public void selfIntroduction()
            {
                System.out.println("I am from " + School.this.name);
            }
        };
        introduction.selfIntroduction();

    }
}
//main函数调用
package com;

public class Main
{
    public static void main(String[] args)
    {
        //创建一个外部类对象
        School school = new School("Nanjing foreign language middle school");
        school.studentInfo();
    }
}
```
结果输出：
```java
I am from Nanjing foreign language middle school
```
**匿名内部类规则总结：**

1. 匿名内部类连类名都没有，所有匿名内部类没有构造函数，匿名内部类也是唯一一种没有构造器的类；

2. 一般来说匿名内部类用来继承其他类或者实现接口，并不需要增加额外的方法，只需要对继承的方法进行实现或者重写，大部分匿名内部类用于接口回调；

3. 匿名内部类和局部内部类只能访问final的局部成员。


### 4.静态内部类

如果内部类不需要引用外部类的对象，则可以将内部类定义为静态内部类，用static关键字修饰内部类即可。
```java
package com;

//外部类
public class School
{
    // 外部类的静态成员
    private static String address = "nanjing road #001";

    // 定义静态内部类
    static class Address
    {
        // 静态内部类的成员
        private String postCode;

        // 静态内部类的构造函数
        public Address(String postCode)
        {
            this.postCode = postCode;
        }

        // 静态内部类成员方法
        public void getFullAddress()
        {
            System.out.println("The full address is " + School.address + ", postCode is " + postCode);
        }
    }

}

//main函数调用
package com;

import com.School.Address;

public class Main
{
    public static void main(String[] args)
    {
        School.Address address = new School.Address("210000");
        address.getFullAddress();
    }
}
```
结果输出：
```java
The full address is nanjing road #001, postCode is 210000
```
**静态内部类规则总结：**

1. 静态内部类不需要依赖外部类的对象；
2. 静态内部类只能访问外部类的静态成员而不能访问其他实例域，很好理解，因为不依赖外部类的实例对象。

### 为什么局部内部类和匿名内部类只能访问final的局部变量呢？
匿名内部类是局部内部类的一种特殊形式，局部内部类可以访问外部类的成员，也可以访问局部变量，这很好理解，因为局部内部类在局部作用域里，但是访问局部变量有一个限制，必须访问声明为final的局部变量，这是为什么呢？

其实在《java核心技术卷一》里，有一段比较详细的解释：

比如在外部类的成员方法start里定义了一个局部内部类：

```java
public void start(int interval, final boolean beep)
   {
       class TimePrinter implements ActionListener
       {
           @Override
           public void actionPerformed(ActionEvent e)
           {
               Date now = new Date();
               System.out.println("At the tone, the time is "+now);
               if(beep)
               {
                   Toolkit.getDefaultToolkit().beep();
               }
           }
       }

       ActionListener listener = new TimePrinter();
       Timer timer = new Timer(interval, listener);
       timer.start();
   }
```
局部内部类TimePrinter使用的是start的参数beep作为调用beep函数的flag判断，beep是一个局部变量，这里声明为final，如果beep是一个普通变量有什么问题吗？

捋一下代码执行流程：

1. 调用start方法；
2. 调用局部内部类TimePrinter的构造函数，创建一个TimePrinter的对象listener；
3. 将listener和interval参数传递给Timer的构造函数，创建一个定时器对象timer；
4. timer.start定时器开始执行，start方法结束，start方法的局部变量beep生命周期结束，beep不复存在了；
5. 定时器到了时间间隔，执行actionPerformed函数方法；
6. 执行到if(beep)语句，似乎beep变量不存在啊，那怎么执行呢……
7. 
其实编译器很聪明的，TimePrinter类在beep被释放之前，会对start方法里的beep变量进行备份。当创建一个TimePrinter对象的时候，beep就会被传给TimePrinter的构造函数进行备份。编辑器会对所有局部变量进行检测，为每一个被访问的局部变量创建备份到构造函数函数。

所以上面的第6步，执行到if(beep)语句，会根据备份的beep变量进行判断是否执行beep方法，从而解决了beep变量与Timer生命周期不一致的问题，但是也带来了新的问题，start方法中的beep变量和actionPerformed方法里的beep变量不是同一个，如果在acitonPerformed里对beep变量做了修改，不就有问题么。。

所以要约束局部类只能访问final的局部变量，这样就不允许对beep变量进行修改，从而使得局部变量和在局部类中建立的对该变量的拷贝保持一致。

### 为什么引入内部类
内部类其实还挺复杂的，费这么大劲引入内部类，自然还是有它的用武之地：

1. 可以将存在一定逻辑关系的类组织在一起，同时对外界隐藏内部类；
2. 当想要定义一个回调函数又不想编写大量代码的时候，使用匿名内部类比较方便；
3. 写事件驱动程序的时候，经常用到内部类。

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
[https://www.cnblogs.com/dolphin0520/p/3811445.html](https://www.cnblogs.com/dolphin0520/p/3811445.html)
