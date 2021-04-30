---
title: Java学习笔记01-Java语言概述
excerpt: Java运行过程，JDK、JRE、JVM
mathjax: true
date: 2021-03-19 19:08:20
tags: Java
categories: Java
keywords: 'Java概述,Java'
---

# 一、软件开发介绍

## 1、常用的DOS命令

* dir：列出当前目录下的文件以及文件夹
* md：创建目录
* rd：删除目录
* cd：进入指定目录
* cd..：退回到上一级目录
* cd\：退回到根目录
* del：删除文件
* exit：退出dos命令行
* echo：输出内容

## 2、常用快捷键

* ← →：移动光标
* ↑ ↓：调阅历史操作命令
* Delete和Backspace：删除字符

#　二、计算机编程语言介绍

* 机器语言
* 汇编语言
* 高级语言：
  * C、Pascal、Fortran面向过程的语言
  * C++面向过程/面向对象
  * Java跨平台的纯面向对象的语言
  * .NET跨语言的平台
  * Python、Scala

# 三、Java语言概述

## 1、Java语言简史

* 1991年Green项目，开发语言最初命名为Oak (橡树)
* 1994年，开发组意识到Oak非常适合于互联网
* 1996年，发布JDK 1.0，约8.3万个网页应用Java技术来制作
* 1997年，发布JDK1.1，JavaOne会议召开，创当时全球同类会议规模之最
* 1998年，发布JDK 1.2，同年发布企业平台J2EE
* 1999年，Java分成J2SE、J2EE和J2ME，JSP/Servlet技术诞生
* **2004年，发布里程碑式版本：JDK1.5，为突出此版本的重要性，更名为JDK 5.0**
* 2005年，J2SE-> JavaSE，J2EE-> JavaEE，J2ME-> JavaME
* 2009年，Oracle公司收购SUN，交易价格74亿美元
* 2011年，发布JDK 7.0
* **2014年，发布JDK8.0，是继JDK 5.0以来变化最大的版本**
* 2017年，发布JDK9.0，最大限度实现模块化
* 2018年3月，发布JDK10.0，版本号也称为18.3
* 2018年9月，发布JDK 11.0，版本号也称为18.9

## 2、Java技术平台

* **Java SE(Java Standard Edition)标准版**：支持面向桌面级应用（如Windows下的应用程序）的Java平台，提供了完整的Java核心API，此版本以前称为J2SE。
* **Java EE(Java Enterprise Edition)企业版**：是为开发企业环境下的应用程序提供的一套解决方案。该技术体系中包含的技术如:Servlet、Jsp等，主要针对于Web应用程序开发。版本以前称为J2EE。
* Java ME(Java Micro Edition) 小型版
* Java Card

# 四、运行机制及运行过程

## 1、Java语言的特点

1. **面向对象**：
   * 两个基本概念：类、对象
   * 三大特征：封装、继承、多态
2. 健壮性：吸收C/C++的优点，去掉了其指针、内存的申请和释放等，提供了相对安全的内存管理和访问机制。
3. 跨平台性：“Write once，Run AnyWhere”，只需要在运行Java应用程序的操作系统上，先安装Java虚拟机(JVM,**J**ava **V**irtual **M**achine)即可。由JVM负责Java程序在系统中的运行。、

## 2、Java两种核心机制

* Java虚拟机（Java Virtual Machine）
* 垃圾收集机制（Garbage Collection）

## 3、Java运行过程

\`*.java `文件经过编译生成`\* .class`文件（字节码文件），然后执行

# 五、Java的环境搭建

## 1、环境搭建

* 下载JDK并安装JDK
* 配置环境变量（为了在任何目录下都能执行java命令）
* 使用`javac java`命令验证是否安装成功
* 使用编辑器或IDE开发

## 2、JDK、JRE、JVM

* JDK（Java Development Kit  Java开发工具包）。JDK是提供给Java开发人员使用的，包含了Java的开发工具（比如编译工具（javac.exe）、打包工具（jar.exe）等），也包含了JRE，安装了JDK就不用再安装JRE。
* JRE（Java Runtime Environment Java运行环境）。包括JVM和Java程序所需的核心类库等。只安装JRE就能够运行Java程序。
* JVM（Java Virtual Machine）是一个虚拟的计算机，具有指令集并使用不同的存储区域。负责执行指令，管理数据、内存、寄存器。

> **JDK = JRE+开发工具集（例如Javac编译工具等）**
>
> **JRE = JVM+Java SE标准类库**

Java SE 8概要图[Java Platform Standard Edition 8 Documentation](https://docs.oracle.com/javase/8/docs/)：

<div align='center'>
    <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-0101_1.png"/>
</div>

# 六、运行HelloWorld

搭建好java环境后，可以在dos窗口编译运行Java程序。

* 将Java代码编写到扩展名为.java的文件中。
* 通过javac命令对该java文件进行编译。
* 通过java命令对生成的class文件进行运行。

> 注意：
>
> 1、Java应用程序的执行入口是main()方法。它有固定的书写格式：
>
> ​					public static void main ( String[]args )  {……}
>
> 2、**一个源文件中只能有一个public类**。其他类的个数不限，如果源文件中包含一个public类，则文件名必须按照该类名命名。

# 七、注释(Comment)

1. 单行注释：`//注释内容`

2. 多行注释：`/*注释内容*/`

3. 文档注释（Java特有）：

   ```java
   /**
   	@author 指定程序作者
   	@version 指定源文件版本
   */
   ```

   

# 八、Java API文档

**API(Application Programming Interface,应用程序编程接口)**是Java提供的基本编程接口。

JDK下载链接：[https://www.oracle.com/java/technologies/javase-downloads.html](https://www.oracle.com/java/technologies/javase-downloads.html)

