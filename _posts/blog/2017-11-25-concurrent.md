---
layout: post
title:  ConcurrentHashMap相关（转载）
categories: Java并发
description: 
keywords: 
---


## ConcurrentHashMap的实现原理与使用

### 为何用ConcurrentHashMap

在并发编程中使用HashMap可能会导致死循环，而使用线程安全的HashTable效率又低下。

1、**线程不安全的HashMap**

在多线程环境下，使用HashMap进行put操作会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap，如以下代码会导致死循环：

`

final HashMap<String, String> map = new HashMap<String, String>(2);
Thread t = new Thread(new Runnable() {
​    @Override
​    public void run() {
​        for (int i = 0; i < 10000; i++) {
​            new Thread(new Runnable() {
​                @Override
​                public void run() {
​                    map.put(UUID.randomUUID().toString(), "");
​                }
​            }, "moon" + i).start();
​        }
​    }
}, "ftf");
t.start();
t.join();

`

HashMap在并发执行put操作是会引起死循环，是因为多线程会导致HashMap的Entry链表形成环形数据结构，一旦形成唤醒数据结构，Entry的next节点永远不为空，就会产生死循环。

2、**效率低下的HashTable**

HashTable使用synchronized来保证线程的安全，但是在线程竞争激烈的情况下HashTable的效率非常低下。当一个线程访问HashTable的同步方法，其他方法访问HashTable的同步方法时，会进入阻塞或者轮询状态。如果线程1使用put进行元素添加，线程2不但不能用put方法添加于元素同是也无法用get方法来获取元素，所以竞争越激烈效率越低。

3、**ConcurrentHashMap的锁分段技术**

HashTable容器在竞争激烈的并发环境效率低下的原因是所有访问HashTable的线程都必须竞争同一把锁，假如容器有多把锁，每一把锁用于锁住容器中一部分数据，那么多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效提高并发访问率，这就是ConcurrentHashMap的锁分段技术。将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一段数据的时候，其他段的数据也能被其他线程访问。

### **ConcurrentHashMap的结构**

![这里写图片描述](http://img.blog.csdn.net/20160717141855166)

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。 

![这里写图片描述](http://img.blog.csdn.net/20160717144944500)

### **java1.8的ConcurrentHashMap源码分析**

#### **重要的内部类**

从Java1.7 版本开始 ConcurrentHashMap 不再采用 Segment 实现，而是改用 Node，Node 是一个链表的结构，每个节点可以引用到下一个节点(next)。

- Node类 
  Node是最核心的内部类，包装了key-value键值对，所有插入ConcurrentHashMap的数据都包装在这里面。 
  它与HashMap中的定义很相似，但是有一些差别它对value和next属性设置了volatile同步锁，它不允许调用setValue方法直接改变Node的value域，它增加了find方法辅助map.get()方法。
- TreeNode类 
  树节点类，另外一个核心的数据结构。 当链表长度过长的时候，会转换为TreeNode。 
  但是与HashMap不相同的是，它并不是直接转换为红黑树，而是把这些结点包装成TreeNode放在TreeBin对象中，由TreeBin完成对红黑树的包装。 
  而且TreeNode在ConcurrentHashMap继承自Node类，而并非HashMap中的集成自LinkedHashMap.Entry
- TreeBin 
  这个类并不负责包装用户的key、value信息，而是包装的很多TreeNode节点。它代替了TreeNode的根节点，也就是说在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象，这是与HashMap的区别。
- ForwardingNode 
  一个用于连接两个table的节点类。它包含一个nextTable指针，用于指向下一张表。而且这个节点的key value next指针全部为null，它的hash值为-1. 
  这里面定义的find的方法是从nextTable里进行查询节点，而不是以自身为头节点进行查找

#### **构造函数**

```java
public ConcurrentHashMap() {
    }
    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                   MAXIMUM_CAPACITY :
                   tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
    }

    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }

    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }


    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
```

Java1.8版本的 ConcurrentHashMap 在构造函数中不会初始化 Node 数组，而是第一次 put 操作的时候初始化。 
整个 Map 第一次 put 的时候，map 中用于存放数据的 Node[] 还是null。

#### **初始化函数initTable**

调用ConcurrentHashMap的构造方法仅仅是设置了一些参数而已，而整个table的初始化是在向ConcurrentHashMap中插入元素的时候发生的。如调用put、computeIfAbsent、compute、merge等方法的时候，调用时机是检查table==null。 
初始化方法主要应用了关键属性sizeCtl 如果这个值 < 0，表示其他线程正在进行初始化，就放弃这个操作。 
在这也可以看出ConcurrentHashMap的初始化只能由一个线程完成。如果获得了初始化权限，就用CAS方法将sizeCtl置为-1，防止其他线程进入。初始化数组后，将sizeCtl的值改为0.75*n

sizeCtl含义 
1.负数代表正在进行初始化或扩容操作 
2.-1代表正在初始化 
3.-N 表示有N-1个线程正在进行扩容操作 
4.正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小，这一点类似于扩容阈值的概念。还后面可以看到，它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。



```java
 /** 
     * Initializes table, using the size recorded in sizeCtl. 
     */  
    private final Node<K,V>[] initTable() {  
        Node<K,V>[] tab; int sc;  
        while ((tab = table) == null || tab.length == 0) {  
                //sizeCtl表示有其他线程正在进行初始化操作，把线程挂起。对于table的初始化工作，只能有一个线程在进行。  
            if ((sc = sizeCtl) < 0)  
                Thread.yield(); // lost initialization race; just spin  
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {//利用CAS方法把sizectl的值置为-1 表示本线程正在进行初始化  
                try {  
                    if ((tab = table) == null || tab.length == 0) {  
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;  
                        @SuppressWarnings("unchecked")  
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];  
                        table = tab = nt;  
                        sc = n - (n >>> 2);//相当于0.75*n 设置一个扩容的阈值  
                    }  
                } finally {  
                    sizeCtl = sc;  
                }  
                break;  
            }  
        }  
        return tab;  
    }  
```

#### **扩容方法transfer**

支持多线程进行扩容操作，并没有加锁 ，这样做的目的不仅仅是为了满足concurrent的要求，而是希望利用并发处理去减少扩容带来的时间影响。 
单线程扩容的大体思想就是遍历、复制的过程。首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素：

1. 如果这个位置为空，就在原table中的i位置放入forwardNode节点，这个也是触发并发扩容的关键点；
2. 如果这个位置是Node节点（fh>=0），如果它是一个链表的头节点，就构造一个反序链表，把他们分别放在nextTable的i和i+n的位置上
3. 如果这个位置是TreeBin节点（fh<0），也做一个反序处理，并且判断是否需要untreefi，把处理的结果分别放在nextTable的i和i+n的位置上
4. 遍历过所有的节点以后就完成了复制工作，这时让nextTable作为新的table，并且更新sizeCtl为新容量的0.75倍，完成扩容。

多线程遍历节点，处理了一个节点，就把对应点的值set为forward，另一个线程看到forward，就向后继续遍历，再加上给节点上锁的机制，就完成了多线程的控制。这样交叉就完成了复制工作。而且还很好的解决了线程安全的问题。 



![这里写图片描述](http://img.blog.csdn.net/20160721110522808)

```java
/**
   * 一个过渡的table表  只有在扩容的时候才会使用
   */
  private transient volatile Node<K,V>[] nextTable;

/**
   * Moves and/or copies the nodes in each bin to new table. See
   * above for explanation.
   */
  private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
      int n = tab.length, stride;
      if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
          stride = MIN_TRANSFER_STRIDE; // subdivide range
      if (nextTab == null) {            // initiating
          try {
              @SuppressWarnings("unchecked")
              Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];//构造一个nextTable对象 它的容量是原来的两倍
              nextTab = nt;
          } catch (Throwable ex) {      // try to cope with OOME
              sizeCtl = Integer.MAX_VALUE;
              return;
          }
          nextTable = nextTab;
          transferIndex = n;
      }
      int nextn = nextTab.length;
      ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);//构造一个连节点指针 用于标志位
      boolean advance = true;//并发扩容的关键属性 如果等于true 说明这个节点已经处理过
      boolean finishing = false; // to ensure sweep before committing nextTab
      for (int i = 0, bound = 0;;) {
          Node<K,V> f; int fh;
          //这个while循环体的作用就是在控制i--  通过i--可以依次遍历原hash表中的节点
          while (advance) {
              int nextIndex, nextBound;
              if (--i >= bound || finishing)
                  advance = false;
              else if ((nextIndex = transferIndex) <= 0) {
                  i = -1;
                  advance = false;
              }
              else if (U.compareAndSwapInt
                       (this, TRANSFERINDEX, nextIndex,
                        nextBound = (nextIndex > stride ?
                                     nextIndex - stride : 0))) {
                  bound = nextBound;
                  i = nextIndex - 1;
                  advance = false;
              }
          }
          if (i < 0 || i >= n || i + n >= nextn) {
              int sc;
              if (finishing) {
                //如果所有的节点都已经完成复制工作  就把nextTable赋值给table 清空临时对象nextTable
                  nextTable = null;
                  table = nextTab;
                  sizeCtl = (n << 1) - (n >>> 1);//扩容阈值设置为原来容量的1.5倍  依然相当于现在容量的0.75倍
                  return;
              }
              //利用CAS方法更新这个扩容阈值，在这里面sizectl值减一，说明新加入一个线程参与到扩容操作
              if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                  if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                      return;
                  finishing = advance = true;
                  i = n; // recheck before commit
              }
          }
          //如果遍历到的节点为空 则放入ForwardingNode指针
          else if ((f = tabAt(tab, i)) == null)
              advance = casTabAt(tab, i, null, fwd);
          //如果遍历到ForwardingNode节点  说明这个点已经被处理过了 直接跳过  这里是控制并发扩容的核心
          else if ((fh = f.hash) == MOVED)
              advance = true; // already processed
          else {
                //节点上锁
              synchronized (f) {
                  if (tabAt(tab, i) == f) {
                      Node<K,V> ln, hn;
                      //如果fh>=0 证明这是一个Node节点
                      if (fh >= 0) {
                          int runBit = fh & n;
                          //以下的部分在完成的工作是构造两个链表  一个是原链表  另一个是原链表的反序排列
                          Node<K,V> lastRun = f;
                          for (Node<K,V> p = f.next; p != null; p = p.next) {
                              int b = p.hash & n;
                              if (b != runBit) {
                                  runBit = b;
                                  lastRun = p;
                              }
                          }
                          if (runBit == 0) {
                              ln = lastRun;
                              hn = null;
                          }
                          else {
                              hn = lastRun;
                              ln = null;
                          }
                          for (Node<K,V> p = f; p != lastRun; p = p.next) {
                              int ph = p.hash; K pk = p.key; V pv = p.val;
                              if ((ph & n) == 0)
                                  ln = new Node<K,V>(ph, pk, pv, ln);
                              else
                                  hn = new Node<K,V>(ph, pk, pv, hn);
                          }
                          //在nextTable的i位置上插入一个链表
                          setTabAt(nextTab, i, ln);
                          //在nextTable的i+n的位置上插入另一个链表
                          setTabAt(nextTab, i + n, hn);
                          //在table的i位置上插入forwardNode节点  表示已经处理过该节点
                          setTabAt(tab, i, fwd);
                          //设置advance为true 返回到上面的while循环中 就可以执行i--操作
                          advance = true;
                      }
                      //对TreeBin对象进行处理  与上面的过程类似
                      else if (f instanceof TreeBin) {
                          TreeBin<K,V> t = (TreeBin<K,V>)f;
                          TreeNode<K,V> lo = null, loTail = null;
                          TreeNode<K,V> hi = null, hiTail = null;
                          int lc = 0, hc = 0;
                          //构造正序和反序两个链表
                          for (Node<K,V> e = t.first; e != null; e = e.next) {
                              int h = e.hash;
                              TreeNode<K,V> p = new TreeNode<K,V>
                                  (h, e.key, e.val, null, null);
                              if ((h & n) == 0) {
                                  if ((p.prev = loTail) == null)
                                      lo = p;
                                  else
                                      loTail.next = p;
                                  loTail = p;
                                  ++lc;
                              }
                              else {
                                  if ((p.prev = hiTail) == null)
                                      hi = p;
                                  else
                                      hiTail.next = p;
                                  hiTail = p;
                                  ++hc;
                              }
                          }
                          //如果扩容后已经不再需要tree的结构 反向转换为链表结构
                          ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                              (hc != 0) ? new TreeBin<K,V>(lo) : t;
                          hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                              (lc != 0) ? new TreeBin<K,V>(hi) : t;
                           //在nextTable的i位置上插入一个链表    
                          setTabAt(nextTab, i, ln);
                          //在nextTable的i+n的位置上插入另一个链表
                          setTabAt(nextTab, i + n, hn);
                           //在table的i位置上插入forwardNode节点  表示已经处理过该节点
                          setTabAt(tab, i, fwd);
                          //设置advance为true 返回到上面的while循环中 就可以执行i--操作
                          advance = true;
                      }
                  }
              }
          }
      }
  }
```

#### **put函数**

put方法依然沿用HashMap的put方法的思想，根据hash值计算这个新插入的点在table中的位置i，如果i位置是空的，直接放进去，否则进行判断，如果i位置是树节点，按照树的方式插入新的节点，否则把i插入到链表的末尾。ConcurrentHashMap中依然沿用这个思想，有一个最重要的不同点就是ConcurrentHashMap不允许key或value为null值。另外由于涉及到多线程，put方法就要复杂一点。在多线程中可能有以下两个情况：

1. 如果一个或多个线程正在对ConcurrentHashMap进行扩容操作，当前线程也要进入扩容的操作中。这个扩容的操作之所以能被检测到，是因为transfer方法中在空结点上插入forward节点，如果检测到需要插入的位置被forward节点占有，就帮助进行扩容。
2. 如果检测到要插入的节点是非空且不是forward节点，就对这个节点加锁，这样就保证了线程安全。尽管这个有一些影响效率，但是还是会比hashTable的synchronized要好得多。

整体流程就是首先定义不允许key或value为null的情况放入 对于每一个放入的值，首先利用spread方法对key的hashcode进行一次hash计算，由此来确定这个值在table中的位置。如果这个位置是空的，那么直接放入，而且不需要加锁操作。 
如果这个位置存在结点，说明发生了hash碰撞，首先判断这个节点的类型。如果是链表节点（fh>0）,则得到的结点就是hash值相同的节点组成的链表的头节点。需要依次向后遍历确定这个新加入的值所在位置。如果遇到hash值与key值都与新加入节点是一致的情况，则只需要更新value值即可。否则依次向后遍历，直到链表尾插入这个结点。 如果加入这个节点以后链表长度大于8，就把这个链表转换成红黑树。如果这个节点的类型已经是树节点的话，直接调用树节点的插入方法进行插入新的值。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
       //不允许 key或value为null  
       if (key == null || value == null) throw new NullPointerException();
       //计算hash值  
       int hash = spread(key.hashCode());
       int binCount = 0;
       for (Node<K,V>[] tab = table;;) {
           Node<K,V> f; int n, i, fh;
           // 第一次 put 操作的时候初始化，如果table为空的话，初始化table  
           if (tab == null || (n = tab.length) == 0)
               tab = initTable();

           //根据hash值计算出在table里面的位置   
           else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
               // 根据对应的key hash 到具体的索引，如果该索引对应的 Node 为 null，则采用 CAS 操作更新整个 table
               // 如果这个位置没有值 ，直接放进去，不需要加锁  
               if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                   break;                   // no lock when adding to empty bin
           }
           //当遇到表连接点时，需要进行整合表的操作  
           else if ((fh = f.hash) == MOVED)
               tab = helpTransfer(tab, f);
           else {
               V oldVal = null;
               // 结点上锁，只是对链表头结点作锁操作
               synchronized (f) {
                   if (tabAt(tab, i) == f) {
                       //fh > 0 说明这个节点是一个链表的节点 不是树的节点  
                       if (fh >= 0) {
                           binCount = 1;
                           //在这里遍历链表所有的结点  
                           for (Node<K,V> e = f;; ++binCount) {
                               K ek;
                               //如果hash值和key值相同  则修改对应结点的value值  
                               if (e.hash == hash &&
                                   ((ek = e.key) == key ||
                                    (ek != null && key.equals(ek)))) {
                                   oldVal = e.val;
                                   if (!onlyIfAbsent)
                                       e.val = value;
                                   break;
                               }
                               Node<K,V> pred = e;
                               //如果遍历到了最后一个结点，那么就证明新的节点需要插入 就把它插入在链表尾部  
                               if ((e = e.next) == null) {
                                   // 插入到链表尾
                                   pred.next = new Node<K,V>(hash, key,
                                                             value, null);
                                   break;
                               }
                           }
                       }
                       //如果这个节点是树节点，就按照树的方式插入值  
                       else if (f instanceof TreeBin) {
                           // 如果是红黑树结点，按照红黑树的插入
                           Node<K,V> p;
                           binCount = 2;
                           if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                          value)) != null) {
                               oldVal = p.val;
                               if (!onlyIfAbsent)
                                   p.val = value;
                           }
                       }
                   }
               }
               if (binCount != 0) {
                   // 如果这个链表结点达到了临界值8，那么把这个链表转换成红黑树
                   if (binCount >= TREEIFY_THRESHOLD)
                       treeifyBin(tab, i);
                   if (oldVal != null)
                       return oldVal;
                   break;
               }
           }
       }
       //将当前ConcurrentHashMap的元素数量+1，table的扩容是在这里发生的
       addCount(1L, binCount);
       return null;
   }
```

#### **协助扩容函数helpTransfer**

这是一个协助扩容的方法。这个方法被调用的时候，当前ConcurrentHashMap一定已经有了nextTable对象，首先拿到这个nextTable对象，调用上面讲到的transfer方法来进行扩容。

```java
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    if (tab != null && (f instanceof ForwardingNode) &&
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
        int rs = resizeStamp(tab.length);//计算一个操作校验码
        while (nextTab == nextTable && table == tab &&
               (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                transfer(tab, nextTab);
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```

#### **红黑树转换**

在putVal函数中，treeifyBin是在链表长度达到一定阈值（8）后转换成红黑树的函数。 但是并不是直接转换，而是进行一次容量判断，如果容量没有达到转换的要求，直接进行扩容操作并返回；如果满足条件才将链表的结构转换为TreeBin ，这与HashMap不同的是，它并没有把TreeNode直接放入红黑树，而是利用了TreeBin这个小容器来封装所有的TreeNode。

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```

#### **get方法**

给定一个key来确定value的时候，必须满足两个条件 key相同 hash值相同，对于节点可能在链表或树上的情况，需要分别去查找。

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    //计算hash值
    int h = spread(key.hashCode());
    //根据hash值确定节点位置
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        //如果搜索到的节点key与传入的key相同且不为null,直接返回这个节点  
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //如果eh<0 说明这个节点在树上 直接寻找
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
         //否则遍历链表 找到对应的值并返回
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

本文系转载，原文：http://blog.csdn.net/itachi85/article/details/51816668