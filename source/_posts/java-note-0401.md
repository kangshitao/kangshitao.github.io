---
title: Java学习笔记04-面向对象编程(上)
excerpt: 面向对象思想，对象，属性，方法，封装，构造器，this，package和import
mathjax: true
date: 2021-03-20 21:50:55
tags: Java
categories: Java 
keywords: Java
---

# 一、面向过程和面向对象

* 面向过程(Procedure Oriented Programming)：强调功能行为，以函数为最小单位，考虑怎么做。

* **面向对象(Object Oriented Programming，OOP)**：将功能封装进对象，强调具备了功能的对象，以类/对象为最小单位，考虑谁来做。

面向对象三大特征：**封装(Encapsulation)**、**继承(Inheritance)**、**多态(Polymorphism)**

# 二、Java基本元素：类和对象

* 类是对一类事物的描述，是**抽象**的、概念上的定义
* 对象是**实际存在**的该类事物的每个个体，也称为**实例(instance)**

设计类，就是设计类的成员：

* `属性` = `成员变量` = `filed` = `域`、`字段`
* `方法` = `成员方法` = `函数` = `method`

创建类的对象 = 类的实例化 = 实例化类

创建类：

```java
修饰符 class 类名{
    属性声明;
    方法声明;
}
//创建Person类
public class Person{
    int age; //成员变量
    String name; //成员变量
    public void eat(){ //方法
        System.out.pringln("eat");
    }
}
```



# 三、对象的创建和使用

通过 `类名 对象名= new 类名();`创建对象，通过`对象名.对象成员`访问对象的属性和方法。

```java
Person person = new Person();
int age = person.age;  //访问类的属性
person.eat();  //访问类的方法
```

类的访问机制：

* 同一个类中，类中的方法可以直接访问类中的成员变量（非static）。
* 不同类中，需要先创建类的对象，通过对象访问类中的成员。

# 四、类的成员之一：属性

声明格式：`修饰符 数据类型 属性名 = 初始化值;`

* 成员变量：方法体外，类内声明的变量。有默认初始值。
* 局部变量：方法体内部声明的变量。没有默认初始值，需要显式初始化(形参除外)。

|          | 成员变量       | 局部变量                               |
| -------- | -------------- | -------------------------------------- |
| 声明位置 | 直接声明在类中 | 方法形参、方法内部、代码块内、构造器内 |
| 修饰符 | private、public、static、final等 | 不能用权限修饰符，可以用final修饰 |
| 初始化值 | 有默认初始化值(同数组元素初始化) |无默认初始化值，必须显式赋值才能使用|
|内存加载位置|堆空间或静态域内|栈空间|

例如：

```java
class Person(){
    public int age; //成员变量
    public String name;
    public void eat(){
        int hour = 3; //局部变量
    }
}
```



# 五、类的成员之二：方法

## 1、方法定义

`方法(method)`用来完成某个功能，实现代码复用，简化代码。

Java里的`方法`不能独立存在，必须定义在类内。

声明格式：

```java
修饰符 返回值类型 方法名(参数类型 参数1,参数类型 参数2,...){
    方法体;
    return 返回值;  
    //返回值类型为void时，可以没有return语句，或使用return;
}
public void showInfo(){
    System.out.println("showInfo");
    //return;
}
public int getAge(){
    return age;
}
```

> 方法中只能调用方法或属性，不能在方法内定义方法。

## 2、方法重载

`重载(overload)`：同一个类中，功能类似，`方法名相同`的多个方法。`参数列表不同`（类型，个数，顺序）。返回值类型可以相同也可以不同，但一般返回值类型是相同的。

**两同一不同**：类、方法名相同，参数列表不同。

是否重载和权限修饰符、返回值类型、形参变量名、方法体都没有关系。

举例：

```java
public class OverLoad {
    public static void main(String[] args) {
        OverLoad test = new OverLoad();
    }
    
    public void getSum(int a, int b){} 

    public void getSum(double a, double b){} //是重载

    public void getSum(String s, int i){} //是重载

    public void getSum(int i, String s){} //是重载，和上一个也构成重载，顺序不同。

    public int getSum(int a, int b, int c){ //是重载，参数个数不同
        return 0;
    }
    public int getSum(int a, int b){//不是重载，参数列表相同
        return 0;
    }
}
```

## 3、可变个数形参

JavaSE 5.0提供了Varargs(variable number of arguments)机制，允许直接定义能和多个实参匹配的形参。

JDK5.0之前，使用数组形参定义方法，传入多个**同一类型**变量:

`public static void test(int a, String[] books);`

JDK5.0之后，使用可变形参，传入多个**同一类型**变量：

`public static void test(int a, String...books);`

* 可变个数形参的方法，和同名的方法构成重载。
* 可变个数形参方法的使用，和使用数组时一样，使用索引获取某个参数。
* 可变形参需要放到参数列表的最后。
* 一个方法的参数中只能有一个可变个数形参。
* 如果有参数列表恰好符合的方法，优先调用。

```java
public class MethodTest {
    public static void main(String[] args) {
        MethodTest m = new MethodTest();
        m.show(1,2); //method 3，优先调用两个参数的方法。
        m.show(new int[]{1,2,3,4}); //调用可变形参的方法
        m.show("sss",1,3,4,5); //method 4，两种参数方法都可以。
    }
    public void show(int i){
        System.out.println("method 1");
    }

    public void show(int ... i){  //可以接收任意个数的参数。也可以接受数组类型的参数。
        System.out.println("可变形参");
    }
    
    public void show(int a, int b){
        System.out.println("method 3");
    }

    public void show(String s, int ... i){  //可变形参必须放在最后。
        for(int n = 0; n<i.length; n++){ //将i当成数组处理即可。
        }
        System.out.println("method 4");
    }
}
```

## 4、值传递

**形参**：方法声明时的参数。

**实参**：方法调用时实际传给形参的参数值。

Java里的方法参数传递方式只有一种：**值传递**。即将实际参数值的副本(复制品)传入方法内，而参数本身不受影响。

* 形参是**基本数据类型**：将实参基本数据类型变量的**“数据值”**传递给形参。
* 形参是**引用数据类型**：将实参引用数据类型变量的**“地址值”**传递给形参。

值传递练习：

```java
public class valueTransfer {
    public static void main(String args[]) {
        valueTransfer t = new valueTransfer();
        t.first();
    }

    public void first() {
        int i = 5;
        Value v = new Value();
        v.i = 25;
        second(v, i);  //经过second函数，v中的i被修改为20
        System.out.println(v.i);
    }

    public void second(Value v, int i) { 
        //参数的v和i是在栈空间中新建的，这里的v指向传入对象v的地址
        i = 0;
        v.i = 20; // 将指向的地址中的i赋值为20
        Value val = new Value();
        v = val;  //v指向对象val的地址。
        System.out.println(v.i + " " + i);  //15 0
    }
}

class Value {
    int i = 15;
}
//以上程序输出结果：
15 0
20
```



# 六、类的成员之三：构造器(构造方法)

类的**构造器(构造方法，constructor)**用于给对象进行初始化。

**构造器特征**：

* 具有与类相同的名称
* 不声明返回值类型
* 不能被`static`、`final`、`synchronized`、`abstract`、`native`修饰，不能有`return`语句返回值

**声明格式**：

```java
权限修饰符 类名(参数列表){
    初始化语句;
} 
public class Animal{
    private int legs;
    private String name;
    public Animal(int l){ //构造器1
        legs = l;
    }
    public Animal(int l, String n){ //构造器2
        legs = l;
        name = n;
    }
}
```

注意：

* Java中，每个类至少有一个构造器。
* 如果没有显式地定义类的构造器，系统默认提供一个空参的构造器。
* 一旦显式地定义了构造器，系统不再提供默认的构造器。
* 一个类中可以定义多个构造器，构成重载。

> 在IDEA中，可以右键→generate→Constructor，自动生成构造器

 

**属性根据以下顺序赋值**

① 默认初始化。

② 显式初始化，如`int age = 1;`

③ 构造器中初始化。

④ 通过`对象.方法` 或 `对象.属性`的方式赋值。

# 七、面向对象特征之一：封装与隐藏

## 1、封装性

类的**封装性**是指将对象内部的复杂性隐藏起来，只对外提供必要的接口，便于调用。

如果直接将属性暴露出来，使用者对属性的直接操作可能导致数据错误、混乱或安全性问题。

Java中通过将数据声明为`私有的(private)`，再提供`公共的(public)`方法`getXxx()`和`setXxx()`实现对该属性的操作，以实现下述目的：

* 隐藏一个类中不需要对外提供的实现细节；
* 使用者只能通过事先定制好的方法来访问数据，可以方便地加入控制逻辑，限制对属性的不合理操作；
* 便于修改，增强代码的可维护性；

例如：

```java
public class PersonTest {
    public static void main(String[] args) {
        Person p = new Person(20); // new 后面的Person()就是构造器。
        p.setAge(25); //将年龄设置为25
        int age = p.getAge();  // age = 25
        //int age = p.age; 无法直接通过对象调用age属性，因为类中的age是私有的
    }
}
class Person{
    private int age;

    public Person(){ //构造器（构造函数）
        System.out.println("constructor");
    }
    public Person(int a){ //重载的构造器
        age = a;
        System.out.println("constructor");
    }
    public int getAge() {
        return age;
    }
    public void setAge(int a) {
        age = a;
    }
    public void eat(){}
}
```

同样地，`getXxx()`和`setXxx()`方法可以使用自动生成的方法。

## 2、四种权限修饰符

权限修饰符可以用来修饰类和方法，其中`class`只能用`public`和`缺省(default)`两种权限修饰符，四种权限修饰符的权限如下：

| 权限修饰符    | 类内部 | 同一个包 | 不同包的子类 | 同一个工程 |
| ------------- | ------ | -------- | ------------ | ---------- |
| private       | √      |          |              |            |
| 缺省(default) | √      | √        |              |            |
| protected     | √      | √        | √            |            |
| public        | √      | √        | √            | √          |

## 3、JavaBean

JavaBean是一种Java语言写成的可重用组件。

所谓JavaBean，是指符合如下标准的Java类：

* 类是`public`的。
* 有一个无参的`public`构造器。
* 有属性，且有对应的`get`、`set`方法。

# 八、this关键字

`this`可以用来修饰`属性`、`方法`、`构造器`。

## 1、修饰属性和方法

`this`修饰属性和方法时，表示当前对象或当前正在构建的对象。使用`this.属性`和`this.方法`的方式，调用当前对象的属性和方法。

通常情况下，省略`this`，特殊情况下，当参数和属性同名时，必须使用`this`进行区分。

## 2、修饰构造器

`this`修饰构造器：

* 在类的构造器中，可以用`this(形参列表)`方式，调用**本类中其他构造器**。
* 构造器不能用`this(形参列表)`方式调用自己。
* 如果一个类中有n个构造器，则最多有n-1个构造器中使用了`this(形参列表)`方式。
* `this(形参列表)`必须声明在当前构造器的**首行**。
* 每个构造器内部，最多只能有一个`this(形参列表)`的方式。且多个构造器之间调用不能形成环，否则会导致死循环。

`this`关键字的用法：

```java
public class Person{  //JavaBean的形式构造Person类
    private String name;
    private int age;
    
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;  //使用this区分类的属性name和形参name
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Person(){} //构造器（构造函数）
    
    public Person(String name){
        this.name = name;
        System.out.println("second constructor");
    }
    public Person(String name, int age){
        this(name); //使用this调用第二个构造器
        this.age = age;
        System.out.println("third constructor");
    }
    public void eat(){}
}

```

# 九、package和import

## 1、package

`package`语句是Java源文件的第一条语句，指明该文件中定义的类所在的包。

比如`pack1\pack2\PackageTest.java`路径下的类`PackageTest`:

```java
package pack1.pack2;
public class PackageTest{
    public static void main(String[] args){
        System.out.println("PackageTest");
    }
}
```

`JDK`中常用的包：

* `java.lang`：包含一些Java语言的核心类，如`String`、`Math`、`Integer`、`System`和`Thread`，提供常用功能.
* `java.net`：包含执行与网络相关的操作的类和接口。
* `java.io`：包含能提供多种输入/输出功能的类。
* `java.util`：包含一些实用工具类，如定义系统特性、接口的集合框架类、使用与日期日历相关的函数。
* `java.text`：包含了一些java格式化相关的类
* `java.sql`：包含了java进行JDBC数据库编程的相关类/接口
* `java.awt`：包含了构成抽象窗口工具集（abstract window toolkits）的多个类，这些类被用来构建和管理应用程序的图形用户界面(GUI)。B/S，C/S

## 2、import

`import`语句用于引入指定包层次下所需要的类。

格式：`import 包名.类名;`

* `import`语句声明在`package`声明和类的声明之间；
* `*`表示导入包下的所有类，比如`import java.util.*;`表示导入`util`包下的所有类或接口。
* 如果是`java.lang`包下，或是当前包下的，可以不使用`import`语句。
* 如果使用**不同包**下的**同名的类**。那么就需要使用**类的全类名（包名.类名）**的方式指明调用的是哪个类。
* 如果已经导入`java.a`包下的类。那么如果需要使用`a`包的子包下的类的话，仍然需要导入。

## 3、MVC设计模式

MVC是常用的设计模式之一，将整个程序分为三个层次：**数据模型层(Model)**、**视图层(View)**和**控制器层(Controller)**。各个层的主要内容如下：

* **模型层(Model)**，主要处理数据：
  * 数据对象封装 ：`model.bean/domain`
  * 数据库操作类：`model.dao`
  * 数据库：`model.db`
* **视图层(View)**，显示数据：
  * 相关工具类：`view.utils`
  * 自定义view：`view.ui`
* **控制层(Controller)**：
  * 应用界面相关：`controller.activity`
  * 存放fragment：`controller.fragment`
  * 显示列表的适配器：`controller.adapte`
  * 服务相关的：`controller.service`
  * 抽取的基类：`controller.base`

MVC模式在项目中的应用：[客户信息管理软件](https://github.com/kangshitao/Java_shangguigu/tree/main/Project2)