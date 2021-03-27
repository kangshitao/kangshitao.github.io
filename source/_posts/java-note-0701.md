---
title: Java学习笔记07-异常处理
excerpt: 异常处理，try-catch-finally，throws，throw，自定义异常类
mathjax: true
date: 2021-03-26 22:37:04
tags: Java
categories: Java
keywords: Java
---

# 一、异常概述与异常体系结构

## 1、异常概述

Java中的异常可以分为两类：

* `Error`：Java虚拟机无法解决的严重问题，如：JVM系统内部错误、资源耗尽等严重情况。比如`StackOverflowError`和`OOM`。对于这类异常一般不编写针对性代码进行处理。
* `Exception`：其他因编程错误或偶然的外在因素导致的一般性问题。可以使用针对性代码进行处理，如
  * 空指针异常：`NullPointerException`
  * 读取不存在的文件：`FileNotFoundException`
  * 类型转换异常：`ClassCastException`
  * 数组索引越界：`ArrayIndexOutOfBoundException`

## 2、异常体系结构

Java中的异常体系结构如下：

`java.lang.Throwable`有以下继承关系:

* `java.lang.Error`：

  * `StackOverflowError`
  * `OutOfMemoryError`
  * `...`

* `java.lang.Exception`：

  * `ClassNotFoundException`
  * `CloneNotSupportedException`
  * `IOException`：
    * `EOFException`
    * `FileNotFoundException`
    * `MalformedURLException`
    * `UnknowHostException`
    * `...`

  * `RuntimeException`:
    * `ArithmeticException`：除数是0时报此异常
    * `ClassCastException`
    * `IllegalArgumentException`
    * `IllegalStateException`
    * `IndexOutOfBoundsException`
    * `NoSuchElementException`
    * `NullPointerException`
    * `...`
  * `...`

可以看到，`Throwable`是异常体系的根父类，其有`Error`和`Exception`两个子类，`Exception`分为**运行时异常(RuntimeException)**和**编译时异常(IOException等)**，其中**运行时异常**又可以称为**非受检(unchecked)异常**，**编译时异常**称为**受检(checked)异常**。

## 3、异常处理

Java中对于异常处理使用的**抓抛模型**：

* 第一步，“抛”出异常：执行过程中，一旦出现异常，会在代码处生成一个对应异常类的对象，并将此对象抛出，一旦抛出异常对象以后，其后的代码不再执行。
  * 抛出异常有两种方式：①系统自动生成异常对象并自动抛出。②手动生成异常对象并抛出(`throw`)
* 第二步，“抓”住异常(**捕获(catch)异常**)，即异常的处理。
  * 对于异常处理有两种方法：①`try-catch-finally`。②使用`throws`将异常交给方法调用者处理。

# 二、异常处理机制之一：try-catch-finally

## 1、try-catch的使用

`try-catch-finally`的使用：

```java
    try{
        //可能出现异常的代码
    }catch(异常类型1 变量名1){
        //处理异常的方式1
    }catch(异常类型2 变量名2){
        //处理异常的方式2
    }catch(异常类型3 变量名3){
        //处理异常的方式3
    }
    ......
    finally{
        //一定会执行的代码
    }

//例如：
	try{
       ...//读取文件的语句
    }catch(IOException e){
        System.out.println(e.getMessage());
    }
```

* `finally`是可选的，`try-catch-finally`结构可以嵌套。
* 使用`try`将可能出现异常的代码包装起来，一旦出现异常，会生成一个对应异常类的对象，根据此对象的类型，去`catch`中匹配。
* `try`中的异常对象匹配到某个`catch`时，就进入`catch`中处理，处理完成后，跳出`try-catch`结构(没有`finally`时)，继续执行其后的代码
* catch中的异常类型如果没有子父类关系，则多个`catch`无关顺序。如果多个`catch`的异常类型有子父类关系，需要将子类写在父类的前面，否则出错。比如`NullPointerException`必须写在`RuntimeException`前面。
* 常用的异常对象处理的方式：
  * `Sting getMessage()`：获取异常信息，返回字符串。
  * `printStackTrace()`：获取异常类名和异常信息，以及异常出现在程序中的位置，返回值`void`。
* `try`结构中声明的变量，在`try`结构外面不能调用。可以将变量声明在外面，在`try`结构中赋值。

## 2、finally

`finally`的用法：

* `finally`中声明的是一定会被执行的代码。即使`catch`中又出现异常、`try`中有`return`语句、`catch`中有`return`语句，`finally`中的语句也会执行。
* 数据库连接、输入输出流、网络编程Socket等资源，JVM不能自动回收，需要手动进行资源的释放。此时的资源释放，需要声明在`finally`中。

## 3、不捕获异常的情况

* 使用`try-catch-finally`处理编译时异常，使程序在编译时不报错，但运行时仍可能报错。相当于使用`try-catch-finally`将**编译时**可能出现的异常延迟到**运行时**出现。
* 由于运行时异常比较常见，Java能够自动捕获，因此，对于运行时异常，通常不使用`try-catch`结构捕获。而对于**编译时异常，必须进行异常处理(捕获异常)**。



# 三、异常处理机制之二：throws

## 1、throws的使用

使用`throws`声明抛出异常是处理异常的第二种方式。如果一个方法中可能生成某种异常，但不能确定如何处理这种异常，就可以使用`throws`显式地声明抛出异常，表示该方法不对这些异常处理，而是由调用者负责处理。

当方法体执行时出现异常，会在异常代码处生成一个异常类对象，如果此对象满足throws后的异常类型，就会被抛出(抛给方法的调用者)，异常代码后面的语句不再执行。

`throws`的使用格式为：`throws 异常类型`，写在方法声明处的方法名后面。

例如：

```java
public void method() throws FileNotFoundException, IOException{
    //读取文件的语句
    File file = new File("xxx.txt");
    ...
}
```

> 可以同时声明多个类型的抛出异常。
>
> 对于重写的方法，子类重写的方法抛出的异常**不大于**父类被重写的方法抛出的异常类型。

## 2、throws和try-catch

`throws`和`try-catch`都是异常处理的两种方式，其区别是：

* `try-catch-finally`真正地将异常处理掉了。
* `throws`的方式只是将异常抛给了方法的调用者，没有真正地处理掉异常。

如何选择两种处理方式？

* 如果父类中被重写的方法没有`throws`方式处理异常，则子类重写的方法不能使用`throws`，如果有异常只能使用`try-catch-finally`的方式。
* 如果执行的方法a中先后调用了另外的几个递进关系的方法，建议这几个方法使用`throws`处理异常，而方法a使用`try-catch-finally`。

# 四、手动抛出异常：throw

Java异常类对象除了系统自动生成以外，还可以手动创建异常类对象并抛出。需要先声明异常类对象，然后通过throw语句抛出：

```java
IOException e = new IOException();  //创建异常类对象
throw e;  //将异常类对象抛出
```

> 手动抛出异常的方式，可以用在`try-catch`结构里和使用了`throws`的方法中。

`throw`的应用：

```java
//场景一：
try {
    System.out.println("try");
    throw new RuntimeException("制造异常");
} catch{
    System.out.println("catch");
}
//场景二：
public void method(int id) throws Exception {
    if(xxx){
        //执行语句
    }else{
        //手动抛出异常对象
        throw new RuntimeException("您输入的数据非法！");
        //throw new MyException("不能输入负数");  //抛出自定义异常类对象
    }
```



# 五、自定义异常类

一般来说，自定义异常类都是`RuntimeException`、`Exception`的子类，过程如下：

* 编写重载的构造器。
* 提供`serialVersionUID`
* 通过`throw`抛出异常类对象

以下是自定义类的举例：

```java
public class MyException extends RuntimeException{
    static final long serialVersionUID = -1234567890L;
    public MyException(){}
    public MyException(String msg){
        super(msg);
    }
}

```

