---
layout: post
title:  ThreadLocal相关知识
categories: Java并发
description: 
keywords: 
---

在之前一篇博客：http://Demo-du.github.io/2017/11/23/bingfa2/ ,最后稍微提到了一些ThreadLocal的知识，在这里补充一下。

## 概述

ThreadLocal官网解释：

This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its {@code get} or {@code set} method) has its own, independently initialized copy of the variable.  {@code ThreadLocal} instances are typically private static fields in classes that wish to associate state with a thread (e.g.,a user ID or Transaction ID)

翻译过来的大概意思就是：ThreadLocal类用来提供线程内部的局部变量。这些变量在多线程环境下访问(通过get或set方法访问)时能保证各个线程里的变量相对独立于其他线程内的变量，ThreadLocal实例通常来说都是private static类型。

总结：ThreadLocal不是为了解决多线程访问共享变量，而是为每个线程创建一个单独的变量副本，提供了保持对象的方法和避免参数传递的复杂性。

ThreadLocal的主要应用场景为按线程多实例（每个线程对应一个实例）的对象的访问，并且这个对象很多地方都要用到。例如：我们很多人登录淘宝购物，服务器就会给每个用户开一个线程，每个线程中创建一个ThreadLocal，里面存用户基本信息等，在很多页面跳转时，会显示用户信息或者得到用户的一些信息等频繁操作，这样多线程之间并没有联系而且当前线程也可以及时获取想要的数据。

## 实现原理

ThreadLocal可以看做是一个容器，容器里面存放着属于当前线程的变量。ThreadLocal类提供了四个对外开放的接口方法，这也是用户操作ThreadLocal类的基本方法： 
(1) void set(Object value)设置当前线程的线程局部变量的值。 
(2) public Object get()该方法返回当前线程所对应的线程局部变量。 
(3) public void remove()将当前线程局部变量的值删除，目的是为了减少内存的占用，该方法是JDK 5.0新增的方法。需要指出的是，当线程结束后，对应该线程的局部变量将自动被垃圾回收，所以显式调用该方法清除线程的局部变量并不是必须的操作，但它可以加快内存回收的速度。 
(4) protected Object initialValue()返回该线程局部变量的初始值，该方法是一个protected的方法，显然是为了让子类覆盖而设计的。这个方法是一个延迟调用方法，在线程第1次调用get()或set(Object)时才执行，并且仅执行1次，ThreadLocal中的缺省实现直接返回一个null。

在ThreadLocal类中有一个静态内部类ThreadLocalMap(其类似于Map)，用键值对的形式存储每一个线程的变量副本，ThreadLocalMap中元素的key为当前ThreadLocal对象，而value对应线程的变量副本，每个线程可能存在多个ThreadLocal。

以下是ThreadLocal的源码：



```java
static class ThreadLocalMap {
  //map中的每个节点Entry,其键key是ThreadLocal并且还是弱引用，这也导致了后续会产生内存泄漏问题的原因。
 static class Entry extends WeakReference<ThreadLocal<?>> {
           Object value;
           Entry(ThreadLocal<?> k, Object v) {
               super(k);
               value = v;
   }
    /**
     * 初始化容量为16，以为对其扩充也必须是2的指数 
     */
    private static final int INITIAL_CAPACITY = 16;
    /**
     * 真正用于存储线程的每个ThreadLocal的数组，将ThreadLocal和其对应的值包装为一个Entry。
     */
    private Entry[] table;


    ///....其他的方法和操作都和map的类似
}
```

ThreadLocal源码如下

```java
/**
 Returns the value in the current thread's copy of this
 thread-local variable.  If the variable has no value for thecurrent thread, it is first initialized to the value returned by an invocation of the {@link #initialValue} method.
  @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();//当前线程
    ThreadLocalMap map = getMap(t);//获取当前线程对应的ThreadLocalMap
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);//获取对应ThreadLocal的变量值
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();//若当前线程还未创建ThreadLocalMap，则返回调用此方法并在其中调用createMap方法进行创建并返回初始值。
}
//设置变量的值
public void set(T value) {
   Thread t = Thread.currentThread();
   ThreadLocalMap map = getMap(t);
   if (map != null)
       map.set(this, value);
   else
       createMap(t, value);
}
private T setInitialValue() {
   T value = initialValue();
   Thread t = Thread.currentThread();
   ThreadLocalMap map = getMap(t);
   if (map != null)
       map.set(this, value);
   else
       createMap(t, value);
   return value;
}
/**
为当前线程创建一个ThreadLocalMap的threadlocals,并将第一个值存入到当前map中
@param t the current thread
@param firstValue value for the initial entry of the map
*/
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
//删除当前线程中ThreadLocalMap对应的ThreadLocal
public void remove() {
       ThreadLocalMap m = getMap(Thread.currentThread());
       if (m != null)
           m.remove(this);
}
```

get()方法是用来获取ThreadLocal在当前线程中保存的变量副本，set()用来设置当前线程中变量的副本，remove()用来移除当前线程中变量的副本，initialValue()是一个protected方法，一般是用来在使用时进行重写的，它是一个延迟加载方法，下面会详细说明。

　　首先我们来看一下ThreadLocal类是如何为每个线程创建一个变量的副本的。

　　先看下get方法的实现：

　　![img](http://images.cnitblog.com/blog/288799/201408/241027152537015.jpg)

 　　第一句是取得当前线程，然后通过getMap(t)方法获取到一个map，map的类型为ThreadLocalMap。然后接着下面获取到<key,value>键值对，注意这里获取键值对传进去的是  this，而不是当前线程t。

　　如果获取成功，则返回value值。

　　如果map为空，则调用setInitialValue方法返回value。

　　我们上面的每一句来仔细分析：

　　首先看一下getMap方法中做了什么：

　　![img](http://images.cnitblog.com/blog/288799/201408/241028044719452.jpg)

　　可能大家没有想到的是，在getMap中，是调用当期线程t，返回当前线程t中的一个成员变量threadLocals。

　　那么我们继续取Thread类中取看一下成员变量threadLocals是什么：

　　![img](http://images.cnitblog.com/blog/288799/201408/241029514406632.jpg)

　　实际上就是一个ThreadLocalMap，这个类型是ThreadLocal类的一个内部类，我们继续取看ThreadLocalMap的实现：

　　![img](http://images.cnitblog.com/blog/288799/201408/241031330495608.jpg)

　　可以看到ThreadLocalMap的Entry继承了WeakReference，并且使用ThreadLocal作为键值。

　　然后再继续看setInitialValue方法的具体实现：

![img](http://images.cnitblog.com/blog/288799/201408/241034465033208.jpg)

　　很容易了解，就是如果map不为空，就设置键值对，为空，再创建Map，看一下createMap的实现：

　　![img](http://images.cnitblog.com/blog/288799/201408/241038005189081.jpg)

　　至此，可能大部分朋友已经明白了ThreadLocal是如何为每个线程创建变量的副本的：

　　首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。

　　初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。

　　然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

总之，为不同线程创建不同的ThreadLocalMap，用线程本身为区分点，每个线程之间其实没有任何的联系，说是说存放了变量的副本，其实可以理解为为每个线程单独new了一个对象。

## 内存泄漏问题

​         在上面提到过，每个thread中都存在一个map, map的类型是ThreadLocal.ThreadLocalMap. Map中的key为一个threadlocal实例. 这个Map的确使用了弱引用,不过弱引用只是针对key. 每个key都弱引用指向threadlocal. 当把threadlocal实例置为null以后,没有任何强引用指向threadlocal实例,所以threadlocal将会被gc回收. 但是,我们的value却不能回收,因为存在一条从current thread连接过来的强引用. 只有当前thread结束以后, current thread就不会存在栈中,强引用断开, Current Thread, Map, value将全部被GC回收. 
　　所以得出一个结论就是只要这个线程对象被gc回收，就不会出现内存泄露，但在threadLocal设为null和线程结束这段时间不会被回收的，就发生了我们认为的内存泄露。其实这是一个对概念理解的不一致，也没什么好争论的。最要命的是线程对象不被回收的情况，这就发生了真正意义上的内存泄露。比如使用线程池的时候，线程结束是不会销毁的，会再次使用的。就可能出现内存泄露。

参考资料：1、《Java并发编程的艺术》

​                    2、http://blog.csdn.net/lhqj1992/article/details/52451136

​                    3、https://www.cnblogs.com/WuXuanKun/p/5827060.html