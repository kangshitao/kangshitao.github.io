---
title: Java学习笔记17-Java9&10&11新特征
excerpt: Java9、10、11新特性，模块化，jShell命令，接口私有方法，var
mathjax: true
date: 2021-04-19 16:56:30
tags: Java
categories: Java
keywords: Java,Java9新特性,Java11新特性,jShell,var
---

# 一、Java 9新特性

Java 9的新特性官方[文档](https://docs.oracle.com/javase/9/whatsnew/toc.htm#JSNEW-GUID-C23AFD78-C777-460B-8ACE-58BE5EA681F6)

从Java 9发布开始，Java的计划发布周期变为6个月。

总的来说，Java 9的新特性主要有：

* **模块化系统**
* **jShell命令**
* 多版本兼容jar包
* **接口的私有方法**
* **钻石操作符的使用升级**
* **语法改进：try语句**
* **String存储结构变更**
* **便利的集合特性：of()**
* **增强的StreamAPI**
* **全新的HTTP客户端API**
* **Deprecated的相关API**
* javadoc的HTML5支持
* Javascript引擎升级：Nashorn
* java的动态编译器

## 1、JDK和JRE目录结构改变

Java 9开始，JDK的目录结构发生改变。

**Java 9之前的JDK目录**：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1701_1.png"/>

**Java 9及以后的JDK目录结构**：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1701_2.png"/>



## 2、模块化系统

Java 9之前的项目，存在的缺陷：

* **Java运行环境的膨胀和臃肿**。每次JVM启动的时候，至少会有30～60MB的内存加载，主要原因是JVM需要加载rt.jar，不管其中的类是否被classloader加载，第一步整个jar都会被JVM加载到内存当中去（而模块化可以根据模块的需要加载程序运行需要的class）
* **当代码库越来越大，创建复杂，代码交错的几率呈指数级的增长**。不同版本的类库交叉依赖导致让人头疼的问题，这些都阻碍了Java开发和运行效率的提升。
* **很难真正地对代码进行封装**，而系统并没有对不同部分（也就是JAR文件）之间的依赖关系有个明确的概念。**每一个公共类都可以被类路径之下任何其它的公共类所访问到，这样就会导致无意中使用了并不想被公开访问的API**。

Java 9提出Jigsaw(积木，拼图)项目的概念，之后的版本改名为Modularity，表示模块化项目。

本质上来说，模块(module)的概念其实就是package外再裹一层，就是用模块来管理各个package，通过声明某个package暴露，不声明默认就是隐藏。因此，模块化使得代码组织上更安全，因为它可以指定哪些部分可以暴露，哪些部分隐藏。
**模块化的实现目标**：

* 模块化的主要目的在于减少内存的开销
* 只须必要模块，而非全部jdk模块，可简化各种类库和大型应用的开发和维护
* 改进Java SE平台，使其可以适应不同大小的计算设备
* 改进其安全性，可维护性，提高性能

**模块化的使用**

模块将由**通常的类**和新的**模块声明文件**（`module-info.java`）组成。该文件位于java代码结构的顶层，描述符明确地定义了模块需要什么依赖关系，以及哪些模块被外部使用。在`exports`子句中未提及的所有包默认情况下将封装在模块中，不能在外部使用。

如图：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1701_3.png"/>

要想在`java9demo`模块中调用`java9test`模块下包中的结构，需要在`java9test`的`module-info.java`中声明：

```java
module java9test{
    //将想要对外暴露的包，使用exports导出。
    //没有使用exports语句导出的包，默认被封装到模块里面
    exports com.atguigu.bean;
}
```

对应地，需要在`java9demo`模块的src下，创建`module-info.java`文件，声明下面语句：

```java
module java9demo{
    //requires指明对其他模块的依赖
    requires java9test;
}
```



## 3、jShell命令

**交互式编程环境REPL**(read-evaluate-print-loop)指的是以交互式的方式对语句和表达式求值。

**jShell**是Java语言在Java 9新增的REPL工具，让Java像脚本语言一样运行。无需创建Java文件就可以运行程序。

jShell也可以从文件中加载语句或者将语句保存到文件中。

jShell也可以使用tab键进行自动补全和自动添加分号。

**使用方法**

启动：在命令提示符窗口中输入`jshell`（Windows的DOS命令不区分大小写）即可打开jShell

使用：

* `/help intro`：获取帮助
* `/list`：控制正在执行的操作
* `/help`：获取有关命令的列表
* `/exit`：退出jShell

> jShell环境下，语句末尾的`;`可以省略，为了提高代码可读性，建议不要省略。
>
> jShell中没有受检异常(编译时异常)



## 4、语法改进：接口的私有方法

Java 8中对于接口添加了静态方法和默认方法，Java 9中加入了私有方法。接口更像一个抽象类了。

## 5、语法改进：钻石操作符使用升级

钻石操作符（diamond operator)：`<>`

Java 9中钻石操作符可以和匿名实现类共同使用：

```java
//匿名操作类后面可以使用<>操作符
Comparator<Object> com = new Comparator<>(){
    @Override
    public int compare(Object o1, Object o2){
        return 0;
    }
}
//Java 9以前这样写会报编译异常：Cannot use“<>”with anonymous inner classes
```



## 6、语法改进：try语句

Java 8中，可以实现资源的自动关闭，但是要求**执行后必须关闭的所有资源必须在try子句中初始化**，否则编译不通过：

```java
try(InputStreamReader reader = new InputStreamReader(System.in)){
    //读取数据细节省略
}catch(IOException e){
    e.printStackTrace();
}
```

Java9中，用资源语句编写try将更容易，**可以在try子句中使用已经初始化过的资源**，但是此时的资源是`final`的，无法再被修改：

```java
InputStreamReader reader = new InputStreamReader(System.in);
OutputStreamWriter writer = new OutputStreamWriter(System.out);
try(reader; writer){
    //reader是final的，不可再被赋值
    //reader = null;  //会报错
    //具体读写操作省略
}catch(IOException e) {
    e.printStackTrace();
}
```



## 7、String存储结构改变

Java 9开始，String（包括StringBuffer、StringBuilder）的底层存储结构不再是char型数组，而是用**byte数组加编码标记**，节省空间。



## 8、集合工厂方法

Java 9之前，如果想创建一个只读、不可改变的集合，需要好几步：

```java
List<String> namesList =newArrayList <>(); //创建集合
namesList.add("Joe"); //添加元素
namesList.add("Bob");
namesList.add("Bill");
namesList = Collections.unmodifiableList(namesList); //更改为不可变集合
System.out.println(namesList);

//下面这种方法得到的也是只读集合
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5); 
```

Java 9为集合类添加了静态方法`of()`，用于构建只读集合。包括`List`、`Set`、`Map`。只读集合如果添加元素，会导致`UnsupportedOperationException`异常。

List中的`of()`方法

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1701_4.png"/>

Map中的`of()`方法

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1701_5.png"/>



## 9、InputStream加强

InputStream添加`transferTo`方法，可以用来将数据直接传输到`OutputStream`，这是在处理原始数据流时非常常见的一种用法:

```java
ClassLoader cl =this.getClass().getClassLoader();
try(InputStream is = cl.getResourceAsStream("hello.txt");
    OutputStream os = new FileOutputStream("src\\hello1.txt")) {
    is.transferTo(os);//把输入流中的所有数据直接自动地复制到输出流中
}catch(IOException e) {
    e.printStackTrace();
}
```

## 10、增强的Stream API

Java 9中，Stream接口中添加了4个新的方法：

* `default Stream<T> takeWhile(Predicate<? super T> predicate)`：返回从开头开始，按照指定规则的尽量多的元素（一旦遇到不符合条件的数据，就停止）。
*  `default Stream<T> dropWhile(Predicate<? super T> predicate)`：返回符合条件的前n个元素之后的元素，与`takeWhile`互补。
* `public static<T> Stream<T> ofNullable(T t)`：创建一个单元素的`Stream`实例，`t`可以为空（Java 8中的`of()`方法不能为单个`null`）
* `public static<T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next)`：`iterate`的重载方法，可以提供一个`Predicate` (判断条件)来指定什么时候结束迭代。

## 11、Optional创建Stream对象

除了对`Stream`本身的扩展，`Optional`和`Stream`之间的结合也得到了改进。

现在可以通过`Optional`的新方法`stream()`将一个`Optional`对象转换为一个(可能是空的) `Stream`对象。

比如：

```java
public class Test{
    public void test(){
        List<String> list = new ArrayList<>();
        list.add("Tom");
        list.add("Jerry");
        Optional<List<String>> optional = Optional.ofNullable(list);
        Stream<List<String>> stream = optional.stream();
        stream.flatMap(x -> x.stream()).forEach(System.out::println);
    }
}
```

## 12、JavaScript引擎升级

Nashorn项目在Java 9中得到改进。

# 二、Java 10新特性

Java 10共定义了109个新特性，其中包含12个**JEP(JDK Enhancement Proposal)**，还有新API和JVM规范以及Java语言规范上的改动。

12个JEP：

* 286:Local-Variable Type Inference：局部变量类型推断
* 296:Consolidate the JDK Forest into a Single Repository：JDK库的合并
* 304:Garbage-Collector Interface：统一的垃圾回收接口
* 307:Parallel Full GC for G1：为G1提供并行的Full GC
* 310:Application Class-Data Sharing：应用程序类数据（AppCDS）共享
* 312:Thread-Local Handshakes  ThreadLocal：握手交互
* 313:Remove the Native-Header Generation Tool (javah)：移除JDK中附带的javah工具
* 314:Additional Unicode Language-Tag Extensions：使用附加的Unicode语言标记扩展
* 316:Heap Allocation on Alternative Memory Devices：能将堆内存占用分配给用户指定的备用内存设备
* 317:Experimental Java-Based JIT Compiler：使用基于Java的JIT编译器
* 319:Root Certificates：根证书
* 322:Time-Based Release Versioning：基于时间的发布版本

其中值得关注的是：**Local-Variable Type Inference，局部变量类型推断**。

## 1、局部变量类型推断

使用`var`代替**局部变量**的显示类型声明，适用于编译器可以推断出类型的地方，包括以下三种情况：

* 局部变量的初始化
* 增强for循环中的索引
* 传统for循环中的索引

代码实现：

```java
public class Test{
    public void test1() {
        //1.声明变量时，根据所附的值，推断变量的类型
        var num = 10;
        var list = new ArrayList<Integer>();
        list.add(123);
        
        //2.遍历操作
        for (var i : list) {System.out.println(i);}
        //3.普通的遍历操作
        for (var i = 0; i < 10; i++) {System.out.println(i);}
    }
}
```

`var`不适用于以下情况：

* 没有初始化的局部变量声明
* 方法的返回类型
* 方法的参数类型
* 构造器的参数类型
* 属性
* catch块

**原理**：在处理`var`时，编译器先是查看表达式右边部分，并根据右边变量值的类型进行推断，作为左边变量的类型，然后将该类型写入字节码当中。正确使用`var`生成的字节码文件和不使用`var`时生成的字节码文件内容是相同的。

**注意**：

* `var`不是一个关键字。因此可以作为方法名或变量名，但是不能用它作为类名。除了不能用作类名以外，别的都可以。
* `var`并不会改变Java是一门静态类型语言的事实。这一特征只发生在编译阶段，对运行时性能不会产生任何影响。编译器在编译阶段自动推断出类型，然后写入字节码文件。反编译出的字节码文件仍是传统带类型的代码。



## 2、集合新增创建不可变集合的方法

Java 9 为集合（`List`,`Set`,`Map`）添加了`of()`方法用于创建不可变集合，Java 10 则添加了`copyOf()`方法，同样用于创建不可变集合，二者的区别如下：

```java
public class Test{
    public void test5(){
        //示例1：
        var list1 = List.of("Java", "Python", "C");
        var copy1 = List.copyOf(list1);  //list1是不可变集合，直接返回list1
        System.out.println(list1 == copy1); // true
        //示例2：
        var list2 = new ArrayList<String>();
        list2.add("aaa");
        //list2是可变集合，底层会调用of方法新建一个只读集合并返回
        var copy2 = List.copyOf(list2); 
        System.out.println(list2 == copy2); // false
        //结果不同。
    }
}
```

上面两种结果不同，是因为`copyOf()`会先判断参数集合是不是`AbstractImmutableList`（不可变）类型的，如果是，则直接返回，否则会调用`of`方法创建一个只读集合并返回。



# 三、Java 11新特性

## 1、概述

Java 11是一个长期支持版本，包括ZGC、Http Client等重要更新，以及17个值得关注的JEP。



## 2、新增字符串处理方法

Java 11新增了一系列字符串处理的方法：

| 方法                     | 描述                 |
| ------------------------ | -------------------- |
| `boolean isBlank()`      | 判断字符串是否为空白 |
| `String strip()`         | 去除首尾空白         |
| `String stripTrailing()` | 去除尾部空格         |
| `String stripLeading()`  | 去除首部空格         |
| `String repeat(int n)`   | 复制字符串n次        |
| `Stream<String> lines()` | 统计行数             |

> `strip`和`trim`的区别是：`trim`无法消除Unicode空白字符`\u2000`，而`strip`可以。

## 3、Optional加强

Optional增加了几个方法，Java 9-11新增的方法：

| 新增方法                                                     | 描述                                                         | 新增的版本 |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :--------- |
| `booleanisEmpty()`                                           | 判断`value`是否为空                                          | JDK 11     |
| `ifPresentOrElse(Consumer<superT> action, Runnable emptyAction)` | `value`非空，执行参数1功能；如果`value`为空，执行参数2功能   | JDK 9      |
| `Optional<T> or (Supplier<?extends Optional<?extendsT>>supplier)` | `value`非空，返回对应的`Optional`；`value`为空，返回形参封装的`Optional` | JDK 9      |
| `Stream<T>stream()`                                          | `value`非空，返回仅包含此`value`的`Stream`；否则，返回一个空的`Stream` | JDK 9      |
| `TorElseThrow()`                                             | `value`非空，返回`value`；否则抛异常`NoSuchElementException` | JDK 10     |



## 4、局部变量类型推断升级

新增了在`var`上添加注解的语法格式：

```java
Consumer<String> con = (@Deprecated var t) -> 
    System.out.println(t.toUpperCase());
```



## 5、全新的HTTP客户端API

Java9开始引入的处理HTTP请求的HTTPClient API，该API支持同步和异步，在Java11中已经为正式可用状态，可以在`java.net`包中找到这个API。

它将替代仅适用于blocking模式的`HttpURLConnection`（`HttpURLConnection`是在HTTP1.0的时代创建的，并使用了协议无关的方法），并提供对WebSocket和HTTP/2的支持。

代码实现：

```java
public class Test{
    //HttpClient替换原有的HttpURLConnection。
    @Test
    public void test4(){
        try {
            HttpClient client = HttpClient.newHttpClient();
            HttpRequest request = HttpRequest.newBuilder(
                URI.create("http://127.0.0.1:8080/test/")).build();
            HttpResponse.BodyHandler<String> responseBodyHandler = 
                HttpResponse.BodyHandlers.ofString();
            HttpResponse<String> response = client.send(
                request, responseBodyHandler);
            String body = response.body();
            System.out.println(body);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void test5(){
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder(
            URI.create("http://127.0.0.1:8080/test/")).build();
        HttpResponse.BodyHandler<String> responseBodyHandler = 
            HttpResponse.BodyHandlers.ofString();
        CompletableFuture<HttpResponse<String>> sendAsync = 
            client.sendAsync(request, responseBodyHandler);
        sendAsync.thenApply(t -> t.body()).
            thenAccept(System.out::println);
        //HttpResponse<String> response = sendAsync.get();
        //String body = response.body();
        //System.out.println(body);
    }
}
```



## 6、更简化的编译运行程序

传统的编译运行过程：

①编译：`javac hello.java`，生成字节码文件`hello.class`

②运行：`java hello`，运行字节码文件

Java 11中简化了编译运行过程，以上两步可以使用`java hello.java`一步代替。

使用简化命令的要求：

* 执行源文件的第一个类，第一个类必须包含主方法。
* 不可以使用其他源文件中的自定义类，可以使用本文件中的自定义类。

## 7、废弃Nashorn引擎

Java 11废弃了Nashorn引擎，可以考虑使用GraalVM

## 8、ZGC

**GC是Java的主要优势之一**。GC停顿太长，会影响应用的响应时间，消除或者减少GC停顿时长，java将对更广泛的应用场景是一个更有吸引力的平台。随着现代系统可用内存不断增长，用户希望JVM能够以高效的方式利用这些内存，并且无需长时间的GC暂停时间。

**ZGC（A ScalableLow-Latency Garbage Collector(Experimental)  ）**，是JDK11最为瞩目的特性。

ZGC是一个并发，基于region，压缩型的垃圾收集器，只有root扫描阶段会STW(stop the world)，因此GC停顿时间不会随着堆的增长和存活对象的增长而变长。

**ZGC优势**：

* GC暂停时间不会超过10ms。
* 既能处理几百兆的小堆,也能处理几个T的大堆(OMG)。
* 和G1相比,应用吞吐能力不会下降超过15%。
* 为未来的GC功能和利用colord指针以及Load barriers优化奠定基础。
* 初始只支持64位系统。
  

ZGC的**设计目标**是：支持TB级内存容量，暂停时间低（<10ms），对整个程序吞吐量的影响小于15%。将来还可以扩展实现机制，以支持不少令人兴奋的功能，例如多层堆（即热对象置于DRAM和冷对象置于NVMe闪存），或压缩堆。

## 9、其他新特性

Java 11的其他新特征：

* Unicode10
* Deprecate the Pack200 Tools and API
* 新的Epsilon垃圾收集器
* 完全支持Linux容器（包括Docker）
* 支持G1上的并行完全垃圾收集
* 最新的HTTPS安全协议TLS1.3
* Java Flight Recorder

# 四、总结展望

## 1、标准化和轻量级的JSON API

一个标准化和轻量级的JSON API被许多Java开发人员所青睐。但是由于资金问题无法在Java当前版本中见到，但并不会削减掉。Java平台首席架构师Mark Reinhold在JDK9邮件列中说：“这个JEP将是平台上的一个有用的补充，但是在计划中，它并不像Oracle资助的其他功能那么重要，可能会重新考虑JDK10或更高版本中实现。”

## 2、新的货币API

对许多应用而言货币价值都是一个关键的特性，但JDK对此却几乎没有任何支持。严格来讲，现有的`java.util.Currency`类只是代表了当前ISO4217货币的一个数据结构，但并没有关联的值或者自定义货币。

JDK对货币的运算及转换也没有内建的支持，更别说有一个能够代表货币值的标准类型了。

如果用Maven的话，可以做如下的添加，即可使用相关的API处理货币：

```java
<dependency>
    <groupId>org.javamoney</groupId>
    <artifactId>moneta</artifactId>
    <version>0.9</version>
</dependency>
```

# 3、展望

* 随着云计算和AI等技术浪潮，当前的计算模式和场景正在发生翻天覆地的变化，不仅对Java的发展速度提出了更高要求，也深刻影响着Java技术的发展方向。传统的大型企业或互联网应用，正在被云端、容器化应用、模块化的微服务甚至是函数(FaaS，Function-as-a-Service)所替代。
* Java虽然标榜面向对象编程，却毫不顾忌的加入面向接口编程思想，又扯出匿名对象之概念，每增加一个新的东西，对Java的根本所在的面向对象思想的一次冲击。反观Python，抓住面向对象的本质，又能在函数编程思想方面游刃有余。**Java对标C/C++，以抛掉内存管理为卖点，却又陷入了JVM优化的噩梦**。选择比努力更重要，选择Java的人更需要对它有更清晰的认识。
* **Java需要在新的计算场景下，改进开发效率**。这话说的有点笼统，我谈一些自己的体会，Java代码虽然进行了一些类型推断等改进，更易用的集合API等，但仍然给开发者留下了过于刻板、形式主义的印象，这是一个长期的改进方向。

# 五、Reference

Java笔记系列，参考自[尚硅谷Java核心教程](https://www.gulixueyuan.com/)，衷心感谢。