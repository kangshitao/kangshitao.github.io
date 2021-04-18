---
title: Java学习笔记09(3)-常用类之Java比较器和其他常用类
excerpt: Comparable与Comparator接口、System类、Math类、BigInteger与BigDecimal
mathjax: true
date: 2021-04-05 15:05:13
tags: Java
categories: Java
keywords: Java,Java比较器,Comparable接口,Comparator接口
---

# 一、Java比较器

Java实现对象排序有两种方式：

* 自然排序：`java.lang.Comparable`
* 定制排序：`java.util.Comparator`

## 1、Comparable

`Comparable`接口强行对实现它的每个类的对象进行整体排序，这种排序称为类的自然排序。

实现`Comparable`的类，必须实现`compareTo(Object obj)`方法，从小到大的排序规则如下：

* 如果当前对象this大于obj，返回正整数。
* 如果当前对象this小于obj，返回负整数。
* 如果当前对象this等于obj，返回0。

> 以上规则是从小到大排列，如果想从大到小排列，则正负和上面相反。

`String`类、包装类等类实现了`Comparable`接口，并实现了`compareTo()`方法，进行了从小到大的排列。

实现`Comparable`接口的对象列表可以通过`Collections.sort`或`Arrays.sort`进行自动排序，比如`String`数组就可以自动比较大小，自动排序：

```java  
public class CompareTest {
    @Test
    public void test(){
        String[] str = new String[]{"DD","BB","AA","CC"};
        //String实现了Comparable接口，String对象可以比较大小
        Arrays.sort(str);  //借助Arrays.sort进行排序
        System.out.println(Arrays.toString(str));
        //输出结果为：[AA, BB, CC, DD]
    }
}
```



## 2、Comparator

当元素的类型没有实现`java.lang.Comparable`接口而又不方便修改代码，或者实现了`java.lang.Comparable`接口的排序规则不适合当前的操作，那么可以考虑使用`Comparator`的对象来排序，强行对多个对象进行整体排序的比较。

实现`Comparator`接口需要实现其`compare(Obj o1, Obj o2)`方法，规则和`compareTo(Object obj)`相同。

同样地，可以将`Comparator`传递给`sort`方法（如`Collections.sort`或`Arrays.sort`），控制排序规则。还可以使用`Comparator`来控制某些数据结构（如有序set或有序映射）的顺序，或者为那些没有自然顺序的对象`collection`提供排序。

举例：

```java
public class CompareTest {
        @Test
    public void test3(){
        String[] str = new String[]{"DD","BB","AA","CC"};
        //实现Comparator接口，自定义排序规则。需要重写compare方法
        //Arrays.sort方法的第二个参数，需要比较器对象，这里使用匿名实现类对象
        Arrays.sort(str, new Comparator() {
            @Override
            public int compare(Object o1, Object o2) {
                //如果使用泛型，这里可以不用强制转型
                if(o1 instanceof String && o2 instanceof String){
                    String s1 = (String) o1;
                    String s2 = (String) o2;
                    return -s1.compareTo(s2);
                }else{ throw new RuntimeException("类型错误");}
            }
        });
        System.out.println(Arrays.toString(str));
        //临时改变了排序规则，此时输出为[DD, CC, BB, AA]
    }
}
```

> `Comparable`接口的方式一旦指定，保证`Comparable`接口实现类的对象在任何位置都可以比较大小。
> `Comparator`接口属于临时性的比较。



# 二、System类

`java.lang.System`类代表系统，系统级的很多属性和控制方法都放置在该类的内部。

`System`类的构造器是`private`的，因此无法创建该类的对象，其内的成员变量和成员方法都是`static`的，可以方便调用。

**成员变量**：

* `in`：标准输入流
* `out`：标准输出流
* `err`：标准错误输出流

**成员方法**：

* `native long currentTimeMillis()`：返回当前的计算机时间，时间的表达格式为当前计算机时间和GMT时间(格林威治时间)1970年1月1号0时0分0秒所差的毫秒数。

* `void exit(int status)`：退出程序。其中status的值为0代表正常退出，非零代表异常退出。使用该方法可以在图形界面编程中实现程序的退出功能等。

* `void gc()`：请求系统进行垃圾回收。至于系统是否立刻回收，则取决于系统中垃圾回收算法的实现以及系统执行时的情况。

* `String getProperty(String key)`：获得系统中属性名为key的属性对应的值。系统中常见的属性名以及属性的作用如下：

  * `java.version`：Java运行时环境版本
  * `java.home`：Java安装目录
  * `os.name`：操作系统的名称
  * `os.version`：操作系统的版本
  * `user.name`：用户名称
  * `user.home`：用户的主目录
  * `user.dir`：用户的当前目录

  代码举例：

  ```java
  public class OtherClass {
      @Test
      public void test1() {
          System.out.println("java的version:" + System.getProperty("java.version"));
          System.out.println("java的home:" + System.getProperty("java.home"));
          System.out.println("os的name:" + System.getProperty("os.name"));
          System.out.println("os的version:" + System.getProperty("os.version"));
          System.out.println("user的name:" + System.getProperty("user.name"));
          System.out.println("user的home:" + System.getProperty("user.home"));
          System.out.println("user的dir:" + System.getProperty("user.dir"));
      }
  }
  /*以上代码输出结果为：
  java的version:15.0.1
  java的home:D:\JDK15
  os的name:Windows 10
  os的version:10.0
  user的name:xxx
  user的home:C:\Users\xxx
  user的dir:D:\IntelliJ IDEA\Project\Java_shangguigu\JavaSenior\day04
  */
  ```

  

# 三、Math类

`java.lang.Math`类提供了一系列静态方法用于科学计算，其方法的参数和返回值类型一般为`double`型：

* `abs`：绝对值
* `acos`/`asin`/`atan`/`cos`/`sin`/`tan`：三角函数
* `sqrt`：平方根
* `pow(double a, double b)`：求a的b次幂
* `log`：自然对数
* `exp`：e为底数
* `max(double a, double b)`：求a、b之间的最大值
* `min(double a, double b)`：求a、b之间的最小值
* `random()`：返回0.0到1.0之间的随机数
* `long round(double a)`：double型数据a转换为long型（四舍五入）
* `toDegrees(double angrad)`：弧度-->角度
* `toRadians(double angdeg)`：角度-->弧度

# 四、BigInteger与BigDecimal

## 1、BigInteger

`Long`是`long`的包装类，最大值为$2^{63}-1$，如果是再大的数，就需要使用`BigInteger`类。`java.math.BigInteger`表示**不可变的任意精度的整数**。`BigInteger`类用`int[]`来模拟非常大的整数，`BigInteger`类对象进行计算的时候只能使用实例方法。

构造器：

* `BigInteger(String val)`：根据字符串构建BigInteger对象

常用方法：

* `public BigInteger abs()`：返回此BigInteger的绝对值的BigInteger。
* `BigInteger add(BigInteger val)`：返回其值为(this + val)的BigInteger
* `BigInteger subtract(BigInteger val)`：返回其值为(this-val)的BigInteger
* `BigInteger multiply(BigInteger val)`：返回其值为(this * val)的BigInteger
* `BigInteger divide(BigInteger val)`：返回其值为(this / val)的BigInteger。整数相除只保留整数部分。
* `BigInteger remainder(BigInteger val)`：返回其值为(this % val)的BigInteger。
* `BigInteger[] divideAndRemainder(BigInteger val)`：返回包含(this / val)后跟(this % val)的两个BigInteger的数组。
* `BigInteger pow(int exponent)`：返回其值为this的exponent次幂的BigInteger。

BigInteger和基本数据类型转换：

- 转换为`byte`：`byteValue()`
- 转换为`short`：`shortValue()`
- 转换为`int`：`intValue()`
- 转换为`long`：`longValue()`
- 转换为`float`：`floatValue()`
- 转换为`double`：`doubleValue()`

> 直接调用以上方法，如果超出范围，不会抛出异常，会导致结果不正确。
>
> 调用`xxxValueExact()`方法，转换超出范围时，会抛出`ArithmeticException`异常，以保证结果准确。

## 2、BigDecimal

`java.math.BigDecimal`表示一个任意大小且精度完全准确的浮点数，同样是不可变的。

`BigDecimal`用`scale()`表示小数位数，小数末尾是0的时候也计算在内。如果调用`scale()`方法，返回值为负数，比如`-2`，表示这个数是整数，并且末尾有两个0。

构造器：

* `public BigDecimal(double val)`
* `public BigDecimal(String val)`

常用方法：

* `public BigDecimal setScale(int newScale)`：设置当前BigDecimal的精度，如果比原始值低，根据指定的处理方式进行四舍五入或者直接截断。
* `public BigDecimal stripTrailingZeros()`：将一个BigDecimal格式化为一个相等的，但去掉了末尾0的BigDecimal
* `public BigDecimal add(BigDecimal augend)`
* `public BigDecimal subtract(BigDecimal subtrahend)`
* `public BigDecimal multiply(BigDecimal multiplicand)`
* `public BigDecimal divide(BigDecimal divisor,int scale,int roundingMode)`：如果除不尽，会报`ArithmeticException`异常，此时必须指定精度以及如何截断。

说明：

1. 通过源码可知，一个`BigDecimal`是通过一个`BigInteger`和一个`scale`来表示的，即`BigInteger`表示一个完整的整数，而`scale`表示小数位数。
2. 比较两个`BigDecimal`数的大小，应该使用`compareTo()`方法，根据两个值的大小分别返回`-1`、`1`和`0`，分别表示小于、大于和等于。
3. `compareTo()`比较的是两个数值的大小，而`equals()`方法不但要求两个`BigDecimal`的值相等，还要求它们的`scale()`相等：

```java
BigDecimal d1 = new BigDecimal("123.456");
BigDecimal d2 = new BigDecimal("123.45600");
System.out.println(d1.equals(d2)); // false, 因为scale不同

// true,因为d2去除尾部0后scale变为2
System.out.println(d1.equals(d2.stripTrailingZeros()));
System.out.println(d1.compareTo(d2)); // 0，表示相等
```

