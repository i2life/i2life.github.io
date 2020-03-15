---
title: 说说java中的多线程(三)
date: 2019-09-22 16:58:44
tags: Java
categories: 
			- Java
			- 基础知识
---

今天总结一下java中跟多线程相关的两个接口：Callable接口和Future接口。

我们知道，创建线程有两种方式，实现Thread类和实现Runnable接口。

但是这两种方法，run方法返回值类型均为void，在线程执行完任务后，无法获取执行结果。

可以通过Callable接口和Future接口获取执行完任务后的结果。

## Callable接口

```java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

可以看到Callable接口里面声明了一个call方法，返回的类型就是传递进来的类型V。

Callable接口的使用一般配合ExecutorService使用，将一个Callable或者Runnable对象传给ExecutorService的submit方法，返回一个Future的对象。

ExecutorService的重载的submit方法如下：

```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```

### Future接口
Future接口保存异步计算的结果，可以对具体的Runnable或者Callable任务的执行结果进行取消、查询是否已完成、获取结果等操作。
Future接口的源码如下：
```java
public interface Future<V> {

    /**
     * Attempts to cancel execution of this task.  This attempt will
     * fail if the task has already completed, has already been cancelled,
     * or could not be cancelled for some other reason. If successful,
     * and this task has not started when {@code cancel} is called,
     * this task should never run.  If the task has already started,
     * then the {@code mayInterruptIfRunning} parameter determines
     * whether the thread executing this task should be interrupted in
     * an attempt to stop the task.
     *
     * <p>After this method returns, subsequent calls to {@link #isDone} will
     * always return {@code true}.  Subsequent calls to {@link #isCancelled}
     * will always return {@code true} if this method returned {@code true}.
     *
     * @param mayInterruptIfRunning {@code true} if the thread executing this
     * task should be interrupted; otherwise, in-progress tasks are allowed
     * to complete
     * @return {@code false} if the task could not be cancelled,
     * typically because it has already completed normally;
     * {@code true} otherwise
     */
    boolean cancel(boolean mayInterruptIfRunning);

    /**
     * Returns {@code true} if this task was cancelled before it completed
     * normally.
     *
     * @return {@code true} if this task was cancelled before it completed
     */
    boolean isCancelled();

    /**
     * Returns {@code true} if this task completed.
     *
     * Completion may be due to normal termination, an exception, or
     * cancellation -- in all of these cases, this method will return
     * {@code true}.
     *
     * @return {@code true} if this task completed
     */
    boolean isDone();

    /**
     * Waits if necessary for the computation to complete, and then
     * retrieves its result.
     *
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     */
    V get() throws InterruptedException, ExecutionException;

    /**
     * Waits if necessary for at most the given time for the computation
     * to complete, and then retrieves its result, if available.
     *
     * @param timeout the maximum time to wait
     * @param unit the time unit of the timeout argument
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     * @throws TimeoutException if the wait timed out
     */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

该接口声明了5个方法：

> cancle 方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行的任务。

> isCancled方法表示任务是否被取消成功；

> isDone方法表示任务是否已经完成；

> get()方法用来获取执行结果，这个方法会产生阻塞，会一直等待直到任务执行完毕才返回；

> get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

###FutureTask
FutureTask是Future接口的唯一实现类，其实现如下：

```java
public class FutureTask<V> implements RunnableFuture<V> 
{
	......
}

public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```

通过源码可以看到，RunnableFuture继承了Runnable和Future接口，所以FutureTask既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值。

### 使用方法
Callable+FutureTask获取线程执行结果

```java
//构造对象实现Callable接口
package com;

import java.util.concurrent.Callable;

public class MyCompute implements Callable<Integer>
{

    @Override
    public Integer call() throws Exception
    {
        System.out.println("compute thread!");
        int result = 0;
        for (int i = 1; i <= 100; i++) {
            result += i;
        }
        return result;
    }
}


//main函数
package com;

import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class Main
{
    public static void main(String[] args)
    {
        // 创建一个Callable接口对象
        MyCompute myCompute = new MyCompute();
        // 通过Callable接口对象创建一个FutureTask对象
        FutureTask<Integer> futureTask = new FutureTask<>(myCompute);
        // FutureTask对象作为Runnable对象创建线程
        Thread thread = new Thread(futureTask);
        thread.start();

        // 通过 futureTask的get方法获取线程执行结果
        try {
            System.out.println("the result of mycompute is " + futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }
}
```
执行结果：
```java
compute thread!
the result of mycompute is 5050
```
main函数也可以通过 ExecutorService来起线程：

```java
package com;

import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class Main
{
    public static void main(String[] args)
    {
        // 创建一个Callable接口对象
        MyCompute myCompute = new MyCompute();
        ExecutorService executorService = Executors.newCachedThreadPool();
        FutureTask<Integer> futureTask = new FutureTask<>(myCompute);
        executorService.submit(futureTask);
        executorService.shutdown();

        try {
            System.out.println("the rusult of mycompute is " + futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```
使用Callable和Future获取执行结果
一般配合ExecutorSercie使用：

```java
package com;

import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class Main
{
    public static void main(String[] args)
    {
        // 创建一个Callable接口对象
        MyCompute myCompute = new MyCompute();
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<Integer> result = executorService.submit(myCompute);
        executorService.shutdown();

        try {
            System.out.println("the result of mycompute is " + result.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

### 执行器
在上面通过Callable和Future获取线程返回结果的时候，我们已经用到了线程池的概念，通过传入一个Callable对象到线程池起线程。

因为创建线程涉及与操作系统的交互，会存在一定的开销，如果程序中创建了大量的生命周期很短的线程，会影响程序的性能，这个时候可以使用线程池。

一个线程池中包含许多运行的空闲线程，将Runnable对象交个线程池，就会有一个线程调用run方法，当run方法退出时，线程不会死亡，而是在线程池中准备为下一个请求提供服务。

通过执行器Executors的静态方法创建线程池，Executors累包含如下静态方法：

```java
newCachedThreadPool;//构建一个线程池，如果有空闲线程可用，则让它执行任务，如果没有空闲线程，则创建一个空线程；
newFixedThreadPool;//构建一个固定大小的线程池，如果提交的任务数多于空闲的线程数，则把得不到执行的任务放到队列中，等其他任务执行完了，再执行队列中的线程；
newSingleThreadExecutor;//构建一个大小为1的线程池，由一个线程执行提交的任务，一个接一个执行；
newScheduledThreadPool;//用于预定执行而构建的线程池；
newSingleThreadScheduledExecutor;//用于与定制型而构建的单线程池
```

线程池使用步骤：

> 1. 调用Executors类中的静态方法创建线程池；
2. 调用submit提交Runnable或Callable对象；
3. 如果想要取消一个任务，或如果提交Callable对象，就保存好返回的Future对象；
4. 当不需要提交任务时，调用shutdown。

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)