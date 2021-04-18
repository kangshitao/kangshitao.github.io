---
title: Java学习笔记15-反射
excerpt: 反射机制，Class类，运行时类，类的加载，利用反射机制获取运行时类的结构，动态代理
mathjax: true
date: 2021-04-18 18:22:07
tags: Java
categories: Java
keywords: Java,反射机制,Class类,运行时类,类的加载,动态代理
---

# 一、Java反射机制概述

**Reflection(反射)**是被视为动态语言的关键，反射机制允许程序在执行期间借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。

类加载完之后，在堆内存的方法区中产生了一个Class类型的对象（一个类只有一个Class对象），这个对象包含了完整的类的结构信息。可以通过这个对象看到类的结构。

反射就是通过实例化对象反向得到其所属类的全部信息：实例化对象-->getClass()方法-->得到完整的“包类”名称。

**动态语言与静态语言**

动态语言是运行时可以改变结构的语言，例如新的函数、对象等，主要有：C#、JavaScript、PHP、Python、Erlang等。

静态语言是运行时结构不可变的语言，比如Java、C、C++

Java虽然不是动态语言，但是可以使用反射机制、字节码操作获得类似动态语言的特性。

# 二、理解Class类并获取Class实例

## 1、Class类

在Object类中定义了如下方法，此方法被所有子类继承：

`public final Class getClass()`

返回值是Class类型，**Class类是用来描述类的类**，此类是Java反射的源头，也就是说可以通过反射求出类的名称。

程序经过javac.exe命令以后，会生成一个或多个字节码文件(.class结尾)。使用java.exe命令对某个字节码文件进行解释运行。相当于将某个字节码文件加载到内存中。此过程就称为**类的加载**。加载到内存中的类，称为**运行时类**，此运行时类，就作为Class的一个**实例**。

加载到内存中的运行时类，会缓存一定的时间。在此时间之内，可以通过不同的方式来获取此运行时类。

对于每个类而言，JRE都为其保留一个不变的**Class类型**的对象。一个Class对象包含了特定某个结构`(class/interface/enum/annotation/primitive type/void/[])`的有关信息。

* Class本身也是一个类。
* Class对象只能由系统建立对象。
* 一个加载的类在JVM中只会有一个Class实例，Class的实例就对应着一个运行时类。
* 一个Class对象对应的是一个加载到JVM中的一个.class文件。
* 每个类的实例都会记得自己是由哪个Class实例所生成。
* 通过Class可以完整地得到一个类中的所有被加载的结构。
* Class类是Reflection的根源，任何想动态加载、运行的类，必须先获得相应的Class对象。

## 2、获取Class类实例

Class对象只能由系统创建，我们四种方式获取此实例，以String类为例：

* 通过类的`class属性`获取，前提是已知具体的类。此方法最安全可靠，程序性能最高：
  * `Class clazz = String.class;`
* 调用实例的`getClass()方法`，前提是已知当前类的实例：
  * `Class clazz = "hello".getClass();`
* 使用Class类的静态方法`forName()`获取，前提是已知一个类的全类名。可能抛出`ClassNotFoundException`异常：
  * `Class clazz = Class.forName("java.lang.String");`
* 其他方式：
  * `ClassLoader cl = this.getClass().getClassLoader();`
  * `Class clazz = cl.loadClass("java.lang.String");`

**哪些类可以有Class对象？**

* `class`：类，外部类，成员类（成员内部类、静态内部类），局部内部类，匿名内部类。
* `interface`：接口
* `[]`：数组
* `enum`：枚举类
* `annotation`：注解@interface
* `primitive type`：基本数据类型
* `void`

# 三、类的加载与ClassLoader的理解

## 1、类的加载过程

程序使用某个类时，如果该类没有加载到内存中，会通过以下三个主要步骤对类初始化：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1501_1.png"/>



* **加载**：将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后**生成一个代表这个类的java.lang.Class对象**，作为方法区中类数据的访问入口（即引用地址）。所有需要访问和使用类数据只能通过这个Class对象。这个加载的过程需要**类加载器**参与。
* **链接**：将Java类的二进制代码合并到JVM的运行状态之中的过程。包括三个阶段：
  * **验证**：确保加载的类信息符合JVM规范，例如：以cafe开头，没有安全方面的问题
  * **准备**：正式为类变量（static）分配内存并设置类变量默认初始值的阶段，这些内存都将在方法区中进行分配。
  * **解析**：虚拟机常量池内的符号引用（常量名）替换为直接引用（地址）的过程。
* **初始化**：执行行类构造器`<clinit>()`方法的过程。类构造器`<clinit>()`方法是由编译期自动收集类中所有类变量的赋值动作和静态代码块中的语句合并产生的（类构造器是构造类信息的，不是构造该类对象的构造器）。当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。虚拟机会保证一个类的`<clinit>()`方法在多线程环境中被正确加锁和同步。

**什么时候会发生类初始化?**

详细可参考《深入理解Java虚拟机 第3版》第7章 虚拟机类加载机制。

**类的主动引用（一定会发生类的初始化）**:

* 当虚拟机启动，先初始化main方法所在的类
* new一个类的对象
* 调用类的静态成员（除了final常量）和静态方法
* 使用java.lang.reflect包的方法对类进行反射调用
* 当初始化一个类，如果其父类没有被初始化，则先会初始化它的父类

**类的被动引用（不会发生类的初始化）**：

* 当访问一个静态域时，只有真正声明这个域的类才会被初始化
* 当通过子类引用父类的静态变量，不会导致子类初始化
* 通过数组定义某个类的引用，不会触发此类的初始化
* 引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了）

## 2、类加载器

**类加载器**的作用：在加载阶段，类加载器将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构然后在堆中生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问入口。
**类缓存**：标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载器中，它将维持加载（缓存）一段时间。不过JVM垃圾回收机制可以回收这些Class对象。

JVM规范定义了如下类型的类加载器：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1501_2.png"/>



引导类加载器用来装载核心类库，无法直接获取。

代码实现：

```java
public class ClassLoaderTest {
    @Test
    public void test1(){
        //对于自定义类，使用系统类加载器进行加载
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader);
        //jdk.internal.loader.ClassLoaders$AppClassLoader@2f0e140b

        //调用系统类加载器的getParent()：获取扩展类加载器
        ClassLoader classLoader1 = classLoader.getParent();
        System.out.println(classLoader1); 
        //jdk.internal.loader.ClassLoaders$PlatformClassLoader@2aaf7cc2

        //调用扩展类加载器的getParent()：无法获取引导类加载器
        //引导类加载器主要负责加载java的核心类库，无法加载自定义类。
        ClassLoader classLoader2 = classLoader1.getParent();
        System.out.println(classLoader2); //null
        ClassLoader classLoader3 = String.class.getClassLoader();
        System.out.println(classLoader3); //null。核心类
    }
}
```

类加载器的`getResourceAsStream(String str)`方法可以用来获取类路径下指定文件的输入流，可以用来加载properties配置文件：

```java
public class ClassLoaderTest {
    @Test
    public void test2() throws Exception {
        Properties pros =  new Properties();
        //此时的文件默认在当前的module下。
        //读取配置文件的方式一：
        //FileInputStream fis = new FileInputStream("jdbc.properties");
        //FileInputStream fis = new FileInputStream("src\\jdbc1.properties");
        //pros.load(fis);

        //读取配置文件的方式二：使用ClassLoader
        //配置文件默认识别为：当前module的src下
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        InputStream is = classLoader.getResourceAsStream("jdbc1.properties");
        pros.load(is);

        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        System.out.println("user = " + user + ",password = " + password);
    }
}
```





# 四、创建运行时类的对象

获得了Class对象以后，可以创建运行时类的对象。

有两种方法创建运行时类对象：

* 调用Class对象的`newInstance()`方法。使用此方法有两个要求：
  * 要求运行时类必须有一个无参数的构造器。
  * 要求运行时类的构造器的访问权限需要足够，一般为public
* 调用Class对象的`getDeclaredConstructor(Class<?>... parameterTypes)`，参数就是运行时类对应构造器的参数，返回`Constructor`类对象，`Constructor`类中也有`newInstance()`方法，可以构造运行时类对象。这种方法不要求运行时类必须有无参构造器。

代码实例，通过反射，创建运行时类对象：

```java
public class NewInstanceTest {
    //反射的动态性
    @Test
    public void test(){
        for(int i = 0;i < 10;i++){
            int num = new Random().nextInt(2);//0,1
            String classPath = "";
            //只有在运行时才知道具体的类
            switch(num){
                case 0:
                    classPath = "java.util.Date";
                    break;
                case 1:
                    classPath = "java.lang.Object";
                    break;
            }
            try {
                Object obj = getInstance(classPath);
                System.out.println(obj);
            } catch (Exception e) {e.printStackTrace();}
        }
    }
    /*
    创建一个指定类的对象。
    classPath:指定类的全类名
     */
    public Object getInstance(String classPath) throws Exception {
       Class clazz =  Class.forName(classPath);
       return clazz.newInstance();//方式一
        //方式二
       //return clazz.getDeclaredConstructor().newInstance();
    }
}
```

上述代码，体现了**反射的动态性，运行时动态创建指定类的对象**，这是反射机制应用最多的地方。



# 五、获取运行时类的完整结构

通过反射不仅可以创建运行时类的对象，还可以获取运行时类的完整结构，包括以下所有结构，这些操作都是基于Class类，使用Class类对象调用Class类的相关方法：

* 运行时类实现的全部接口
* 所继承的父类
* 全部的构造器
* 全部的方法
* 全部的Field

## 1、获取运行时类实现的接口

调用`public Class<?>[]getInterfaces()`方法获取运行时类实现的所有接口。

## 2、获取运行时类继承的父类

调用`public Class<? Super T>getSuperclass()`返回此Class所表示的实体（类、接口、基本类型）的父类的Class。

## 3、获取运行时类的构造器

获取运行时类的构造器：

* `public Constructor<T>[] getConstructors()`：返回此Class对象所表示的类的所有public构造方法。
* `public Constructor<T>[] getDeclaredConstructors()`：返回此Class对象表示的类声明的所有构造方法。

返回的`Constructor`类，有以下方法：

* `public int getModifiers()`：取得修饰符:
* `public StringgetName()`：取得方法名称:
* `public Class<?>[] getParameterTypes()`：取得参数的类型
* `public T newInstance(Object ... initargs)`：构建运行时类的实例

## 4、获取运行时类的方法

获取运行时类的方法：

* `public Method[] getDeclaredMethods()`：返回此Class对象所表示的类或接口的全部方法
* `public Method[] getMethods()`：返回此Class对象所表示的类或接口的public的方法

返回的`Method`类中有以下方法：

* `public Class<?> getReturnType()`：取得全部的返回值
* `public Class<?>[] getParameterTypes()`：取得全部的参数
* `public int getModifiers()`：取得修饰符
* `public Class<?>[] getExceptionTypes()`：取得异常信息

## 5、获取运行时类的Field

获取Field：

* `public Field[] getFields()`：返回此Class对象所表示的类或接口的public的Field。
* `public Field[] getDeclaredFields()`：返回此Class对象所表示的类或接口的全部Field。

返回的`Field`类中有以下方法：

* `public int getModifiers()`：以整数形式返回此Field的修饰符
* `public Class<?> getType()`：得到Field的属性类型
* `public String getName()`：返回Field的名称。

## 6、获取运行时类其他结构

**获取注解相关内容**

* `public <A extends Annotation> A getAnnotation(Class<T> annotationClass)`
* `public <A extends Annotation> A getDeclaredAnnotation(Class<A> annotationClass)`

**泛型相关**

* `public Type getGenericSuperclass()`：获取父类泛型类型
* `ParameterizedType`：泛型类型，继承了Type接口。`ParameterizedTypeImpl`类实现了此接口。
* `getActualTypeArguments()`：`ParameterizedType`中定义的`Type[]`类型数组，用于获取实际的泛型类型参数数组

**类所在的包**

* `public Package getPackage()`

# 六、调用运行时类的指定结构

使用反射获取不仅可以一次获取所有的结构，也可以获取指定的某个结构，然后调用此结构。

## 1、调用指定方法

可以使用反射获取指定的方法，并调用此方法。步骤如下：

* `getMethod(Stringname,Class…parameterTypes)`方法取得指定的Method对象，并设置此方法操作时所需要的参数类型。
* 使用`Object invoke(Objectobj, Object[]args)`进行调用，并向方法中传递要设置的obj对象的参数信息。

> invoke方法的说明：
>
> 1. Object对应原方法的返回值，若原方法无返回值，此时返回null
> 2. 若原方法若为静态方法，此时形参Object obj可为null
> 3. 若原方法形参列表为空，则Object[] args为null
> 4. 若原方法声明为private,则需要在调用此invoke()方法前，显式调用方法对象的setAccessible(true)方法，设置可见性，然后即可访问private的方法。

代码实例：

```java
public class ReflectionTest{
    /*
    Person类中有非静态方法show，和静态方法showDesc()
    private static void showDesc()
    */
    public void testMethod() throws Exception {
        Class clazz = Person.class;
        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();
        /*
        1.获取指定的某个方法
        getDeclaredMethod():
        参数1：指明获取的方法的名称  
        参数2：指明获取的方法的形参列表
         */
        //获取Person类中的show方法
        Method show = clazz.getDeclaredMethod("show", String.class);
        //2.保证当前方法是可访问的
        show.setAccessible(true);
        /*
        3. 调用方法的invoke()方法:
        参数1：方法的调用者，指明是哪个对象的方法  
        参数2：给方法形参赋值的实参
        invoke()的返回值即为对应类中调用的方法的返回值。
         */
        //类似于String nation = p.show("CHN");
        Object returnValue = show.invoke(p,"CHN"); 
        System.out.println(returnValue);
        
        /*
        调用静态方法
        */
        //调用showDesc方法
        Method showDesc = clazz.getDeclaredMethod("showDesc");
        showDesc.setAccessible(true);
        //如果调用的运行时类中的方法没有返回值，则此invoke()返回null
        Object returnVal = showDesc.invoke(Person.class);
        //这两种写法都可以，参数为空即可，不需要指明对象，
        //因为静态方法不需要通过对象调用，类中的静态方法是已知的。
        //Object returnVal2 = showDesc.invoke(clazz);
        //Object returnVal3 = showDesc.invoke(null);
        System.out.println(returnVal);//null
    }
}
```



## 2、调用指定属性

同样，反射机制可以调用指定的某个属性，并且可以直接通过Field类操作类中的属性，通过Field类提供的set()和get()方法就可以完成设置和取得属性内容的操作。

**获取指定属性** ：

* `public Field getField(String name)`：返回此Class对象表示的类或接口的指定的public的Field。
* `public Field getDeclaredField(String name)`：返回此Class对象表示的类或接口的指定的Field。

**Field中的方法**：

* `public Object get(Object obj)`：取得指定对象obj上此Field的属性内容
* `public void set(Object obj,Object value)`：设置指定对象obj上此Field的属性内容

代码实例：

```java
public class ReflectionTes{
    @Test
    public void testField1() throws Exception {
        //创建Class类实例
        Class clazz = Person.class;
        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();
        //1. 获取运行时类中指定变量名的属性，获取Person中的name属性
        Field name = clazz.getDeclaredField("name");
        //2.保证当前属性是可访问的
        name.setAccessible(true);
        //3.获取、设置对象p的name属性值为“Tom”
        name.set(p,"Tom");
        System.out.println(name.get(p));  // Tom
    }
}
```



## 3、调用指定构造器

通过反射获取指定的构造器，也可以创建运行时类的对象。

**获取指定构造器**：

* `public Constructor<T> getConstructor(Class<?>... parameterTypes)`：获取public的参数类型为指定类型的构造器。
* `public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)`：获取运行时类声明的**参数类型为指定参数**的构造器，

代码实例：

```java
public class ReflectionTest{
    //Person类中有以下构造器
    //private Person(String name)
     @Test
    public void testConstructor() throws Exception {
        Class clazz = Person.class;
        /*
        1.获取指定的构造器
        getDeclaredConstructor():参数：指明构造器的参数列表
         */
        Constructor constructor = clazz.getDeclaredConstructor(String.class);
        //2.保证此构造器是可访问的
        constructor.setAccessible(true);
        //3.调用此构造器创建运行时类的对象
        Person per = (Person) constructor.newInstance("Tom");
        System.out.println(per);
    }
}
```

**关于setAccessible方法的说明**

`Method`和`Field`、`Constructor`对象都有`setAccessible()`方法。

`setAccessible`用于启动和禁用访问安全检查的开关：

* 参数值为`true`指示反射的对象在使用时应该取消Java语言访问检查。有以下作用：
  * 提高反射的效率。如果代码中必须用反射，而该句代码需要频繁的被调用，需要设置为`true`。
  * 使得原本无法访问的私有成员也可以访问。
* 参数值为`false`指示反射的对象应该实施Java语言访问检查。

# 七、反射的应用：动态代理

## 1、动态代理概述

**代理模式**：使用一个代理将对象包装起来,然后用该代理对象取代原始对象。任何对原始对象的调用都要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。代理类和被代理类要实现同一个接口。

**静态代理**：代理类和目标对象的类都是在编译期间确定下来，不利于程序的扩展。每一个代理类只能为一个接口服务，这样程序开发中必然产生过多的代理。

**动态代理**：通过一个代理类完成全部的代理功能。在程序运行时根据需要动态创建目标类的代理对象。

动态代理使用场合：

* 调试
* 远程方法调用

动态代理相比于静态代理的优点：抽象角色中（接口）声明的所有方法都被转移到调用处理器一个集中的方法中处理，这样，可以更加灵活和统一的处理众多的方法。

一种**静态代理**实现举例：

```java
package com.atguigu.java;

/*
静态代理举例
特点：代理类和被代理类在编译期间，就确定了。
 */
//共同接口
interface ClothFactory{
    void produceCloth();
}

//代理类
class ProxyClothFactory implements ClothFactory{
    private ClothFactory factory;//用被代理类对象进行实例化
    public ProxyClothFactory(ClothFactory factory){
        this.factory = factory;
    }
    @Override
    public void produceCloth() {
        System.out.println("代理工厂做一些准备工作");
        factory.produceCloth();
        System.out.println("代理工厂做一些后续的收尾工作");
    }
}

//被代理类
class NikeClothFactory implements ClothFactory{
    @Override
    public void produceCloth() {
        System.out.println("Nike工厂生产一批运动服");
    }
}

public class StaticProxyTest {
    public static void main(String[] args) {
        //创建被代理类的对象
        ClothFactory nike = new NikeClothFactory();
        //创建代理类的对象
        ClothFactory proxyClothFactory = new ProxyClothFactory(nike);
        //调用代理类的方法
        proxyClothFactory.produceCloth();
    }
}
/*输出结果

代理工厂做一些准备工作
Nike工厂生产一批运动服
代理工厂做一些后续的收尾工作

*/
```



## 2、动态代理实现

Java中提供了`Proxy`类，专门用于完成代理操作。其是所有动态代理类的父类。通过此类为一个或多个接口动态地生成实现类。

`Proxy`类提供用于创建动态代理类和动态代理对象的静态方法:

* `static Class<?> getProxyClass(ClassLoader loader, Class<?>...interfaces)`：创建
  一个动态代理类所对应的Class对象
* `static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces
  InvocationHandler h)`：直接创建一个动态代理对象。第一个参数是类加载器，第二个参数是被代理类实现的全部接口，第三个参数是`InvocationHandler`接口的实现类实例。

动态代理实现步骤：

* 创建一个实现接口`InvocationHandler`的类，它必须实现`invoke`方法，以完成代理的具体操作。
* 创建被代理的类以及接口。
* 通过`Proxy`的静态方法`new ProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)`创建一个Subject接口代理。
* 通过`Subject`代理调用`RealSubject`实现类的方法。

以上步骤很难体现出动态代理的优势，下面的例子是更实用的动态代理机制的代码实现：

```java
package com.atguigu.java;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/*
  动态代理的举例
 */
//步骤2.创建实现的共同接口和被代理类
interface Human{ 
    String getBelief();
    void eat(String food);
}
//被代理类
class SuperMan implements Human{
    @Override
    public String getBelief() {
        return "I believe I can fly!";
    }
    @Override
    public void eat(String food) {
        System.out.println("我喜欢吃" + food);
    }
}

class HumanUtil{
    public void method1(){
        System.out.println("*****通用方法一*****");
    }
    public void method2(){
        System.out.println("*****通用方法二*****");
    }
}

/*
要想实现动态代理，需要解决的问题？
问题一：如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象。
问题二：当通过代理类的对象调用方法a时，如何动态的去调用被代理类中的同名方法a。
 */

//步骤3.创建Subject接口代理。
class ProxyFactory{
    //调用此方法，返回一个代理类的对象。解决问题一
    public static Object getProxyInstance(Object obj){//obj:被代理类的对象
        MyInvocationHandler handler = new MyInvocationHandler();
        handler.bind(obj);
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(),
                                      obj.getClass().getInterfaces(),
                                      handler);}
}

//步骤1.创建实现InvocationHandler接口的类
class MyInvocationHandler implements InvocationHandler{
    private Object obj;//需要使用被代理类的对象进行赋值
    public void bind(Object obj){
        this.obj = obj;
    }
    //当我们通过代理类的对象，调用方法a时，就会自动的调用如下的方法：invoke()
    //将被代理类要执行的方法a的功能就声明在invoke()中
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) 
        throws Throwable 
    {
        HumanUtil util = new HumanUtil();
        util.method1();
        //method:即为代理类对象调用的方法，此方法也就作为了被代理类对象要调用的方法
        //obj:被代理类的对象
        Object returnValue = method.invoke(obj,args);
        util.method2();
        //上述方法的返回值就作为当前类中的invoke()的返回值。
        return returnValue;
    }
}

public class ProxyTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        //proxyInstance:代理类的对象。通过指定的被代理类，创建代理类对象
        Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superMan);
        //当通过代理类对象调用方法时，会自动的调用被代理类中同名的方法
        String belief = proxyInstance.getBelief();
        System.out.println(belief);
        proxyInstance.eat("火锅");
    }
}
/*输出结果
*****通用方法一*****
*****通用方法二*****
I believe I can fly!
*****通用方法一*****
我喜欢吃火锅
*****通用方法二*****
*/
```

## 3、动态代理与AOP

AOP（Aspect Orient Programming），面向切片编程。

使用Proxy生成一个动态代理时，往往并不会凭空产生一个动态代理，这样没有太大的意义。通常都是为指定的目标对象生成动态代理。

这种动态代理在AOP中被称为AOP代理，AOP代理可代替目标对象，AOP代理包含了目标对象的全部方法。但AOP代理中的方法与目标对象的方法存在差异：**AOP代理里的方法可以在执行目标方法之前、之后插入一些通用处理**。