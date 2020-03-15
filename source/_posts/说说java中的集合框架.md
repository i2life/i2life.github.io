---
title: 说说java中的集合框架
date: 2019-09-22 12:39:41
tags: Java
categories: 
			- Java
			- 基础知识
---
选择合适的数据结构是优雅的解决问题的关键之一。

java提供了一组集合框架帮助我们实现常用的数据结构，今天就对java中的集合进行一个总结。

## 集合框架
java集合类库构成了集合类的框架。java集合框架将接口与实现分离，接口定义集合的基本方法，在实现类里对这些方法进行实现。为了减少实现类要实现接口的所有方法带来的繁琐，java集合框架还定义了一些抽象类，实现类也可以直接继承抽象类，这样实现类可以只实现想实现的方法。

java集合包含两个基本的接口:Collection和Map。Collection又分为三个子类：List、Set、Queue。常用的集合实现类有：ArrayList、LinkedList、HashSet、LinkedHashSet、HashMap以及LinkedHashMap等。

## Collection接口
Collection接口是单元素集合的父接口，定义了很多方法：


对于这些方法的详细说明可以参照API文档，下面对Collection接口的三个子接口:List、Set、Queue进行介绍。

### List
List接口是Collection的子接口，List接口可以用来定义一个有序的、元素可重复的集合。

List集合中每个元素都有对应的索引，可以通过索引来访问指定位置的集合元素，List集合默认按照元素的添加顺序设置元素的索引，比如第一个添加的元素索引为0……

#### ArrayList
ArrayList和Vector类都是基于数组实现的List集合，所以ArrayList和Vector类封装了一个动态的、允许再分配的Object[]数组。ArrayList或Vector对象使用initalCapacity参数来设置该数组的长度，当向ArrayList或Vector中添加元素超过了该数组的长度时，他们的initalCapacity会自动增加。

**遍历ArrayList的方法一：**
迭代器遍历
```java
package com;

import java.util.ArrayList;
import java.util.Iterator;

public class Main
{
    public static void main(String[] args)
    {
        ArrayList<String> names = new ArrayList<>();
        names.add("a");
        names.add("b");
        Iterator iter = names.iterator();
        while ((iter).hasNext()) {
            System.out.println(iter.next());
        }
    }
}
```

**遍历ArrayList的方法二：**
索引遍历
```java
package com;

import java.util.ArrayList;
import java.util.Iterator;

public class Main
{
    public static void main(String[] args)
    {
        ArrayList<String> names = new ArrayList<>();
        names.add("a");
        names.add("b");

        for (int i = 0; i < names.size(); i++) {
            System.out.println(names.get(i));
        }
    }
}
```
**遍历ArrayList的方法三：**
foreach循环
```java
for(String name:names)
{
	System.out.println(name);
}
```
通过索引遍历ArrayList的效率最高，迭代器访问效率最低。

**ArrayList和Vector的区别：**

> 1. ArrayList是线程不安全的，Vector是线程安全的；

> 2. Vector的性能比ArrayList差。

#### LinkedList
LinkedList是基于链表实现的List集合，插入和删除元素的效率较高。所以需要经常在集合中间插入或删除元素，可以考虑选用LinkedList集合。

#### Set
Set接口也是扩展自Collection接口，与List不同，Set不允许重复的元素，Set接口有3个具体的实现类：HashSet、LinkedHashSet、TreeSet。

##### HashSet
HashSet使用Hash算法来存储集合中的元素，具有很好的存取和查找性能。
**HashSet的特点：**

> 1. 不能保证元素的顺序；
> 2. HashSet不是同步的，如果多个线程同时访问一个HashSet，则必须通过同步机制来保证线程安全；
> 3. 集合元素值可以是null。

##### LinkedHashSet
LinkedHashSet是用一个链表实现来对HashSet进行扩展，它支持对规则集内的元素排序，使得元素是以插入的顺序来保存的。

当遍历LinkedHashSet时，将会按照元素的添加顺序来访问集合中的元素，由于要维护元素的插入顺序，所以性能略低于HashSet，但在迭代访问Set里的全部元素时，有很好的性能。

##### TreeSet
TreeSet是一个有序的Set，可以确保集合元素处于排序状态，不是插入顺序。

#### Queue
Queue是一种先进先出的数据结构，Queue接口也是扩展自Collection接口。

因为LinkedList类实现了Deque接口，所以我们通常可以使用LinkedList来创建一个队列：

```java
package com;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.Queue;

public class Main
{
    public static void main(String[] args)
    {
        Queue<String> queue = new LinkedList<>();
        queue.offer("a");
        queue.offer("b");
        while (queue.size() > 0) {
            System.out.println(queue.poll());
        }
    }
}
```

## Map接口
Map是用来存储键值对的容器类，在Map中键可以是任意类型的对象，但不能有重复的键，每个Key都对应一个value。

Map接口有3个具体的实现类：HashMap、LinkedHashMap、TreeMap。

### HashMap
看一下HashMap的构造函数：

```java
//默认构造函数
HashMap()

//指定容量大小的构造函数
HashMap(int capacity)

//指定容量大小和加载因子的构造函数
HashMap(int capacity, float loadFactor)

//包含子Map的构造函数
HashMap(Map<? extends K, ? extends V> map)
```

从构造函数中，了解到两个重要的元素：容量大小(capacity)和加载因子(loadFactor)。
容量是哈希表的容量，初始容量是哈希表在创建时的容量(即DEFAULT_INITIAL_CAPACITY=1<<4)。

加载因子是哈希表在其容量自动增加前可以达到多满的一种尺度。当哈希表中的条目数超过了加载因子与当前容量的乘积时，则要对该哈希表进行resize操作，从而哈希表将具有大约两倍的桶数。

通常，默认加载因子是0.75，这是在时间和空间成本上的一种折中。加载因子过高虽然减少了空间开销，但同时也增加了查询成本，在设置容量时应该考虑到映射中所需要的条目数和加载因子，以便最大限度地减少resize操作次数。

**HashMap的遍历方式：**

> 1. 遍历HashMap的键值对；
第一步：根据EntrySet()获取HashMap的键值对的Set集合。
第二步：通过Iterator迭代器遍历第一步得到的集合。

> 2. 遍历HashMap的键
第一步：根据KeySet()获取HashMap的“键”的Set集合。
第二步：通过Iterator迭代器遍历第一步得到的集合。

> 3. 遍历HashMap的值
第一步：根据value()获取HashMap的值的集合。
第二步：通过Iterator迭代器遍历第一步得到的集合。

### LinkedHashMap
HashSet有一个linkedHashSet子类，HashMap也有一个LinkedHashMap子类；LinkedHashMap使用双向链表来维护Key-value对的次序。

LinkedHashMap需要维护元素的插入顺序，因此性能略低于HashMap的性能；但是因为它以链表来维护内部顺序，所以在迭代访问Map里的全部元素时有较好的性能，迭代输出LinkedHashMap的元素时，将会按照添加key-value对的顺序输出。

本质上来讲，LinkedHashMap=散列表+循环双向链表。

### TreeMap
TreeMap是SortedMap接口的实现类，TreeMap是一个有序的key-value集合，它是通过红黑树来实现的，每个key-value对即作为红黑树的一个节点。

## 线程安全的集合
上面主要对java集合框架中的一些常用接口和类进行了介绍，包括Collection和Map接口及他们的抽象类和常用的具体实现类。下面介绍一下其他几个特殊的集合类：

### Vector
前面已经提到，java设计者在对之前的容器进行重新设计时保留了一些数据结构，其中就有Vector，用法上Vector与ArrayList基本一致，不同之处在与Vector使用了关键字synchronized将访问和修改向量的方法都变成同步的了，所以Vector是线程安全的。

### Stack
Stack，栈类，继承自Vector。

### HashTable
HashTable和前面介绍的HashMap比较类似，它也是一个散列表，存储的内容是键值对映射，不同之处在于，HahsTable继承自Ditionary，HashTable中的函数都是同步的，HashTable也是线程安全的。

### ConcurrentHashMap
Concurrent，并发，从名字就可以看出ConcurrentHashMap是HashMap的线程安全版，同HashMap相比，ConcurrentHashMap不仅保证了访问的线程安全性，而且在效率上也有较大提高。

### CopyOnWriteArrayList
CopyOnWriteArrayList是一个线程安全的List接口的实现，它使用了ReenTrantLock锁来保证在并发情况下提供高性能的读取

## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)