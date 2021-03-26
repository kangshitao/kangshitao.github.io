---
title: Java学习笔记06-面向对象编程(下)
excerpt: static，fianl，单例模式，抽象类，接口，代码块，内部类
mathjax: true
date: 2021-03-26 11:49:40
tags: Java
categories: Java
keywords: Java
---

# 一、关键字：static

## 1、介绍

`static`：静态的，可以用来修饰属性、方法、代码块、内部类。

被`static`修饰的成员，有以下特点：

* 随着类的加载而加载。
* 优先于对象存在。
* 修饰的成员，被所有对象共享。
* 访问权限允许时，可不创建对象，直接被类调用。

> 有关`static`修饰的成员特点，都可以从生命周期的角度来解释。

## 2、修饰属性

`static`修饰的属性称为**静态变量(类变量，类属性)**，静态变量**随着类的加载而加载**，可以通过`类名.静态变量`的方法调用。静态变量的加载早于对象的创建，且在**内存中只会存在一份**，保存在**方法区**的**静态域**中。常见的静态变量：`System.out`、`Math.PI`等。

**静态变量(类变量)** vs **非静态变量(实例变量)**：

* 静态变量：可以通过`类.静态变量`和`对象.静态变量`两种方法调用。多个对象共享一个静态变量，通过一个对象修改此属性，其他对象调用时是被修改后的值。
* 实例变量：只能通过`对象.实例变量`的方式调用。每个对象都独立地拥有一套类中的实例变量，实例对象归具体的某个对象所有，修改某个对象的实例变量，不会影响其他对象同样的变量。

## 3、修饰方法

**静态方法**vs**非静态方法**：

* 静态方法：可以通过`类.方法`和`对象.方法`两种方法调用。静态方法只能调用静态方法和属性，可以通过创建对象的方法调用非静态方法和属性。静态方法不能使用`this`、`super`关键字。
* 非静态方法：既可以调用静态方法，也可以调用非静态方法。

> 静态方法不可以被重写。

## 4、使用场景

1. 判断属性是否要声明为静态的：
   * 属性可以被多个对象共享，不会随着对象的不同而不同，比如银行利率。
   * 类中的常量一般声明为`static final`的。
2. 判断方法是否要声明为静态的：
   * 操作静态属性的方法一般声明为静态的。
   * 工具类中的方法习惯上声明为静态的，比如`Math`、`Arrays`、`Collections`

## 5、单例模式

**单例模式**，就是采取一定的方法保证在整个的软件系统中，对某个类只能存在一个对象实例，并且该类只提供一个取得其对象实例的方法。

设计单例模式根据以下思路：

1. 想让类只能存在一个对象实例，就要将构造器权限设置为`private`，在类内部创建此对象。这样外部不能用`new`操作符创建新的对象。
2. 外部想要得到这个类的对象，就需要调用`public`的`static`方法得到此对象。
3. 获取对象的方法是`static`的，则指向类内部产生的该类对象的变量也必须是`static`的。

根据以上思路，有两种单例模式写法，`饿汉式`和`懒汉式`：

```java
//饿汉式单例模式
class Bank{
    //1.私有化类的构造器
    private Bank{}
    
    //2.创建类的私有对象属性
    //4.方法是静态的，则此对象属性必须是静态的
    private static Bank instance = new Bank();
    
    //3.提供公共的静态方法，返回类的对象
    public static Bank getInstance(){
        return instance;
    }
}

//懒汉式单例模式
class Bank{
    //1.私有化类的构造器
    private Bank{}
    
    //2.创建类的私有对象属性,先不初始化
    //4.方法是静态的，则此对象属性必须是静态的
    private static Bank instance;
    
    //3.提供公共的静态方法，返回类的对象
    public static Bank getInstance(){
        if (instance == null){
            instance = new Bank();
        }
        return instance;
    }
}
```

饿汉式单例模式vs懒汉式单例模式：

* 饿汉式：无论有没有使用，都创建对象实例，对象加载时间较长；但饿汉式单例模式线程安全的。
* 懒汉式：延迟对象的创建；以上写法是线程不安全的，可以修改为线程安全的。

单例模式的应用：

* 网站的计数器
* 应用程序的日志应用
* 数据库连接池
* 读取配置文件的类
* Application
* Windows任务管理器
* Windows回收站

# 二、理解main方法

`main()`方法是程序的入口，其形式为：`public static void main(String[] args){}`，从形式上可知：

* `main()`方法也是普通的静态方法，可以通过`类名.main()`的方式调用。`main()`方法调用其他非静态方法需要创建对象，然后使用`对象.非静态方法`的方式调用。
* `main()`有参数，可以作为与控制台交互的方式。

# 三、类的成员之四：代码块

1. 代码块没有名字，自动执行。

2. 作用：代码块用于对Java类或对象进行初始化。
3. 一个类中可以定义多个代码块，按照先后顺序执行。
4. 代码块如果有修饰符，只能使用`static`修饰。
5. **静态代码块**：
   * 内部可以有输出语句。
   * **随着类的加载而执行**，只会执行一次。
   * 作用：用来初始化类的信息，比如对类变量赋值。
6. **非静态代码块**：
   * 内部可以有输出语句。
   * 随着对象的创建而执行，每创建一次对象就执行一次。
   * **非静态代码块先于构造器执行**。
   * 作用：可以在创建对象时，对对象的属性进行初始化。
7. 静态代码块和静态方法类似，只能直接调用静态方法和属性。
8. 代码块应用举例：

```java
public class BlockTest {
    public static void main(String[] args) {
        Student student = new Student();
    }
    static { //先加载BlockTest类，再加载Person类
        System.out.println("BlockTest block");
    }
}

class Person{
    private String name;
    public Person(){}
    //非静态代码块
    {System.out.println("Person block");}
    
    //静态代码块
    static{System.out.println("Person static block");}
}

class Student extends Person{
    private int number;
    public Student(){
        System.out.println("Student constructor");
    }
    
    {System.out.println("Student block");}
    
    static {System.out.println("Student static block");}
}
/* 
以上程序运行结果：
BlockTest block         //先加载主类
Person static block	//声明子类对象变量时，先加载父类，执行父类的静态代码块
Student static block	//然后加载子类，执行子类的静态代码块
Person block		//执行new语句通过构造器创建对象时，先执行父类非静态代码块
Student block		//然后执行子类代码块
Student constructor     //最后执行构造器
/*
```

以上结论可以总结为：**由父及子，静态先行**。



**属性赋值的方式以及顺序**：

1. 默认初始化
2. 显式初始化；多个代码块赋值(这两种方式根据代码顺序执行)
3. 构造器初始化
4. 创建对象以后，通过对象调用属性或方法进行赋值



# 四、关键字：final

`final`：最后的，可以用来修饰`类`、`方法`、`变量`。

* `final`修饰`类`：表示此`类`不能被其他类继承，比如`String`类、`System`类、`StringBuffer`类等
* `final`修饰`方法`：表明此`方法`不能被重写，比如`Object`类的`getClass()`方法
* `final`修饰`变量`：包括`属性`和`局部变量`，表示变量的值不允许被修改，此时的变量称为`常量`。
  * `final`修饰`属性`，可以直接赋值，也可以先声明，然后在`代码块`中、`构造器`中再初始化赋值。
  * `构造器`中初始化常量，可以通过`形参`为每个对象设置不同的常量值，比如身份证号。
  * `final`修饰`局部变量`时，对于方法内部的`局部变量`，使用`final`修饰变为`常量`；对于`final`修饰的`形参`，调用方法的时候赋值，然后不允许再修改。

> `static final`用来修饰的`属性`，称为`全局常量`。

# 五、抽象类与抽象方法

`abstract`：抽象的，可以用来修饰`类`、`方法`。

## 1、抽象类

`abstract`修饰的类称为抽象类。

* 抽象类**不能实例化对象**，可以使用多态。
* 抽象类中**一定有构造器**，便于子类实例化时调用。
* 实际开发中会提供抽象类的子类，让子类对象实例化，完成相关操作。

## 2、抽象方法

`abstract`修饰的方法称为`抽象方法`。

* 抽象方法只有方法的声明，没有方法体（没有`{}`），例：`public void showInfo();`
* 包含抽象方法的类一定是抽象类，反之，抽象类可以没有抽象方法。
* 只有子类**实现**（类似于重写）了父类中的**所有抽象方法**(包括**直接父类**和**间接父类**的所有抽象方法)，此子类才可以实例化，否则该子类也是抽象类。

> 1.`abstract`不能修饰变量、构造器、代码块。
>
> 2.因为抽象方法必须要被实现(重写)，所以`abstract`不能用来修饰`private`方法、`static`方法、`final`的方法。同样，`abstract`也不能修饰`final`的类。

## 3、匿名子类

定义抽象类`Person`的匿名子类：

```java
Person p = new Person(){
    @Override
    public abstract void eat(){ 
        System.out.println("eat");
    }
}; //匿名子类需要重写父类的抽象方法

abstract class Person{
    public abstract void eat();
}
```



# 六、接口(interface)

`interface`：接口。接口定义的是一组规则，是和类并列的结构。继承是一种”是不是“的关系，而接口是”能不能“的关系。定义接口：`interface Flyable{}`

* ` JDK 7`及以前，接口中只能定义全局常量和抽象方法。
  * 全局常量：默认权限是`public static final`，关键字可以省略不写。全局常量可以通过`接口.常量`调用。
  * 抽象方法：默认权限是`public abstract`，关键字同样可以不写
* `JDK 8`及以后，接口中除了能够定义全局常量和抽象方法以外，还能够定义静态方法，默认(`default`)方法。
  * 接口中定义的静态方法，只能使用接口调用，即`接口.静态方法`。
  * 实现类如果调用接口的默认方法，使用`接口.super.默认方法`的方式调用。
  * 通过实现类的对象，可以调用接口的默认方法。默认方法可以被重写，但不是必须的，但是接口中的抽象方法必须被重写。
  * 如果子类继承的父类和实现的接口中声明了同名同参的方法，如果子类没有重写，默认调用的是父类中的方法。(类优先原则)
  * 如果实现类同时实现了多个接口，这多个接口中定义了同名同参的默认方法，实现类必须重写此方法，否则报错。(接口冲突)

* 接口中**不能定义构造器**，即接口不能够实例化。
* 接口通过被类`实现(implements)`来使用。实现类必须实现接口的所有抽象方法，此类才能被实例化，否则此类必须定义为抽象类。
* Java类可以同时实现多个接口，弥补了Java无法多继承的缺陷。`implements`关键字在`extends`后面，比如父类B的子类A，实现了C、D两个接口：`class A extends B implements C,D{}`
* 接口和接口之间可以继承，而且可以多继承。接口的具体使用体现了多态性。
* 接口的匿名实现类，格式和抽象类的匿名子类相同。

# 七、类的成员之五：内部类

Java中允许将一个`类A`声明在另一个`类B`中，则`类A`是`内部类`，`类B`是`外部类`。`内部类`和`外部类`的类名不能相同。

根据声明的位置不同，内部类又分为`成员内部类`和`局部内部类`(方法内、代码块内、构造器内)

## 1、成员内部类

成员内部类直接声明在类的内部，有两种身份：**作为类的成员**，**作为一个类**。

作为类的成员，具有类的成员的特征：

* 可以调用外部类的结构
* 可以被`static`修饰
* 可以被四种不同的权限修饰符修饰

作为一个类，具有类的功能：

* 成员内部类内可以定义属性、方法、构造器等，和一般的类定义相同
* 可以被`final`修饰，表示不可以被继承。不加`final`则表示可以被继承
* 可以被`abstract`修饰

> 1.非static的成员内部类中的成员不能声明为static的，只有外部类，或者static的成员内部类中才可以声明static成员。
>
> 2.外部类访问成员内部类的成员，通过`内部类.成员`或`内部类对象.成员`的方式。
>
> 3.成员内部类可以直接使用外部类的所有成员，包括私有的数据。

## 2、局部内部类

**局部内部类**是声明在方法内、代码块内、构造器内的类。

局部内部类只能在声明它的方法或代码块中使用，但是局部内部类的对象可以通过外部方法的返回值返回使用，返回值类型只能是局部内部类的父类或父接口类型。

* 局部内部类可以使用外部类的成员，包括私有的。
* 局部内部类可以使用外部方法的局部变量，但必须是`final`的。JDK 8之后可以省略`final`关键字。
* 和局部变量类似，局部内部类不能使用四种权限修饰符。
* 局部内部类不能使用`static`修饰，也不能包含静态成员。

## 3、内部类的使用

实例化**成员内部类**的对象：

* 静态成员内部类：`外部类.内部类 对象名 = new 外部类.内部类();`
* 非静态成员内部类：①创建外部类对象：`外部类 p = new 外部类();`②`外部类.内部类 对象名 = p.new 内部类();`

在成员内部类中区分调用外部类的结构：

* `this.属性`表示内部类的属性
* `外部类.this.属性`表示调用外部类的属性

局部内部类中的使用案例：如下面代码中的`public Comparable getComparable(){}`

内部类的使用代码：

```java
class Person{
    //静态成员内部类
    static class Brain{
    }
    
    //非静态成员内部类
    class Heart{ }

    public void method(){
        int num = 10;
        class AA{
            int aa = num;
        } //方法中的局部内部类
        //num = 20; //编译错误。局部内部类调用了num，则num是final的，不可以被修改。
    }
    
    {
        class BB{} //代码块中的局部内部类
    }
    
    public Person(){
        class CC{} //构造器中的局部内部类
    }

    public Comparable getComparable(){  //返回一个实现了Comparable接口的类的对象
        //方式一，定义局部内部类
        /*
        class MyComparable implements Comparable{
            public int compareTo(Object o){
                return 0;
            }
        }
        return new MyComparable();
        */

        //方式二：使用匿名实现类，返回匿名实现类的匿名对象
        return new Comparable() {
            @Override
            public int compareTo(Object o) {return 0;}
        };
    }
}
```



> 内部类在编译后，也会生成字节码文件：
>
> ①成员内部类：`外部类$内部类名.class`
>
> ​							例如：`Person$Brain.class`
>
> ②局部内部类：`外部类$数字 内部类名.class`
>
> ​							例如：`Person$1AA.class`

## 4、匿名内部类

**匿名内部类**不能定义任何静态成员、方法和类，只能创建匿名内部类的一个实例。一个匿名内部类一定是在`new`的后面，用其隐含实现一个接口或实现一个类。

格式：

`new 父类构造器(实参列表)|实现接口(){`

​		`	//匿名内部类的类体部分`

​	`}`

匿名内部类特点：

* 匿名内部类必须继承父类或实现接口。
* 只能有一个对象。
* 对象只能使用多态形式引用。

匿名内部类举例：

```java
interface Product{
	public double getPrice();
	public String getName();
}
public class AnonymousTest{
	public void test(Product p){
		System.out.println("购买了一个" + p.getName() + "，花掉了" + p.getPrice());
	}
	public static void main(String[] args) {
		AnonymousTest ta = new AnonymousTest();
		//调用test方法时，需要传入一个Product参数，
		//此处传入其匿名实现类的实例，匿名内部类。
		ta.test(new Product(){ 
			public double getPrice(){
				return 567.8;
			}
			public String getName(){
				return "AGP显卡";
			}
		});
	}
}
```

