---
title: 说说java中的泛型(二)
date: 2019-09-22 12:24:01
tags: Java
categories: 
			- Java
			- 基础知识
---

## 类型变量的限定
上一篇中，我们知道了泛型类、泛型接口、泛型的方法的定义和使用。

现在我们定义一个泛型方法，找到两个数中的最小值:

```java
public static <T> T getMin(T num1, T num2)
{
   return num1.compareTo(num2) < 0 ? num1 : num2;
}
```
如果直接这样定义，编译器会报错,无法解析compareTo方法。

因为num1类型是一个泛型T，编译器不能确定该对象是否实现了compareTo方法，那么怎么来保证T类型的变量一定实现了compareTo方法呢？通过类型限定：

```java
public static <T extends Comparable> T getMin(T num1, T num2)
{
   return num1.compareTo(num2) < 0 ? num1 : num2;
}
```
T extends Comparale限定T必须是实现了Comparable接口的类型。

一个类型变量可以有多个限定，基本语法如下：

```java
//限定类型用&分隔，类型变量用，分隔
<T extends Type1 & Type2, U extends Type3>
```

由于TypeT可根据java的继承规则，可以是类也可以是接口，所以这里用关键字extends，表示T是Type的字类型。

**根据java的继承规则，限定列表中可以有多个接口类型，但是最多只能有一个类，如果一个类作为限定，则类必须是限定列表的第一个。**

## 类型擦除
通过上面的一些例子，我们能看到泛型的用处，但是，泛型特性并不是java一开始就有的特性，而是在java1.5才引入的，那么为了跟引入泛型特性之前的代码保持兼容，就必须的做些事情。

因为java1.5前的代码并没有泛型的概念，所以java泛型只能用于在编译期间的静态类型检查，通过编译生成的代码要进行类型擦除，相当于擦掉泛型的痕迹，这样到了运行期JVM就感知不到泛型的存在了，从而能够跟以前的代码保持兼容。

简单理解就是，为了跟以前的代码保持兼容，java虚拟机没有泛型类型对象，所以无论何时定义一个泛型类型，通过编译器后都要进行类型擦除。

**类型擦除的规则：**

> 擦除类型变量，并替换为限定类型列表里的第一个限定类型，如果没有给出限定类型，则用Object替换。

### example 1:有限定列表的类型擦除
```java
public static <T extends Comparable> T getMin(T num1, T num2)
{
    return num1.compareTo(num2) < 0 ? num1 : num2;
}
```
类型擦除后：
```java
public static Comparable getMin(Comparable num1, Comparable num2)
{
    return num1.compareTo(num2) < 0 ? num1 : num2;
}
```
### example 2:无限定列表的类型擦除
```java
package com;

public class Result<T>
{
    private T value;

    public T getValue()
    {
        return value;
    }

    public void setValue(T value)
    {
        this.value = value;
    }
}
```
类型擦除后：
```java
package com;

public class Result
{
    private Object value;

    public Object getValue()
    {
        return value;
    }

    public void setValue(Object value)
    {
        this.value = value;
    }
}
```
### 类型擦除跟多态的冲突
类型擦除带来一个问题，会跟多态冲突，我们看一个导致冲突的例子：

定义一个泛型类Person，有一个contactPerson成员，由于联系人可能是一个字符串表示电话号码，也可能是一个Father对象表示Father作为联系人的相关信息，也可能是一个Mother对象等，所以这里定义contactPerson类型为泛型。

```java
package com;

public abstract class Person<T>
{
    private T contactPerson;

    public T getContactPerson()
    {
        return contactPerson;
    }

    public void setContactPerson(T contactPerson)
    {
        this.contactPerson = contactPerson;
    }
}
```
定义一个学生类继承Person类，我们在学生类里覆盖setContactPerson方法，我们认为如果是学生，就把联系人定位老师，在setContactPerson方法里添加是”contact person is teacher“的描述：

```java
package com;

public class Student extends Person<String>
{
    public void setContactPerson(String contactPerson)
    {
        super.setContactPerson("contact person is teacher " + contactPerson);
    }
}
```
在main函数如下调用：

```java
package com;

public class Main
{
    public static void main(String[] args)
    {
        Student xiaoming = new Student();
        Person<String> p1 = xiaoming;
        p1.setContactPerson("025-11111111");
    }
}
```
Person<String> p1 = xiaoming;这里将父类的变量指向子类的引用，子类对父类的方法进行重写，这其实就是多态，p1.setContactPerson("025-11111111");调用的是子类对象的方法。

类型擦除之后：
父类Person

```java
package com;

public abstract class Person
{
    private Object contactPerson;

    public Object getContactPerson()
    {
        return contactPerson;
    }

    public void setContactPerson(Object contactPerson)
    {
        this.contactPerson = contactPerson;
    }
}
```
子类Student

```java
package com;

public class Student extends Person
{
    public void setContactPerson(String contactPerson)
    {
        super.setContactPerson("contact person is teacher " + contactPerson);
    }
}
```
我们看到经过类型擦除，父类的方法是public void setContactPerson(Object contactPerson),子类的方法是public void setContactPerson(String contactPerson)，子类方法和父类方法只是函数名相同，但是方法签名里的参数类型不一样了，这时候按照传统的理解是函数重载，而不是重写了，我们本意是实现多态，我们发现类型擦除导致了跟多态的冲突。

那么我们看一下我们上面的分析是不是对的，进行如下的test：

```java
package com;

public class Main
{
    public static void main(String[] args)
    {
        Student xiaoming = new Student();
        Person p1 = xiaoming;
        p1.setContactPerson(new Object());
    }
}
```
传入Obejct类型的参数，这个时候如果是重载的话，就会调用父类的setContactPerson方法，但是出现了运行异常：

```java
Exception in thread "main" java.lang.ClassCastException: java.base/java.lang.Object cannot be cast to java.base/java.lang.String
	at com.Student.setContactPerson(Student.java:3)
	at com.Main.main(Main.java:9)
```
无法将Object类型转换为String类型，而且报错的位置是Student的方法，这说明并没有发生重载，仍然是重写，调用的仍然是子类的方法，那这是为什么呢？

这是因为JVM自动采用了一种**桥方法**的办法来解决类型擦除跟多态的冲突。

对Student.class文件反编译一下:javap -c Student.class，得到如下结果：

```java
Compiled from "Student.java"
public class com.Student extends com.Person<java.lang.String> {
  public com.Student();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method com/Person."<init>":()V
       4: return

  public void setContactPerson(java.lang.String);
    Code:
       0: aload_0
       1: aload_1
       2: invokedynamic #2,  0              // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;)Ljava/lang/String;
       7: invokespecial #3                  // Method com/Person.setContactPerson:(Ljava/lang/Object;)V
      10: return

  public void setContactPerson(java.lang.Object);
    Code:
       0: aload_0
       1: aload_1
       2: checkcast     #4                  // class java/lang/String
       5: invokevirtual #5                  // Method setContactPerson:(Ljava/lang/String;)V
       8: return
}
```
我们看到Student类反编译后里面有两个方法：

```java
7: invokespecial #3                  // Method com/Person.setContactPerson:(Ljava/lang/Object;)V
5: invokevirtual #5                  // Method setContactPerson:(Ljava/lang/String;)V
```
并且可以看出这两个方法的关系是:7: invokespecial #3 // Method com/Person.setContactPerson:(Ljava/lang/Object;)V调用5: invokevirtual #5 // Method setContactPerson:(Ljava/lang/String;)V。

也就是说JVM自动生成了一个额外的桥方法，也就是类型擦除后，Student类的真实面目是这样的：

```java
package com;

public class Student extends Person
{
    public void setContactPerson(String contactPerson)
    {
        super.setContactPerson("contact person is teacher " + contactPerson);
    }
    
    public void setContactPerson(Object contactPerson)
    {
    	setContactPerson((String)contactPerson);
    }
}
```
这样子类和父类都拥有public void setContactPerson(Object contactPerson)方法，即使传入的参数类型是Object对象，调用的也是子类方法，只不过子类方法再调用了一次public void setContactPerson(String contactPerson)，通过上面的分析，可以看到JVM就是通过桥方法解决类型擦除跟多态之前的冲突的。

进一步考虑一下，setContactPerson是有参数列表的，如果子类Student还重写了父类的getContactPerson()方法呢，这个时候类型擦除了之后，父类的getContactPerson为：

```java
public Object getContactPerson()
{
	return contactPerson;
}
```
子类的getContactPerson方法为：

public String getContactPerson()
{
	retrn super.getContactPerson();
}
只有返回值不一样，这其实就是正常的重写，按道理不需要桥方法过渡了，我们看一下编译后的代码：

```java
Compiled from "Student.java"
public class com.Student extends com.Person<java.lang.String> {
  public com.Student();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method com/Person."<init>":()V
       4: return

  public java.lang.String getContactPerson();
    Code:
       0: aload_0
       1: invokespecial #2                  // Method com/Person.getContactPerson:()Ljava/lang/Object;
       4: checkcast     #3                  // class java/lang/String
       7: areturn

  public java.lang.Object getContactPerson();
    Code:
       0: aload_0
       1: invokevirtual #4                  // Method getContactPerson:()Ljava/lang/String;
       4: areturn
}
```
我们看到JVM仍然为getContactPerson生成了一个桥方法：public java.lang.Object getContactPerson();，那这个时候Student类进行类型擦除后，有两个getContactPerson()方法，它们的区别仅仅是返回值类型不一样，按道理这编译都会报错的，会报方法冲突的。
**
这是因为在虚拟机中用参数列表和返回值类型一起来确定一个方法，所以不会有方法冲突。所以这里跟编译器只通过参数列表来确定一个方法是不同的。**

**关于类型擦除的几点总结：**

> 1. 虚拟机中没有泛型，只有普通类和方法；
> 2. 所有类型参数都用限定类型或者Object替换；
> 3. JVM通过桥方法来保持多态；
> 4. 为保持类型安全性，必要时插入强制类型转换。

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)