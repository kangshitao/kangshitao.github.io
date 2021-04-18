---
title: Java学习笔记03-数组
excerpt: 一维数组，二维数组，排序算法，Arrays工具类
mathjax: true
date: 2021-03-20 21:13:12
tags: Java
categories: Java
keywords: Java,Java数组
---

# 一、数组概述

* 数组中的元素必须是**同一类型**。
* 数组本身是**引用数据类型**，数组的元素可以是**任何数据类型**。
* 数组长度一旦确定就**不能修改**。

**数组在内存中，数组首地址值存放在栈中，数组中的内容存放在堆中。 **

# 二、一维数组

初始化方式：

* 动态初始化：`int[] arr = new int[5];`
* 静态初始化：`int[] arr = new int[]{1,2,3,4,5};`或`int[] arr = {1,2,3,4,5};`

> 中括号可以写在类型后面，也可以写在数组名后面，比如`int arr[]`也是正确的

数组的**默认初始化值**：

* 整型：`0`
* 浮点型：`0.0`
* `char`型：`0`
* `boolean`型：`false`
* 引用类型：`null`

# 三、二维数组

初始化方式：

* 动态初始化：`int[][] arr = new int[5][4];`，`int[][] arr = new int[5][];`，
* 静态初始化：`int[][] arr = new int[][]{{1,2,3},{2,3},{3}};`或`int[] arr = {1,2,3,4,5};`

> 中括号可以写在类型后面，也可以写在数组名后面，`int arr[][]`和`int []arr[]`都是正确的写法。
>
> 动态初始化时，第一个维度必须指定，第二个维度可以先不指定。`int arr[][] arr = new int[][3]`非法。

# 四、数组涉及的常见算法

* 查找算法
* 排序算法，可以参考[排序算法总结](https://kangshitao.github.io/2020/12/27/rank-algorithm/)

# 五、Arrays工具类

`java.util.Arrays`类是操作数组的工具类，以下几个是常用的几个方法：

* `boolean equals(int[] a, int[] b)`：判断两个数组是否相等
* `String toString(int[] a)`：输出数组信息
* `void fill(int[] a, int val)`：将指定值填充到数组之中
* `void sort(int[] a)`：对数组进行排序
* `int binarySearch(int[] a, int key)`：对排序后的数组进行二分法检索指定的值

# 六、数组的常见异常

* 数组索引越界异常：`ArrayIndexOutOfBoundsException`
* 空指针异常：`NullPointerException`