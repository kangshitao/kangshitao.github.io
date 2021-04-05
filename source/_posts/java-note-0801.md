---
title: Java学习笔记08-多线程
excerpt: 进程、线程、线程同步和通信、Thread、synchronized、wait()、notify()、notifyAll()
mathjax: true
date: 2021-04-03 10:07:20
tags: Java
categories: Java
keywords: Java
---

# 一、程序、进程、线程

* **程序(program)**：是为完成特定任务、用某种语言编写的一组指令的集合，即指一段静态的代码，静态对象。
* **进程(process)**：是程序的一次执行过程，是正在运行的一个程序。
  * 是一个动态的过程，有自身的产生、存在和消亡的过程——即**生命周期**。
  * 程序是**静态**的，进程是**动态**的。
  * **进程是资源分配的单位**，系统在运行时会为每个进程分配不同的内存区域。
* **线程(thread)**：进程可进一步细化为线程，是一个**程序内部的一条执行路径**。 
  * 若一个进程同一时间并行执行多个线程，就是支持**多线程**的。
  * **线程是调度和执行的单位**，每个线程拥有独立的**运行栈**和**程序计数器(pc)**，线程切换开销小。 
  * 一个进程的多个线程共享进程的**堆**和**方法区**，每个线程有自己的**程序计数器、虚拟机栈和本地方法栈**，这些线程从同一堆中分配对象，可以访问相同的变量和对象。
  * 多个线程操作共享的系统资源可能会带来安全的隐患。

> 一个Java程序java.exe，至少有三个线程：main()主线程、gc()垃圾回收线程、异常处理线程。

JVM结构：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-0801_1.jpg'/>
</div>

1. **方法区(Method Area)**：存储类结构信息，包括常量池、静态变量、构造函数等。JVM规范把方法区描述为堆的一个逻辑部分，但它有个别名non-heap(非堆)。方法区还包含运行时常量池
2. **堆(Heap)**：存储Java实例或者对象，是GC的主要区域。
3. **虚拟机栈(VM Stack，Stack，Java Stack)**：即Java栈，每当创建一个线程时，JVM就会为这个线程创建一个其私有的Java栈。在这个栈中包含多个栈帧，每运行一个方法就创建一个栈帧，用于存储局部变量表、操作栈、方法返回值等。每一个方法从调用直至执行完成的过程，就对应一个栈帧在Java栈中入栈到出栈的过程。
4. **程序计数器(PC Register)**：保存当前线程执行的内存地址。由于JVM程序是多线程执行的（线程轮流切换），所以为了保证线程切换回来后，还能恢复到原先状态，就需要一个独立的计数器，记录之前中断的地方。
5. **本地方法栈(Native Method Stack)**：和Java栈的作用类似，只不过是为JVM使用到的native方法服务的。

参考：[java中进程与线程的概念](https://www.zhihu.com/question/264627396)

并行和并发：

* 并行：多个CPU同时执行多个任务。
* 并发：一个CPU(采用时间片)同时执行多个任务。

# 二、线程的创建和使用

线程的创建方式共有四种：继承`Thread`类、实现`Runnable`接口、实现`Callable`接口(5.0新增)、使用线程池(5.0新增)。

## 1、Thread类

创建线程的第一种方式是继承`Thread`类：

* 创建继承`Thread`类的子类。
* 重写`run()`方法，将此线程执行的操作声明的此方法中。
* 创建子类的对象，然后使用此对象调用`start()`方法。

> ①`start()`方法的作用：启动当前线程；调用当前线程的`run()`方法。
>
> ②启动多线程，必须调用`start()`方法，不能通过直接调用`run()`的方式启动线程。
>
> ③不能让已经`start()`的线程再`start()`，即一个线程不能启动多次，否则报`IllegalThreadStateException`异常。
>
> ④如果想创建多个线程，需要创建多个对象。

实现代码：

```java
public class ThreadTest {
    public static void main(String[] args) {
        //3.创建Thread类的子类的对象
        MyThread1 mt = new MyThread1();
        //4.通过此对象调用start()方法
        mt.start(); //主线程使用mt对象调用了start方法，启动新线程。
        // start方法能够使线程执行，JVM调用这个线程的run方法

        //这里是主线程运行
        //同时运行两个线程，输出顺序就不确定，每个线程的输出也可能交叉
        for( int i = 0; i<100;i++){
            if(i % 2 != 0)
                System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
// 1.创建一个继承于Thread类的子类
class MyThread1 extends Thread{
    //2.重写Thread的run()
    @Override
    public void run() {
        for(int i = 0; i<100;i++){
            if(i % 2 == 0)
                System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
```

`Thread`类的相关方法：

* `start()`：启动线程，并执行对象的run()方法
* `run()`：线程被调度时执行的操作
* `currentThread()`：返回当前线程，在Thread子类中就是this，通常用于主线程和Runnable实现类
* `getName()`：返回线程名称
* `setName()`：设置线程名称
* `yield()`：线程让步，释放当前cpu的执行权。暂停当前正在执行的线程，把执行机会让给优先级相同或更高的线程。
* `join()`：在线程A中调用线程B的`join()`方法，此时A进入阻塞状态，直到线程B完全执行完以后，线程A才结束阻塞状态
* `sleep(long millitime)`：让当前线程指定时间内放弃对CPU的控制。指定的毫秒时间内，当前线程是阻塞状态。
* `isAlive()`：判断线程是否活着
* `stop()`：已过时。强制结束当前线程
* `suspend()`：已过时
* `resume()`：已过时

线程的优先级：

`MAX_PRIORITY`：10

`MIN_PRIORITY`：1

`NORM_PRIORITY`：5

方法：

* `getPriority()`：返回线程优先级
* `setPriority(int newPriority)`：改变线程优先级

> 线程创建时继承父线程的优先级
>
> 低优先级的线程只是获得调度的概率低，并不是说低优先级线程一定在高优先级线程执行完之后执行。

**守护线程**和**用户线程**

* 二者几乎每个方面都是相同的，唯一的区别是判断JVM何时离开。
* 守护线程是用来服务用户线程的，可以通过在`start()`方法前调用`thread.setDaemon(true)`把用户线程变成守护线程。
* 守护线程比如Java垃圾回收。当JVM中都是守护线程时，当前JVM将退出

## 2、Runnable接口

创建线程的第二种方式是实现`Runnable`接口：

* 创建类，实现实现`Runnable`接口。
* 重写`run()`方法。
* 创建实现类对象，作为参数传入`Thread`类的构造器。
* `Thread`类的对象调用`start()`方法启动线程。

> ①实现接口的方法，避免了单继承的局限性。
>
> ②多个线程共享一个接口实现类的对象，非常适合多个相同线程处理同一份资源。
>
> ③和方法一相比，实现`Runnable`接口的方法优先使用。

实现代码：

```java
public class ThreadTest2 {
    public static void main(String[] args) {
        MyThread2 mt = new MyThread2();
        Thread t1 = new Thread(mt,"Thread-1");
        Thread t2 = new Thread(mt,"Thread-2");
        //启动多个线程，只需要新建Thread对象，两个线程共享Runnable实现类对象
        t1.start();
        t2.start();
    }
}
class MyThread2 implements Runnable{
    @Override
    public void run() {
        for(int i = 0; i< 100; i++)
            System.out.println(Thread.currentThread().getName());
    }
}
```



## 3、Callable接口

创建线程的第三种方法是实现`Callable`接口(JDK 5.0新增)：

* 声明实现`Callable`接口的实现类。
* 重写`call()`方法。
* 声明实现类对象，比如`mt3`。
* 声明`FutureTask`类对象`ft`，构造器参数为实现类对象`mt3`。
* 声明`Thread`类对象`t`，构造器参数为`FutureTask`类对象`ft`。
* 通过对象`t`调用`start()`方法，启动并运行线程。

> ①`Future`接口可以对具体`Runnable`、`Callable`任务的执行结果进行取消、查询是否完成、获取结果等。
>
> ②`FutureTask`是`Future`接口唯一的实现类。
>
> ③`FutrueTask`同时实现了`Runnable`、`Future`接口，它**既可以作为`Runnable`被线程执行**，**也可以作为`Future`得到`Callable`的返回值**。

实现`Callable`接口和实现`Runnable`接口相比，前者更强大，因为：

* `call()`方法可以有返回值。
* `call()`可以抛出异常，被外面的操作捕获，获取异常信息
* `Callable`支持泛型。

实现代码：

```java
public class ThreadThree {
    public static void main(String[] args) {
        //创建实现类的对象
        NumThread numThread = new NumThread(); 
        //创建FutureTask类对象，将实现类对象作为参数
        FutureTask futureTask = new FutureTask(numThread); 
        //创建Thread类对象，将futureTask作为Runnable被线程执行
        new Thread(futureTask).start();  

        //FutureTask作为Future得到Callable的返回值
        try {//get的返回值即numThread重写的call的返回值
            Object num = futureTask.get();  
            System.out.println(num);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
class NumThread implements Callable{ //实现Callable接口，并重写call方法
    //call()函数有返回值，且能够抛出异常
    @Override
    public Object call() throws Exception {
        int sum = 0;
        for (int i = 0; i <= 100; i++) {
            if(i % 2 == 0){
                System.out.println(i);
                sum += i;
            }
        }
        return sum;
    }
}
```



## 4、线程池

创建线程的第四种方式是使用线程池(JDK 5.0新增)。

经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影响很大。线程池的思想是提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁、实现重复利用。

线程池的优势：

* 提高响应速度（减少了创建新线程的时间）。
* 降低资源消耗（重复利用线程池中线程，不需要每次都创建）。
* 便于线程管理。

JDK 5.0 提供了线程池相关的API：`ExecutorService`接口和`Executors`工具类。

* `ThreadPoolExecutor`是`ExecutorService`的常见实现类，其中的常用方法：
  * `corePoolSize`：核心池大小
  * `maximumPoolSize`：最大线程数
  * keepAliveTim`e：线程没有任务时，最多保持多长时间后会终止

* `ExecutorService`：真正的线程池接口，常见的实现类为`ThreadPoolExecutor`
  * `void execute(Runnable command)`:执行任务/命令，一般用来执行`Runnable`
  * `<T>Future<T> submit(Callable<T> task)`:执行任务，有返回值，一般用来执行`Callable`
  * `void shutdown()`：关闭连接池。

* `Executors`：工具类、线程池的工厂类，用于创建并返回不同类型的线程池。
  * `Executors.newCachedThreadPool()`：创建一个可根据需要创建新线程的线程池
  * `Executors.newFixedThreadPool(n)`：创建一个可重用的固定线程数的线程池
  * `Executors.newSingleThreadExecutor()`：创建一个只有一个线程的线程池
  * `Executors.newScheduledThreadPool(n)`：创建一个线程池，可安排在给定延迟后运行命令或者定期地执行

使用步骤：
    1.提供指定线程数量的线程池：调用工具类的方法创建线程池，返回ThreadPoolExecutor对象。
    2.执行指定的线程的操作，需要提供实现Runnable接口或Callable接口实现类的对象。
    3.关闭线程池。

具体代码：

```java 
public class ThreadPool {
    public static void main(String[] args) {
        //1.提供指定线程数量的线程池
        ExecutorService service = Executors.newFixedThreadPool(10);
        //ThreadPoolExecutor类实现了ExecutorService接口
        ThreadPoolExecutor service1 = (ThreadPoolExecutor) service;
        //通过service1，可以设置线程池的属性...
        service1.setMaximumPoolSize(15);

        //2.执行指定的线程的操作，需要提供实现Runnable接口或Callable接口实现类的对象
        service.execute(new NumThreadFour()); //适合于Runnable,分配线程1
        service.execute(new NumThreadFour2()); //适合于Runnable，分配线程2
        service.submit(new FutureTask(new NumThreadFour3())); //适合于Callable，分配线程3
        //3.关闭线程池
        service.shutdown();
    }
}
class NumThreadFour implements Runnable {...}
class NumThreadFour2 implements Runnable {...}
class NumThreadFour3 implements Callable {...}
```





# 三、线程的生命周期

一个线程的完整生命周期有六种状态，在`Thread`类中的`State`枚举类定义了线程的五种状态：

* **新建(NEW)**：执行new指令，新创建的线程的状态
* **运行(RUNNABLE)**：运行状态包括就绪(READY)和正在运行(RUNNING)两个状态。
  * **就绪(READY)**：执行start方法以后进入就绪状态，已经具备了运行的条件，只有分配到CPU资源以后才进入RUNNING状态。
  * **正在运行(RUNNING)**：就绪的线程分配到CPU资源(时间片)时，便进入RUNNING状态，`run()`方法定义了线程的操作和功能。
* **阻塞(BLOCKED)**：线程进入synchronized方法/代码块中时，要判断同步锁是否被释放，等待同步锁被释放的状态就是阻塞状态。获取到同步锁以后进入就绪状态。
* **等待(WAITING)**：等待状态下的线程不会被分配CPU时间片，必须被显式唤醒，即`notify()`/`notifyAll()`，否则会处于无限期等待的状态
* **超时等待(TIMED_WAITING)**：超时等待状态下的线程不会被分配CPU时间片，无须被显式唤醒，在达到一定时间后会自动唤醒。
* **终止(TERMINATED)**：线程的`run()`方法完成时，或者被强制终止，或出现异常导致结束，线程进入终止状态。

>1、执行`Object`中的`wait()`方法后，线程释放同步锁并进入等待状态，只有被显示唤醒，比如`notify()`/`notifyAll()`，才能进入阻塞状态，等待同步锁被释放然后进入就绪状态。
>
>2、执行`sleep()`、`join()`、IO操作以后，线程进入超时等待状态，等时间到或者操作结束，自动进入就绪状态。

**线程的状态转换图**(图中的阻塞状态包括了阻塞、等待、超时等待三个状态)：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-0801_2.png'/>
</div>



# 四、线程的同步

## 1、介绍

多条语句在操作共享数据时，如果某个线程操作尚未完成时，其他线程参与进来，会导致共享数据出现错误，这就导致了线程安全问题。

比如有三个窗口共同卖100张票，如下的代码就会有线程安全问题：

```java
public class WindowTest2 {
    public static void main(String[] args) {
        Window2 w2 = new Window2();
        //创建三个线程，即三个窗口
        Thread t1 = new Thread(w2,"窗口1");
        Thread t2 = new Thread(w2,"窗口2");
        Thread t3 = new Thread(w2,"窗口3");
        t1.start();
        t2.start();
        t3.start();
    }
}
class Window2 implements Runnable{
    private int ticket = 100;
    @Override
    public void run() {
        while(true){
            if(ticket > 0){
                System.out.println(Thread.currentThread().getName()+"票号为:"+ticket);
                ticket--;
            }else break;
        }
    }
}
/*
三个线程共享100张票，在其中一个窗口操作的同时，其他线程参与进来，导致票数出现异常
以下是部分执行结果：
窗口2票号为:23
窗口2票号为:7
窗口2票号为:6
窗口1票号为:8
窗口3票号为:19
窗口1票号为:4
窗口1票号为:2
窗口2票号为:5
窗口1票号为:1
窗口3票号为:3
*/
```

为了解决类似于以上的线程安全问题，需要保证当前线程操作时，其他线程不能参与进来，直到当前线程操作完，其他线程才可以操作。即使当前线程出现了阻塞，也不能被改变。

Java中使用**同步机制**解决线程安全问题，包括**同步代码块**、**同步方法**、**Lock锁(JDK 5.0新增)**三种方法实现同步机制。其中同步代码块和同步方法都是使用`synchronized`关键字。

## 2、方法一：同步代码块

格式和代码：

```java
//格式：
synchronized(同步监视器){
    //需要被同步的代码
}
//具体使用：
public class TicketSell {
    public static void main(String[] args) {
        Window w = new Window();
        //创建三个线程，即三个窗口
        Thread t1 = new Thread(w, "窗口1");
        Thread t2 = new Thread(w, "窗口2");
        Thread t3 = new Thread(w, "窗口3");
        t1.start();
        t2.start();
        t3.start();
    }
}

class Window implements Runnable {
    private int ticket = 100;
    Object obj = new Object(); //同步监视器(同步锁)
    @Override
    public void run() {
        while (true) {
            //synchronized (this){ //也可以使用当前对象
            //synchronized (Window.class){  //当前类也可以作为同步锁
            synchronized (obj) { //以下为同步代码
                if (ticket > 0) {
                    System.out.println(Thread.currentThread().getName() + 
                                       "票号为:" + ticket);
                    ticket--;
                } else break;
            }
        }
    }
}
```

说明：

* 操作共享数据的代码，即为需要被同步的代码，同步代码不能多也不能少，否则容易导致错误。
* 共享数据：多个线程共同操作的数据，比如本例的ticket。
* 同步监视器，即“同步锁”：任何一个类的对象都可以充当同步锁。同步锁被某个线程持有的时候，只有当其执行完同步代码，然后释放同步锁以后，其他线程才可以继续执行。
* 要求多个线程必须共用同一把同步锁，即**多个线程的同步锁必须是同一个对象**，才能保证同步。
* 实现`Runnable`接口创建多线程的方式中，因为只创建一个实现类对象，可以考虑使用`this`充当同步锁。而在继承`Thread`类创建多线程的方式中，要慎用`this`作为锁，根据实际情况判断`this`是不是唯一的。
* 也可以考虑当前类作为同步锁(类也是对象，即`当前类.class`。

## 3、方法二：同步方法

如果操作共享数据的代码完整的声明在一个方法中，可以使用`synchrozied`将此方法声明为同步的。可以将多个方法都使用`synchronized`监视起来，解决这几个方法的线程安全问题。

格式和使用：

```java
//格式：
权限修饰符 synchronized 返回值类型 方法名(参数){
    //操作共享数据的代码
}

//具体使用：
class Window3 implements Runnable {
    private int ticket = 100;

    @Override
    public void run() {
        while (true) {
            show();
        }
    }
	//同步方法。非静态方法默认的同步监视器是this
    private synchronized void show() {
        if (ticket > 0) {
            System.out.println(Thread.currentThread().getName() + "票号为:" + ticket);
            ticket--;
        }
    }
}
```

说明：

* 同步方法仍然涉及到同步监视器，不需要显式声明。
* 对于非静态的同步方法，同步监视器是this
* 对于静态的同步方法，同步监视器是当前类本身(`类名.class`，类也是对象)。

## 4、方法三：Lock锁

Lock锁是JDK 5.0新增的解决线程安全问题的一种方式，通过显式定义同步锁对象来实现同步。同步锁使用Lock对象充当。

`java.util.concurrent.locks.Lock`接口是控制多个线程对共享资源进行访问的工具。锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应先获得Lock对象。
`ReentrantLock`类实现了`Lock`接口，它拥有与`synchronized`相同的并发性和内存语义，在实现线程安全的控制中，比较常用的是`ReentrantLock`，可以显式加锁、释放锁。

使用方法：

```java
//首先需要声明ReentrantLock对象
//然后调用lock方法加锁
//最后调用unlock方法释放锁
class Window implements Runnable {
    private int ticket = 100;
    private ReentrantLock lock = new ReentrantLock(); //步骤一
    @Override
    public void run() {
        while (true) {
            lock.lock(); //步骤二，加锁
            try {
                if (ticket > 0) {
                    System.out.println(Thread.currentThread().getName() 
                                       + ":" + ticket);
                    ticket--;
                } else break;
            } finally {
                lock.unlock(); //步骤三，解锁
            }
        }
    }
}
```

使用Lock接口进行同步的时候，同样需要保证多个线程使用同一个ReentrantLock对象。

比较`synchronized`和`Lock`的异同：

* `synchronized`是隐式锁，在执行完相应的同步代码后，自动释放同步锁(同步监视器)。
* `Lock`是显式锁，需要手动启动同步（执行`Lock()`），结束时也需要手动关闭（执行`unLock()`），`try-catch`结构中，解锁代码需要写在`finally`中，保证`unlock()`一定会被执行。
* `Lock`只有代码块锁，`synchronized`有代码块锁和方法锁。

三种实现同步的方式，使用的优先顺序为：Lock-->同步代码块(已经进入了方法体，分配了相应资源)-->同步方法



## 5、死锁

同的线程分别占用对方需要的同步资源(同步锁)不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁。A线程持有锁a，等待操作b，而B线程持有锁b，等待操作a(此时a被A当作同步锁持有，没有被释放)，这就形成了死锁。

举例：

```java
public class DeadLock {
    public static void main(String[] args) {
        StringBuffer s1 = new StringBuffer();
        StringBuffer s2 = new StringBuffer();
        new Thread() {//Thread的匿名实现类
            @Override
            public void run() {
                synchronized (s1) {
                    s2.append("a"); //持有同步锁 s1，等待s2被释放
                    System.out.println(getName() + ":" + s1);
                    System.out.println(getName() + ":" + s2);
                }
            }
        }.start();
        new Thread() { //Thread的匿名实现类
            @Override
            public void run() {
                synchronized (s2) {
                    s1.append("b");//持有同步锁 s2，等待s1被释放，造成死锁
                    System.out.println(getName() + ":" + s1);
                    System.out.println(getName() + ":" + s2);
                }
            }
        }.start();
    }
}
```



解决方法：
1.专门的算法、原则
2.尽量减少同步资源的定义
3.尽量避免嵌套同步

# 五、线程的通信

线程的通信涉及到三个方法：

* `wait()`：使当前线程进入等待状态(排队等待同步资源)，并释放同步锁(同步监视器)。
* `notify()`：唤醒排队等待同步资源的一个进程，如果有多个线程排队等待同步资源，唤醒优先级高的线程。
* `notifyAll()`：唤醒所有排队等待同步资源的线程。

> 调用以上三个方法的必要条件是当前线程具有对当前对象的监控权，即持有同步锁。这三个方法只能在同步代码块或同步方法中使用。
>
> 任意对象都能作为同步锁，因此以上三个方法在Object类中声明。

`sleep()`和`wait()`两个方法的异同：

* 都可以使当前线程进入阻塞状态，`sleep`使线程进入的是**超时等待(TIMED_WAITING)**状态，`wait`使线程进入**等待(WAITING)**状态。
* 声明位置不同，`sleep`声明在`Thread`类中，`wait`方法声明在`Object`类中。
* 调用的要求不同。`sleep`可以在任何需要的场景下使用。`wait`只能在同步代码块/同步方法中。
* 是否释放同步锁：两个方法都使用在同步代码块/同步方法中时，`sleep`不会释放同步锁，`wait`会释放同步锁。

总结：

能够释放锁的操作有以下几种：

* 线程的同步方法、同步代码块执行结束。
* 执行过程中遇到`break`、`return`终止了执行操作。
* 有未处理的`Error`和`Exception`，导致异常结束。
* 执行线程的`wait()`方法，使线程释放锁，并进入等待状态。

>`sleep`、`yield`方法以及`suspend`方法不会释放同步锁。
>
>尽量避免使用`suspend`和`resume`方法控制线程。