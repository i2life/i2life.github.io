---
title: 说说java中的接口与回调
date: 2019-08-10 16:40:04
tags: Java
categories: 
			- Java
			- 基础知识
---
## 接口与回调
callback是一种常见的设计模式，在这种模式中，可以指定在某个特定事件发生后应该采取的动作。比如说，有一个老师发布了一项任务，让一个学生去跑50m，跑完后学生主动告诉老师跑的时间，老师进行成绩登记。在这个过程中，首先老师给学生发布任务，可以理解是老师调用学生，然后学生执行完跑步这个动作后，需要主动通知老师跑的时间，这个时候是学生反过来调用老师，这个就是比较通俗的回调逻辑了。

通过上面的例子，我们可以很明显的看到，这里有一个循环调用，老师调用学生，学生反调老师，如果只定义一个Teacher类和一个Student类，就会存在循环依赖，如果Teacher和Student分属不同的组件，就会导致组件间的循环依赖，这样会导致编译错误，因为编译器不知道先编译哪个组件。

那么通过接口就可以很好的解决这个问题了：

可以让Teacher实现一个接口ScoreInterface，在该接口里定义一个登记分数的方法recordScore(Double timeSpend)，老师在发布任务方法pubulishTask()里调用学生的doRun()方法，让学生执行跑步动作，调用doRun()方法的时候，Teacher把自己的实例对象以ScoreInterface接口对象的形式传给Student对象；

Student对象执行doRun()方法，执行完后，根据doRun()方法传过来的ScoreInterface接口对象调用recordScore(Double timeSpend)方法，从而实现把跑的时间通知给老师，让老师登记分数这样一个动作。

这样就避免了循环依赖，Teacher依赖Student和ScoreInterface，Student依赖ScoreInterface，Teacher和Student之间就不存在相互依赖了。

## 代码实现
```java
//定义接口ScoreInterface
package com.publicInterface;

public interface ScoreInterface
{
  void recordScore(Double timeSpend);
}

//定义一个Teacher类，实现该接口
package com.teacher;

import com.Person;
import com.publicInterface.ScoreInterface;
import com.student.Student;

public class Teacher implements ScoreInterface
{
    public void publishTask(Student student, int length)
    {
        System.out.println("Teacher publish the task, run " + length + "ｍ.");
        student.doRun(this, length);
    }

    @Override
    public void recordScore(Double timeSpend)
    {
        if(timeSpend<7)
            System.out.println("Well done. Excellent.");
        else
            System.out.println("Good");
    }
}
//定义学生类
package com.student;

import com.publicInterface.ScoreInterface;

public class Student
{
    public void doRun(ScoreInterface scoreInterface, int length)
    {
        if (length == 50) {
            System.out.println("Begin to Run.");
            System.out.println("Run finished, my time spend is 6.8s.");
        }
        scoreInterface.recordScore(6.8);
    }
}
//main函数
package com;

import com.student.Student;
import com.teacher.Teacher;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main
{
    public static void main(String[] args)
    {
        Teacher teacher = new Teacher();
        Student student = new Student();
        teacher.publishTask(student, 50);
    }
}
```
结果输出：
```java
Teacher publish the task, run 50ｍ.
Begin to Run.
Run finished, my time spend is 6.8s.
Well done. Excellent.
```
## 参考文献
[java核心技术.卷一](https://www.douban.com/link2/?url=https%3A%2F%2Fbook.douban.com%2Fsubject%2F3146174%2F&query=java%E6%A0%B8%E5%BF%83%E7%BB%93%E6%9D%9F&cat_id=1001&type=search&pos=1)
