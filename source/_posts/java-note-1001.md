---
title: Java学习笔记10-枚举类与注解
excerpt: 自定义枚举类、enum、注解(Annotation)
mathjax: true
date: 2021-04-05 16:50:08
tags: Java
categories: Java
keywords: Java,Java枚举类,注解
---

# 一、枚举类

## 1、什么是枚举类

类的对象只有**有限个**，**确定的**，称此类为**枚举类**。比如星期、性别、季节、xx状态等。

当需要定义一组常量时，强烈建议使用枚举类。

如果枚举类中只有一个对象，则可以作为单例模式的实现方式。

属性：

* 枚举类对象的**属性不应允许被改动**，所以应该使用`private final`修饰。
* 枚举类中使用`private final`修饰的属性应该在构造器中为其赋值。
* 若枚举类显式地定义了带参数的构造器，则在列出枚举值时也应该对应地传入参数。

## 2、自定义枚举类

JDK 5.0之前，没有`enum`关键字，只能自定义枚举类，步骤为：

* 私有化类的构造器，保证不能在类的外部创建其对象。
* 在类的内部创建枚举类的实例，声明为：`public static final`
* 对象如果有实例变量，应该声明为`private final`，并在构造器中初始化。

举例说明：

```java
public class SeasonTest {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        System.out.println(spring);
    }
}
class Season{
    //1.声明对象属性为private final
    private final String seasonName; //季节名字
    private final String seasonDec;  //季节描述
    //2.构造器应该是private的
    private Season(String seasonName,String seasonDec){
        this.seasonName = seasonName;
        this.seasonDec = seasonDec;  //{seasonName='春天', seasonDec='春暖花开'}
    }
    //3.创建确定个数的实例public static final
    public static final Season SPRING = new Season("春天","春暖花开");
    public static final Season SUMMER = new Season("夏天","夏日炎炎");
    public static final Season AUTUMN = new Season("秋天","秋高气爽");
    public static final Season WINTER = new Season("冬天","冰天雪地");

    //4.根据需求，可以选择性地创建其他函数
    public String getSeasonName() { return seasonName;}

    public String getSeasonDec() { return seasonDec;}
    
    @Override
    public String toString() {
        return "{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDec='" + seasonDec + '\'' +
                '}';
    }
}
```



## 3、使用enum定义枚举类

JDK 5.0新增了`enum`定义枚举类的方式，使用`enum`定义的枚举类默认继承了`java.lang.Enum`类，因此不能再继承其他类。

使用说明：

* 枚举类的构造器只能使用`private`权限修饰符。
* 枚举类的所有实例必须在枚举类中显式列出（**以`,`分隔，以`;`结尾**）。列出的实例，系统会自动添加`public static final`修饰。
* **必须在枚举类的第一行声明枚举类对象**。

JDK 5.0中，`switch`表达式中可以使用枚举类对象，`case`子句可以直接使用枚举值的名字，无需添加枚举类作为限定。

使用`enum`定义枚举类：

```java
enum Season{
    //1.提供当前枚举类的对象，多个对象之间用","隔开，末尾对象";"结束
    SPRING("春天","春暖花开"),
    SUMMER("夏天","夏日炎炎"),
    AUTUMN("秋天","秋高气爽"),
    WINTER("冬天","冰天雪地");
    //2.声明Season对象的属性:private final修饰
    private final String seasonName;
    private final String seasonDesc;

    //3.构造器应该是private的，enum里面可以省略private关键字
    Season(String seasonName,String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }
    
    //可以根据需要声明其他方法
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }
}
```



## 4、Enum类

`Enum`类有以下几个常用方法：

* `values()`：静态方法，枚举类调用此方法返回包含此枚举类中所有枚举对象的**枚举类型的对象数组**。这个方法是Java编译器在编译枚举类的时候自动添加的方法。
* `valueOf(String str)`：把一个字符串str转为对应的枚举类对象。要求str‘必须是枚举类对象的“名字”。如不是，会有运行时异常`IllegalArgumentException`。
* `toString`：可以被重写
* `public final boolean equals`：枚举类中可以直接使用`==`比较两个枚举常量是否相等，Enum类中的`equals`方法也是直接使用`==`实现的，此方法是为了在Set、List和Map中使用。
* `public final int hashCode`
* `getDeclaringClass`：得到枚举常量所属枚举类型的Class对象，可以用来判断两个枚举常量是否属于同一个枚举类型
* `name`：得到当前枚举常量的名字。
* `ordinal`：得到当前枚举常量的次序。
* `compareTo`：枚举类型实现了Comparable接口，比较两个枚举常量的大小(按照声明的顺序排列)
* `protected final Object clone`：枚举类型不能被Clone，为了防止子类实现克隆方法，Enum实现了一个仅抛出`CloneNotSupportedException`异常的`final`方法`clone()`。

代码举例：

```java
public class SeasonTest2 {
    public static void main(String[] args) {
        //toString方法：
        System.out.println(Season2=.SPRING); //SPRING
        //父类是Enum类
        System.out.println(Season2.class.getSuperclass()); //class java.lang.Enum

        //values方法
        Season[] values = Season.values();
        for(Season i: values){
            System.out.println(i);
        }
        /*输出：
        SPRING
        SUMMER
        AUTUMN
        WINTER
        */

        //valueOf(String str)方法,返回枚举类中，对象名是str的枚举类对象
        Season winter = Season.valueOf("WINTER");
        System.out.println(winter.toString()); //WINTER
        //等同于Systou.out.println(winter)

        //枚举类在switch中的应用
        Season season = Season.SPRING;
        switch (season){
            case SPRING:
                System.out.println("春天"); //输出：春天
                break;
            case SUMMER:
                System.out.println("夏天");
                break;
            case AUTUMN:
                System.out.println("秋天");
                break;
            case WINTER:
                System.out.println("冬天");
                break;
        }
    }
}
```



## 5、枚举类实现接口

和普通Java类一样，枚举类可以实现一个或多个接口：

* 若每个枚举值在调用实现的接口方法呈现相同的行为方式，则只要统一实现该方法即可。
* 若需要每个枚举值在调用实现的接口方法**呈现出不同的行为方式**,则可以让**每个枚举值分别来实现该方法**。

比如下面的代码：

```java 
public class SeasonTest2 {
    public static void main(String[] args) {
        //values方法
        Season2[] values = Season2.values();
        for(Season2 i: values){
            System.out.println(i);
            i.info();
        }
    }
}

enum Season2 implements Info{
    //每个枚举变量单独实现Info接口中的方法
    SPRING ("春天","春暖花开"){
        public void info(){
            System.out.println("spring");
        }
    },
    SUMMER ("夏天","夏日炎炎"){
        public void info(){
            System.out.println("summer");
        }
    },
    AUTUMN ("秋天","秋高气爽"){
        public void info(){
            System.out.println("autumn");
        }
    },
    WINTER("冬天","冰天雪地"){
        public void info(){
            System.out.println("winter");
        }
    };

    private final String seasonName;
    private final String seasonDec;
    Season2(String seasonName,String seasonDec){
        this.seasonName = seasonName;
        this.seasonDec = seasonDec;
    }

}
interface Info{
    void info();
}
/*
输出结果为：
SPRING
spring
SUMMER
summer
AUTUMN
autumn
WINTER
winter
*/
```



# 二、注解

## 1、概述

JDK 5.0开始支持注解（Annotation），注解是代码里的特殊标记，可以在编译、类加载、运行时被读取，并执行相应的处理。通过使用注解，可以在不改变原有逻辑的情况下，在源文件中嵌入一些补充信息。代码分析工具、开发工具和部署工具可以通过这些补充信息进行验证或者部署。

Annotation可以像修饰符一样被使用,可用于**修饰包,类,构造器,方法,成员变量,参数,局部变量的声明**，这些信息被保存在Annotation的“name=value”对中。

在JavaSE中，注解的使用目的比较简单，例如标记过时的功能，忽略警告等。在JavaEE/Android中注解占据了更重要的角色，例如用来配置应用程序的任何切面，代替JavaEE旧版中所遗留的繁冗代码和XML配置等。

**框架 = 注解 + 反射 + 设计模式**

## 2、常见注解

常见的注解主要用三种用处：

1、生成文档相关的注解。

* @author标明开发该类模块的作者，多个作者之间使用,分割

* @version标明该类模块的版本@see参考转向，也就是相关主题

* @since表示从哪个版本开始增加的

* @param对方法中某参数的说明，如果没有参数就不能写

* @return对方法返回值的说明，如果方法的返回值类型是void就不能写

* @exception对方法可能抛出的异常进行说明，如果方法没有用throws显式抛出的异常就不能写

  其中

  * @param@return和@exception这三个标记都是只用于方法的。
  * @param的格式要求：@param形参名形参类型形参说明
  * @return的格式要求：@return返回值类型返回值说明
  * @exception的格式要求：@exception异常类型异常说明
  * @param和@exception可以并列多个

例如：

```java
/**
*@author xxx
*@version 1.0
*@see Math.java
*/
```



2、编译时进行格式检查（JDK内置的三个基本注解）

* `@Override`：限定重写父类方法，该注解只能用于方法。
* `@Deprecated`：用于表示所修饰的元素(类、方法等)已过时。通常是因为所修饰的结构危险或存在更好的选择
* `@SuppressWarnings`：抑制编译器警告。

3、跟踪代码依赖性，实现替代配置文件的功能。例如Serverlet3.0中注解使得不再需要在web.xml文件中进行Servlet部署：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1001_1.jpg'/>
</div>

## 3、自定义注解

自定义注解使用`@interface`关键字，并且自动继承了`java.lang.annotation.Annotation`接口。

* 注解的成员变量在Annotation定义中以**无参数方法**的形式来声明。其方法名和返回值定义了该成员的名字和类型，称之为配置参数。类型只能是**八种基本数据类型、String类型、Class类型、enum类型、Annotation类型以及以上类型的数组**。

* 定义注解成员变量时如果指定初始值，可以使用`default`关键字。
* 如果只有一个参数成员，建议使用参数名为`value`
* 如果定义的注解含有配置参数，那么**使用时必须指定参数值**，除非它有默认值。格式是`参数名=参数值`，如果只有一个参数成员，且名称为`value`，可以省略`value=`
* 没有成员定义的`Annotation`称为`标记`；包含成员变量的`Annotation`称为`元数据Annotation`。

> 自定义注解必须配上注解的信息处理流程才有意义。

自定义注解举例：

```java
public class Hello {
    //使用是指定配置参数
    @Report(type = 3,level = "level",value = "string")
    public int n;
    
    //指定部分参数
    @Report(level = "level",value = "string")
    public int p;

    @Report(value = "string") // 只对value赋值，等同于@Report("string")
    public int x;

    @Report  //对所有参数使用默认值
    public int y;
}
//以下是自定义的一个注解，如果是public修饰的，必须单独放在一个java文件中
//三个配置参数都有默认值，因此使用的时候可以指定，也可以不指定
public @interface Report {
    int type() default 0;  //类型为int，配置参数名为type，默认值是0
    String level() default "info";
    String value() default "";
}
```



## 4、元注解

用于修饰其他注解的注解，就是`元注解(meta-annotation)`。

JDK 5.0提供了4个标准的元注解：

* `@Retention`：指定被修饰的注解的生命周期。
* `@Target`：指定被修饰的注解能够用于修饰哪些程序元素。
* `@Documented`：表示所修饰的注解在被`javadoc`解析时保留下来。
* `@Inherited`：被修饰的注解将具有继承性。

**1、@Retention**

`@Retentio`元注解用于定义`Annotation`的生命周期，其包含一个`RetentionPolicy`类型的成员变量，使用`@Retention`元注解时，必须为该成员变量指定值，有以下三种值(枚举变量)：

* `RetentionPolicy.SOURCE`：在源文件中有效（即源文件中保留），编译器直接丢弃这种策略的注释
* `RetentionPolicy.CLASS`：在class文件中有效（即class保留），运行Java程序时，JVM不会保留这种注解。**这是默认值**。
* `RetentionPolicy.RUNTIME`：**在运行时有效（即运行时保留），运行Java程序时，JVM会保留注释。程序可以通过反射获取该注释**。

如下图：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1001_2.jpg'/>
</div>

使用举例：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    int type() default 0;
    String level() default "info";
    String value() default "";
}
```



**2、@Target**

`@Target`用于指定被修饰的`Annotation`能用于修饰哪些元素，`@Target`包含一个名为value的数组成员变量，可以取以下的值：

| 取值                         | 含义                                     |
| ---------------------------- | ---------------------------------------- |
| `ElementType.CONSTRUCTOR`    | 用于描述构造器                           |
| `ElementType.PACKAGE`        | 用于描述包                               |
| `ElementType.FIELD`          | 用于描述域(包括枚举常量)                 |
| `ElementType.TYPE`           | 用于描述类、接口(包括注解类型)或enum声明 |
| `ElementType.METHOD`         | 用于描述方法                             |
| `ElementType.LOCAL_VARIABLE` | 用于描述局部变量                         |
| `ElementType.PARAMETER`      | 用于描述参数                             |

**3、@Documented**

`@Documented`用于指定被改元注解修饰的注解类将被`javadoc`工具提取成文档。默认情况下，`javadoc`不包括注解。

定义为`Documented`的注解必须设置`Retention`的值为`RUNTIME`。

**4、@Inherited**

`@Inherited`修饰的注解具有继承性，其子类自动具有该注解，即该注解类的子类可以继承父类级别的注解。

## 5、反射获取注解信息

JDK 5.0在java.lang.reflect包下新增了AnnotatedElement接口,该接口代表**程序中可以接受注解的程序元素**。

当一个Annotation类型被定义为运行时Annotation后,该注解才是运行时可见,当class文件被载入时保存在class文件中的Annotation才会被虚拟机读取。

程序可以调用AnnotatedElement对象的如下方法来访问Annotation信息：

* `getAnnotation(Class<T> annotatoinClass)`
* `getAnnotations()`
* `getDeclaredAnnotations()`
* `isAnnotationPresent(Class<? extends Annotation> annotationClass)`

## 6、JDK 8中注解新特征

JDK 8.0新增了**可重复注解**和**类型注解**。

**1、可重复注解**

* a. 在`MyAnnotation`上声明`@Repeatable`，成员值为`MyAnnotations.class`，此时的`MyAnnotations`为容器注解。
* b. 要求`MyAnnotation`的`@Target`和`@Retention`等元注解与`MyAnnotations`相同。

使用示例：

```java
//注意：这三个类都是public的，需要分别写在三个文件中
@Inherited
@Retention(RetentionPolicy.RUNTIME) 
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
public @interface MyAnnotations {
    MyAnnotation[] value();
}

@Inherited
@Repeatable(MyAnnotations.class) //Repeatable的参数是容器注解的类型
@Retention(RetentionPolicy.RUNTIME)
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE,
         TYPE_PARAMETER,TYPE_USE})
public @interface MyAnnotation {
    String value() default "hello";
}

//此时MyAnnotation注解可以重复使用
@MyAnnotation(value="hi")  
@MyAnnotation(value="abc")
public class Person{}
```



**2、类型注解**

JDK 8.0之后，元注解`@Target`的参数类型`ElementType`枚举值多了两个：`TYPE_PARAMETER`和`TYPE_USE`。在JDK 8.0之前，注解只能是在声明的地方所使用，Java8开始，注解可以应用在任何地方。

* `ElementType.TYPE_PARAMETER`：表示该注解能写在类型变量的声明语句中（如：泛型声明）。
* `ElementType.TYPE_USE`：表示该注解能写在使用类型的任何语句中。

比如，同样使用上面的`@MyAnnotation`，其元注解`@Target`的参数包括了`TYPE_PARAMETER`和`TYPE_USE`，表明该注解可以写在类型变量的声明语句中，也能写在使用类型的任何语句中：

```java
class Test<@MyAnnotation T>{ //写在声明语句中
    public void show() throws @MyAnnotation RuntimeException{
        ArrayList<@MyAnnotation String> list = new ArrayList<>();
        int num = (@MyAnnotation int) 10L; //写在任何语句中
    }
}
```

