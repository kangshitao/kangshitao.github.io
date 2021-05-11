---
title: JVM垃圾回收相关概念补充
excerpt: 内存溢出、内存泄露、STW、GC的并行与并发、强引用、软引用、弱引用、虚引用 
mathjax: true
date: 2021-05-11 18:09:54
tags: ['Java','JVM']
categories: JVM
keywords: Java垃圾回收，STW，Java四种引用，Java内存泄露与内存溢出
---



# 一、System.gc()的理解

默认情况下，调用`System.gc()`或者`Runtime.getRuntime().gc()`（`System.gc()`的底层方法）方法，会显式触发Full GC，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。

`System.gc()`调用附带一个免责声明，无法保证对垃圾收集器的调用，也就是说，这个方法只是提醒垃圾收集器进行垃圾回收，具体什么时候会进行垃圾回收就不一定了，此方法不保证一定会回收。

JVM实现者可以通过`System.gc()`调用来决定JVM的GC行为。而一般情况下，垃圾回收应该是自动进行的，无须手动触发，否则就太麻烦了。在一些特殊情况下，如我们正在编写一个性能基准，我们可以在运行之间调用`System.gc()`。

例一：

```java
public class SystemGCTest {
    public static void main(String[] args) {
        new SystemGCTest();
        System.gc();//提醒jvm的垃圾回收器执行gc,但是不确定是否马上执行gc
        //与Runtime.getRuntime().gc();的作用一样。

        //此方法会强制调用失去引用对象的finalize()方法
        //System.runFinalization();
    }

    @Override
    protected void finalize() throws Throwable {
        System.out.println("SystemGCTest 重写了finalize()");
    }
}
/*
如果不使用System.runFinalization()方法，由于System.gc()不保证一定执行垃圾回收，
因此执行上述代码，finalize()方法可能会被调用，也可能不会被调用
*/
```



例二，调用`System.gc()`进行垃圾回收：

```java
public class LocalVarGC {
    //buffer不会被回收
    public void localvarGC1() {
        byte[] buffer = new byte[10 * 1024 * 1024];//10MB
        System.gc(); 
    }

    //buffer会被回收
    public void localvarGC2() {
        byte[] buffer = new byte[10 * 1024 * 1024];
        buffer = null;
        System.gc();
    }
    
    /*buffer不会被回收。
    查看字节码文件可知，局部变量表中存放了this和buffer两个变量,
    垃圾回收时发现buffer在变量表中，属于GC roots，因此不会被回收
    */
    public void localvarGC3() {
        {
            byte[] buffer = new byte[10 * 1024 * 1024];
        }
        System.gc();
    }

    /*buffer会被回收。
    查看字节码文件可知，首先局部变量表中存放了this和buffer两个变量,
    然后，在代码块执行完后，超出了buffer的作用范围，value重用了buffer的变量槽
    因此垃圾回收时，buffer会被回收。
    可以参考《深入理解JVM 第三版》P296
    */
    public void localvarGC4() {
        {
            byte[] buffer = new byte[10 * 1024 * 1024];
        }
        int value = 10;
        System.gc();
    }

    //在localvarGC1中时，buffer没有被回收，localvarGC5中，buffer被回收。
    public void localvarGC5() {
        localvarGC1();
        System.gc();
    }

    public static void main(String[] args) {
        LocalVarGC local = new LocalVarGC();
        local.localvarGC1();
    }
}
```



# 二、内存溢出与内存泄漏

## 1、内存溢出

内存溢出（OutOfMemoryError）：没有空间内存，并且垃圾收集器也无法提供更多内存。

JVM堆、方法区、虚拟机栈和本地方法栈都可能出现OOM问题，其中堆内存溢出的情况最为常见

> 栈除了存在OOM的情况，当还存在StackOverflow的情况。如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常；如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出OOM异常。也就是说，如果栈不支持扩展，除非在创建线程申请内存时就因无法获得足够内存而出现OOM异常，否则在线程运行时是不会因为扩展而导致内存溢出的，只会因为栈容量无法容纳新的栈帧而导致StackOverflowError异常。

导致OOM的原因有：

* **Java虚拟机的堆内存设置不够**。可以通过`-Xms`、`-Xmx`参数来调整

* **代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）**。比如JDK6之前的永久代，空间很小，且使用的是JVM内存，容易出现OOM问题。

  > 在抛出OOM之前，通常会触发一次GC，尽其所能去清理出空间。例如在引用机制分析中，涉及到JVM会去尝试回收软引用指向的对象等。在java.nio.BIts.reserveMemory()方法中，我们能清楚的看到，System.gc()会被调用，以清理空间。

* 不是在任何情况下垃圾收集器都会被触发。比如，当分配一个超大对象，其所需空间超过堆的最大值时，JVM可以判断出垃圾收集并不能解决这个问题，所以直接抛出OutOfMemoryError。

## 2、内存泄漏

内存泄露（Memory Leak），也称作“存储渗漏”，**严格上来讲**，内存泄漏指的是对象不会被程序使用了，但是GC又不能回收他们。比如不使用的对象没有断开和GC Roots引用链的连接，所以这些对象还是可达状态，无法被回收。

Java中会出现的内存泄漏情况：

* 单例模式。单例的生命周期和应用程序是一样长的，所以单例程序中，如果持有对外部对象的引用的话，那么这个外部对象是不能被回收的，则会导致内存泄漏的产生。`java.lang.Runtime`类就是单例模式。
* 一些提供close的资源未关闭导致内存泄漏。比如数据库连接（`dataSourse.getConnection()`)，网络连接(`socket`)和IO连接必须手动close，否则是不能被回收的。

> 引用计数算法无法解决循环引用问题，会导致内存泄露，但是Java中不会出现这种内存泄漏，因为Java没有使用引用计数算法。



实际情况中，一些不太好的实践或疏忽导致对象的生命周期变得很长甚至导致OOM，也可以叫做*宽泛意义上的“内存泄漏”*。

尽管内存泄漏并不会立刻引起程序崩溃，但是一旦发生内存泄漏，程序中的可用内存就会被逐步蚕食，直至耗尽所有内存，最终出现OOM异常，导致程序崩溃。

> 这里的存储空间并不是指物理内存，而是指虚拟内存大小，这个虚拟内存大小取决于磁盘交换区设定的大小。 

内存泄露并不一定会（或者说立即）导致内存溢出。



# 三、Stop The World

Stop-The-World，简称STW，指的是GC事件发生过程中，会产生应用程序的停顿。**停顿产生时整个应用程序所有工作线程都会被暂停，没有任何响应**，有点像卡死的感觉，这个停顿称为STW。比如：

* 可达性分析算法中枚举根节点(GC Roots)会导致所有Java执行线程停顿。
  * 分析工作必须在一个能确保一致性的快照中进行。
  * 一致性指整个分析期间整个执行系统看起来像被冻结在某个时间点上。
  * 如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证。

被STW中断的应用程序线程会在完成GC之后恢复，频繁中断会让用户感觉像是网速不快造成电影卡带一样，影响用户体验。所以我们需要减少STW的发生。

STW的发生和采用哪款GC无关，所有的GC都有这种情况。即使是号称停顿时间可控，或者（几乎）不会发生停顿的CMS、G1、ZGC收集器也不能完全避免STW情况发生，枚举根节点时也必须要停顿的。只能说垃圾回收器越来越优秀，回收效率越来越高，尽可能地缩短了暂停时间。

STW是JVM在后台自动发起和自动完成的。在用户不可见的情况下，把用户正常的工作线程全部停掉。

开发中不要用`system.gc ();`，这个方法会触发Full GC，进而导致STW的发生。



# 四、垃圾回收的并行与并发

## 1、并发

并发(Concurrent)是指**一个时间段中**有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在**同一个处理器**上运行。

并发不是真正意义上的同时进行，只是CPU把一个时间段划分成几个时间片段(时间区间)，然后在这几个时间区间之间来回切换，由于CPU处理的速度非常快，只要时间间隔处理得当，即可让用户感觉是多个应用程序同时在进行。

> 在某个时刻，实际只有一个程序正在运行。

比如：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/gc-related-concepts-supplement_1.png"/>



## 2、并行

并行(Parallel)是指当系统有一个以上CPU时，当一个CPU执行一个进程时，另一个CPU可以执行另一个进程，两个进程互不抢占CPU资源，可以同时进行。

决定并行的因素不是CPU的数量，而是CPU的核心数量，比如一个CPU多个核也可以并行。

并行适合科学计算，后台处理等弱交互场景。

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/gc-related-concepts-supplement_2.png"/>



并行和并发的对比：

* 并发指的是**多个事件在同一时间段内同时发生**。并行指的是**多个事件在同一时刻同时发生**。
* 并发的多个任务之间是互相抢占资源的。并行的多个任务之间是不互相抢占资源的。
* 只有在多CPU或者一个CPU多核的情况中，才会发生并行。否则，看似同时发注的事情，其实都是并发执行的。



## 3、垃圾回收的并行和并发

对于垃圾收集器来说，并行和并发的概念具体指的是：

* 并行(Parallel)：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。如ParNew、Parallel scavenge、Parallel old收集器。
* 串行(Serial)：相较于并行的概念，单线程收集。如果内存不够，则程序暂停，启动JVM垃圾回收器进行垃圾回收。回收完，再启动
  程序的线程。比如Serial、Serial Old收集器。
* 并发(Concurrent)：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行)，垃圾回收线程在执行时不会停顿用户程序的运行。用户程序在继续运行，而垃圾收集程序线程运行于另一个CPU上。比如CMS、G1收集器。



# 五、再谈引用

JDK1.2之前，Java对象只有“被引用”和“未被引用”两种状态，这些对象要么被回收，要么不能被回收。对于那些“食之无味弃之可惜”的对象，比如我们希望能描述这样一种对象：当内存空间足够时，能保存在内存中，如果内存空间在进行垃圾回收后仍然非常紧张，那就可以抛弃这些对象，类似于缓存功能，这种情况下只有“被引用”和“未被引用”两种状态是不够的。

JDK 1.2之后，Java对引用的概念进行了扩充，将引用分为强引用(Strongly Reference)、软引用(Soft Reference)、弱引用(Weak Reference)和虚引用(Phantom Reference)4种，这4种引用强度依次逐渐减弱。

如图（其中FinalReference表示终结器引用）：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/gc-related-concepts-supplement_3.png"/>



4种引用关系概述：

* **强引用(StrongReference)**：最传统的“引用”的定义，是指在程序代码之中普遍存在的引用赋值，即类似`Object obj=new O bject()`这种引用关系。无论任何情况下只要强引用关系还存在，垃圾收集器永远不会回收掉被强引用的对象。**不回收**
* **软引用(SoftReference)**：用于描述一些“还有用，但非必须的对象”。被软引用关联着的对象，在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收后还没有足够的内存，才会抛出内存溢出异常。**内存不足即回收**
* **弱引用（weakReference)**：用于描述非必须对象。被弱引用关联的对象只能生存到下一次垃圾收集之前。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。**发现即回收**
* **虚引用(PhantomReference)**：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。**对象回收跟踪**



## 1、强引用

强引用关联对象的主要特点是**不回收**。

在Java程序中，最常见的引用类型是强引用（普通系统99%以上都是强引用)，也就是我们最常见的普通对象引用，也是默认的引用类型。

当在Java语言中使用new操作符创建一个新的对象，并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。

**强引用的对象是可触及的（可达的），垃圾收集器永远不会回收强引用的对象**。

对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为`null`，就是可以当做垃圾被收集了，当然具体回收时机还是要看垃圾收集策略。

相对的，软引用、弱引用和虚引用的对象是软可触及、弱可触及和虚可触及的，在一定条件下，都是可以被回收的。所以，**强引用是造成Java内存泄漏的主要原因之一**。

看下面的例子：

```java
public class StrongReferenceTest {
    public static void main(String[] args) {
        StringBuffer str = new StringBuffer ("Hello");
        StringBuffer str1 = str;
        
        str = null;//取消str对“Hello”的引用
        System.gc(); 
        
        try {//暂停一段时间，保证GC会被触发
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //“Hello”对象不会被回收，因此可以打印出内容
        System.out.println(str1); //"Hello"
    }
}
```

上述程序执行时，主动触发GC机制，正常来讲，GC会将str关联的“Hello”对象回收，但“Hello”被str1强引用关联着，因此实际上“Hello”对象不会被回收。

本例中的两个引用，都是强引用，强引用具备以下特点：

* 强引用可以直接访问目标对象。
* 强引用所指向的对象在**任何时候都不会被系统回收**，虚拟机宁可抛出OOM异常，也不会回收强引用所指向对象。
* 强引用可能导致内存泄漏。



## 2、软引用

软引用关联对象的主要特点是**内存不足即回收**， 内存足够时不会回收软引用关联的对象。

软引用是用来描述一些还有用，但非必需的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收（第一次回收是指回收不可达对象），如果这次回收还没有足够的内存，才会抛出内存溢出异常。

> 这时出现OOM异常，并不是软引用对象导致的，因为软引用对象二次回收已经被回收掉了。



软引用通常用来实现内存敏感的缓存。比如高速缓存，如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。

垃圾回收器在某个时刻决定回收软可达的对象的时候，会清理软引用，并可选地把引用存放到一个引用队列(Reference Queue) 。

类似弱引用，区别是Java虚拟机会尽量让软引用的存活时间长一些，迫不得已才清理。

案例：

```java
public class SoftReferenceTest {
    public static class User {
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public int id;
        public String name;

        @Override
        public String toString() {
            return "[id=" + id + ", name=" + name + "] ";
        }
    }

    public static void main(String[] args) {
        //创建对象，建立软引用
        SoftReference<User> userSoftRef = 
            new SoftReference<User>(new User(1, "John"));
        /*上面的一行代码，等价于如下的三行代码
        User u1 = new User(1,"songhk");//先建立一个强引用
        //将创建软引用，指向强引用关联的对象
        SoftReference<User> userSoftRef = new SoftReference<User>(u1);
        u1 = null;//取消强引用，只保留软引用对对象的关联
        */

        //从软引用中获得对象
        System.out.println(userSoftRef.get());//[id=1, name=John]

        System.gc();
        System.out.println("After GC:");
        //由于堆空间内存足够，不会回收软引用的可达对象。
        System.out.println(userSoftRef.get());
        try {//模拟堆内存不足的情况，-Xms10m,-Xmx10m
            //会发生OOM的情况
            byte[] b = new byte[1024 * 1024 * 7];
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            //再次从软引用中获取数据
            //OOM之前，垃圾回收器会回收软引用的可达对象。输出null
            System.out.println(userSoftRef.get());//null
        }
    }
}
```

执行以上程序，发现当内存足够时，软引用关联的对象并没有被回收，接下来试图创建大对象，导致堆空间内存不足，触发GC，这时会将软引用关联的对象回收掉。最后的结果输出为`null`，说明软引用对象确实被回收掉了。

> 并不是要发生OOM时才会回收软引用关联的对象，只要堆内存不能存储软引用关联的对象时，就会将软引用关联的对象回收。



## 3、弱引用

弱引用关联对象的主要特点是**发现即回收**。

弱引用也是用来描述那些非必需对象，**只被弱引用关联的对象只能生存到下一次垃圾收集发生为止**。在系统GC时，**只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象**。

但是，由于垃圾回收器的线程通常优先级很低，因此，并不一定能很快地发现持有弱引用的对象。在这种情况下，弱引用对象可以存在较长的时间。

弱引用和软引用一样，在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收情况。

软引用、弱引用都非常适合来保存那些可有可无的缓存数据。如果这么做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用。

弱引用的案例：

```java
public class WeakReferenceTest {
    public static void main(String[] args) {
        //构造了弱引用
        WeakReference<User> userWeakRef = 
            new WeakReference<User>(new User(1, "John"));
        //从弱引用中获取对象
        System.out.println(userWeakRef.get()); //[id=1, name=John]

        System.gc();
        //不管当前内存空间足够与否，都会回收它的内存
        System.out.println("After GC:");
        //重新尝试从弱引用中获取对象
        System.out.println(userWeakRef.get()); //null
    }
}
```

执行上述代码，最后一行输出结果为`null`，说明弱引用关联的对象被回收了。只要GC回收执行，弱引用关联的对象就会被回收。



## 4、虚引用

虚引用关联对象的主要特点是**对象回收跟踪**。

虚引用也称为“幽灵引用”或者“幻影引用”，是所有引用类型中最弱的一个。

一个对象是否有虚引用的存在，完全不会决定对象的生命周期。如果一个对象**仅持有虚引用，那么它和没有引用几乎是一样的，随时都可能被垃圾回收器回收**。

它不能单独使用，也**无法通过虚引用来获取被引用的对象**。当试图通过虚引用的get()方法取得对象时，总是nu11。

**为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程**。比如:能在这个对象被收集器回收时收到一个系统通知。

虚引用必须和**引用队列**一起使用。虚引用在创建时必须提供一个引用队列作为参数。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象后，将这个虚引用加入引用队列，以通知应用程序对象的回收情况。

由于虚引用可以跟踪对象的回收时间，因此，也可以将一些资源释放操作放置在虚引用中执行和记录。

案例：

```java
public class PhantomReferenceTest {
    public static PhantomReferenceTest obj;//当前类对象的声明
    static ReferenceQueue<PhantomReferenceTest> phantomQueue = null;//引用队列

    //创建线程，一直检测引用队列，如果虚引用对象被回收了，引用队列不为空，打印回收记录
    public static class CheckRefQueue extends Thread {
        @Override
        public void run() {
            while (true) {
                if (phantomQueue != null) {
                    PhantomReference<PhantomReferenceTest> objt = null;
                    try {
                        objt = (PhantomReference<PhantomReferenceTest>) phantomQueue.remove();
                    } catch (InterruptedException e) {e.printStackTrace();}
                    if (objt != null) {
                        System.out.println("追踪垃圾回收过程：PhantomReferenceTest实例被GC了");
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        Thread t = new CheckRefQueue();
        t.setDaemon(true);//设置为守护线程：当程序中没有非守护线程时，守护线程也就执行结束。
        t.start();

        phantomQueue = new ReferenceQueue<PhantomReferenceTest>();
        obj = new PhantomReferenceTest();
        //构造了 PhantomReferenceTest 对象的虚引用，并指定了引用队列
        PhantomReference<PhantomReferenceTest> phantomRef = new 
            PhantomReference<PhantomReferenceTest>(obj, phantomQueue);

        try {
            //无法通过虚引用获取其关联的对象
            System.out.println(phantomRef.get()); //null
            //将强引用去除
            obj = null;
            System.out.println("调用GC");
            System.gc(); //一旦将obj对象回收，就会将此虚引用存放到引用队列中。
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 可用");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
/*执行结果
null
调用GC
追踪垃圾回收过程：PhantomReferenceTest实例被GC了
obj 是 null
*/
```



## 5、终结器引用

**终结器引用（Final Reference）**用以实现对象的`finalize ()`方法。无需手动编码，其内部配合引用队列使用。

在GC时，终结器引用入队。由`Finalizer`线程通过终结器引用找到被引用对象并调用它的`finalize ()`方法，第二次GC时才能回收被引用对象。



