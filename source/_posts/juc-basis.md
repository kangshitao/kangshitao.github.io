---
title: Java并发编程-JUC基础
excerpt: JUC、锁、多线程、并发编程、阻塞队列、线程池等
mathjax: true
date: 2021-07-25 22:30:24
tags: ['JUC','Java']
categories: 并发编程
keywords: JUC,Lock,synchronized,阻塞队列,线程池
---



# 一、JUC基础

## 1.1 JUC简介

` java.util.concurrent`简称JUC，是一个处理线程的工具包，JDK 1.5出现。



## 1.2 进程与线程

参考[进程与线程](https://kangshitao.github.io/2021/05/21/computer-os-review/#%E4%BA%8C%E3%80%81%E8%BF%9B%E7%A8%8B%E4%B8%8E%E7%BA%BF%E7%A8%8B)

`Thread`类中定义的进程几种状态：

* `NEW`：线程还未开始时的状态
* `RUNNABLE`：线程可运行的状态（就绪状态）
* `BLOCKED`：线程为了进入同步方法或同步代码块，而等待监视器锁(monitor lock)的状态。
* `WAITING`：由于调用`wait`\`join`\`LockSupport.park`方法进入的等待状态，直到被唤醒为止。
* `TIMED_WAITING`：由于调用`wait`\`join`\`LockSupport.parkNanos`\`LockSupport.Until`方法进入的超时等待状态，等待具体的时间后就会**自动停止等待。**
* `TERMINATED`：终止状态，线程执行完成。

## 1.3 多线程锁

**1、平锁和非公平锁**

* **非公平锁**：可能造成一个线程独占资源，其他线程被饿死的情况；但效率高

* **公平锁**：只有当前线程在等待队列的第一个位置时，才会执行；各个线程都能获取到锁；效率较低.



**2、可重入锁**

**“可重入锁”**（递归锁） 指的是自己可以再次获取自己的内部锁。进入最外层的锁，内层也可以直接获取当前的锁，不需要等外层释放锁。

`synchronized`和`Lock`都是可重入锁.

`synchronized`是隐式的；`Lock`是显式的。Lock的上锁和解锁必须成对出现，如果没有解锁，可能会影响别的线程。



**3、死锁**

两个或两个以上的进程在执行过程中，因为争夺资源而造成互相等待的现象；如果没有外力干涉，他们无法继续进行下去。

产生死锁的原因：系统资源不足；进程运行推进顺序不合适；资源分配不当。死锁的产生条件和解决方法：[操作系统-死锁](https://kangshitao.github.io/2021/05/21/computer-os-review/#2-7-%E6%AD%BB%E9%94%81)

验证是否是死锁：

* jps命令找到要验证的进程。
* jstack对指定id的进行进行分析，这是jvm自带的堆栈跟踪工具。



## 1.4 wait/sleep

`wait`和`sleep`的区别：

* `sleep`是Thread类的静态方法，而`wait`是Object类的方法，任何对象实例都能调用。
* `sleep`不会释放锁，调用它不要求必须占用锁，因此可以在任何地方使用。`wait`会释放锁，但调用它的前提是当前线程已经占有锁(即代码要在`synchronized`中，`Lock`锁需要使用`Condition`的`await`方法)。
* 他们都可以被`interrupted`方法中断。
* 他们都是在哪里执行，就在哪里醒来。即原地唤醒。

> 为什么sleep方法定义在Thread类中，而wait方法定义在Object类中？
>
> 调用对象的wait方法，会导致**当前线程**进入等待状态，并且释放这个锁，前提是要求当前线程必须持有这个对象锁。二者的施加者是不同的；二者本质区别是一个是线程的运行状态控制，一个是线程之间的通讯问题。



## 1.5 并发和并行

**并发（concurrent）**指的是多个程序或进程同时运行，对于单核心CPU来说，同一时刻只能允许一个线程，其并发就表示多个线程采用时间片方式执行多个线程。**同一时刻多个线程访问同一个资源，多个线程对一个点**，比如抢票、电商秒杀。

**并行**是指多项工作一起执行，之后再汇总。





## 1.6 多线程编程

Java创建线程的几种方式：[线程的创建和使用](https://kangshitao.github.io/2021/04/03/java-note-0801/#%E4%BA%8C%E3%80%81%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%88%9B%E5%BB%BA%E5%92%8C%E4%BD%BF%E7%94%A8)，主要有以下四种：

* 继承Thread类
* 实现Runnable接口
* 实现Callable接口
* 使用线程池



Java多线程编程的一般步骤：

1、创建资源类，定义属性和操作方法

2、在资源类中操作方法：判断、操作、通知

3、创建多个线程，调用资源类的操作方法

4、防止虚假唤醒问题，将`wait`使用在`while`中，这样每次唤醒后都做判断。

> 如果在if中使用wait，这样只会判断一次，之后唤醒就不会再判断了





# 二、synchronized与Lock接口

## 2.1 synchronized

synchronized是Java中的关键字，是一种同步锁。使用方法参考：[线程的同步](https://kangshitao.github.io/2021/04/03/java-note-0801/#%E5%9B%9B%E3%80%81%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%90%8C%E6%AD%A5)

补充：synchronized并不属于方法定义的一部分，因此synchronized关键字不能被继承。如果在父类中的某个方法使用了synchronized关键字，而在子类中覆盖了这个方法，在子类中的这个方法默认情况下并不是同步的，而必须显式地在子类的这个方法中加上synchronized关键字才可以。

当然，还可以在子类方法中调用父类中相应的方法，这样虽然子类中的方法不是同步的，但子类调用了父类的同步方法，因此，子类的方法也就相当于同步了。



**关于synchronized锁**

* 如果一个类中有多个`synchronized`方法，对于类的对象，某一时刻内，只能有一个线程调用其中的一个同步方法，其他线程都必须等待。因为锁的是当前对象`this`，当前对象被锁定后，其他对象无法获取这个锁，也就无法使用其中的方法。
* 如果是一个类的两个不同的对象作为锁，则二者互不影响。`synchronized`实现同步的基础是**每个对象都可以作为锁**。
* 对于**普通同步方法**，锁的是当前实例对象。
* 对于**静态同步方法**，锁的是当前类的`Class`对象
* 对于**同步方法块**，锁的是括号内指定的对象。
* 对于`synchronized`的锁问题，需要判断其锁对象是否相同来分析，如果多个线程要用同一把锁，必然有竞态条件，如果用的不是同一把锁，则不会有关系。比如一个类的普通同步方法和一个静态同步方法，如果两个线程一个要访问普通同步方法，另一个线程访问静态同步方法，二者不会竞争，因为用的不是同一把锁。



## 2.2 Lock接口

### 2.2.1 Lock

Lock接口是使用最多的方式，作用是用来**手动获取和释放锁**。

一般来说Lock的加锁语句放在try结构外面，这样如果加锁失败，就不会执行finally中的释放锁的操作。并且，需要在加锁和try之间不建议有别的操作，如果有异常会导致不能释放锁。参考[Lock.lock()为什么在try之前执行](https://blog.csdn.net/E_N_T_J/article/details/105943325)

释放锁语句放在finally里面，保证锁一定会被释放，防止死锁的发生。



### 2.2.2 Condition

如果使用Lock锁，想要实现类似于notify方法的功能，就要使用Condition接口类对象。

```java
Lock lock = new ReentrantLock();   //创建Lock对象
Condition condition = lock.newCondition();  //获取Condition对象
```

Condition中的两个常用方法：

* `await()`：相当于`wait()`，会使当前线程等待，同时会释放锁，其他线程调用当前Condition对象的`signal()`方法时才会继续执行。

* `signal()`相当于`notify()`。

调用这两个方法前，需要线程持有相关的Lock锁，调用`await()`后，线程会释放这个锁，而调用某个Condition对象的`signal()`方法时，会从当前Condition对象的等待队列中，唤醒一个线程，当某个线程获取锁成功就会继续执行。



## 2.3 ReentryantLock

`ReentryantLock`表示可重入锁，是Lock的一个实现类，我们可以用它声明一个Lock对象。

使用`ReentryantLock`可以指定公平锁和非公平锁。





## 2.4 synchronized和Lock的区别



* synchronized是Java语言的关键字，是Java语言内置的。而Lock是一个接口，通过这个类可以实现同步访问；

* **synchronized不需要手动释放锁**，同步方法或同步代码块执行完之后，系统会自动让线程释放对锁的占用；**Lock必须手动释放锁**，如果没有主动释放锁，就有可能导致出现死锁现象。
* Lock可以让等待锁的线程响应中断，synchronized不能，会导致线程一直等待下去。
* Lock可以知道有没有成功获取锁，synchronized不能。
* 竞争资源激烈时，Lock的性能优于synchronized

* 二者都是独占锁。



## 2.5 案例：多个售票员售票

模拟3个售票员售30张票，售完为止：

`synchronized`方式

```java
//创建资源类
class Ticket{
    private int number = 30;//定义属性：票数
    //操作方法
    public synchronized void sale(){
        //判断：是否还有票
        if(number>0){
            System.out.print("当前"+number+"张--");
            number--;
            System.out.println(Thread.currentThread().getName()+": 卖出1，剩余"+number+"张");
        }
    }
}

public class SaleTicket {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        //创建资源类对象
        //创建3个线程，代表3个售票员
        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i=0;i<30;i++){
                    ticket.sale();
                }
            }
        },"售票员A").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i=0;i<30;i++){
                    ticket.sale();
                }
            }
        },"售票员B").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i=0;i<30;i++){
                    ticket.sale();
                }
            }
        },"售票员C").start();
    }

```

`Lock`锁的方式：

```java
class LTicket{
    private int number=30;
    //创建可重入锁对象
    private final ReentrantLock lock = new ReentrantLock();
    //编写方法
    public void sale(){
        //上锁
        lock.lock();  //一般将上锁操作写在try外面,且和try直接不能有其他操作
        try{
            //做操作
            if(number>0){
                System.out.print("当前"+number+"张--");
                number--;
                System.out.println(Thread.currentThread().getName()+": 卖出1，剩余"+number+"张");
            }
        }finally {  //将解锁操作放到finally中，保证一定能够解锁
            //解锁
            lock.unlock();
        }
    }
}

public class SaleTicket_L {
    public static void main(String[] args) {
        LTicket lTicket = new LTicket();
        //调用start方法不一定是马上创建线程，需要底层操作系统情况。
        new Thread(()->{for(int i=0;i<30;i++) lTicket.sale();},"售票员A").start();
        new Thread(()->{for(int i=0;i<30;i++) lTicket.sale();},"售票员B").start();
        new Thread(()->{for(int i=0;i<30;i++) lTicket.sale();},"售票员C").start();
    }
}
```







# 三、线程通信



线程通信主要是通过`wait()、notify()、notifyall()`方法，或者是`Condition`的`await()、siginal()、signalAll()`这些方法。前者适用于`sychronized`方式，而后者是`Lock`锁的方式。



## 3.1 案例：线程通信

创建两个线程A和B，一个线程对当前数值+1，另一个线程对当前数字-1，利用线程间的通信，使这两个线程交替操作。

方式一：使用`synchronized`：

```java
//一、创建资源类，定义属性和操作方法
//操作方法中判断、操作、通知
class Share{
    //初始值
    private int number = 0;
    //+1的方法
    public synchronized void increat() throws InterruptedException {
        /*
        if只会判断一次，wait在哪里睡就在哪里醒，因此如果使用if，则会导致虚假唤醒问题。
        解决虚假唤醒问题，需要使用while代替if，因为while每次都会判断
         */
        while(number !=0){
            wait(); //如果值不是0，就释放锁,进入阻塞状态，等待被唤醒
        }
        number++;
        System.out.println(Thread.currentThread().getName()+"::"+number);
        notifyAll();
    }
    //-1的方法
    public synchronized void decreat() throws InterruptedException {
        while(number !=1){
            wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName()+"::"+number);
        notifyAll();
    }

}
public class ThreadDemo1 {
    public static void main(String[] args) {
        Share share = new Share();
        new Thread(() -> {
            for(int i=1;i<=10;i++){
                try {
                    share.increat();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"Thread A").start();
        new Thread(() -> {
            for(int i=1;i<=10;i++){
                try {
                    share.decreat();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"Thread B").start();
    }
}
```

方式二：使用`Lock`锁：

```java
//第一步：创建资源类，定义属性和操作方法
class Share{
    private int number = 0;
    //创建Lock对象
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    //定义方法
    public void incr() throws InterruptedException {
        lock.lock();//上锁
        try {
            while(number!= 0){condition.await();}//判断
            number++;//操作
            System.out.println(Thread.currentThread().getName()+"::"+number);
            condition.signalAll();//通知
        } finally {
            lock.unlock();//解锁
        }
    }

    public void decr() throws InterruptedException {
        lock.lock();
        try{
            while(number!=1){condition.await();}
            number--;
            System.out.println(Thread.currentThread().getName()+"::"+number);
            condition.signalAll();
        }finally {
            lock.unlock();
        }
    }
}
public class ThreadDemo1_L {
    public static void main(String[] args) {
        Share share = new Share();
        //创建A、B两个线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i=0;i<10;i++){
                    try {
                        share.incr();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        },"Thread A").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i=0;i<10;i++){
                    try {
                        share.decr();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        },"Thread B").start();
    }
}
```



## 3.2 案例：线程间定制化通信

一般的线程通信我们不能控制他们的执行次数和顺序，如果想要实现多个线程按照指定的顺序执行，就需要使用定制化通信。比如下面的例子。

案例：A线程打印5次，B线程打印10次，C线程打印15次，重复以上步骤n次

```java
/*线程通信案例2：
实现线程间的定制化通信：A线程打印5次，B线程打印10次，C线程打印15次，重复以上步骤n次
实现方法为设置标志位flag，1表示A执行，2表示B执行，3表示C执行
 */
class ShareResource{
    private int flag = 1;  // 1-A,2-B,3-C
    private Lock lock = new ReentrantLock(); //声明锁
    //为了实现定制化通信，需要创建3个condition；保证唤醒的是指定的线程
    private Condition c1 =  lock.newCondition();  //相当于钥匙，用于通信
    private Condition c2 =  lock.newCondition();
    private Condition c3 =  lock.newCondition();

    public void print5(int loop) throws InterruptedException {
        lock.lock();
        try{
            while(flag!=1){
                c1.await();
            }
            for(int i=0;i<5;i++){
                System.out.println(Thread.currentThread().getName()+"::"+i+",当前轮数:"+loop);
            }
            flag = 2; //修改标志位是2
            c2.signal();  //通知B线程；signal表示唤醒当前Condition对象中的等待队列中的一个线程。
        }finally {
            lock.unlock();
        }
    }

    public void print10(int loop) throws InterruptedException {
        lock.lock();
        try{
            while(flag!=2){
                c2.await();
            }
            for(int i=0;i<10;i++){
                System.out.println(Thread.currentThread().getName()+"::"+i+",当前轮数:"+loop);
            }
            flag = 3; //修改标志位是2
            c3.signal();  //通知C线程；signal表示唤醒当前Condition对象中的等待队列中的一个线程。
        }finally {
            lock.unlock();
        }
    }

    public void print15(int loop) throws InterruptedException {
        lock.lock();
        try{
            while(flag!=3){
                c3.await();
            }
            for(int i=0;i<15;i++){
                System.out.println(Thread.currentThread().getName()+"::"+i+",当前轮数:"+loop);
            }
            flag = 1; //修改标志位是2
            c1.signal();  //通知A线程；signal表示唤醒当前Condition对象中的等待队列中的一个线程。
        }finally {
            lock.unlock();
        }
    }
}

public class ThreadDemo2_L {
    public static void main(String[] args) {
        ShareResource shareResource = new ShareResource();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i=0;i<10;i++){
                    try {
                        shareResource.print5(i);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        },"A").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i=0;i<10;i++){
                    try {
                        shareResource.print10(i);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        },"B").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i=0;i<10;i++){
                    try {
                        shareResource.print15(i);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        },"C").start();
    }
}
```





# 四、集合的线程安全

## 4.1 集合线程安全问题

对于非线程安全的集合类，如果多个线程对其进行并发读写，就会出现并发异常：

```java
public class ThreadDemo4_L {
    public static void main(String[] args) {
        ThreadDemo4_L demo4L = new ThreadDemo4_L();
        demo4L.testHashMap();
    }
    public void testHashMap(){
        Map<String,String> map = new HashMap<>();
        for(int i=1;i<=100;i++){
            new Thread(new Runnable() {
                @Override
                public void run() {
                    //向hashmap中写入随机数
                    map.put(UUID.randomUUID().toString().substring(0,8),"t");
                    System.out.println(map);
                }
            },String.valueOf(i)).start();
        }
    }
}
```

以上程序，并发情况下，对HashMap进行读写，会出现`ConcurrentModificationException`异常，表示出现了读写冲突。

对于不同的集合类，一般有对应的线程安全类，或者是其他的解决方法。



## 4.2 ArrayList

ArrayList是线程不安全的，如果多个线程同时添加数据，读取的时候就会出现`ConcurrentModificationException`异常。

解决方法：

* 使用`Vector`代替`ArrayList`
* 使用`Collections`中的`synchronizedList`代替`ArrayList`
* 使用`CopyOnWriteArrayList`代替`ArrayList`



`CopyOnWriteArrayList`是JUC包下的一个`ArrayList`线程安全类，其功能和`ArrayList`一样，但是其使用**写时复制**的思想，保证了其是**线程安全**的，有以下特点：

* 适合于**读操作多于写操作**、**List较小**的情况。
* 写时复制，在进行写操作的时候，会复制一份，在复制出来的容器中添加元素，添加完以后，将原容器的引用指向新的容器。
* `CopyOnWriteArrayList`底层使用`volatile`修饰数组，保证当前线程读的时候总能看到线程对数组最后的写入。避免了脏数据读取。
* `CopyOnWriteArrayList`使用互斥锁来保护数据，在“添加、修改、删除”数据时，会先获取互斥锁，修改完毕后，先将数据更新到volatile数组中，然后释放锁，以此保护数据。



## 4.3 HashSet

JUC中对应的HashSet的线程安全类为：

`java.util.concurrent.CopyOnWriteArraySet`

## 4.4 HashMap

JUC中对应的HashMap的线程安全类为：

`java.util.concurrent.ConcurrentHashMap`       



# 五、Callable&Future

实现`Callable`接口来创建线程最大的特点可以返回结果。

`Runnable`接口和`Callable`接口对比：

| 对比项       | Runnable.run() | Callable.call() |
| ------------ | -------------- | --------------- |
| 是否有返回值 | 否             | 是              |
| 是否抛出异常 | 否             | 是              |

`Callable`不能直接替换`Runnable`，因为`new Thread()`这个构造器只接受`Runnable`类型的参数，如果是`Callable`的实现类怎么办呢？

我们可以使用`Futrure`接口的实现类对象，`Future`接口中定义了5个方法：

```java
public interface Future<V> {
    //用于停止任务
    boolean cancel(boolean mayInterruptIfRunning);
    
    //如果任务在完成前被取消，则返回true
    boolean isCancelled();
    
    //如果任务完成，返回true，否则返回false
    boolean isDone();
    
    //用户获取任务的结果
    V get() throws InterruptedException, ExecutionException;
    
    //在指定的等待时间内，获取结果，如果超时还没有获取到结果，则抛出超时异常
    V get(long timeout, TimeUnit unit) throws InterruptedException,
    ExecutionException, TimeoutException;
}
```



`FutureTask`类实现了`Future`和`Runnable`接口，同时具备了这两个类的功能，并且其构造函数的参数是`Callable`类型，因此我们可以借助这个类来使用`Callable`接口创建线程：

```java
//实现Callable接口
class MyThread2 implements Callable {

    @Override
    public Integer call() throws Exception {
        System.out.println(Thread.currentThread().getName()+"--come in callable");
        return 200;
    }
}

public class Demo1 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        
        //创建FutureTask对象并传入Callable实现类对象
        FutureTask<Integer> futureTask1 = new FutureTask<>(new MyThread2());
        new Thread(futureTask1,"futureTask1").start();
    }
}
```



FutureTask中的`get()`方法如果被调用时计算未完成，会进入阻塞状态，直到计算完成，才会返回结果或者抛出异常。因此，`FutureTask`多用于耗时的计算任务，主线程可以再完成其他任务之后，再使用`get()`获取结果。

`get()`方法只计算一次，再次调用时会直接返回计算好的结果，因此`get()`方法通常放到最后。





# 六、JUC三大辅助类

JUC中提供了三种常用的辅助类，通过这些辅助类可以很好的解决线程数量过多时，手动操作`Lock`锁的频繁操作：

* `CountDownLatch`：减少计数
* `CyclicBarrier`：循环栅栏
* `Semaphore`：信号量



## 6.1 CountDownLatch

CountDownLatch类可以设置一个计数器，然后通过`countDown`方法进行**减1**操作，使用`await()`方法等待计数器小于或等于0时，调用`await()`方法之后的语句。

* `countDown()`方法会将计数器减1
* `await()`方法会将当前线程阻塞，直到计数器的值小于等于0



案例：当所有学生都离开教室以后关灯。

```java
/*
场景：当教室的n个人都离开后，才可以锁门
*/
public class countDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        //1.创建CountDownLatch对象并设置初始计数值，假设有6个学生
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for(int i=1;i<=6;i++){
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+":离开");
                countDownLatch.countDown();  //每次一个线程离开后，计数器减一
            },String.valueOf(i)).start();
        }
        //只要计数器大于0，当前线程就一直等待；直到计数器等于0，当前线程才被唤醒
        countDownLatch.await(); 
        System.out.println(Thread.currentThread().getName()+"可以关灯了");
    }
}
```





## 6.2 CyclicBarrier

`CyclicBarrier`的构造方法第一个参数是目标障碍数，每次执行其中的`await()`方法，都会使障碍数**加1**，当障碍数达到目标障碍数，就会执行`await()`方法之后的语句。

> 线程调用`await()`表示自己已经到达栅栏。当CyclicBarrier达到了指定的栅栏数，才会执行后面的语句。



案例：集齐七颗龙珠召唤神龙

```java
//场景：集齐七颗龙珠召唤神龙
public class CyclicBarrierDemo {
    private static final int NUMBER = 7;
    public static void main(String[] args) {
        //新建循环栅栏对象，设置一个固定值，并指定达到这个固定值以后，需要做的事
        CyclicBarrier cyclicBarrier = new CyclicBarrier(NUMBER, () -> {
            System.out.println("召唤神龙!");
        });

        //集齐七颗龙珠的过程
        for(int i=1;i<=7;i++){
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"星龙珠已收集到");
                try {
                    //await表示已经到达栅栏，每执行一次，计数加一
                    cyclicBarrier.await();   
                    //如果没有到达指定的7颗，cyclicBarrier会一直处于等待状态
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```



## 6.3 Semaphore

`Semaphore`的构造方法中传入的第一个参数是最大信号量，类似于线程池中最大的线程数量，每个信号量初始化为一个最多只能分发一个许可证，线程通过`acquire()`方法获得许可证，`release()`方法释放许可证。

案例：6辆车抢3个车位

```java
//场景：6辆汽车，停到3个车位
public class SemaphoreDemo {
    public static void main(String[] args) {
        //创建semaphore，模拟3个车位，即设置3个许可量，即同时只能有3个线程获取到许可量
        Semaphore semaphore = new Semaphore(3);
        //模拟6辆汽车
        for(int i=1;i<=6;i++){
            new Thread(()->{
                try {
                    //获取许可量，即占用车位
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"抢占到了车位");
                    //模拟停车时间
                    TimeUnit.SECONDS.sleep(new Random().nextInt(5));
                    System.out.println(Thread.currentThread().getName()+"离开了车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //释放许可量
                    semaphore.release();
                }
            },i+"号车").start();
        }
    }
}
```



# 七、读写锁

## 7.1 读写锁概念



悲观锁：每次操作都上锁，不支持并发

乐观锁：支持并发。通过版本号进行控制，每次提交事务都会比较当前版本号和自己的版本号，如果一致，则提交成功，如果版本号高于自己的版本号，说明别人已经优先一步做了修改，自己提交失败。

表锁：不会发生死锁

行锁：会发生死锁

读锁：共享锁。会发生死锁

写锁：独占锁。会发生死锁



**进入读锁的前提条件**

* 没有其他线程的写锁
* 没有写请求，或者写请求和当前持有锁的线程是同一个，即可重入锁。

**进入写锁的前提条件**

* 没有其他线程的读锁
* 没有其他线程的写锁



**读写锁**：一个资源可以被多个读线程访问，或者可以被一个写线程访问；但是不能同时存在读和写线程。即读写互斥，读读共享。

读写锁缺点：

* 容易造成锁饥饿现象。比如一直读，不能写
* 读的时候不能写，只能读完再写。而写的时候可以读。



JUC提供了`ReentrantReadWriteLock`，这个类可以提供**读写锁**。读写锁有以下特征：

* 公平选择性：支持公平和非公平锁（默认）两种方式，非公平锁的吞吐量优于公平锁。
* 重进入：读锁和写锁都支持线程重进入。
* **锁降级**：遵循获取写锁、获取读锁再释放写锁的顺序，**写锁能够降级为读锁**。写操作的时候可以进行读操作；但是读锁不能升级为写锁，因此读操作的时候不能进行写。



##  7.2 ReadWriteLock

`ReadWriteLock`表示读写锁接口，`ReentrantReadWriteLock`是它的一个实现类，其中提供了`readLock()`和`writeLock()`两个方法用来获取**读锁**和**写锁**：

```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     * @return the lock used for writing
     */
    Lock writeLock();
}
```



读写锁将文件的读写操作分开，分成2个锁来分配给线程，使多个线程可以同时进行读操作。

`ReentrantReadWriteLock`实现了`ReadWriteLock`接口，并提供了更多丰富的方法。

案例：使用`ReentrantReadWriteLock`对哈希表进行读写操作

```java
//场景：模拟缓存机制
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        //创建线程放数据
        for(int i=1;i<=5;i++){
            final int num = i;
            new Thread(()->{
                myCache.put(num+"",num+"");
            },"线程"+i).start();
        }
        //创建线程取数据
        for(int i=1;i<=5;i++){
            final int num = i;
            new Thread(()->{
                myCache.get(num+"");
            },"线程"+i).start();
        }
    }
}

//使用读写锁，每次读和写之前都上锁，保证了不会出现还没写完就读的情况
class MyCache{
    private volatile Map<String,Object> map = new HashMap<>();

    //创建读写锁对象
    private ReadWriteLock rwl = new ReentrantReadWriteLock();

    //放数据
    public void put(String key, Object value) {
        rwl.writeLock().lock(); //加上一个写锁
        try {
            System.out.println(Thread.currentThread().getName()+"正在写"+key);
            TimeUnit.MILLISECONDS.sleep(300);
            map.put(key,value);
            System.out.println(Thread.currentThread().getName()+"写完了"+key);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //释放写锁
            rwl.writeLock().unlock();
        }
    }
    
    //取数据
    public Object get(String key)  {
        //添加读锁
        rwl.readLock().lock();
        Object o = null;
        try {
            System.out.println(Thread.currentThread().getName()+"正在读"+key);
            TimeUnit.MILLISECONDS.sleep(300);
            o = map.get(key);
            System.out.println(Thread.currentThread().getName()+"读完了"+key);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //释放读锁
            rwl.readLock().unlock();
        }
        return o;
    }
}
/*运行结果
线程1正在写1
线程1写完了1
线程3正在写3
线程3写完了3
线程4正在写4
线程4写完了4
线程5正在写5
线程5写完了5
线程2正在写2
线程2写完了2
线程1正在读1
线程4正在读4
线程3正在读3
线程2正在读2
线程5正在读5
线程2读完了2
线程5读完了5
线程4读完了4
线程3读完了3
线程1读完了1
*/
```

可以看到，读锁可以被多个线程共享，因此出现了多个线程同时进行读操作的情况。而写锁是互斥锁，同时只能一个线程持有，因此只能等当前线程写操作执行完释放锁以后，其他线程才可以获取到写锁。



# 八、阻塞队列

## 8.1 BlockingQueue

JUC提供了`BlockingQueue`接口表示**阻塞队列**。

阻塞队列，首先是一个队列，其通过一个共享的队列，使数据从队列的一端输入，从另外一端输出。

**当队列是空的，从队列中获取元素的操作将会被阻塞，直到其他线程往空队列中添加了新元素。**

**当队列是满的，向队列中添加元素的操作将会被阻塞，直到其他其他线程从队列中移除一个或多个元素**。

使用阻塞队列，我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为阻塞队列自动判断。以常见的生产者-消费者模型为例，当生产者将队列填满产品时，就会进入阻塞状态，直到队列中有元素被“消费”；而当队列没有数据时，消费者线程就会被自动阻塞，直到有产品入队。

`BlockingQueue`接口中的核心方法主要包括添加、移除、检查元素三种，每种方法有多个具体的方法，其差别如下：

| 方法类型 | 抛出异常    | 返回特殊值 | 阻塞     | 超时                 |
| -------- | ----------- | ---------- | -------- | -------------------- |
| 插入数据 | `add(e)`    | `offer(e)` | `put(e)` | `offer(e,time,unit)` |
| 移除数据 | `remove()`  | `poll()`   | `take()` | `poll(time,unit)`    |
| 检查数据 | `element()` | `peek()`   | 不可用   | 不可用               |

抛出异常：方法执行失败就会抛出异常，比如队列满时添加数据，或者队列空时移除数据。

返回特殊值：对于`offer()`方法，添加成功返回`true`，失败则返回`false`；`poll()`方法，移除成功则返回移除的元素，否则返回`null`

阻塞：队列满时，添加数据就会进入阻塞状态；队列空时，移除数据会进入阻塞状态。

超时：带有阻塞时间的方法，在阻塞到指定的时长后，退出线程。



## 8.2 常见的阻塞队列

### 8.2.1 ArrayBlockingQueue

`ArrayBlockingQueue`是**由数组结构组成的有界阻塞队列**.

底层使用数组实现，长度固定，声明时必须指定大小，先进先出。

创建时，可以指定公平锁或者非公平锁。

### 8.2.2 LinkedBlockingQueue

`LinkedBlockingQueue`是**由链表结构组成的有界阻塞队列**。

底层使用链表结构(Node节点)，可以手动指定固定的大小，默认为`Integer.MAX_VALUE`。

以生产者消费者模型为例，`ArrayBlockingQueue`在生产者和消费者操作的时候，共用同一个锁对象，因此它们不是完全并行的；`LinkedBlockingQueue`对于生产者和消费者使用独立的锁来控制数据同步，在高并发情况下提高整个队列的并发性能。



### 8.2.3 DelayQueue

`DelayQueue`是**使用优先级队列实现的延迟无界阻塞队列**

`DelayQueue`中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。

`DelayQueue`长度无限制，意味着添加数据的进程用于不会被阻塞，只有获取数据的线程才可能会被阻塞。



### 8.2.4 PriorityBlockingQueue

`PriorityBlockingQueue`是**支持优先级排序的无界阻塞队列**

基于优先级队列实现，长度无限制，不会阻塞添加数据的进程。

内部控制同步的锁是公平锁。

使用`PriorityBlockingQueue`要注意的是，如果生产者的速度快于消费者，会导致可用堆内存空间耗尽。



### 8.2.5 SynchronousQueue

`SynchronousQueue`是**不存储元素的阻塞队列**

`SynchronousQueue`不存储元素，或者说只有单个元素。是一个无缓冲区的阻塞队列，类似于消费者生产出一个产品以后，亲自交到消费者手中以后才生产下一个，而没有类似其他阻塞队列的“缓冲区”可以存放生产好的产品。

一个线程的插入操作，必须等待其他线程的移除操作执行完后才能执行，相当于队列只能同时有一个元素。

`SynchronousQueue`有公平锁和非公平锁模式。



### 8.2.6  LinkedTransferQueue

`LinkedTransferQueue`是**由链表组成的无界阻塞队列**

由链表结构组成，相较于其他阻塞队列，多了`tryTransfer`和`transfer`方法。

`LinkedTransferQueue`采用一种**预占模式**，消费者取数据时，如果队列为空，会生成一个节点（节点元素为`null`），这个消费者线程会等待在这个节点上，当生产者线程入队时，发现这个节点后，就直接将数据填充到该节点，并唤醒在此等待的消费者线程，消费者线程取走元素。



### 8.2.7 LinkedBlockingDeque

`LinkedBlockingDeque`是由**链表组成的双向有界阻塞队列**

和`LinkedBlockingQueue`相比，`LinkedBlockingDeque`是由链表组成的双向队列，同样是有界的，最大容量为最大整型数值。

数据可以从队列两端入队出队。



# 九、线程池

## 9.1 使用线程池

Java中的线程池主要是`Executor`这个接口及其子类。继承体系如下：

![线程池接口体系](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/juc-basis_1.png)



`Executor`接口中只有一个`execute()`方法，用于执行`Runnable`实现类的线程。

`ExecutorService`是真正意义上的线程池接口，定义了`submit()`、`shutdown()`等线程池相关的方法。

`ThreadPoolExecutor`是`ExecutorService`的一个实现类，其定义了对线程池操作的各种方法，比如设置核心池大小，最大线程数等方法。创建线程池就是创建这个类的对象。



`Executors`类是线程池的一个工具类，提供了多种方法可以创建多种线程池：

* `newCachedThreadPool()`：创建一个**可缓存线程池**，如果线程池长度超过了处理需要的长度，可灵活回收空闲线程，如果没有可回收的线程，则新建线程。
  * 线程池数量不固定，不限制数量，但最大值为`Integer.MAX_VALUE`
  * 线程池中的线程可进行缓存重复利用和回收。
  * 线程池中没有可用线程时，才会创建新线程。
* `newFixedThreadPool()`：创建一个**可重用固定线程数的线程池**，以无界队列方式运行这些线程。
  * 线程池中的线程处于一定的量，可以很好的控制线程的并发量
  * 线程可以重复被使用，在显式关闭之前，都将一直存在
  * 超出一定量的线程被提交时需要在队列中等待。
* `newSingleThreadExecutor()`：创建一个**只有一个活动线程的线程池**，以无界队列方式运行该线程，保证同时只有一个线程在工作。
  * 线程池中同时最多只执行一个线程，之后提交的线程都会排到队列中。
* `newScheduleThreadPool()`：创建一个**带有延时执行命令的线程池**。线程池支持定时以及周期性执行任务。
* `newWorkStealingPool()`：创建一个**拥有多个任务队列的线程池**，创建当前可用cpu核数的线程来并行执行任务。JDK 8.0出现，适用于耗时、可并行执行的场景。



案例：银行窗口办理业务，假设有5个窗口，10个客户

```java
// 以银行窗口办理业务为例
public class ThreadPoolDemo1 {
    public static void main(String[] args) {
        //一池5线程，表示5个窗口
        ExecutorService threadPool1 = Executors.newFixedThreadPool(5);  
        try{
            //假设有10个顾客
            for(int i=1;i<=10;i++){
                threadPool1.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"正在办理业务");
                });
            }
        }finally {
            //关闭线程池
            threadPool1.shutdown();
        }
    }
}
```





## 9.2 线程池原理及参数

<font color=red>线程池参数</font>



`Executors`工具类中创建线程池，都是调用的`ThreadPoolExecutor`类的构造参数来创建的，只是根据不同的需求，传入的参数不同。

`ThreadPoolExecutor`构造器：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```



其中的7个参数：

* `int corePoolSize`：核心线程数量、常驻线程数量
* `int maximumPoolSize`：最大支持线程数量
* `long keepAliveTime`：线程没有任务时的存活时间（只针对除了核心线程以外的线程）
* `TimeUnit unit`：`keepAliveTime`的时间单位，比如秒、毫秒等
* `BlockingQueue<Runnable> workQueue`：阻塞队列。如果常驻线程数量用完，其他请求就会放到阻塞队列中。当阻塞队列再满的时候，如果有其他请求进入，才创建新的线程，直到达到最大支持的数量。
* `ThreadFactory threadFactory`：线程工厂，用于创建线程
* `RejectedExecutionHandler handler`：拒绝策略，表示什么时候拒绝线程。



当提交的任务数，大于线程池最大支持的线程数量时，就会触发线程池的拒绝策略。

**拒绝策略**有以下四种：

* AbortPolicy。默认策略。直接抛出`RejectedExecutionException`异常阻止系统正常运行。
* CallerRunsPolicy：既不会抛弃任务，也不会抛出异常，而是将这些任务回退到调用者，从而降低新任务的流量。
* DiscardOldestPolity：抛弃队列中等待最久的任务，然后把当前任务加入队列中并尝试再次提交当前任务。
* DiscardPolicy： 默默地丢弃无法处理的任务，不予任何处理也不抛出异常。



<font color=red>线程池工作原理</font>

假设当前线程池的常驻线程数为2，最大线程数为5，阻塞队列固定大小为3，下面分析运行流程。

![线程池运行原理](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/juc-basis_2.png)



1、提交两个任务，此时常驻线程被占用。

2、当常驻线程都被占用时，再次提交任务3,4,5会进入阻塞队列。

3、当阻塞队列满的时候，再提交任务6,7,8的时候，才会创建新线程。

4、当创建了3个线程后，达到了最大支持线程数量，此时提交任务9，就会触发拒绝策略。

当非核心线程超过存活时间后，会被销毁。



## 9.3 自定义线程池

虽然`Executor`工具类提供了各种创建线程池的方法，但是一般我们不会使用，根据《阿里巴巴Java开发手册》，开发中创建线程池必须手动创建，手动指定线程池的参数。

这是因为`Executors`返回的线程池对象有以下弊端：

* `FixedThreadPool`和`SingleThreadPool`：
  允许的请求队列长度为`Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致OOM。
* `CachedThreadPool`和`ScheduledThreadPool`：
  允许的创建线程数量为`Integer.MAX_VALUE`，可能会创建大量的线程，从而导致OOM。

即使用工具类创建的线程池可能会因为堆积大量请求而导致OOM。

手动创建线程池，需要指定7个参数。

案例：使用自定义线程池实现银行窗口办理业务场景模拟。

```java
//自定义线程池
public class ThreadPoolDemo2 {
    public static void main(String[] args) {
        //自定义线程池,
        //常驻线程数量为2，最大为5，超时时间为0秒，阻塞队列为3，拒绝策略为AbortPolicy
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 5, 0L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());

        //使用线程池。模拟10个客户
        try{
            for(int i=1;i<=10;i++){
                threadPoolExecutor.execute(() -> System.out.println(
                    Thread.currentThread().getName()+"正在办理业务"));
            }
        }finally {
            threadPoolExecutor.shutdown();
        }
    }
}
```

上述代码中，当最大线程数为5，阻塞队列容量为3时，面对10个任务请求，会触发拒绝策略， 如果将最大线程数或者队列容量提高，保证二者之和大于等于最大任务请求，就不会触发拒绝策略。实际大小需要根据实际情况而定。
