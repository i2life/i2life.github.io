---
title: 说说java中的clone以及浅拷贝和深拷贝
date: 2019-08-10 16:31:37
tags: Java
categories: 
			- Java
			- 基础知识
---
## java中的clone方法
超类Object有一个protected的方法clone()，该方法返回Object对象的一个拷贝。我们知道所有的类都继承自Object，也就是所有的类都有clone方法，是不是直接调用clone()方法就可以对一个对象进行克隆呢？
然而并不是，在《java核心技术》里有这么一句话：”子类只能调用受保护的clone方法克隆它自己”，啥意思，这句话琢磨了半天，clone方法是protected的没问题啊，protected的访问范围包括同一个包的其他类和该类的子类，所有的类都是Object类的子类，所以所有的子类可以正常访问clone方法，没什么问题啊，我们写一个具体的例子看一看：

```java
//定义一个Boy类
public class Boy
{
    private String name;
    private int age;

    public Boy(String name, int age)
    {
        this.name = name;
        this.age = age;
    }

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
//main函数调用
package com;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
public class Main
{
    public static void main(String[] args)
    {
        Boy kobe = new Boy("kobe",25);
        Boy bryant = (Boy)kobe.clone();
    }
}
```
我们发现报了编译错误：
```java
Error:(12, 31) java: clone() 在 java.lang.Object 中是 protected 访问控制
```
似乎有点明白”子类只能调用受保护的clone方法克隆它自己”这句话了，要克隆Boy对象，那么只能在Boy类里面调用clone方法，在Main类里调用clone方法只能克隆Main对象。

那么对代码做一下修改，在Boy类里面调用clone方法：
```java
package com;
public class Boy
{
    private String name;
    private int age;

    public Boy(String name, int age)
    {
        this.name = name;
        this.age = age;
    }

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

    //定义一个public方法，调用clone方法对boy对象进行克隆
    public Boy copyOfBoy() throws CloneNotSupportedException {
      return (Boy)this.clone();
    }
}

//main函数里调用copyOfBoy是否能对Boy对象进行克隆呢
package com;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
public class Main
{
    public static void main(String[] args) throws CloneNotSupportedException {
        Boy kobe = new Boy("kobe",25);
        Boy bryant = (Boy)kobe.copyOfBoy();
        System.out.println("bryant's name is: "+bryant.getName());
    }
}
```
我们觉得这样应该是可行的，但是抛了异常：
```java
Exception in thread "main" java.lang.CloneNotSupportedException: com.Boy
	at java.base/java.lang.Object.clone(Native Method)
	at com.Boy.copyOfBoy(Boy.java:36)
	at com.Main.main(Main.java:11)
```
CloneNotSuported，这个是由于java的克隆机制决定的，因为Boy类没有实现接口Cloneable，java规定要想对一个类对象进行克隆，这个类必须要实现Cloeable接口。关于这个约束，似乎没少被喷，引用自某乎”这是java的一个设计缺陷，是个很糟糕的设计，如果java在今天被重新设计的话，多半就不会设计成这个样子了”。

但是既然有这么个机制约束，我们还是得弄清楚，为什么会抛这个异常。java对象要支持clone功能，但是不是所有的类都可以克隆，所有java就想让用户自己来标记哪些类可以克隆，如果这个类需要克隆，就给这个类一个标记，但是并没有像final这些关键字一样提供一个关键字来修饰类的克隆属性，java是通过让类实现一个接口的方式进行标记该类是否可被克隆，也就是Cloneable接口，像Cloneable这种接口就是一种标记接口。顺便说两句标记接口：java中把没有定义任何方法和常量的接口称为标记接口，也就是说这种接口不像正常的接口那样是用来描述类的某种功能的，只是单纯的对类进行标记，常用的三个标记接口为RandomAccess、Cloneable、Serializable。

那么我们基本上就知道了java进行clone的一个过程：JVM首先检查要clone的对象的类有没有打上Cloneable的标记，如果有这个标记，就进行clone，如果没有这个标记则抛异常，所有上面抛出的异常也就不难理解了。

那么对Boy类实现Cloneable接口即可：
```java
package com;

public class Boy implements Cloneable
{
    private String name;
    private int age;

    public Boy(String name, int age)
    {
        this.name = name;
        this.age = age;
    }

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

    //定义一个public方法，调用clone方法对boy对象进行克隆
    public Boy copyOfBoy() throws CloneNotSupportedException {
      return (Boy)this.clone();
    }
}

package com;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main
{
    public static void main(String[] args) throws CloneNotSupportedException {
        Boy kobe = new Boy("kobe",25);
        Boy bryant = (Boy)kobe.copyOfBoy();
        System.out.println("kobe's name is: "+kobe.getName());
        System.out.println("bryant's name is: "+bryant.getName());
        //对克隆对象进行修改
        bryant.setName("bryant");
        System.out.println("kobe's name is: "+kobe.getName());
        System.out.println("bryant's name is: "+bryant.getName());
    }
}
```
结果输出：
```java
kobe's name is: kobe
bryant's name is: kobe
kobe's name is: kobe
bryant's name is: bryant
```
我们发现，成功地对Boy对象进行了拷贝，并且改变bryant的成员，kobe的成员并没有受到影响（这就是跟引用的区别了，如果直接让bryant=kobe，看似是拷贝，但是改变bryant的成员，kobe的成员也会同步被修改）。

但是呢，这里我们调用的方法是copyOfBoy，名字太别扭了，既然是clone，那么如果调用kobe.clone()方法不是更合适么，把方法名copyOfBoy替换成clone即可，其实就是对父类的clone方法进行了重写，注意一下，copyOfBoy里调用的是this.clone()，如果将copyOfBoy替换成clone，this.clone()需要替换成super.clone()，表示调用的是父类的clone方法，不然就形成递归了，会导致栈溢出。

**总结一下，对象克隆的步骤：**

> step1:implements Cloneable接口；
step2:重写clone方法为public。

## 浅拷贝和深拷贝
上面我们已经很清楚的实现了对象clone，改变bryant的name，kobe的成员没有受到影响，但是呢，如果Boy包含可变对象的成员变量呢，这个时候clone对象修改该可变成员，原对象的成员是否受到影响呢？我们看一看具体的代码：

```java
//定义一个coach类
package com;

public class Coach
{
    private String name;

    public Coach(String name)
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
//Boy新增一个coach成员
package com;

public class Boy implements Cloneable
{
    private String name;
    private int age;
    private Coach coach;

    public Boy(String name, int age)
    {
        this.name = name;
        this.age = age;
    }

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

    //定义一个public方法，调用clone方法对boy对象进行克隆
    public Boy clone() throws CloneNotSupportedException {
      return (Boy)super.clone();
    }

  public Coach getCoach() {
    return coach;
  }

  public void setCoach(Coach coach) {
    this.coach = coach;
  }
}
//对复制后的对象的coach成员进行修改，原对象的coach成员是否受影响
package com;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main
{
    public static void main(String[] args) throws CloneNotSupportedException {
        Boy kobe = new Boy("kobe",25);
        Coach coach = new Coach("Jakson");
        kobe.setCoach(coach);
        Boy bryant = (Boy)kobe.clone();

        System.out.println("kobe's coach name is: "+kobe.getCoach().getName());
        System.out.println("bryant's coach name is: "+bryant.getCoach().getName());

        //对bryant的coach成员进行修改
        bryant.getCoach().setName("Warton");

        System.out.println("kobe's coach name is: "+kobe.getCoach().getName());
        System.out.println("bryant's coach name is: "+bryant.getCoach().getName());

    }
}
```
结果输出：
```java
kobe's coach name is: Jakson
bryant's coach name is: Jakson
kobe's coach name is: Warton
bryant's coach name is: Warton
```
我们发现改变bryant的coach的名字，kobe的coach的名字也被同步修改了，这说明clone方法默认是浅拷贝。

为了实现深拷贝，需要对每一个可变对象成员对象均进行clone：
```java
package com;

public class Coach implements Cloneable
{
    private String name;

    public Coach(String name)
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

    public Coach clone() throws CloneNotSupportedException
    {
        return (Coach) super.clone();
    }
}

package com;

public class Boy implements Cloneable
{
    private String name;
    private int age;
    private Coach coach;

    public Boy(String name, int age)
    {
        this.name = name;
        this.age = age;
    }

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

    //重写clone方法,对coach成员也进行clone
    public Boy clone() throws CloneNotSupportedException {
      Boy result = (Boy)super.clone();
      result.coach = (Coach)this.coach.clone();
      return result;
    }

  public Coach getCoach() {
    return coach;
  }

  public void setCoach(Coach coach) {
    this.coach = coach;
  }
}


package com;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main
{
    public static void main(String[] args) throws CloneNotSupportedException {
        Boy kobe = new Boy("kobe",25);
        Coach coach = new Coach("Jakson");
        kobe.setCoach(coach);
        Boy bryant = (Boy)kobe.clone();

        System.out.println("kobe's coach name is: "+kobe.getCoach().getName());
        System.out.println("bryant's coach name is: "+bryant.getCoach().getName());

        //对bryant的coach成员进行修改
        bryant.getCoach().setName("Warton");

        System.out.println("kobe's coach name is: "+kobe.getCoach().getName());
        System.out.println("bryant's coach name is: "+bryant.getCoach().getName());

    }
}
```
结果输出：
```java
kobe's coach name is: Jakson
bryant's coach name is: Jakson
kobe's coach name is: Jakson
bryant's coach name is: Warton
```
我们发现改变bryant的coach成员，kobe的coach成员并没有受到影响，至此已经实现了深拷贝。但是假如一个对象有好多可变对象成员，要实现深拷贝，岂不是要对每一个成员均进行clone，不是很麻烦吗？是的，需要对每一个成员都要clone，java还有另外一种方式实现深拷贝，Serializable序列化，关于这个下回再说。

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)