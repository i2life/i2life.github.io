---
title: 说说java中的代理
date: 2019-09-21 22:31:58
tags: Java
categories: 
			- Java
			- 基础知识
---

代理是java中很重要的一个概念，尤其在学习java框架的时候，像spring的AOP都是基于动态代理实现的，所以今天就说说java中的代理机制。

代理可以分为静态代理和动态代理。

## 静态代理
举个例子，我们知道科比的经纪人是佩林卡，现在是休赛期，腾讯想邀请科比来中国打一场名人赛。那么关于赛制、合同什么的，腾讯肯定是先联系经纪人佩林卡，然后通过佩林卡向科比传达这个信息，那么我们就可以把佩林卡当作是科比的一个代理。

看一下静态代理的代码实现：

```java
//创建一个接口BasketballMatch，里面包含一个方法basketballShowInChina()
package com;

public interface BasketballMatch
{
  void basketballShowInChina();
}
```
球员要具备打比赛的能力，所以定义一个BasketballPlayer类，实现上面的接口，并实现basketballShowInChina方法
```java
package com;

public class BasketballPlayer implements BasketballMatch
{
    private String name;

    public BasketballPlayer(String name)
    {
        this.name = name;
    }

    @Override
    public void basketballShowInChina()
    {
        System.out.println(name + " will play basketball show in Shanghai.");
    }
}
```
经纪人虽然不直接打比赛，但是经纪人有能力找到打比赛的人，所以经纪人相当于间接具备打比赛的能力，定义一个PlayerProxy类，也实现上面的接口，并实现basketballShowInChina方法。代理类里面定义一个接口成员指向一个BasketballPlayer对象，因为经纪人并不能直接打比赛，所以经纪人实现basketballShowInChina方法是通过调用BasketballPlayer对象的basketballShowInChina方法。
```java
package com;

public class PlayerProxy implements BasketballMatch
{
    private BasketballMatch basketballMatch;

    public PlayerProxy(BasketballMatch basketballMatch)
    {
        this.basketballMatch = basketballMatch;
    }

    @Override
    public void basketballShowInChina()
    {
        basketballMatch.basketballShowInChina();
    }
}
```
main函数调用：
```java
package com;
public class Main
{
    public static void main(String[] args)
    {
        BasketballMatch basketballMatch = new BasketballPlayer("kobe");
        PlayerProxy peLinka = new PlayerProxy(basketballMatch);
        peLinka.basketballShowInChina();
    }
}
```
结果输出：
```java
kobe will play basketball show in Shanghai.
```
**总结：**
代理模式包括如下几个角色：

> Subject:抽象主题角色，可以是接口也可以是抽象类，对应上面的BasketballMatch接口；
> 
> RealSubject:真实主题角色，业务逻辑的具体执行者，对应上面的BasketballPlayer类；
> 
> ProxySubject:代理主题角色，内部含有RealSubject的引用，负责对真实角色的调用，并在真实主题角色处理前后做预处理和善后工作，对应上面的PlayerProxy类。

**为什么要引入代理模式？**

> 职责清晰，真实角色只需要关注业务逻辑的实现，非业务逻辑部分通过代理类完成即可；
高扩展性，不管真实角色如何变化，由于接口是固定的，代理类无需做任何改动。


## 动态代理
相对于静态代理，动态代理的代理类并不是提前定义的，也就是说不会提前定义PlayerProxy类，而是动态生成代理类，但是需要构造一个调度处理器来动态生成代理类。

抽象主题角色和真实角色都没有变化，唯一变化的是代理角色以及调用的方式：

```java
package com;

public interface BasketballMatch
{
  void basketballShowInChina();
}

package com;

public class BasketballPlayer implements BasketballMatch
{
    private String name;

    public BasketballPlayer(String name)
    {
        this.name = name;
    }

    @Override
    public void basketballShowInChina()
    {
        System.out.println(name + " will play basketball show in Shanghai.");
    }
}

//定义一个调度处理器，为了生成动态代理类
package com;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class DynamicProxy implements InvocationHandler
{
    private BasketballMatch basketballMatch;

    public DynamicProxy(BasketballMatch basketballMatch)
    {
        this.basketballMatch = basketballMatch;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
    {
        return (Object) method.invoke(basketballMatch, args);
    }
}

//main函数调用
package com;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class Main
{
    public static void main(String[] args)
    {
        BasketballMatch basketballMatch = new BasketballPlayer("kobe");

        InvocationHandler handler = new DynamicProxy(basketballMatch);
        //动态生成代理类
        BasketballMatch currentProxy = (BasketballMatch) Proxy.newProxyInstance(
                basketballMatch.getClass().getClassLoader(), basketballMatch.getClass().getInterfaces(), handler);
        //调用代理类的方法
        currentProxy.basketballShowInChina();
    }
}
```
结果输出：
```java
kobe will play basketball show in Shanghai.
```
## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
