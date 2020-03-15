---
title: 说说java中的接口和抽象类
date: 2019-08-10 16:14:34
tags: Java
categories: 
			- Java
			- 基础知识
---

在java中，抽象类和interface是面向对象编程的两个比较重要的概念，抽象类和interface都是为了对对象进行更好的抽象。二者有一些相似的地方，但也有很多不同之处，所以总结一下java的接口和抽象类。

## 抽象类
抽象类的概念的提出，其实是为了对对象进行一个更高层次的抽象。比如学生和老师都可以继承自Person这么一个类，在Person类里可以定义一些通用属性，比如姓名年龄等，这些都比较好定义，在Perosn类里还可以顶一个通用方法thisIsMyRole(),如果是学生则输出身份信息是学生，如果是老师，则输出身份信息是老师，那么这个方法在Person类里就不好定义了，因为在Person类还不知道具体是什么角色，应该输出什么信息。

所以java提出了抽象方法的概念：通过关键字abstract进行修饰，只有方法声明而没有具体实现。基本格式为abstract void fucntionName();。

抽象方法只是充当着占位的角色，具体实现在子类实现。

java规定，包含一个或多个抽象方法的类必须声明为抽象类，因为抽象类包含有未具体实现的方法，所以抽象类不能实例化，抽象类存在的意义就是被继承。

一般我们理解包含抽象方法的类为抽象类，实际上只要是被abstract修饰的类都是抽象类，即使该类不包含抽象方法，所以不包含抽象方法的类也可以是抽象类。

除了抽象方法以外，抽象类还可以包含实例域和实例方法。

### example

```java
//定义一个抽象类Person，包含一个抽象方法thisIsMyRole()
package com;

public abstract class Person
{

    private String name;
    private int age;

    public abstract void thisIsMyRole();

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public int getAge()
    {
        return age;
    }

    public void setAge(int age)
    {
        this.age = age;
    }
}
//定义Student类和Teacher类均继承自Person，分别实现thisIsMyRole方法
package com;

public class Teacher extends Person
{
    public Teacher(String name, int age)
    {
        super.setName(name);
        super.setAge(age);
    }

    public void thisIsMyRole()
    {
        System.out.println("the role of " + super.getName() + " is teacher.");
    }

}

package com;

public class Student extends Person
{
    public Student(String name, int age)
    {
        super.setName(name);
        super.setAge(age);
    }

    public void thisIsMyRole()
    {
        System.out.println("the role of " + super.getName() + " is student.");
    }

}

//main函数调用
package com;

public class Main
{
    public static void main(String[] args)
    {
        Teacher mrZhang = new Teacher("MrZhang", 30);
        Student xiaoming = new Student("xiaoming", 15);

        mrZhang.thisIsMyRole();
        xiaoming.thisIsMyRole();
    }
}
```
结果输出：
```java
the role of MrZhang is teacher.
the role of xiaoming is student.
```
关于抽象类总结
> 
1.一般，对于一个父类，如果它的某个方法在父类中不好实现，或者说实现没有什么具体的意义，必须根据子类的实际情况进行不同的实现，则可以将这个方法定义为abstract方法，包含该方法的类定义为abstarct类；
2.抽象方法必须为public或者protected（如果为默认或者private，则不能被子类继承）；
3.抽象类不能被实例化；
4.如果子类继承一个抽象类，则子类要实现父类的抽象方法，如果子类没有实现父类的抽象方法，则子类也必须声明为抽象类。

## 接口
interface技术主要用来描述类具有什么功能，但是不给出每个功能的具体实现。interface可以理解成对对象行为（或对象功能）的一个抽象。

基本格式：

```java
public interface interfaceName
{
	//接口中只能包含常量，默认均为public static final
    double PI = 3.14;
    
    //接口中只能包含abstract方法，默认均为public abstract方法
    void function1();
}

//一个类可以实现多个接口
public className implements interface1,interface2
{
	//需要实现接口的每一个方法
}
```
## 抽象类和接口的区别
### 语法层面的区别
> 1.抽象类可以包含具体的方法实现，接口中的方法均为public abstract；
2.抽象类可以博阿含不同类型的成员，接口中的成员只能是public static final；
3.抽象类可以包含实例域或静态方法，接口中不能包含实例域或静态方法；
4.抽象类只能单一继承，接口可以多继承。

### 设计层面的区别
> 1.抽象类是对对象的抽象，是is-A的关系，子类是父类的一种；而接口是对行为的抽象，是has-A的关系，一个类实现了一个接口，可以认为该类具有这个接口的能力。比如Student和Teacher都可以继承自Person，学生和老师都是Person这种理解没有问题，打篮球是一种能力，可以把打篮球封装成一个接口BasketballLever,一个Student或Teacher对象实现了这个接口，说明该学生或者老师具有一定篮球水平的能力。

> 2.抽象类是一种模板式设计，比如要新增一个共有属性，在Person类里面添加一个属性就可以；而接口是一种辐射式设计，比如在BasketballLever接口里新增一个方法灌篮dunk，那么所有实现了BasketballLever接口的类都要进行相应的改动，这些类都要对dunk方法进行实现。

## 参考资料
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
[http://www.cnblogs.com/dolphin0520/p/3811437.html](http://www.cnblogs.com/dolphin0520/p/3811437.html)