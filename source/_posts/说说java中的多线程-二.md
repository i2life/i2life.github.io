---
title: 说说java中的多线程(二)
date: 2019-09-22 16:47:49
tags: Java
categories: Java
---
## 线程同步
在前面提到过，多线程会有线程安全的问题，如果多个线程同时去访问共享资源（并发访问），可能导致程序运行错误。

如何解决由于并发访问而导致的线程安全问题呢？

一般都是采用线程同步互斥访问的方法来处理，”序列化访问临界资源”，即在同一时间，只允许一个线程访问临界资源。

java中通常有两种方法来实现线程同步：synchronized机制和同步阻塞队列。

### synchronized机制
有一个互斥锁的概念，对临界资源加上互斥锁后，线程只有获得了该临界资源的互斥锁才能访问该资源。

在java中每一个对象都有一个内部的互斥锁，如果一个方法用synchronized关键字声明，则对象的锁会保护整个方法，那么如果需要调用该方法，必须获得内部的锁对象。

内部对象锁只有一个相关条件（相关条件：就是线程获得锁对象之后，可能有一些条件判断才能继续执行，而这些条件又必须其他的线程执行才能满足，这个时候就需要对当前对象进行释放锁）。

看一个具体的例子：
```java
package com;

public class PrintNum
{
    public static void printNum()
    {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " print num " + i);
        }
    }

}


//定义线程类
package com;

public class MyThread implements Runnable
{

    @Override
    public void run()
    {
      PrintNum.printNum();
    }
}

//main函数中起两个线程，并start
package com;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.Queue;

public class Main
{
    public static void main(String[] args)
    {
        Thread threadOne = new Thread(new MyThread());

        threadOne.start();

        Thread threadTwo = new Thread(new MyThread());
        threadTwo.start();

    }
}
```
运行结果：
```java
Thread-1 print num 0
Thread-0 print num 0
Thread-1 print num 1
Thread-0 print num 1
Thread-1 print num 2
Thread-0 print num 2
Thread-1 print num 3
Thread-0 print num 3
Thread-1 print num 4
Thread-0 print num 4
```
线程0和线程1交叉打印数字。

现在将printNum方法声明为synchronized：
```java
package com;

public class PrintNum
{
    public synchronized static void printNum()
    {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " print num " + i);
        }
    }

}
```
运行结果：
```java
Thread-0 print num 0
Thread-0 print num 1
Thread-0 print num 2
Thread-0 print num 3
Thread-0 print num 4
Thread-1 print num 0
Thread-1 print num 1
Thread-1 print num 2
Thread-1 print num 3
Thread-1 print num 4
```
使用了synchronized关键字，线程进行了同步处理，线程0运行完了，释放内部锁之后，线程1才能执行。

**关于synchronized关键字的几点说明：**

> 1) 当一个线程正在访问对象的synchronized方法，其他线程不能再访问该对象的其他synchronized方法，因为一个对象只有一个互斥锁；
2) 当一个线程正在访问对象的synchronized方法，其他线程可以访问该对象的非synchronized方法；
3) 访问同一个类的不同对象实例的synchronized方法，不存在线程安全问题。

### 同步阻塞机制（也称为synchronized代码块）
java中还有另外一种获取锁的方式：synchronized代码块。

有的时候把一整个方法声明为synchronized并不一定合适，比如该方法中只有一部分代码需要同步互斥访问，这个时候可以使用synchronized代码块，只对需要同步的那部分代码进行同步。

举例子，上面的代码可以将printNum方法声明synchronized代码块：
```java
package com;

public class PrintNum
{
    private static Object object = new Object();
    public static void printNum()
    {
        synchronized (object) 
        {
            for (int i = 0; i < 5; i++) 
            {
                System.out.println(Thread.currentThread().getName() + " print num " + i);
            }
        }
    }

}
```
说明一点，这里定义的是static方法，并没有定义PrintNum对象，所以不光对象实例有锁，类也有锁。

**Tips:**

> 1. 如果一个线程执行对象的非static synchronized方法，另一个线程执行该类的static synchronized方法，不存在互斥问题，因为访问static synchronized方法占用的是类锁，访问非static synchronized方法占用的是对象锁，所以不存在资源互斥的问题；
2. 对于synchronized方法或者synchronized代码块，当出现异常时，JVM会自动释放当前线程占用的锁，因此不会由于异常导致出现死锁现象。

### volatile域
volatile也是java提供的一种轻量级的同步机制。

有的时候，仅仅为了读写一两个实例域就使用同步锁，开销会比较大，那么就可以考虑使用volatile域。

volatile为实例域的同步访问提供了一种免锁机制，如果声明一个域为volatile，那么编译期虚拟机就知道该域是被另一个线程并发更新的。

**并发编程中的三个概念：**

> 1. 原子性：一个操作要么全部执行并且执行的过程中不允许中断，要么不执行；
> 2. 可见性：是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程也能立即看到修改的值；
> 3. 有序性：是指程序执行的顺序按照代码的先后顺序执行。
volatile可以保证共享变量的可见性和有序性，不能保证共享变量的原子性。

被volatile修饰的变量，会强制将修改的值立即写入主存，这样其他线程就可以知道该变量的值得到了修改，从而保证了可见性；被volatile修饰的变量，禁止指令重排，保证了有序性。

### 阻塞队列
队列分为非阻塞队列和阻塞队列。非阻塞队列的一个问题是，不会对当前线程产生阻塞，那么在面对类似消费者-生产者模型时，就必须额外的实现同步机制来保证线程安全。

阻塞队列，可以对当前线程产生阻塞，如果一个线程从空的阻塞队列中读取元素，或者往一个满的阻塞队列中放入元素，线程都会被阻塞，直到阻塞队列满足当前的操作，被阻塞的队列会自动被唤醒。

#### 常见的阻塞队列
**LinkedBlockingQueue**:基于链表实现的阻塞队列，如果没有指定队列容量大小，默认大小为Integer.MAX_VALUE;
**ArrayBlockingQueue**:基于数组实现的阻塞队列，在构造该队列对象时，必须指定容量，并且有一个可选的参数来指定时候需要公平性，如果设置了公平参数，那么等待了最长时间的线程会得到处理，一般，公平性会牺牲性能，所以默认是非公平的。
**PriorityBlockingQueue**:上面两种队列都是先进先出的队列，PriorityBlockingQueue是带优先级的队列，元素按照优先级顺序从队列移出。且该队列是无界阻塞队列，容量没有上限，上面两种队列都是有界队列。
**DelayQueue**:基于PriorityQueue，一种延时阻塞队列，DelayQueue中的元素只有当指定的时间到了，才能够从队列中获取到该元素，DelayQueue也是无界队列。

#### 非阻塞队列和阻塞队列的方法
**1. 非阻塞队列的几个主要方法：**
add(E e)：将元素e插入到队列尾部，如果插入成功，返回true，如果插入失败（如队列满），则抛出异常；
remove():移除队列首元素，若移除成功，则返回true，否则抛异常；
offer(E e)：将元素e插入到队列尾部，插入成功，返回true，否则返回false；
poll():移除并获取队列首元素，获取成功返回队首元素，失败返回null；
peek():获取队列首元素，若成功，则返回队首元素，失败返回null。

对于非阻塞队列，一般情况下建议使用offer、poll和peek三个方法，不建议使用add和remove方法，因为这样可以通过返回值判断操作成功与否。注意非阻塞队列中的方法都没有进行同步处理。

**2. 阻塞队列的几个主要方法：**
阻塞队列包括非阻塞队列中的大部分方法，上面列举的5个方法在阻塞队列中都存在，但是要注意这5个方法在阻塞队列中都进行了同步处理，除此之外，阻塞队列提供了另外4个非常有用的方法：

put(E e)：向队尾存入元素，如果队列满，则等待；
take()：从队首取元素，如果队列空，则等待；
offer(E e, long timeout, TimeUnit unit)：向队尾存入元素，如果队列满，等待一定时间，到达等到时间上限，如果还没有插入成功，则返回false，否则返回true；
poll(long timeout, TimeUnit unit)：从队首取元素，如果空，则等待一定时间，如果还是没有取到，则返回null，否则返回取到的元素。

### 线程安全的集合
上面讨论的阻塞队列就是线程安全的集合，java.util.concurrent还提供了一些其他的线程安全的集合：
线程安全的map：
ConcurrentHashMap:构造一个可以被多线程访问的hashmap
ConcurrentSkipListMap:构造一个可以被多线程安全访问的有序map

线程安全的set：
ConcurrentSkipListSet:构造一个可以被多线程安全访问的有序集

线程安全的queuue：
ConcurrentLinkedQueue:构造一个可以被多个线程安全访问的无边界的非阻塞队列

### 同步包装器
Vector和Hashtable提供了线程安全的动态数组和散列表的实现，ArrayList和HaspMap不是线程安全的，但是可以通过同步包装器变成线程安全的：
```java
List<T> synchArrayList = Collections.synchronizedList(new ArrayList <>());
Map<K,V> synchHashMap = Collections.synchronizedMap(new HashMap <>());
```

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
