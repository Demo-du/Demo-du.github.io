---
layout: post
title:  设计模式之观察者模式
categories: 设计模式
description: 
keywords: 
---

## 概述

观察者模式（又被称为发布-订阅（Publish/Subscribe）模式，属于行为型模式的一种，它定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态变化时，会通知所有的观察者对象，使他们能够自动更新自己。

结构如下图所示：

![图 6-11 观察者模式的结构图](http://img.blog.csdn.net/20161111191040882)

在观察者模式中有如下角色：

- Subject：抽象主题（抽象被观察者），抽象主题角色把所有观察者对象保存在一个集合里，每个主题都可以有任意数量的观察者，抽象主题提供一个接口，可以增加和删除观察者对象。
- ConcreteSubject：具体主题（具体被观察者），该角色将有关状态存入具体观察者对象，在具体主题的内部状态发生改变时，给所有注册过的观察者发送通知。
- Observer：抽象观察者，是观察者者的抽象类，它定义了一个更新接口，使得在得到主题更改通知时更新自己。
- ConcrereObserver：具体观察者，实现抽象观察者定义的更新接口，以便在得到主题更改通知时更新自身的状态。

## 具体实现

这里以师生关系为例,老师和学生是一对多的关系,老师给学生布置作业,这个动作作为主题事件,每当老师布置一道题时,就要自动通知到所有的学生把该题记下来,然后再布置下一道题..

**Subject接口:**

```java
package com.wang.observer;
//主题接口
 interface Subject {
     //添加观察者
     void addObserver(Observer obj);
     //移除观察者
     void deleteObserver(Observer obj);
     //当主题方法改变时,这个方法被调用,通知所有的观察者
     void notifyObserver();
}
```

**Oserver接口:**

```java
package com.wang.observer;

interface Observer {
    //当主题状态改变时,会将一个String类型字符传入该方法的参数,每个观察者都需要实现该方法
    public void update(String info);
}
```

**Subject接口实现类TeacherSubject:**

```java
package com.wang.observer;

import java.util.ArrayList;
import java.util.List;

public class TeacherSubject implements Subject {
    //用来存放和记录观察者
    private List<Observer> observers=new ArrayList<Observer>();
    //记录状态的字符串
    private String info;
    
    @Override
    public void addObserver(Observer obj) {
        observers.add(obj);
    }

    @Override
    public void deleteObserver(Observer obj) {
        int i = observers.indexOf(obj);
        if(i>=0){
            observers.remove(obj);
        }
    }

    @Override
    public void notifyObserver() {
        for(int i=0;i<observers.size();i++){
            Observer o=(Observer)observers.get(i);
            o.update(info);
        }
    }
    //布置作业的方法,在方法最后,需要调用notifyObserver()方法,通知所有观察者更新状态
    public void setHomework(String info){
        this.info=info;
        System.out.println("今天的作业是"+info);
        this.notifyObserver();
    }

}
```

**Observer接口实现类StudentObserver:**

```java
package com.wang.observer;

public class StudentObserver implements Observer {

    //保存一个Subject的引用,以后如果可以想取消订阅,有了这个引用会比较方便
    private TeacherSubject t;
    //学生的姓名,用来标识不同的学生对象
    private String name;
    //构造器用来注册观察者
    public Student(String name,Teacher t) {
        this.name=name;
        this.t = t;
        //每新建一个学生对象,默认添加到观察者的行列
        t.addObserver(this);
    }


    @Override
    public void update(String info) {
        System.out.println(name+"得到作业:"+info);
        
    }

}
```

**测试类TestObserver:**

```java
package com.wang.observer;

public class TestObserver {

    public static void main(String[] args) {
        
        TeacherSubject teacher=new TeacherSubject();
        StudentObserver zhangSan=new StudentObserver("张三", teacher);
        StudentObserver LiSi=new StudentObserver("李四", teacher);
        StudentObserver WangWu=new StudentObserver("王五", teacher);
        
        teacher.setHomework("第二页第六题");
        teacher.setHomework("第三页第七题");
        teacher.setHomework("第五页第八题");
    }
}
```

**结果：**

```
今天的作业是第二页第六题
张三得到作业:第二页第六题
李四得到作业:第二页第六题
王五得到作业:第二页第六题
今天的作业是第三页第七题
张三得到作业:第三页第七题
李四得到作业:第三页第七题
王五得到作业:第三页第七题
今天的作业是第五页第八题
张三得到作业:第五页第八题
李四得到作业:第五页第八题
王五得到作业:第五页第八题
```

## 使用观察者模式的场景和优缺点

#### **使用场景**

- 关联行为场景，需要注意的是，关联行为是可拆分的，而不是“组合”关系。
- 事件多级触发场景。
- 跨系统的消息交换场景，如消息队列、事件总线的处理机制。

#### **优点**

解除耦合，让耦合的双方都依赖于抽象，从而使得各自的变换都不会影响另一边的变换。

#### **缺点**

在应用观察者模式时需要考虑一下开发效率和运行效率的问题，程序中包括一个被观察者、多个观察者，开发、调试等内容会比较复杂，而且在Java中消息的通知一般是顺序执行，那么一个观察者卡顿，会影响整体的执行效率，在这种情况下，一般会采用异步实现。

