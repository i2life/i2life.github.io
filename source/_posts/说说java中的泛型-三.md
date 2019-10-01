---
title: 说说java中的泛型(三)
date: 2019-09-22 12:32:07
tags: Java
categories: Java
---

## 泛型中的继承规则
我们定义这样一个泛型类：

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
那么`Result<Person>`和`Result<Student>`有什么关系呢，其中`Student`是`Person`类的子类。

答案是他们没有关系，`Result<Student>`并不是`Result<Person>`的子类，通常无论S和T有什么关系，`Result<S>`和`Result<T>`都没有什么关系。

泛型类可以扩展或实现其他泛型类或泛型接口，这跟普通的类和接口没有区别。

## 通配符类型
### 通配符的子类型限定

通过泛型类型的继承规则，我们知道，`Result<Student>`和`Result<Person>`并没有什么关系，换言之他们是不同的类型，下面我们想定义一个printName方法：

```java
//首先定义Student类和Person类
package com;

public class Person
{
    private String name;

    public Person(String name)
    {
        this.name = name;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }
}

package com;

public class Student extends Person
{
    public Student(String name)
    {
        super(name);
    }
}
```

定义printName函数：

```java
package com;

public class Main
{
    public static void main(String[] args)
    {
        Person xiaoming = new Person("xiaoming");

        Student xiaoqiang = new Student("xiaoqiang");

        Result<Person> input1 = new Result<>();
        input1.setValue(xiaoming);

        Result<Student> input2 = new Result<>();
        input2.setValue(xiaoqiang);

        printName(input1);

        printName(input2);//报编译错误
    }

    public static void printName(Result<Person> input)
    {
        System.out.println(input.getValue().getName());
    }
}
```
我们发现printName(input2);语句报编译错误，如果我们想打印Result<Student>类型入参的姓名，得再定义一个printName方法，参数类型为Result<Student>。

这样就显得很冗余，那么可以考虑使用通配符来解决：

```java
public static void printName(Result<? extends Person> input)
{
   System.out.println(input.getValue().getName());
}
```

`? extends Person`表示`Person`的所有子类。

### 通配符的超类型限定
通配符除了可以指定子类型限定，还可以指定超类型限定，语法如下：
```java
? super Student
```
表示Student类的所有超类型。

## 泛型类型的约束与局限
最后总结一下泛型的约束与局限性，大多数约束都是由于类型擦除导致的。

> 1. 不能使用基本类型实例化类型参数。
因此没有Result<double>，只有Result<Double>，这是因为类型擦除导致的，擦除之后Result类的成员为Object类型，而Object不能存储double这样的基本类型。

> 2. 运行时类型查询只适用原始类型。
因为虚拟机中的对象总有一个特定的非泛型类型，所以所有的类型查询只产生原始类型。
比如if(a instanceof Pair<String>，实际上等价于if(a instanceof Pair<T>。

> 3. 不能抛出也不能捕获泛型类异常。
事实上，泛型类扩展Throwable都不合法：public class Problem<T> extends Exception{...}不能通过编译。

> 4. 参数化类型的数组不合法
Pair<String>[] table = new pair<String>[10]是错误的。

> 5. 不能实例化类型变量。
不能使用像new T();T.class之类的表达式。

> 6. 泛型类的静态上下文中类型变量无效
```java
public class Singleton<T>
{
	public static T getSingleInstance()//Error
    {...}
}
```


## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
