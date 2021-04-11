---
title: Java学习笔记09(1)-常用类之String
excerpt: String类，StringBuffer，StringBuilder
mathjax: true
date: 2021-04-03 21:08:54
tags: Java
categories: Java
keywords: Java
---

# 一、String类

## 1、字符串的使用

`String`类的特征：

* `String`类声明为`final`的，不可以被继承。
* `String`类实现了`Serializable`接口，表示字符串是支持序列化的。
* `String` 类实现了`Comparable`接口，表示字符串可以比较大小
* `String`类内部定义了`final char[] value`，用于存储字符串数据。数组不能被重新赋值，数组元素不能被修改。（JDK 9.0改为了`byte`数组）
* `String`代表不可变的字符序列，**不可变性**，以下操作需要重写指定内存区域进行赋值，不能使用原有的`value`进行赋值：
  * 当对字符串重新赋值时
  * 当对现有的字符串进行连接操作时
  * 当调用String的replace方法修改指定的字符/字符串时

## 2、字符串创建和存储

字符串有两种创建方式：

* 字面量定义：比如 `String s1 = "hello";`
* 创建`String`类对象：比如：`String s2 = new String("hello");`

字面量定义的字符串存储在**字符串常量池（位于方法区）**中，目的是共享；通过创建对象构建的字符串存储在**堆**中，堆中的地址值又指向字符串常量池中的值。

> 字符串常量池中不会保存内容相同的字符串，每个字符串只保存一份。
>
> 通过创建String类对象构造的字符串，内存中一共创建了两个对象，一个String类对象，一个底层的数组对象。

两种方式定义的字符串，在内存中的区别：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-0901_1.png'/>
</div>



具体案例：

```java
public class Test(){
    public static void main(String[] args){
        String s1 = "hello"; //字面量方式定义
        String s2 = "hello";
        String s3 = new String("hello");  //创建对象的方式定义
        String s4 = new String("hello");
        
        //字面量定义的字符串，直接指向常量池，所以地址相等
        System.out.println(s1 == s2);  //true
        //new出来的每个对象有单独的空间和地址，因此地址不同
        System.out.println(s3 == s4); //false
        System.out.println(s1 == s3); //false
        System.out.println(s1 == s4); //false
        //只要其中一个是变量，地址就不相等
        System.out.println("hello"+"java" == "hello"+"java"); //true
        System.out.println("hello"+"java" == "hellojava");  //true
        System.out.println((s1+"java") == (s1+"java")); //false
        System.out.println((s1+"java") == (s2+"java")); //false
        //此时的s5是新创建的字符串，s4本身并没有改变
        String s5 = s4.replace('h','a'); 
        System.out.println(s4);  //hello
        System.out.println(s5);  //aello
    }
}
```

总结：

* 常量(字面量定义的，或者final修饰的，都是常量)与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量。
* 只要其中有一个是变量，结果就在堆中，地址也不同。比如`s1 + s2`、`s1+"abc"`、`s1+="abc"`等，用`==`比较是地址，结果是`false`，比如`s1+"abc" == s1+"abc"`为`false`
* 如果拼接的结果调用`intern()`方法，返回值就在常量池中。



# 二、String类常用方法

String类的常用方法：

`int length()`：返回字符串的长度：`return value.length`
`char charAt(int index)`：返回某索引处的字符：`return value[index]`
`boolean isEmpty()`：判断是否是空字符串：`return value.length==0`
`String toLowerCase()`：使用默认语言环境，将String中的所有字符转换为小写
`String toUpperCase()`：使用默认语言环境，将String中的所有字符转换为大写
`String trim()`：返回字符串的副本，忽略前导空白和尾部空白
`boolean equals(Object obj)`：比较字符串的内容是否相同
`boolean equalsIgnoreCase(String anotherString)`：与equals方法类似，忽略大小写
`String concat(String str)`：将指定字符串连接到此字符串的结尾。等价于用“+”
`int compareTo(String anotherString)`：比较两个字符串的大小
`String substring(int beginIndex)`：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。
`String substring(int beginIndex,int endIndex)`：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。
`boolean endsWith(String suffix)`：测试此字符串是否以指定的后缀结束
`boolean startsWith(String prefix)`：测试此字符串是否以指定的前缀开始
`boolean startsWith(String prefix,int toffset)`：测试此字符串从指定索引开始的子字符串是否以指定前缀开始
`boolean contains(CharSequence s)`：当且仅当此字符串包含指定的char值序列时，返回true
`int indexOf(String str)`：返回指定子字符串在此字符串中第一次出现处的索引
`int indexOf(String str, int fromIndex)`：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
`int lastIndexOf(String str)`：返回指定子字符串在此字符串中最右边出现处的索引
`int lastIndexOf(String str, int fromIndex)`：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索

> 注：`indexOf`和`lastIndexOf`方法如果未找到都是返回-1 

`String replace(char oldChar,char newChar)`：返回一个新的字符串，它是通过用newChar替换此字符串中出现的所有oldChar得到的。
`String replace(CharSequence target,CharSequence replacement)`：使用指定的字面值替换序列替换此字符串所有匹配字面值目标序列的子字符串。
`String replaceAll(String regex,String replacement)`：使用给定的replacement替换此字符串所有匹配给定的正则表达式的子字符串。
`String replaceFirst(String regex,String replacement)`：使用给定的replacement替换此字符串匹配给定的正则表达式的第一个子字符串。
`boolean matches(String regex)`：告知此字符串是否匹配给定的正则表达式。
`String[] split(String regex)`：根据给定正则表达式的匹配拆分此字符串。
`String[] split(String regex,int limit)`：根据匹配给定的正则表达式来拆分此字符串，最多不超过limit个，如果超过了，剩下的全部都放到最后一个元素中。

# 三、String类与基本数据类型之间的转换

## 1、字符串与基本数据类型、包装类

* 字符串-->基本数据类型、包装类:
  * `Integer`包装类的`public static int parseInt(String s)`可将由“数字”字符组成的字符串转换为整型。类似地,使用`java.lang`包中的`Byte`、`Short`、`Long`、`Float`、`Double`类调相应的类方法可以将由“数字”字符组成的字符串，转化为相应的基本数据类型。
* 基本数据类型、包装类-->字符串：
  * 调用`String`类的`public String valueOf(int n)`可将`int`型转换为字符串。相应的`valueOf(byte b)`、`valueOf(long l)`、`valueOf(float f)`、`valueOf(double d)`、`valueOf(boolean b)`可由参数的相应类型到字符串的转换。

## 2、字符串与字符数组

* 字符数组-->字符串：
  * `String`类的构造器：`String(char[])`和`String(char[]，int offset，int length)`分别用字符数组中的全部字符和部分字符创建字符串对象。
* 字符串-->字符数组：
  * `public char[] toCharArray()`：将字符串中的全部字符存放在一个字符数组中。
  * `public void getChars(int srcBegin, int srcEnd, char[]dst, int dstBegin)`：提供了将指定索引范围内的字符串存放到数组中的方法。

## 3、字符串与字节数组

* 字节数组-->字符串：
  * `String(byte[])`：通过使用平台的默认字符集解码指定的byte数组，构造一个新的String。
  * `String(byte[]，int offset，int length)`：用指定的字节数组的一部分，即从数组起始位置offset开始取length个字节构造一个字符串对象。
* 字符串-->字节数组：
  * `public byte[] getBytes()`：使用平台的默认字符集(编码方式)将此String编码为byte序列，并将结果存储到一个新的byte数组中。
  * `public byte[] getBytes(String charsetName)`：使用指定的字符集将此String编码到byte序列，并将结果存储到新的byte数组。

> 字节数组转换为字符串的过程，就是解码的过程，反之，字符串变为字节数组是编码的过程。
>
> 编码和解码默认使用平台默认字符集。
>
> 编码和解码使用的字符集(比如都是UTF-8)应该相同，否则出现乱码。

# 四、StringBuffer和StringBuilder

`StringBuffer`和`StringBuilder`都是**可变的字符序列**，对字符串修改，不会产生新的对象，直接在原来的字符串对象上进行修改。

`String`、`StringBuffer`、`StringBuilder`的异同：

* `String`： 1.0 ，不可变的字符序列；底层使用char[]来保存
* `StringBuffer`：1.0 ，可变的字符序列；**线程安全**的，效率低；底层使用char[]来保存
* `StringBuilder`：JDK 5.0新增 ，可变的字符序列；**线程不安全**，效率高；底层使用char[]来保存

>JDK9.0之后，三者的底层存储结构都改为了byte[]数组
>
>效率从高到低排列：`StringBuilder` > `StringBuffer`> `String`

`StringBuffer`和`StringBuilder`在使用方法上类似，提供的方法也相同。这里以`StringBuffer`为例，讲解其扩容机制以及常用的方法。

## 1、构造StringBuffer/StringBuilder对象

`StringBuffer`类不同于`String`，其对象必须使用构造器生成。有三个构造器：

* `StringBuffer()`：初始容量为16的字符串缓冲区。
* `StringBuffer(intsize)`：构造指定容量的字符串缓冲区。
* `StringBuffer(String str)`：将内容初始化为指定字符串内容。

举例：

```java 
public class test(){
    public static main(String args[]){
        StringBuffer sb1 = new StringBuffer("abc");  //abc
        sb1.setCharAt(0,'m'); //直接修改sb1，不会创建新的对象
        System.out.println(sb1);  //mbc  

        StringBuffer sb2 = new StringBuffer();
        System.out.println(sb2.length());  //0
        StringBuffer sb3 = new StringBuffer();
        String s = null;
        sb3.append(s); //append方法可以将null当作字符串添加进去
        System.out.println(sb3);  //此时的sb3为字符串"null"
        System.out.println(sb3.length()); // 4
        //StringBuffer sb4 = new StringBuffer(null); //构造的时候不行
    }
}
```

`String`、`StringBuffer`、`StringBuilder`三者之间的相互转换：

* `String`-->`StringBuffer`、`StringBuilder`，直接调用两者的构造器：`StringBuffer sb = new StringBuffer(str);`
* `StringBuffer`、`StringBuilder`-->`String`：
  * 调用两者的`toString()`方法
  * 调用`String`的构造器。`String str = new String(sb);`

## 2、扩容机制

对于`StringBuffer`和`StringBuilder`来说，如果添加的数据，底层数组装不下，就需要扩容底层的数组。

默认情况下，扩容为原来容量的2倍+2（JDK 7.0 ，不同JDK版本可能不同），同时将原有数组中的元素复制到新的数组中。

> 关于为什么+2，各种解释：
>
> 1、考虑到在创建Sf，Sb设置的初始长度不大时（例如1），+2 可以很大地提升扩容的效率，减少扩容的次数
>
> 2、在旧版本的JDK扩容语句是 (value.length + 1) * 2 先加一再乘2，推测原意思是扩容的话至少增添一个空间再乘2，兼顾到扩容的次数和要减少扩容过大浪费的空间
>
> 3、newCapacity(int)的传入参数有可能是0，那么在参数是0的情况下，0<<1运算结果也是0，如果没+2，那么在创建数组的时候会创建出MAX_ARRAY_SIZE大小，所以作为设计的安全性考虑，选择了+2。
>
> 4、在使用StringBuffer的时候，append()之后，我们一般会在后面在加上一个分隔符，例如逗号，也就是再加上一个char，而char在java中占2个字节，避免了因为添加分隔符而再次引起扩容

举例：

```java
String str = new String();  //底层操作：new char[0]
String str1 = new String("abc");//底层操作：new char[]{'a','b','c'}

StringBuffer sb = new StringBuffer();   //sb.length()是0
//默认创建长度为16的内容为空的数组，即char[] value = new char[16]
//这里sb.length()的结果是0，返回长度不是底层value数组的长度
sb.append('a');  //底层操作：value[0] = 'a';

StringBuffer sb2 = new StringBuffer("abc"); 
//构建一个长度为"abc".length()+16的数组，即：
//char[] value = new char["abc".length()+16]
```

>为了提高效率，建议使用`StringBuffer(int capacity)` 或 `StringBuilder(int capacity)`指定初始容量，尽量避免自动扩容。



## 3、常用方法

* `String Buffer append(xxx)`：提供了很多的append()方法，用于进行字符串拼接。如果参数是null，会被当成字符串添加进去
* `String Buffer delete(int start,int end)`：删除指定位置的内容
* `String Buffer replace(int start,int end,String str)`：把[start,end)位置替换为str
* `String Buffer insert(int offset,xxx)`：在指定位置插入xxx
* `String Buffer reverse()`：把当前字符序列逆转
* `public int indexOf(String str)`
* `public String substring(int start,int end)`
* `public int length()`
* `public char charAt(int n )`
* `public void setCharAt(int n ,char ch)`