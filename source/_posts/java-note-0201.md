---
title: Java学习笔记02-Java基本语法
excerpt: Java基本语法，变量、运算符、程序流程控制
mathjax: true
date: 2021-03-19 20:42:47
tags: Java
categories: Java
keywords: 'Java基本语法,Java'
---

#  一、关键字和保留字

## 1、关键字

[Java关键字](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/_keywords.html)被Java语言赋予了特殊含义，用做专门用途。一共有如下关键字

* 用于定义数据类型：`class`、`interface`、`enum`、`byte`、`short`、`int`、`long`、`float`、`double`、`char`、`boolean`、`void`
* 用于定义流程控制：`if`、`else`、`switch`、`case`、`default`、`while`、`do`、`for`、break`、continue`、`return`
* 用于定义访问权限修饰符：`private`、`protected`、`public`
* 用于定义类，函数，变量修饰符：`abstract`、`final`、`static`、`synchronized`
* 用于定义类与类之间关系：`extends`、`implements`
* 用于定义建立实例及引用实例，判断实例：`new`、`this`、`super`、`instanceof`
* 用于异常处理：`try`、`catch`、`finally`、`throw`、`throws`
* 用于包：`package`、`import`
* 其他修饰符：`native`、`strictfp`、`transient`、`volatile`、`assert`

## 2、保留字

Java保留字指的是现有Java版本尚未使用，但以后版本可能会作为关键字使用。

有两个保留字：goto、const

# 二、标识符

标识符（Identifier）是对各种**变量**、**方法**和**类**等要素命名时使用的字符序列。

1. 标识符命名规则：

   * 由26个英文字母大小写，0-9，_或$组成。
   * 不能以数字开头。
   * 不可以使用关键字和保留字，但能包含关键字和保留字。
   * Java中严格区分大小写，长度无限制。
   * 不能包含空格。

2. Java中命名规范

   * 包名：aaabbbccc
   * 类名、接口名：AaaBbbCcc
   * 变量名、方法名：aaaBbbCcc
   * 常量名：AAA_BBB_CCC

   > 起名时要遵循“**见名知意**”的原则。

   

# 三、变量

## 1、变量的分类

变量是程序中最基本的存储单元，包含**变量类型**、**变量名**和存储的**值**。

根据数据类型分：

<div align='center'>
    <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-0201_01.png"/>
</div>
​	根据声明的位置：

<div align='center'>
   <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-0201_02.jpg"/>
</div>




> **成员变量**是方法体外，类体内声明的变量。
>
> **局部变量**是方法体内部声明的变量。

局部变量除形参外，需要显式初始化。而成员变量有默认的初始化值。

需要注意的点：

> `long`型常量需要在后面加‘l’或‘L’
>
> `float`型常量需要在后面加‘f’或‘F’
>
> Java中整型变量默认为`int`型，浮点型默认是`doule`型

## 2、自动类型转换

### 自动类型转换

以下几种类型从左到右自动类型提升(转换)：

**(byte，char，short) → int → long → float → double**

* `byte`，`short`，`char`之间不会相互转换，他们三者在计算时首先转换为`int`型
* `boolean`类型不能与其他数据类型运算
* 任何基本数据类型和`String`类型进行连接运算(+)时，基本数据类型会自动转化为`String`类型

### 强制类型转换

强制类型转换是自动类型转换的**逆过程**。需要使用强制类型转换符：`()`，例`int a = (int) 3.14;`



# 四、运算符

## 1、算数运算符

算数运算符包括：`+`、`-`、`*`、`/`、`%`、`++`、`--`、`+(字符串连接)`

## 2、赋值运算符

赋值运算符包括：`=`、`+=`、`-=`、`*=`、`/=`、`%=`

> 这里的`+=`、`-=`、`*=`、`/=`、`%=`以及`++`、`--`运算符，都不会改变变量本身的类型。比如：
>
> ```java
> short s = 10;
> s = s + 1;  //编译错误，因为s+1已经被自动提升为int类型。
> s = (short)(s + 1); //使用强制类型转换，正确
> s++;  //自增运算符不会改变数据类型，正确
> s += 1; //正确。
> ```
>
> 

## 3、比较运算符

比较运算符包括：`==`、`!=`、`>`、`<`、`>=`、`<=`、`instanceof`

## 4、逻辑运算符

逻辑运算符包括：`&`、`&&`(短路与)、`|`、`||`(短路或)、`!`、`^(异或)`

## 5、位运算符

位运算符包括：`<<`、`>>`、`>>>(无符号右移)`、`&(按位与)`、`|(按位或)`、`^(按位异或)`、`~(按位取反)`

## 6、三元运算符

`(条件表达式) ? 表达式1 : 表达式2;`表示如果条件表达式为`true`，则返回表达式1，否则返回表达式2。

> 这里的表达式1和表达式2必须是同种类型，或者能够自动类型提升。比如表达式1是`int`类型，表达式2是`double`类型，则表达式1会被自动转换为`double`类型。

# 五、程序流程控制

## 1、顺序结构

Java程序从上到下逐行地执行

## 2、分支结构

* **if语句**

用法：

```java
//用法1：
if(条件表达式){
	执行代码块;
}
//用法2：
if(条件表达式){
	执行代码块1;
}else{
	执行代码块2;
}
//用法3：
if(条件表达式){
	执行代码块1;
}else if{
	执行代码块2;
}
......
else{
	执行代码块3;
}

```

* **switch语句**

用法：

```java
switch(表达式){
case 常量1:
	语句1;
	//break;
case 常量2:
	语句2;
	//break;
......
case 常量N:
	语句N;
	//break;
default:
	语句;
	//break;
}
```

> `switch(表达式)`中的表达式必须是`byte`、`short`、`char`、`int`、`枚举`、`String`这几种类型之一。
>
> `case`子句中的值必须是常量，不能是变量名或不确定的表达式值。
>
> 同一个`switch`语句，所有`case`子句中的常量值互不相同。
>
> 如果没有`break`，程序在运行第一个执行语句以后，会继续向下执行(即使`case`值不同，也会将下面的所有`case`中语句运行完)。
>
> `default`语句是可选的，位置也是灵活的。没有匹配`case`时，会执行`default`的语句。
>
> `switch`语句效率稍高，情况少的时候尽量不用`if`，选择`switch`语句。



##  3、循环结构

* **for语句**

用法：

```java
for(1.初始化部分; 2.循环条件; 4.迭代部分){
	3.循环体部分;
}
```



* **while语句** 

用法：

```java
//用法1
1.初始化部分
while(2.循环条件){
	3.循环体部分;
	4.迭代部分;
}
//用法2
1.初始化部分
do{
	3.循环体部分;
	4.迭代部分;
}while(2.循环条件)
```

## 4、break和continue

`break`用于终止某个语句块的执行，只能用于`switch`语句和循环语句，表示跳过循环体，终止循环。

`continue`只能用在循环体中，表示跳过当前循环，继续下一次循环。

> `break`和`continue`用于循环体内时，都可以使用`label`指明具体要跳过的循环体是哪个。如果没有明确指明，则都默认对当前循环体起作用，**就近原则**。







