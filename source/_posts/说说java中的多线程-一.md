---
title: 说说java中的多线程(一)
date: 2019-09-22 16:40:15
tags: Java
categories: Java
---
## 多线程的基本概念
今天周末，难得清闲，靠着窗户，打开电脑，一边看文档，一边听音乐，还可以开个迅雷下个小电影，惬意！

电脑之所以能同时做这么多事，其实是多进程的功劳，adobe进程负责打开pdf文档，网易云音乐进程负责播放音乐，迅雷进程负责下载电影，各个进程互不干扰。

有个算法好像要参考另外一篇文档，于是我们打开另外一个pdf，这个时候我们并不需要关掉原来的pdf，这是因为每个进程又引入了多线程技术。

当然你可能会问，新打开一个pdf为什么不是重新起了一个adobe进程呢，这个好办，打开计算机的任务管理器看一下当前系统的进程，你会发现只有一个adobeReader进程。

再举个例子，你想下载一个文件，那么在迅雷再创建一个下载任务就可以，一个迅雷进程执行了多个下载任务，这也是多线程。

**进程**：进程是操作系统进行资源分配的基本单位，每个进程拥有独立的内存地址空间，并且只能使用它自己的内存空间，各个进程间互不干扰，进程让操作系统的并发成为可能。

**线程**：线程是操作系统进行调度的基本单位，一个进程虽然包括多个线程，但是这些线程是共同享有进程占有的资源和空间地址的，线程让进程的并发成为可能。

由于线程共享进程的资源和内存空间，那么多个线程如果同时访问某个共享资源怎么办呢，所以引发了进程间的同步问题，这也是多线程编程的难点。然而，也正因为线程是资源共享的，所以线程间通信更容易、线程切换更高效，线程也就比进程更加轻量化。

## 创建多线程
在java中创建线程有两种方法：

> 1)继承Thread类；
2)实现Runnable接口。

### 继承Thread类
> 1. 创建一个类继承Thread;
2. 重新run方法；
3. 创建一个该类对象；
4. 类对象.start()启动线程。

```java
package com;

public class CreateThreadOne extends Thread
{
    @Override
    public void run()
    {
        System.out.println("create thread method one.");
    }
}


public class Main
{
    public static void main(String[] args)
    {
        CreateThreadOne threadOne = new CreateThreadOne();
        threadOne.start();
    }
}
```

### 实现Runnable接口

> 1. 创建一个类实现Runnable接口；
2. 实现run函数；
3. 创建该类对象；
4. 通过该类对象创建一个线程对象；
5. 线程对象.start()启动线程。

```java
package com;

public class CreateThreadTwo implements Runnable
{

    @Override
    public void run()
    {
        System.out.println("create thread method two.");
    }
}


public class Main
{
    public static void main(String[] args)
    {
        CreateThreadTwo threadTwo = new CreateThreadTwo();
        Thread thread = new Thread(threadTwo);
        thread.start();
    }
}
```

### 线程状态
从new一个线程到线程的生命周期结束，会经历多个状态，一般线程主要包括以下几个状态：

> 1. New(新生)
2. Runnable(就绪)
3. Running(运行)
4. Block(阻塞)
5. Time waiting(计时等待)
6. Waiting(等待)
7. Dead(消亡)

## 线程属性
线程属性一般包括线程的优先级、线程是否为守护线程、线程组以及处理未捕获异常的处理器。

### 线程优先级
线程调度器根据线程优先级调度线程，默认情况下，一个线程继承它的父线程的优先级，可以通过setPriority()设置线程的优先级。

```java
thread.setPriority(3);//设置线程优先级
thread.MIN_PRIORITY;//线程最小优先级，１
thread.MAX_PRIORITY;//线程最大优先级，１０
thread.NORM_PRIORITY;//线程的默认优先级，５
thread.yield();//线程让步，让CPU处理其他线程
```

### 守护线程
守护线程和用户线程的区别：
守护线程依赖于创建它的线程，而用户线程则不依赖。
举个例子，如果在main线程中创建了一个守护线程，那么当main方法运行完毕之后，守护线程也会随着消亡，而用户线程则不会，用户线程会一直运行直到运行完毕。在JVM中，像垃圾收集器线程就是守护线程。

```java
thread.setDaemon(true);//将线程设置为守护线程，该方法必须在线程启动前调用
isDaemon();//判断线程是否为守护线程
```


### 未捕获异常处理器
线程的run方法不能抛出任何已检查异常，但是遇到未检查异常，线程会终止。
JVM提供了线程的未捕获异常处理机制，通过未捕获异常处理器进行处理。

### 设置未捕获异常处理器
通过Thread的setUncaughtExceptinHandler方法设置线程的未捕获异常处理器：

```java
public void setUncaughtExceptionHandler(UncaughtExcpetionHandler eh);
```
该处理器必须属于一个实现Thread.UncaughtExceptionhandler接口的类，该接口只有一个方法：
```java
void uncaughtException(Thread t, Throwable e);
```
如下代码，给线程设置了一个未捕获异常处理器：
```java
package com;

public class CreateThreadTwo implements Runnable
{

    @Override
    public void run()
    {
        int[] num =
        { 1, 2, 3 };
        for (int i = 0; i <= 3; i++) {
            System.out.println(num[i]);
        }
    }
}

package com;

public class MyUncaughtExcepitonHandler implements Thread.UncaughtExceptionHandler
{

    @Override
    public void uncaughtException(Thread t, Throwable e)
    {
        System.out.println("catch a uncaughtException.");
        System.out.println(e.fillInStackTrace());
    }
}


package com;

public class Main
{
    public static void main(String[] args)
    {
        CreateThreadTwo threadTwo = new CreateThreadTwo();
        Thread thread = new Thread(threadTwo);
        thread.setUncaughtExceptionHandler(new MyUncaughtExcepitonHandler());
        thread.start();

    }
}
```
运行结果：
```java
1
2
3
catch a uncaughtException.
java.lang.ArrayIndexOutOfBoundsException: 3
```
捕获了一个未处理异常：java.lang.ArrayIndexOutOfBoundsException: 3

除了上述方法，也可以setDefaultUncaughtExceptionhandler为所有线程安装一个默认的处理器。

### 线程组
看一下Thread类的getUncaughtExeptionHandler方法的源码：
```java
public UncaughtExceptionHandler getUncaughtExceptionHandler() {
        return uncaughtExceptionHandler != null ?
            uncaughtExceptionHandler : group;
    }
```
可以看到，如果没有给线程设置未捕获异常处理器，则使用线程所在的线程组对象来处理未捕获异常。

看ThreadGroup的源码：

```java
public class ThreadGroup implements Thread.UncaughtExceptionHandler
{

    ........
    
	public void uncaughtException(Thread t, Throwable e) {
        if (parent != null) {
            parent.uncaughtException(t, e);
        } else {
            Thread.UncaughtExceptionHandler ueh =
                Thread.getDefaultUncaughtExceptionHandler();
            if (ueh != null) {
                ueh.uncaughtException(t, e);
            } else if (!(e instanceof ThreadDeath)) {
                System.err.print("Exception in thread \""
                                 + t.getName() + "\" ");
                e.printStackTrace(System.err);
            }
        }
    }
}
```
ThreadGroup实现了UncaughtExceptionHandler接口，所以可以用来处理未捕获异常。

uncaughtException的处理逻辑如下：

> 1. 如果该线程有父线程组，则调用父线程组的uncaughtException方法；
2. 否则，如果返回的默认处理器不为空，则调用默认处理器的uncaughtException方法；
3. 否则，如果Throwalbe e是ThreadDeath的实例，do nothing;
4. 否则打印线程的名称以及栈跟踪。

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)