---
title: 深入理解String的intern()方法
excerpt: 从JVM的角度理解字符串的创建和拼接过程，理解intern()方法
mathjax: true
date: 2021-05-10 19:40:39
tags: ['JVM','Java']
categories: JVM
keywords: JVM,String,intern
---



# 一、String基本特征

1. Java中的字符串String，使用`""`引起来表示。两种实例化方式：

   * `String s1 = “hello”;`
   * `String s2 = new String("hello");`

2. String声明为final的，不可被继承

3. String类实现了Serializable接口、Comparable接口。

4. String类重写了`equals()`方法，代码如下：

   ```java
   //JDK 15
   public boolean equals(Object anObject) {
       if (this == anObject) {
           return true;
       }
       if (anObject instanceof String) {
           String aString = (String)anObject;
           if (!COMPACT_STRINGS || this.coder == aString.coder) {
               return StringLatin1.equals(value, aString.value);
           }
       }
       return false;
   }
   ```

5. **JDK8之前，String使用char型数组存放数据，JDK9改为byte型数组**。修改原因，官方描述如下，[参考](http://openjdk.java.net/jeps/254)：

   ***

   We propose to change the internal representation of the String class from a UTF-16 char array to a byte array plus an encoding-flag field. The new String class will store characters encoded either as ISO-8859-1/Latin-1 (one byte per character), or as UTF-16 (two bytes per character), based upon the contents of the string. The encoding flag will indicate which encoding is used.

   String-related classes such as AbstractStringBuilder, StringBuilder, and StringBuffer will be updated to use the same representation, as will the HotSpot VM's intrinsic string operations.

   This is purely an implementation change, with no changes to existing public interfaces. There are no plans to add any new public APIs or other interfaces.

   The prototyping work done to date confirms the expected reduction in memory footprint, substantial reductions of GC activity, and minor performance regressions in some corner cases. 

   ***

6. String代表不可变的字符序列，详细使用可以参考[Java学习笔记09(1)-常用类之String](https://kangshitao.github.io/2021/04/03/java-note-0901/)。

7. 通过字面量定义的字符串存储在**字符串常量池(String Pool)**（JDK7将字符串常量池移到了堆中）中，目的是共享；通过`new`创建对象的方式构建的字符串存储在**堆**（堆中字符串常量池以外的空间）中。

8. **字符串常量池中是不会存储相同内容的字符串**：

   * String Pool 是一个固定大小的`Hashtable`（底层采用数组和链表实现，**Hashtable无法扩容**，这一点不同于HashMap）。如果放进String Pool的String非常多， 就会造成严重的哈希冲突，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用`String.intern()`时性能会大幅下降。
   * 使用` - XX:StringTableSize`可设置StringTable的长度
   * 在JDK 6中`StringTable`的值默认大小为`1009`，所以如果常量池中的字符串过多就会导致效率下降很快。对于`StringTableSize`的设置没有要求。
   * 在JDK 7中，`StringTable`的长度默认值是`60013`。`StringTableSize`的设置没有要求。
   * 从JDK 8开始，`StringTable`长度可设置的**最小值**要求为`1009`。



# 二、String的内存分配

Java中，8中基本数据类型的常量池都是系统协调的，**String类型的常量池比较特殊**，它的主要使用方法有两种：

* 直接使用`""`号，即**字面量的方式，声明出来的字符串对象会直接存储在常量池**中。
  * 比如：`String s = "hello";`
* 如果**不是使用`""`的方式（使用new方式，或者和变量拼接等方式）声明的String对象是放在堆中的**。可以使用String类提供的`intern()`方法，`intern()`方法会从字符串常量池中查询当前字符串是否存在，如果存在，直接返回字符串，若不存在就会将当前字符串放入常量池中并返回。



JDK 6及以前，字符串常量池存放在永久代。

**JDK 7中，字符串常量池被移到了堆中**（原因：permSize默认空间比较小；永久代垃圾回收频率低）：

* 所有的字符串都保存在Heap中，和其他普通对象一样，调优时仅需调整堆大小即可。
* 这次改动让我们重新考虑在JDK 7中使用`String.intern();`。因为调整以后，**常量池中不需要必须存储一份对象了，可以直接存储堆中的引用**。也就是说，调用此方法时，**如果堆中已经存在相等的String对象，会直接保存对象的引用，而不会重新创建对象**。这一点的体现可以参考下文的例题。

JDK 8中，字符串常量池依然是在堆中。



# 三、String的基本操作

1. 情况一：Java语言规范（[链接](https://docs.oracle.com/javase/specs/jls/se15/html/jls-3.html#jls-3.10.5)）里要求，完全相同的字符串字面量，应该包含同样的unicode字符序列，并且必须是指向同一个String类实例。

   ```java
   public class StringTest4 {
       public static void main(String[] args) {
           System.out.println();//3142个已加载字符串常量
           System.out.println("1");//3143
           System.out.println("2");//3144
           System.out.println("3");;//3145
           //如下的字符串"1" 到 "3"不会再次加载
           System.out.println("1");//3145
           System.out.println("2");//3145
           System.out.println("3");//3145
       }
   }
   ```

2. 情况二：

   ```java
   class Memory {
       public static void main(String[] args) {//line 1
           int i = 1;//line 2
           Object obj = new Object();//line 3
           Memory mem = new Memory();//line 4
           mem.foo(obj);//line 5
       }//line 9
   
       private void foo(Object param) {//line 6
           String str = param.toString();//line 7
           System.out.println(str);
       }//line 8
   }
   ```

   上述程序的内存结构如下：

   <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/understandint-string-intern_1.png"/>



# 四、字符串拼接操作

## 1、字符串拼接的特征

字符串拼接有以下特征：

1. 常量与常量的拼接，结果在常量池，原理是**编译期优化**，生成字节码文件的时候，已经生成了常量的拼接结果。

   ```java
   public class StringTest {
       @Test
       public void test1(){
           String s1 = "a" + "b" + "c";//编译期优化：等同于"abc"
           String s2 = "abc"; //"abc"一定是放在字符串常量池中，将此地址赋给s2
           System.out.println(s1 == s2); //true
           System.out.println(s1.equals(s2)); //true
       }
   }
   /*
   在前端编译生成的字节码文件中，已经将s1优化为“abc”
   如下为部分字节码文件内容：
    0 ldc #7 <abc>    可见，s1的值已经优化为"abc"
    2 astore_1
    3 ldc #7 <abc>
    5 astore_2
    6 getstatic #9 <java/lang/System.out>
    9 aload_1
   10 aload_2
   ...
   */
   ```

   

2. 常量池中不会存在相同内容的常量。

3. 如果**拼接的字符串里有变量，结果在堆中**（堆中字符串常量池以外的堆空间），相当于`new`了新对象。拼接的原理是StringBuilder。

   ```java
   public class StringTest{
       public void test3(){
           String s1 = "a";
           String s2 = "b";
           String s3 = "ab";
           String s4 = s1 + s2;
           System.out.println(s3 == s4);//false
       }
   }
   /*
   上述的s1 + s2 的具体执行细节：(变量s是为了容易理解定义的，实际上是匿名的）
   ① StringBuilder s = new StringBuilder();
   ② s.append("a");
   ③ s.append("b");
   ④ s.toString();  --> 约等于 new String("ab")
   补充：在jdk5.0之后使用的是StringBuilder,在jdk5.0之前使用的是StringBuffer
   
   s3在堆中的字符串常量池中，而s4在堆中字符串常量池以外的位置，因此二者地址不相等
   */
   ```

   上述程序的字节码文件如下：

   ```java
   0 ldc #47 <a>
   2 astore_1
   3 ldc #49 <b>
   5 astore_2
   6 ldc #51 <ab>
   8 astore_3
   9 new #33 <java/lang/StringBuilder>  
   12 dup
   13 invokespecial #35 <java/lang/StringBuilder.<init>>
   16 aload_1
   17 invokevirtual #36 <java/lang/StringBuilder.append> 
   20 aload_2
   21 invokevirtual #36 <java/lang/StringBuilder.append> 
   24 invokevirtual #40 <java/lang/StringBuilder.toString> 
   27 astore 4
   29 getstatic #9 <java/lang/System.out>
   32 aload_3
   33 aload 4
   35 if_acmpne 42 (+7)
   38 iconst_1
   39 goto 43 (+4)
   42 iconst_0
   43 invokevirtual #15 <java/io/PrintStream.println>
   46 return
   ```

   通过变量拼接字符串的过程，可以总结如下，当遇到有变量的字符串拼接时：

   * 首先使用`new`方式创建`StringBuilder`对象（JDK 5.0之前是StringBuffer）
   * 然后调用`append()`方法，将要拼接的字符串内容添加到`StringBuilder`对象中
   * 最后调用`StringBuilder`对象的`toString()`方法，将其转换为`String`类型，赋给指定的变量。

   

   此外，还需要注意的是，如果字符串拼接是变量，但是是用`final`修饰的，此时这种变量就是常量，同样在编译期优化，结果存放到字符串常量池中：

   ```java
   public class StringTest{
       public void test4(){
           final String s1 = "a"; //常量
           final String s2 = "b";
           String s3 = "ab";
           String s4 = s1 + s2; //此时的s4等价于"a"+"b"
           System.out.println(s3 == s4);//true
       }
   }
   ```

   建议：针对于`final`修饰类、方法、基本数据类型、引用数据类型的量的结构时，能用`final`时尽量使用。这样前端编译期就能赋值，提高运行效率。

4. 如果拼接的结果调用`intern()`方法，则主动将常量池中没有的字符串对象放入池中，并返回此对象的地址（引用，reference），具体参加下一节内容。



## 2、‘+’号和append方法对比

对于字符串拼接，可以使用`+`号直接拼接字符串，也可以在`StringBuffer`/`StringBuilder`对象中使用`append()`方法，二者的效率对比如下：

```java
public class StringTest{
    public void test6(){
        long start = System.currentTimeMillis();
        method1(100000);//3989ms
        //method2(100000);//7ms
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为：" + (end - start));
    }
    
    public void method1(int highLevel){
        String src = "";
        for(int i = 0;i < highLevel;i++){
            src = src + "a";//每次循环都会创建一个StringBuilder、String
        }
    }

    public void method2(int highLevel){
        //只需要创建一个StringBuilder
        StringBuilder src = new StringBuilder();
        for (int i = 0; i < highLevel; i++) {
            src.append("a");
        }
    }
}
```

以上结果表明，使用`StringBuffer`/`StringBuilder`对象中的`append()`方法进行字符串拼接，效率要远高于`+`号拼接String的方式。

* 使用`+`号拼接时，**每次**都需要创建一个`StringBuffer`/`StringBuilder`对象，然后调用`append()`方法添加内容，最后调用`toString()`方法生成`String`对象并重新赋值。
* 使用`append()`方式，只创建了一个`StringBuffer`/`StringBuilder`对象，因此效率要高很多。
* 此外，用String的字符串拼接方式，内存中由于创建了较多的`StringBuilder`和`String`的对象，内存占用更大，如果进行GC，需要花费额外的时间。

因此，实际开发中，如果需要大量的字符串操作，尽量使用`StringBulider`。

如果确定字符串长度不高于某个值的情况下，建议使用带参的构造器实例化`StringBuilder`，底层会一次性申请指定长度的数组，这样可以避免频繁扩容带来的效率降低。



# 五、intern()的使用

## 1、intern（）方法

JDK源码中`intern()`方法的定义和描述：

```java
/*
Returns a canonical representation for the string object.
A pool of strings, initially empty, is maintained privately by the class String.
When the intern method is invoked, if the pool already contains a string equal to
this String object as determined by the equals(Object) method, then the string from
the pool is returned. Otherwise, this String object is added to the pool and a
reference to this String object is returned.

It follows that for any two strings s and t, s.intern() == t.intern() is true if and
only if s.equals(t) is true.

All literal strings and string-valued constant expressions are interned. String
literals are defined in section 3.10.5 of the The Java Language Specification.

Returns:
a string that has the same contents as this string, but is guaranteed to be from a
pool of unique strings.
*/
public native String intern();
```

根据描述可知，当`str`调用`str.intern()`时，如果字符串常量池中已经有和`str`相等的字符串（使用`equals()`方法判断），则返回当前字符串（或者说字符串对象的引用）；如果没有`str`字符串的话，会在常量池中生成，然后再返回此字符串。

`s.intern() == t.intern()`等价于`s.equals(t) 为 true`

如果不是用`""`声明的String对象，可以使用String提供的`intern()`方法，`intern()`方法会从**字符串常量池**中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中，比如：

```java
//返回的结果是字符串常量池中的引用，即myInfo指向了字符串常量池中的“hello"
String myInfo = new String("hello").intern();
```



也就是说，如果在任意字符串上调用`intern()`方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是true：

```java
("a" + "b" + "c").intern() == "abc";  //结果为true
```

通俗点讲，`Interned String`确保字符串在字符串常量池里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。

进一步来说，例如，想要保证变量`s`指向的是字符串常量池中的数据，比如"hello"这个字符串，有两种方式：

* 直接使用字面量定义的方式，`String s = "hello";`
* 调用String类的`intern()`方法，比如`String s = new String("hello").intern();`

> 无论字符串是怎样方式定义或生成的，只要调用`intern()`方法，返回的一定是此字符串在字符串常量池中的引用



## 2、关于new String()

问题引申一：如果使用`new String("ab")`方式创建字符串对象，会创建几个对象呢？答案是两个

通过字节码可以看出：

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("ab");
    }
}
//以上代码的字节码文件如下：
 0 new #7 <java/lang/String>  //1.创建String对象
 3 dup
 4 ldc #9 <ab>   //2.字符串常量池中的对象（数组）
 6 invokespecial #11 <java/lang/String.<init>>
 9 astore_1
10 return
//根据字节码文件可知，底层创建了String对象和字符串常量池中的对象"ab"
```

**字符串常量池中创建了"ab"字符串对象**。



问题引申二：如果使用`new String("a")+new String("b")`方式创建字符串对象，会创建几个对象呢？答案是6个

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("a") + new String("b");
    }
}
//以上程序字节码文件为：
 0 new #7 <java/lang/StringBuilder>  //对象1
 3 dup
 4 invokespecial #9 <java/lang/StringBuilder.<init>>
 7 new #10 <java/lang/String> //对象2
10 dup
11 ldc #12 <a>  //对象3
13 invokespecial #14 <java/lang/String.<init>>
16 invokevirtual #17 <java/lang/StringBuilder.append>
19 new #10 <java/lang/String>  //对象4
22 dup
23 ldc #21 <b>  //对象5
25 invokespecial #14 <java/lang/String.<init>>
28 invokevirtual #17 <java/lang/StringBuilder.append>
31 invokevirtual #23 <java/lang/StringBuilder.toString> //对象6
34 astore_1
35 return
```

通过字节码文件可知，一共创建了6个对象，分别为：

* `new StringBuilder()`，字节码第0行
* `new String("a")`，字节码第7行
* 常量池的"a"，字节码第11行
* `new String("b")`，字节码第19行
* 常量池的“b“，字节码第23行
* `toString()`方法创建的`String`对象，字节码第31行

这里的`StringBuilder`的`toString()`方法创建了一个`String`对象：

```java
//StringBuilder中toString()方法的源码。JDK15
public String toString() {
    // Create a copy, don't share the array
    return isLatin1() ? StringLatin1.newString(value, 0, count)
        : StringUTF16.newString(value, 0, count);
}
//参数中的value是一个数组
```

值得注意的是，这里`toString()`方法只创建了一个`String`对象，没有在字符串常量池中创建"ab"字符串对象，因为其直接使用数组创建字符串，而不是使用字面量的方式，所以**字符串常量池中没有"ab"这个字符串**。



## 3、通过实例进一步理解

例一，参考[深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)：

```java
public static void main(String[] args) {
    String s = new String("1");
    s.intern();
    String s2 = "1";
    System.out.println(s == s2);  //JDK6 false; JDK7 false

    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4); //JDK6 false; JDK7 true
}
```

可以看到，在JDK6中结果都是`false`，而在JDK7及以后的版本中结果为`false`和`true`。这是为什么呢？

* 在JDK6及以前的版本中，常量池放在永久区（Perm区），永久区和Heap区是完全分开的。前面说过，使用`""`号声明的字符串直接在字符串常量池中生成，而new出来的String对象放在堆中。因此`s`和`s3`都是指向堆中对象的一个引用地址，`s2`和`s4`是方法区字符串常量池中对象地址，无论如何都不相等。

* 对于JDK7及以后的版本：

  * 先看 s 和 s2 对象。 `String s = new String("1");` 第一句代码，生成了2个对象，即常量池中的“1” 和堆中的String对象。其中s是指向堆中的String对象的引用。`s.intern();` 这一句是 s 对象去字符串常量池中寻找后发现 “1” 已经在常量池里了。因此`String s2 = "1";` 这句代码生成一个 s2的引用指向字符串常量池中的“1”对象。 结果就是 s 和 s2 的引用地址明显不同。
  * 对于 s3和s4。`String s3 = new String("1") + new String("1");`这句代码生成了字符串常量池中的“1” ，此时s3引用对象内容是”11”，存放在堆中。**但此时常量池中是没有 “11”对象的**。接下来`s3.intern();`将 s3中的“11”字符串放入 String 常量池中，因为此时常量池中不存在“11”字符串，常规做法是跟 jdk6 那样，在常量池中生成一个 “11” 的对象，关键点是**jdk7 常量池中可以直接存储堆中的引用**。因此常量池中存放的是引用，这份引用指向 s3 引用的对象。 也就是说引用地址是相同的。最后`String s4 = "11";` 这句代码中”11”是显式声明的，因此会直接去常量池中创建，创建的时候发现已经有这个对象了，此时也就是指向 s3 引用对象的一个引用，所以 s4 引用就指向和 s3 一样了，最后比较 `s3 == s4` 是 true。

  <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/understanding-string-intern_2.png"/>

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/understanding-string-intern_3.png"/>



例二，将例一中的`intern()`语句下调一行：

```java
public static void main(String[] args) {
    String s = new String("1");
    String s2 = "1";
    s.intern();
    System.out.println(s == s2); //JDK6 false; JDK7 false

    String s3 = new String("1") + new String("1");
    String s4 = "11";  //在字符串常量池中生成了“11”
    s3.intern();
    System.out.println(s3 == s4); //JDK6 false; JDK7 false
}
```

此时不同版本的结果都是`false`：

*  对于s 和 s2 ，`s.intern();`，这一句往后放不会有什么影响，因为对象池中在执行第一句代码`String s = new String("1");`的时候已经生成“1”对象了。下边的s2声明都是直接从常量池中取地址引用的。 s 和 s2 的引用地址是不会相等的。
* 对于s3和s4，首先执行`String s4 = "11";`声明 s4 的时候常量池中是不存在“11”对象的，执行完毕后，“11“对象是 s4 声明产生的新对象，方法返回字符串常量池中“11”的引用赋给s4。然后再执行`s3.intern();`时，常量池中“11”对象已经存在了，因此 s3 和 s4 的引用是不同的。s3是指向堆中对象的引用，而s4是字符串常量池中的引用，所以结果是`false`

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/understanding-string-intern_4.png"/>



## 4、总结

String中`intern()`的作用：

* 在JDK 6中，`intern()`尝试将字符串对象放入字符串常量池。
  * 如果常量池中有，则不会放入，返回已有的对象地址
  * 如果没有，会把此**对象复制一份**，放入常量池，并返回常量池中对象地址。
* 在JDK7及以后，`intern()`尝试将字符串对象放入字符串常量池。
  * 如果常量池中有，则不会放入，返回已有的对象地址
  * 如果没有，会把此**对象的引用地址复制一份**，放入常量池，并返回常量池中对象引用地址。



## 5、练习

练习1：

```java
public class StringExer1 {
    public static void main(String[] args) {
        String s = new String("a") + new String("b");//new String("ab")
        //在上一行代码执行完以后，字符串常量池中并没有"ab"

        /*
        jdk6中：在串池中创建一个字符串"ab"
        jdk7中：串池中没有创建字符串"ab",
        而是创建一个引用，指向new String("ab")，将此引用返回
        */
        String s2 = s.intern();
        System.out.println(s2 == "ab");//jdk6:true  jdk7:true
        System.out.println(s == "ab");//jdk6:false  jdk7:true
        //"ab"与String s3 = "ab"写法结果相同
    }
}
```



练习2：

```java
public class StringExer2 {
    public static void main(String[] args) {
        //执行完以后，不会在字符串常量池中会生成"ab"
        String s1 = new String("a") + new String("b");
        s1.intern();
        String s2 = "ab";
        System.out.println(s1 == s2); //jdk6 false; jdk7 true
    }
}
```



练习3：

```java
public class StringExer3 {
    public static void main(String[] args) {
        //执行完以后，会在字符串常量池中会生成"ab"
        String s1 = new String("ab");
        s1.intern();
        String s2 = "ab";
        System.out.println(s1 == s2); //jdk6 false; jdk7 false
    }
}
```



## 6、intern（）的使用

实际开发中，如果程序中存在大量重复字符串，使用`intern()`会节省大量空间，与JDK版本无关。

```java
public class StringIntern2 {
    static final int MAX_COUNT = 1000 * 10000;
    static final String[] arr = new String[MAX_COUNT];
    
    public static void main(String[] args) {
        Integer[] data = new Integer[]{1,2,3,4,5,6,7,8,9,10};
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
            //两种创建字符串的方式，分别为不使用intern和使用intern
            //arr[i] = new String(String.valueOf(data[i % data.length]));
            arr[i] = new String(String.valueOf(data[i % data.length])).intern();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为：" + (end - start));
    }
}
```

通过运行程序可知，使用intern()方法时，占用的空间大小相较于不使用intern()时要小很多，主要原因是因为：

* 不使用`intern()`创建字符串对象，会在堆中生成大量的String对象，且不能回收，导致空间占用很大。

* 使用`intern()`方法创建对象，如果字符串常量池中已经存在要创建的对象，则数组元素直接指向字符串常量池中的字符串对象，无需在堆中维护过多的String对象（或者说创建之后可以被回收销毁），因此空间占用小。

  > 注意，两种方式在字符串常量池中创建的字符串个数是相等的，区别只是堆中字符串对象的数量不同

实际应用：在大的网站平台，需要内存中存储大量字符串，这时候如果字符串都调用`intern()`方法，会明显降低内存大小。

关于intern()方法错误的使用，也可以参考[深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)



# 六、G1中的String去重操作

Garbage First收集器对String的去重操作，参考官方说明[JEP 192: String Deduplication in G1](http://openjdk.java.net/jeps/192)

**背景**：

对许多Java应用（有大的也有小的）做的测试得出以下结果:

* 堆存活数据集合里面string对象占了25%
* 堆存活数据集合里面重复的string对象有13.5%
* string对象的平均长度是45

许多大规模的Java应用的瓶颈在于内存，测试表明，在这些类型的应用里面，Java堆中存活的数据集合差不多25%是string对象。更进一步，这里面差不多一半string对象是重复的，即`string1.equals(string2 )=true`。堆上存在重复的string对象必然是一种内存的浪费。在G1垃圾收集器中实现自动持续对重复的strfing对象进行去重，这样就能避免浪费内存。

**实现**：

* 当垃圾收集器工作的时候，会访问堆上存活的对象。对每一个访问的对象都会检查是否是候选的要去重的string对象。
* 如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去引用和它一样的string对象。
* 使用一个hashtable来记录所有的被string对象使用的不重复的char数组。当去重的时候，会查这个hashtable，来看堆上是否已经存在一个一模一样的char数组。
* 如果存在，string对象会被调整为引用那个数组，释放对原来的数组的引用，最终会被垃圾收集器回收掉。
* 如果查找失败，char数组会被插入到hashtable，这样以后的时候就可以共享这个数组。

**命令行选项**：

- `UseStringDeduplication` (`bool`) ：开启String 去重，默认是不开启的
- `PrintStringDeduplicationStatistics` (`bool`) ：打印详细的去重统计信息
- `StringDeduplicationAgeThreshold` (`uintx`) ：达到这个年龄的String对象被认为是去重的候选对象。