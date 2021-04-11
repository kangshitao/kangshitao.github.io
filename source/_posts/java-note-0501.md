---
title: Java学习笔记05-面向对象编程(中)
excerpt: 继承性，多态性，重写，super关键字，Object类，包装类
mathjax: true
date: 2021-03-21 17:00:21
tags: Java
categories: Java
keywords: Java
---

# 一、面向对象特征之二：继承性

多个类中存在相同属性和行为时，将这些内容抽取到单独一个类中，那么多个类无需再定义这些属性和行为，只要**继承(extends)**那个类即可。

此处的多个类称为**子类(派生类)**，单独的这个类称为**父类(基类或超类)**。可以理解为:“子类is a父类”，比如`Student`类继承`Person`类，可以说Student is Person。

## 1、优势

继承有以下作用：

* 减少了代码冗余，提高代码复用性。
* 便于功能的扩展。
* 为多态性的使用提供了前提。

## 2、使用

格式：`class A extends B{}`

* A：子类、派生类、subclass
* B：父类、超类、基类、superclass
* 子类A继承父类B，则子类A中获取了父类B中声明的结构、属性、方法。
* 子类能继承到父类的**私有方法和属性**，只是由于封装性的影响，子类无法显式调用。
* 子类继承父类以后，还可以声明自己特有的属性和方法，实现功能的扩展。
* 子类和父类的关系，是对父类的“扩展”，不等同于子集和集合的关系。

## 3、规定

* 一个类可以被多个子类继承，但一个类只能有一个父类。
* 子父类是一个相对的概念，可以多层继承。A→B→C，其中A是C的间接父类，B是C的直接父类。
* 子类继承父类以后，就获取了直接父类以及所有间接父类声明的属性和方法。
* 如果没有显式地声明父类，则此类继承与`java.lang.Object`类。
* 所有的java类(除`java.lang.Object`类以外），都直接或间接地继承于`java.lang.Object`类，也就是说所有的java类都有`java.lang.Object`类声明的功能。

继承(extends)的使用：

```java
public class ExtendTest {
    public static void main(String[] args) {
        Person p = new Person();
        p.age = 1;
        Student s = new Student();
        s.age = 2; //父类的default权限的属性，子类可以直接调用
    }
class Person {
    String name;
    int age;
    public Person() {}
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public void eat() {
        System.out.println("eat");
    }
    public void sleep() {
        System.out.println("sleep");
    }
}
class Student extends Person {
    String major; //Student类特有的属性
    public Student() {}
    public Student(String name, int age, String major) {
        this.name = name;  //Student继承了Person类，因此可以使用Person类的属性
        this.age = age;  
        this.major = major;
    }
    public void study() { //Student类特有的方法
        System.out.println("study");
    }
    public void showInfo(){
        System.out.println("name:"+name+"age:"+age);
    }
}
```



# 二、方法的重写(override/overwrite)

## 1、定义

**重写(override/overwrite)**：在子类根据需要对从父类中继承来的方法进行改造，也称为方法的重置、覆盖。在程序执行时，子类的方法将覆盖父类的方法。

重写以后，子类对象调用此同名同参方法时，实际执行的是子类重写的方法。

## 2、要求

* 子类重写的方法与父类被重写的方法，方法名和形参列表相同。
* 重写方法的**权限不小于**父类被重写的方法，不能重写父类的`private`方法。
* 子类重写的方法抛出的异常**类型不大于**父类被重写方法的抛出的异常类型。
* 返回值类型：
  * 如果父类方法返回值类型是`void`，则子类重写方法的返回值必须是`void`。
  * 如果父类方法返回值类型是`A类`，则子类重写方法的返回值可以是`A类`或`A类的子类`。
  * 如果父类方法返回值类型是**基本数据类型**，则子类重写方法的返回值类型必须是**相同的基本数据类型**。
* 重写只是对`非static`方法来说的，对于`static`方法不是重写。

## 3、重载和重写的区别

* **重载**：类名、方法名相同，参数列表不同。不表现为多态性。 重载方法的调用地址在编译期就绑定了，称为“早绑定”或“静态绑定”。
* **重写**：类名、方法名相同，参数列表相同。表现为多态性。运行时才确定要调用的方法，称为“晚绑定”或“动态绑定”。

# 三、super关键字

`super`理解为“父类的”，可以用来调用父类的：`属性`、`方法`、`构造器`。

## 1、super调用属性和方法

* 可以在子类的方法或构造器中，通过`super.属性`或`super.方法`的方式，显式地调用父类中声明的属性或方法。但通常情况下省略`super.`。
* 当子类和父类中定义了**同名的属性**，如果想在子类中调用父类中的属性，必须使用`super.属性`的方式，表明调用的是父类中声明的属性。

## 2、super调用构造器

* 可以在子类中使用`super(形参列表)`的方式，调用父类中指定的构造器。
* `super`调用构造器必须声明在子类构造器的首行。
* 子类所有构造器**默认**访问父类中的**空参构造器**，除非显式使用`super(形参列表)`调用父类构造器，或者使用`this(形参列表)`调用本类的构造器，且二者只能存在一种。
* 在类的构造器中，至少有一个构造器使用了`super(形参列表)`，调用父类中的构造器。

## 3、super和this的对比

|            |                           `this`                           |                   `super`                    |
| :--------: | :--------------------------------------------------------: | :------------------------------------------: |
|  访问属性  | 访问**本类**中的属性，如果本类没有此属性则从父类中继续查找 |           直接访问**父类**中的属性           |
|  调用方法  | 访问**本类**中的方法，如果本类没有此方法则从父类中继续查找 |           直接访问**父类**中的方法           |
| 调用构造器 |          调用**本类**构造器，必须放在构造器的首行          | 调用**父类**构造器，必须放在子类构造器的首行 |



# 四、子类对象实例化过程

1. **从结果上看**：子类继承父类以后，就获取了父类中声明的属性或方法。创建子类对象，在**堆**空间中，就会加载**所有父类中声明的属性**。

2. **从过程上看**：当通过子类的构造器创建子类对象时，一定会直接或间接地调用其父类的构造器，进而向上调用父类的父类的构造器......，直到调用了`java.lang.Object`类中空参的构造器为止。因为加载过所有的父类的结构，所以才可以看到内存中有父类中的结构，子类对象才可以考虑进行调用。

   > 虽然创建子类对象时，调用了父类的构造器，但只是创建过一个对象，即`new`出来的对象。

# 五、面向对象特征之三：多态性

## 1、多态的使用

Java中的**多态性(Polymorphism)**，是面向对象中最重要的概念。

对象的多态性：**父类的引用指向子类的对象**，比如`Person p = new Student();`

多态的使用(**虚拟方法调用**)：对于多态，编译时只能调用父类中声明的方法，但在运行时，实际执行的是子类重写的方法，即**编译看左边，运行看右边**。

编译时类型和运行时类型不一致，就出现了对象的多态性。

* 对象的多态性，只适用于方法，不适用于属性（编译和运行都看左边）。
* 多态性是**运行时**的行为（动态绑定），即只有在运行时才知道具体执行哪个方法，在编译时是不知道的。
* 子类可以看作是特殊的父类，所以父类类型的引用可以指向子类的对象，即**向上转型(upcasting)**。
* 当一个对象声明为父类的类型，实际引用的是子类对象，那么该变量不能访问子类中的属性和方法(实际在内存中是加载的，只是不能调用）。

多态性的例子：

```java
public class PersonTest {
    public static void main(String[] args){
        //对象的多态性：父类的引用指向子类的对象。
        Person pm = new Man();
        Person pw = new Woman();
        //多态的引用：当调用子父类同名同参数的方法时，实际执行的是子类重写的父类的方法——虚拟方法调用
        pm.eat();  //Man:eat
        pw.eat();  //Woman：eat
        
        //多态的另一种使用方法，传入不同子类对象时，调用其重写的方法，提高了代码复用性
        PersonTest test = new PersonTest();
        test.walkPrint(new Man());  //Man：walk
        test.walkPrint(new Woman());  //Woman：walk
        
    }
    public void walkPrint(Person person){
        person.walk();
    }
}

class Person {
    private String name;
    private int age;
    public Person() {}
    public void eat(){System.out.println("Person:eat"); }
    public void walk(){System.out.println("Person:walk");}
    
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
    public int getAge() {return age;}
    public void setAge(int age) {this.age = age;}
}

class Man extends Person{
    private boolean isSmoking;
    public void earnMoney(){System.out.println("Man:earnMoney");}
    public void eat(){ System.out.println("Man:eat");}
    public void walk(){ System.out.println("Man:walk");}
}

class Woman extends Person{
    private boolean isBeauty;
    public void goShopping(){System.out.println("Woman:goShopping");}
    public void eat(){System.out.println("Woman:eat");}
    public void walk(){System.out.println("Woman:walk");}
}
```

## 2、instanceof操作符

`x instanceof A`用于检验x是否是类A的对象，返回`boolean`值。

要求`x`所属的类与`A类`必须是子类和父类的关系，否则编译错误。

如果`x`属于`类B`，`B extends A`，则`x instanceof B`和`x instanceof A`都为`True`。

```java
//对于Person类、Man类，Woman类如下关系：
//Man extends Person，Woman extends Person
Man man = new Man();
Person person = new Person();
Person pm = new Man();
man instanceof Man; //true
man instanceof Person; //true
man instanceof Woman;  //编译错误，Man类和Woman类没有关系，不是子类和父类的关系
person instanceof Person;  //true
person instanceof Man;  //false
pm instanceof Person;  //true
pm instanceof Man;  //true
pm instanceof Woman;  //false
```

## 3、对象类型转换(Casting)

类似于基本数据类型的类型转换，对象也有类型转换。

* **基本数据类型**：
  * 自动类型转换(提升)：小的数据类型自动转换为大的数据类型，比如`int`型自动转换为`double`型：`double d = 12;`。
  * 强制类型转换：大的数据类型强制转换(casting)为小的数据类型，比如`double`强转为`int`型：`int a = (int)12.0;`。
* **对象类型**：
  * 向上转型：子类对象赋给父类对象的引用，即多态性，比如`Person person = new Man();`
  * 向下转型：父类对象强制转换为子类对象，比如：`Person person = new Man();Man man=(Man)person;`，将父类对象`person`强制转换为子类类型对象`man`，向下转型后，`man`对象就可以使用`Man类`定义的属性和方法。

> 自动类型提升和向上转型都可以自动进行。
>
> 基本数据类型的强制类型转换可能会带来精度损失。
>
> 无继承关系的引用类型间的转换是非法的，编译错误。
>
> 对象类型向下转型时，可以使用`instanceof`关键字进行判断，结果是`true`时才可以进行转换。

# 六、Object类

* `Object`类是所有Java类的根父类。
* 如果类的声明中没有使用`extends`关键字指明父类，则默认父类为`java.lang.Object`。
* `Object`类中的功能(属性、方法)具有通用性，`Object`类没有属性，但有常用的方法，比如`equals()`、`toString()`、`getClass()`、`hashCode()`、`clone()`、`finalize()`、`wait()`、`notify()`、`notifyAll()`。
* `Object`类只声明了一个空参的构造器。



# 七、包装类

对于Java中的8种基本数据类型，Java提供了各自对应的包装类，使基本数据类型的变量具有了类的特征。

| 基本数据类型 | 包装类      |
| ------------ | ----------- |
| `byte`       | `Byte`      |
| `short`      | `Short`     |
| `int`        | `Integer`   |
| `long`       | `Long`      |
| `float`      | `Float`     |
| `double`     | `Double`    |
| `boolean`    | `Boolean`   |
| `char`       | `Character` |



基本数据类型、包装类、String三者之间的相互转换。

* 基本数据类型和包装类之间的相互转换

  * 基本数据类型→包装类，调用包装类的构造器或`valueOf()`方法：

    * `Integer i = new Integer(12);`
    * `Integer i = Integer.valueOf(12);`

  * 包装类→基本数据类型，调用包装类的`xxxValue()`方法：

    * `int a = i.intValue();`
    
    > JDK 5.0 以后，有了**自动装箱(基本数据类型转换为包装类)**和**自动拆箱**功能，包装类和基本数据类型可以直接相互转换，比如`Integer i = 12;`，因此，包装类和基本数据类型可以看出一个整体。

* 基本数据类型、包装类和String类型之间的相互转换

  * 基本数据类型、包装类→String类型，可以使用`+`连接符，或者String类的`valueOf()`方法。
    * 方法1：`String s  =  12 + "";`
    * 方法2：`String s  = String.valueOf(12);`
  * String类型→基本数据类型、包装类，使用包装类的构造器或者parseXxx()方法：
    * 例1：`int i = new Integer("12");`
    * 例2：`int i = Integer.parseInt("12");`

特殊说明，`Integer`内部定义了`IntegerCache`结构，`IntegerCache`定义了`Integer[]`，保存了`-128~127`范围的整数，使用自动装箱的时候可以直接使用数组的元素，如果不在此范围内，则会`new`一个对象。比如下面的例子：

```java
Integer m = 1;
Integer n = 1;
System.out.println(m == n); //true
Integer x = 128;
Integer y = 128;
//数字128超出了IntegerCache数组的范围，此时x和y是两个不同的对象
System.out.println(x == y); //false
```



# 八、== 和equals()的区别

* `==`是运算符，可以使用在基本数据类型和引用数据类型中：
  * 对于基本数据类型，`==`操作符比较两个变量保存的数据是否相等。
  * 对于引用数据类型，`==`比较的是两个对象的地址值是否相同，即两个引用是否指向同一个对象实体。
  * `==`操作符两边变量的类型不一定相同，但必须要兼容，能够比较，比如`10 == 10.0`结果为`true`，`int`和`char`类型也能够比较，`97 == 'a'`结果为`true`。
* `equals()`是方法，只能用于引用数据类型。
  * `Object`类中的equals()方法使用的是==运算符，因此比较的是对象的地址。
  * 像`String`、`Date`、`File`、`包装类`等类，重写了`Object`类中的`equals()`方法。重写以后，比较的是两个对象的“**实体内容(各种属性)**”是否相同。
  * 通常，自定义的类如果使用`equals()`方法，也是比较对象的内容是否相同，需要对`equals()`重写。重写`equals()`方法也可以使用自动生成的方法。

> 判断某个类中的`equals()`方法的功能，需要看其是否对`equals()`进行了重写，以及怎样重写的。