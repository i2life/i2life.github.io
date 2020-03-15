---
title: 说说java中的泛型(一)
date: 2019-09-22 12:16:20
tags: Java
categories: 
			- Java
			- 基础知识
---
## 泛型概念
### 什么是泛型？
泛型的本质就是参数化类型。我们知道定义一个变量的时候会为其指定一个类型，比如定义Student类的对象Student xiaoming，Student即为变量xiaoming的类型，现在我们把这个具体的类型参数化，也就是说用参数变量来代替xiaoming的类型，这就是参数化类型。

当参数化类型被应用在类、接口、方法中的时候，对应的即为泛型类、泛型接口、泛型方法。

### 为什么要引入泛型？
不难理解，泛型编程可以使得编写的代码能被不同的类型对象所重用，只要传递不同的类型参数实参给类型参数就可以实现代码重用。

比如我们想定义一个集合分别存储String对象和Student对象，我们可不想设计一个StringArray来存String对象，再另外设计一个StudentArray来存Student对象。

我们知道ArrayList类就可以解决这个问题：

```java
public class Main
{
    public static void main(String[] args)
    {
        ArrayList arrayList = new ArrayList<>();
        Student xiaoming = new Student();
        arrayList.add("hello");
        arrayList.add(xiaoming);

        String hello = (String)arrayList.get(0);
        String world = (String)arrayList.get(1);
    }
}
```
我们可以向ArrayList对象存入String对象和Student对象，但是有两个问题：

> 1. arrayList.get()获取元素的时候，需要进行强制类型转换，否则会有编译问题；
2. 我们看到String world = (String)arrayList.get(1);并没有报编译错误，但是在运行的时候，会报类型转换异常。

```java
Exception in thread "main" java.lang.ClassCastException: com.Student cannot be cast to java.base/java.lang.String
	at com.Main.main(Main.java:18)
    ```
之所以有这两个问题，是因为不知道arrayList存入的元素类型，为了在编译阶段就解决这些问题，引入泛型，给ArrayList传入类型参数：
```java
ArrayList<String> arrayList = new ArrayList <>();
```

指定了类型参数，那么arrayList.add(xiaoming)就不会报编译错误；而且get方法也不需要进行强制类型转换了。

## 泛型类
将类的成员变量类型参数化，即为泛型类。

定义一个普通类Result

```java
package com;

public class Result
{
    private String value;

    public String getValue()
    {
        return value;
    }

    public void setValue(String value)
    {
        this.value = value;
    }
}
```
有一个String类型的成员变量，我们将其参数化，即得到泛型类：
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
复用泛型类：

```java
package com;

public class Main
{
    public static void main(String[] args)
    {
        Result<String> stringResult = new Result<>();
        Result<Integer> integerResult = new Result<>();
        stringResult.setValue("hello");
        integerResult.setValue(1);

        System.out.println(stringResult.getValue());
        System.out.println(integerResult.getValue());
    }
}
```
这样就实现了对泛型类的复用，否则就得定义一个类处理String成员，再定义另外一个类来处理Integer成员，这里引入了类型变量T，用<>括起来放在类名后面，泛型类也可以有多个类型变量，比如public class Result<T,U>{...}。java中用E表示集合的元素类型，用K和V表示map中的key和value的类型，用T以及临近的U和S表示任意类型。

## 泛型接口
泛型接口和泛型类比较类似，将成员或方法类型参数化，看一个泛型接口的例子：

```java
package com;

public interface Compute<T>
{
    T add(T a, T b);
}
```
定义一个类实现该接口：

```java
package com;

public class Student implements Compute<Integer>
{
    @Override
    public Integer add(Integer a, Integer b)
    {
        return a + b;
    }
}

//main函数调用
package com;

public class Main
{
    public static void main(String[] args)
    {
        Student xiaoming = new Student();
        System.out.println(xiaoming.add(10, 20));
    }
}
```
## 泛型方法
将方法的形参以及返回值参数类型化，即得到泛型方法：

```java
public static <T> T getMiddle(T[] num)
{
   return num[num.length / 2];
}
```
这样只要是想获取数组中间元素，给getMiddle方法，传入该数组即可：

```java
package com;

public class Main
{
    public static void main(String[] args)
    {
        String[] names =
        { "james", "kobe", "mcGrady" };
        Integer[] numbers =
        { 1, 2, 3, 4, 5 };
        System.out.println(getMiddle(names));
        System.out.println(getMiddle(numbers));
    }

    public static <T> T getMiddle(T[] num)
    {
        return num[num.length / 2];
    }
}
```
结果输出：

```java
kobe
3
```
泛型方法可以定义在普通类中，也可以定义在泛型类中。

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)