---
title: JVM-不同JDK版本中方法区的演变
excerpt: JDK6，JDK7，JDK8及以后的方法区的变化
mathjax: true
date: 2021-05-09 11:27:46
tags: [JVM,java]
categories: JVM
keywords: JVM,方法区,永久代,元空间
---

# 1、方法区的演变

方法区（Method Area）存放的内容为：类型信息、常量、静态变量、即时编译器编译后的代码缓存、域信息、方法信息等。

HotSpot虚拟机中，**方法区(Method Area)**在JDK8中经历了重要的变化。

JDK 8之前，HotSpot虚拟机中，**使用永久代实现方法区**，永久代与方法区的概念并不等价，永久代只是方法区的实现而已。

方法区只是逻辑上的分区概念，真正实现方式是永久代或元空间。

从JDK6版本有了移除永久代的计划，直到JDK 8版本，永久代被彻底移除，取而代之的是**元空间（Meta-space）**，具体演变过程如下（参考《深入理解Java虚拟机 第三版》P46；JVM**[教程](https://www.bilibili.com/video/BV1PJ411n7xZ?p=97)**）：

* JDK 6版本及之前，使用永久代实现方法区，此时的永久代，或者说方法区，使用的是JVM内存。

* JDK 7版本，把原本在永久代的**字符串常量池**、**静态变量**等移出到了**堆**中，永久代仍然使用JVM内存。

  > 静态变量仅是指的引用名（变量本身），只是引用名的位置发生了改变，new出来的对象实体，一直是保存在堆中。参考《深入理解Java虚拟机》P152的案例。

* JDK 8版本，将永久代中剩余的内容，包括类型信息、字段、方法、常量等，移出到了**元空间**中。彻底废弃了永久代的概念。元空间使用的是**本地内存（Native Memory）**。注意，JDK8中仍然有方法区的概念，不过是实现方法变成了本地内存的元空间。

如图：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/method-area-evolution_1.jpg"/>



# 2、为什么要废弃永久代

关于为什么要废弃永久代，使用本地内存的元空间代替，官方解释原因参考([来源](http://openjdk.java.net/jeps/122))：

**Motivation**

This is part of the JRockit and Hotspot convergence effort. JRockit customers do not need to configure the permanent generation (since JRockit does not have a permanent generation) and are accustomed to not configuring the permanent generation.

上述内容简单来说，是因为Oracle官方想要将JRockit虚拟机的优秀功能移植到HotSpot虚拟机中，而JRockit虚拟机不存在永久代的概念，为了保持一致，将HotSpot虚拟机中的永久代移除。

这种说法并未从深层次解释原因，关于为什么不使用永久代的原因，需要从永久代本身的一些特点来解释：

* **为永久代设置空间大小是很难确定的**。某些场景下，如果动态加载类过多，容易产生永久代(Perm)的OOM问题。因为永久代有`-XX:MaxPermSize`参数，限制了永久代的上限空间，32位系统默认大小是4GB。而JRockit虚拟机没有这种限制，只要不超过线程可用本地内存上限，就不会有问题。**元空间不在虚拟机中，而是使用本地内存，默认情况下只受本地内存大小限制**。

  > HotSpot虚拟机中的永久代，设计初衷是将收集器的分代机制扩展至方法区，即使用永久代实现方法区，方便垃圾收集器能够像管理Java堆一样管理这部分内存，省去专门为方法区编写内存管理代码的工作。现在回头看来，这种做法并不是好主意。

* **对永久代的调优很困难**。对方法区的垃圾回收，涉及到常量池回收和**类型的卸载**（类的回收）等，类型回收的条件相当苛刻，比较消耗时间。



# 3、为什么要移动字符串常量池



JDK 7中将String Table（字符串常量池）放到了堆空间中。因为永久代的回收效率很低，在full GC时才会回收永久代。而full GC在老年代空间不足、永久代空间不足时才会触发，这就导致StringTable的回收效率并不高。开发中会有大量的字符串被创建，如果回收效率低，会导致永久代内存不足。放到堆里面，能及时回收内存。