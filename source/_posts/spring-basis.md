---
title: Spring5配置与使用
excerpt: IOC容器，依赖注入，Bean自动装配，动态代理与AOP，声明式事务，Spring与MyBatis整合
mathjax: true
date: 2021-06-27 20:06:31
tags: ['SSM','Spring']
categories: SSM框架
keywords: Spring,IOC,AOP,动态代理,Bean
---



# 一、Spring简介

## 1.1 什么是Spring？

Spring 是一种轻量级开发框架（或者说是一种容器），旨在提高开发人员的开发效率以及系统的可维护性。Spring 官网：https://spring.io/

我们一般说 Spring 框架指的都是 Spring Framework，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发。这些模块是：核心容器、数据访问/集成,、Web、AOP（面向切面编程）、工具、消息和测试模块。比如：Core Container 中的 Core 组件是Spring 所有组件的核心，Beans 组件和 Context 组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。

Spring 官网列出的 Spring 的 6 个特征:

- **核心技术** ：依赖注入(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
- **测试** ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
- **数据访问** ：事务，DAO支持，JDBC，ORM，编组XML。
- **Web支持** : Spring MVC和Spring WebFlux Web框架。
- **集成** ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
- **语言** ：Kotlin，Groovy，动态语言。



**Spring是一个轻量级的IOC和AOP的框架。**



## 1.2 Spring的组成

下图对应的是 Spring4.x 版本。目前最新的5.x版本中 Web 模块的 Portlet 组件已经被废弃掉，同时增加了用于异步响应式处理的 WebFlux 组件。

![Spring主要模块](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/spring-basis_1.png)

- **Spring Core：** 基础，可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IoC 依赖注入功能。
- **Spring Aspects** ： 该模块为与AspectJ的集成提供支持。
- **Spring AOP** ：提供了面向切面的编程实现。
- **Spring JDBC** : Java数据库连接。
- **Spring JMS** ：Java消息服务。
- **Spring ORM** : 用于支持Hibernate等ORM工具。
- **Spring Web** : 为创建Web应用程序提供支持。
- **Spring Test** : 提供了对 JUnit 和 TestNG 测试的支持。
- 

## 1.3 Spring的优点

* Spring是一个开源免费的框架(容器)
* Spring是一个轻量级的、**非入侵式**的框架（AOP是非入侵式的，不需要对原有的代码进行修改）
* **控制反转(Inversion of Control ,IoC)**，**面向切面编程(Aspect-Oriented Programming，AOP)**
* 支持声明式事务。



## 1.4 Spring弊端

Spring经过长时间的发展，配置十分繁琐，人称"配置地狱"。

直到SpringBoot的出现，解决了这一问题。SpringBoot是一个快速开发的脚手架。



# 二、IOC

**控制反转loC(Inversion of Control)**，是一种设计思想，**DI(依赖注入)**是实现loC的一种方法，也有人认为DI只是loC的另一种说法。没有loC的程序中，我们使用面向对象编程，对象的创建与对象间的依赖关系完全硬编码在程序中，对象的创建由程序自己控制，控制反转后将对象的创建转移给第三方，个人认为控制反转就是：获得依赖对象的方式反转了。简而言之就是，**对象由Spring创建、管理和装配。**

IOC是Spring的核心内容，IOC可以有多种方式实现，比如XML配置，注解，甚至可以零配置实现IOC。

采用XML方式配置Bean的时候，Bean的定义信息是和实现分离的，而采用注解的方式可以把两者合为一体，Bean的定义信息直接以注解的形式定义在实现类中，从而达到了零配置的目的。

**控制反转是一种通过描述(XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是loC容器，其实现方法是依赖注入(Dependency Injection,Dl) ，控制反转是一种思想，依赖注入是一种具体实现方式。**

## 2.1 IOC基础

以传统的项目为例，假设现在有以下程序：

* DAO层：`UserDao`接口；接口的实现类`UserDaoImpl`

* Service层：`UserService`接口；接口的实现类`UserServiceImpl`

在`UserServiceImpl`中，有这样的代码：

```java
public class UserServiceImpl implements UserService{
    private UserDao userDao = new UserDaoImpl();
    @Override
    public void getUser() {
        userDao.getUser();
    }
}
```

`UserServiceImpl`中通过创建`UserDaoImpl`对象，完成相应的功能。

Controller层的程序，通过创建`UserServiceImpl`对象并调用方法完成功能。

```java
UserService userService = new UserServiceImpl();
//然后通过UserService调用方法完成功能
```



如果现在有第二个`UserDao`接口的实现类`UserDaoImpl2`，此时如果想要使用`UserDaoImpl2`中的方法，必须修改`UserServiceImpl`中的代码：

```java
private UserDao userDao = new UserDaoImpl2();
```

如果实现类有很多，就需要大量修改底层代码。

如何修改代码使得程序能够自动适应不同的需求呢？可以在`UserServiceImpl`中添加一个`set`方法，代替手动`new`的方式：

```java
public class UserServiceImpl implements UserService{
    private UserDao userDao;
    //private UserDao userDao = new UserDaoImpl();

    //利用set方法,动态实现值的注入
    //程序由主动创建对象，变为了被动接收对象。这就是控制反转的思想（IOC）
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void getUser() {
        userDao.getUser();
    }
}
```

这样，只要传入不同的UserDao的实现类，就能完成不同的功能。程序由主动创建对象，变为了被动接收对象。这就是IOC思想的原型。

主动权由原来的业务层(service)，变为了用户层。



## 2.2 使用IOC创建对象

使用Spring的IOC容器，创建并获取对象的基本流程如下，以创建一个HelloSpring程序为例。



**1、首先创建POJO类，`Hello.java`：**

```java
public class Hello {
    private String str;
    public String getStr() {
        return str;
    }
    ...//
}
```



**2、然后创建`applicationContext.xml`配置文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 
    id相当于变量名，根据此id获取对象
    class是全限类名，表示获取的是哪个类的对象
    property用于设置（注入）对象中属性的值,这里的name必须和属性名相同
    -->
    <bean id="hello" class="com.kang.pojo.Hello">
        <property name="str" value="Spring"/>
    </bean>
    
    <!-- more bean definitions go here -->
</beans>
```

顾名思义，`<bean>`标签的`class`必须是非抽象类，即能够实例化的类，不能是接口和抽象类。因为Spring要实例化这些类。



**3、最后实例化容器，并获取对象：**

```java
public static void main(String[] args) {
    //实例化容器，获取Spring的上下文对象
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    //对象现在都在Spring容器中管理了，使用getBean方法从容器里取对象
    Hello hello = context.getBean("hello",Hello.class);
    System.out.println(hello.toString());
}
//以上代码打印结果为：Hello{str='Spring'}
```



在配置文件中注册的bean，当实例化容器的时候，都会生成一个对象保存到容器中，然后使用`getBean()`方法获取指定的对象实例。

多次获取同一个id的对象，默认是单例模式，得到的都是同一个对象。

`ApplicationContext`是Spring的一个核心接口（或容器），允许容器通过应用程序上下文环境创建、获取、管理bean，是为应用程序提供配置的中央接口。其有多个实现类，根据不同的使用方式需要实例化不同类型的容器。

上述例子是使用Spring的一个简单案例，从中我们可以得出：

* hello对象是由Spring创建的，而不是我们手动创建的（使用`<bean>`)
* hello对象的属性是由Spring容器设置的（使用`<property>`）

这个过程就是控制反转。



## 2.3 IOC创建对象的方式



**1、如果不显式指定使用有参构造器，IOC默认使用无参构造器创建对象。**

使用无参构造器的前提是bean类中必须有无参构造器。



**2、如果想要使用有参构造器创建对象，有以下几种方式传参：**

* a. 通过**索引**为参数赋值，创建对象

  ```xml
  <!-- 索引从0开始 -->
  <bean id="user" class="com.kang.pojo.User">
      <constructor-arg index="0" value="usernamevalue"/>      
  </bean>
  ```

  这里的索引从"0"开始。



* b. 通过**类型**为参数赋值，创建对象

  ```xml
  <bean id="user" class="com.kang.pojo.User">
      <constructor-arg type="java.lang.String" value="usernamevalue"/>
  </bean>
  ```

  要想使用这种方法，必须保证各个参数可以通过类型区分开。基本类型可以直接用，引用类型必须使用全限类名。

  

* c. 通过**参数名称**赋值，创建对象

  ```xml
  <bean id="user" class="com.kang.pojo.User">
      <constructor-arg name="name" value="usernamevalue"/>
  </bean>
  ```

* d. 通过**引用**赋值，创建对象。

  ```java
  package x.y;
  public class ThingOne {
      public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
          // ...
      }
  }
  ```

  对应的配置文件：

  ```xml
  <beans>
      <bean id="thingOne" class="x.y.ThingOne">
          <constructor-arg ref="thingTwo"/>
          <constructor-arg ref="thingThree"/>
      </bean>
      <bean id="thingTwo" class="x.y.ThingTwo"/>
      <bean id="thingThree" class="x.y.ThingThree"/>
  </beans>
  ```

  

# 三、Spring配置文件详解

下面介绍Spring核心配置文件中的常用标签，后面的使用到再介绍。



## 3.1 alias

`<alias`起别名，比如：

```xml
<bean id="user" class="com.kang.pojo.User">
    <constructor-arg name="name" value="kangkang"/>
</bean>
<!--给user起别名，和直接通过id获取效果是一样的-->
<alias name="user" alias="userAlias"/>
```

其中在`<bean>`标签中的`name`属性也能够为对象起别名：

```xml
<!-- 通过name属性起别名，可以同时起多个别名，用逗号隔开 -->
<bean id="user" class="com.kang.pojo.User" name="user2,user3">
    <constructor-arg name="name" value="kangkang"/>
</bean>
```





## 3.2 Bean

Spring中将需要实例化的类使用`<bean>`注册为容器中的bean：

```xml
<bean id="user" class="com.kang.pojo.User" name="user2,user3">
    <constructor-arg name="name" value="kangkang"/>
</bean>
```

其中常用的变量有：

* `id`: bean的唯一标识，相当于对象名。
* `class`：bean对象所对应的全限类名，包名+类名。**必须是普通的类，不能是抽象类或者接口，因为容器要创建对象，抽象类和接口无法创建对象**。
* `name`：可以省略。也是别名，并且可以同时取多个别名，可以空格或`,`分隔。通过这些别名获取对象时，得到的都是同一个对象。
* `scope`：指定该bean的作用域，官方给定了六种作用域，比如`prototype`/`sigleton`等，具体参考下文内容。
* `autowire`：用于指定自动装配的方式，具体参考下文内容。



## 3.3 import

`import`一般用于团队开发，用于将多个配置文件导入合并在一起。

假设有一个总配置文件`applicationContext.xml`，每个开发人员都有自己的一个配置文件，则可以在总配置文件中将各个配置文件导入，合并为一个配置文件。这样在实例化容器时，只需要传入总配置文件，就可以创建导入的所有配置文件中的`bean`对象。

总配置文件`applicationContext.xml`引入多个配置文件：

```xml
<import resource="beans1.xml"/>
<import resource="beans2.xml"/>
<import resource="beans3.xml"/>
```

实例化容器：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

如果不同配置文件中，存在相同`id`的`bean`对象，则会根据导入的顺序依次进行覆盖，以最后一个为准。



# 四、依赖注入

**依赖注入(Dependency Injection，DI)**是一个过程，通过该过程，对象只能通过构造函数参数，工厂方法的参数或在构造或创建对象实例后在对象实例上设置的属性来定义其依赖关系（即其需要的其他对象），从工厂方法返回。然后，容器在创建 bean 时注入那些依赖项。从根本上讲，此过程是通过使用**类的构造器**或**服务定位器模式**来自己控制其依赖关系的实例化或位置的 Bean 本身的逆过程（因此称为**控制反转**）。

依赖注入使得代码更简洁，当为对象提供依赖项时，去耦会更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。

DI主要有两种方式：**基于构造器的依赖注入**和**基于Setter的依赖注入**。



## 4.1 构造器注入

使用构造器实现依赖注入，可以参考前面2.1中IOC创建对象的例子。

**1、基于无参构造器的依赖注入**，是在调用无参数构造函数或无参数`static`工厂方法以实例化 bean 之后，然后在bean上调用 `set` 方法来给属性赋值。

比如：

```xml
<!-- 通过反射调用无参构造器创建对象，然后调用set方法为name属性赋值 -->
<bean id="student" class="com.kang.pojo.Student">
    <property name="name" value="Jack"/>
</bean>
```

这种方式要求类中必须有对应的`setXxx()`方法



**2、基于有参构造器的依赖注入，通过有参构造器给属性赋值。**

主要是在`<bean>`中使用`<constructor-arg>`标签表示使用有参构造器。

* a. 通过**索引**为参数赋值，创建对象。

  > 这里的索引从"0"开始。

* b. 通过**类型**为参数赋值，创建对象。

  > 要想使用这种方法，必须保证各个参数可以通过类型区分开。基本类型可以直接用，引用类型必须使用全限类名。

* 通过**参数名称**赋值，创建对象。

  ```xml
  <bean id="user" class="com.kang.pojo.User">
      <constructor-arg name="name" value="usernamevalue"/>
  </bean>
  ```

  

* 通过**引用**赋值，创建对象。

  ```xml
  <beans>
      <bean id="thingOne" class="x.y.ThingOne">
          <constructor-arg ref="thingTwo"/>
          <constructor-arg ref="thingThree"/>
      </bean>
      <bean id="thingTwo" class="x.y.ThingTwo"/>
      <bean id="thingThree" class="x.y.ThingThree"/>
  </beans>
  ```

  



## 4.2 通过Setter方式注入

通过setter方式注入，也就是通过无参构造器创建对象后，调用`set`方法注入，因此使用setter方式注入，要求bean中必须要有`set`方法和一个**无参构造器**。

使用Setter方式注入：

```xml
<property name="id" value="1"/>
<property name="name" value="Java" />
<property name="age" value="18" />
```



**复杂类型的依赖注入**

举例：

1、定义类`Address`：

```java
public class Address {
    private String address;

    public String getAddress() {return address;}

    public void setAddress(String address) {this.address = address;}

    @Override
    public String toString() {
        return "Address{" +
            "address='" + address + '\'' +
            '}';
    }
}
```



2、实体类`Student`：

```java
public class Student {
    private String name;
    private Address address;
    private String[] books;
    private List<String> hobbies;
    private Map<String,String> card;
    private Set<String> games;
    private Properties info;
    private String wife;
    ...//
}
```



3、`applicationContext.xml`配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.kang.pojo.Address">
        <property name="address" value="xx省xx市xx区"/>
    </bean>
    <bean id="student" class="com.kang.pojo.Student">
        <!-- 1、普通值注入 -->
        <property name="name" value="kangst"/>
        <!--另一种写法-->
        <!--        <property name="name">-->
        <!--            <value>kangst</value>-->
        <!--        </property>-->

        <!-- 2、Bean注入 -->
        <property name="address" ref="address"/>
        <!--另一种写法-->
        <!--        <property name="address">-->
        <!--            <ref bean="address"/>-->
        <!--        </property>-->

        <!-- 3、数组注入 -->
        <property name="books">
            <array>
                <value>红楼梦</value>
                <value>西游记</value>
                <value>三国演义</value>
                <value>水浒传</value>
            </array>
        </property>

        <!-- 4、list注入 -->
        <property name="hobbies">
            <list>
                <value>hobby1</value>
                <value>hobby2</value>
                <value>hobby3</value>
            </list>
        </property>

        <!-- 5、map注入 -->
        <property name="card">
            <map>
                <entry key="身份证" value="12345678"/>
                <entry key="银行卡" value="6345112384034"/>
            </map>
        </property>

        <!-- 6、set注入 -->
        <property name="games">
            <set>
                <value>Game1</value>
                <value>Game2</value>
            </set>
        </property>

        <!-- 7、Properties注入 -->
        <property name="info">
            <props>
                <prop key="username">root</prop>
                <prop key="password">123456</prop>
            </props>
        </property>

        <!-- 8、null值注入 (和空串不同,空串直接赋值为""即可)-->
        <property name="wife">
            <null/>
        </property>
    </bean>
</beans>
```

4、测试

```java
public void test1(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    Student student = (Student) context.getBean("student");
    System.out.println(student.toString());
    /*输出结果
        Student{name='kangst',
        address=Address{address='xx省xx市xx区'},
        books=[红楼梦, 西游记, 三国演义, 水浒传],
        hobbies=[hobby1, hobby2, hobby3],
        card={身份证=12345678, 银行卡=6345112384034},
        games=[Game1, Game2],
        info={password=123456, username=root},
        wife='null'}
     */
}
```





## 4.3 其他方式注入

#### p-命名空间

p表示properties，允许使用`bean`元素的属性(而不是嵌套的`<property>`元素)来声明 `Bean`的属性值，或同时使用这两者。

**要求类中必须有`setXxx()`方法。**

如下，可以同时使用p-命名空间和`<property>`两种方式。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--p-命名方式和传统方式结合使用-->
    <bean id="user" class="com.kang.pojo.User" p:name="用户名">
        <property name="age" value="20"/>
    </bean>
</beans>
```



注意，使用p-命名空间时，需要导入`xmlns:p="http://www.springframework.org/schema/p"`约束



#### c-命名空间

c表示constructor，构造器。表示使用**有参构造器**注入属性值。**要求类中必须有有参构造器**。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- 传统使用构造器的声明方式 -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-命名空间声明方式 -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```



同样，需要导入`xmlns:c="http://www.springframework.org/schema/c"`约束





## 4.4 理解IOC和DI

**`IOC`：当某个角色(比如一个Java实例，调用者)需要另一个角色(另一个Java实例，被调用者)的协助时，在 传统的程序设计过程中，通常由调用者来创建被调用者的实例。但在Spring里，创建被调用者的工作不再由调用者来完成，因此称为控制反转**

**`DI`：创建被调用者实例的工作通常由Spring容器来完成，然后注入调用者，这个操作称为依赖注入**。

控制反转是一种思想，依赖注入是实现IOC的一种行为。



**控制反转（IOC）**

传统程序中，我们在类内部通过new的方式，主动创建其依赖对象，导致类和类之间高耦合。在Spring中，将创建和查找依赖对象的控制权交给了IOC容器，由容器负责控制对象的创建和注入组合对象，程序想要什么资源必须从容器中获取，即对资源的控制权转变了，这就是控制反转。IOC有效降低了耦合性。

谁控制谁？

* IOC容器控制对象；



控制什么？

* 控制了外部资源的获取（比如依赖对象，文件资源等）



为何是反转？

* 传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；



哪些方面反转了？

* 依赖对象的获取被反转了。



举个例子？

* 比如用户类需要依赖用户信息类，传统做法是程序（比如客户端类）创建用户类和用户信息类，然后将用户信息类主动注入得到用户类。而在Spring中，使用IOC容器，IOC容器创建用户类，发现用户类需要有依赖对象注入，然后创建用户信息类，并将其注入到用户类。此时的客户端类直接从容器中获取用户类即可。



**依赖注入**

组件之间的依赖关系由容器在**运行期**决定，即由容器**动态**的将某个依赖关系注入到组件中。即动态的向某个对象提供它所需要的其他对象。



谁注入谁？

* IOC容器注入将对象所依赖的对象或资源注入到这个对象中。



注入什么？

* 注入某个对象所需要的外部资源，或者说是属性，比如对象、资源、常量数据等。



# 五、Bean作用域与生命周期

## 5.1 Bean 作用域

Bean Scope（Bean作用域），即Bean对象的作用范围。Spring官方规定了Bean的六种作用域。

**1、singleton，单例模式（默认）：**

```xml
<bean id="user" class="com.kang.pojo.User" scope="singleton"/>
```

​	IOC容器**仅创建一个**Bean实例，并且IOC容器**每次返回的都是同一个Bean实例**，singleton是默认的作用域。



**2、prototype，原型模式：**

```xml
<bean id="user" class="com.kang.pojo.User" scope="prototype"/>
```

​	IOC容器**可以创建多个**Bean实例，**每次返回的都是一个新的实例**。



**3、request**

​	仅对HTTP请求产生作用，每次HTTP请求都会创建一个自己的Bean。

**4、session**

​	每个Session中只有一个共享的Bean实例。不同Session使用不同的实例。

**5、application**

​	作用域为整个web应用，即`ServletContext`

**6、websocket**

​	作用域为`WebSocket`



其中后四种仅在web应用中有效。

Spring还支持自定义范围。



## 5.2 Spring 中的单例 bean 的线程安全问题

当多个线程操作同一个对象的时候，对这个对象的成员变量的写操作会存在线程安全问题。

但是，一般情况下，我们常用的 `Controller`、`Service`、`Dao` 这些 Bean 是无状态的。无状态的 Bean 不能保存数据，因此是线程安全的。

> 无状态的bean只有普通的对数据的操作方法，没有数据存储的功能，比如UserDao
>
> 有状态的bean具有数据存储功能，比如User

常见的有两种解决办法：

* 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。
* 改变Bean的作用域为 `prototype`，保证每次请求都会创建一个新的 bean 实例，避免线程安全问题。



## 5.3 Bean的生命周期

简单来说，Bean的生命周期有四个阶段：

* **实例化**：容器通过获取`BeanDefinition`对象中的信息进行实例化，仅仅是简单的实例化，并未进行依赖注入。

  * 对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 
  * 对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。

  > 实例化对象被包装在`BeanWrapper`对象中（可以认为是Bean的原生态）

* **属性赋值**：也就是依赖注入的过程。Spring根据`BeanDefinition`中的信息，通过`populateBean()`方法为属性赋值。

* **初始化**：比如

  * 如果实现了`BeanNameAware`接口，就调用它的`setBeanName()`方法，传入Bean的名字。
  * 如果实现了其他的`*Aware`接口，同样调用其方法。
  * 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
  * 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
  * 如果 Bean 在配置文件中的定义包含` init-method` 属性，执行指定的方法。
  * 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法

* **销毁**：容器关闭时销毁bean。销毁时，如果实现了`DisposableBean`接口，就执行`destroy()`方法，如果配置文件中包含`destroy-method`属性，就调用指定方法。

  

![Bean声明周期](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/spring-basis_7.jpg)



这里涉及到了两个主要的接口：

* `BeanPostProcessor`接口，自定义处理，该接口提供了两个方法，即前置处理和后置处理方法。这个接口可以影响多个Bean。
* `*Aware`相关接口，只会调用一次。主要是用于从Spring容器中拿到一些资源，增强Bean的能力。



详细内容参考[Bean的生命周期](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/Spring%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93?id=_55-spring-%e4%b8%ad%e7%9a%84-bean-%e7%94%9f%e5%91%bd%e5%91%a8%e6%9c%9f)、[Spring中的bean生命周期是怎样的](https://www.zhihu.com/question/38597960/answer/1063970966)



# 六、Bean的自动装配



## 6.1 介绍



自动装配是Spring满足bean依赖的一种方式，自动装配就是**实现对Java类属性的自动注入**，也就是说为当前类中的类型为`bean`的属性自动注入值。

Spring会在上下文中自动寻找，并自动给bean装配属性。

Spring中bean的三种装配方式：

* xml中手动配置。（第四节依赖注入，使用`ref`）
* java中使用注解显式配置。
* 隐式自动装配。



本章使用到的测试类：Person类、Dog类、Cat类。

```java
//Person类
public class Person {
    private Cat cat;
    private Dog dog;
    private String name;
    ...//
}

//Dog类
public class Dog {}

//Cat类
public class Cat {}
```



首先看一下xml手动装配时的配置：

```xml
<bean id="cat" class="com.kang.pojo.Cat"/>
<bean id="dog" class="com.kang.pojo.Dog"/>

<bean id="person" class="com.kang.pojo.Person">
    <property name="name" value="kang"/>
    <property name="dog" ref="dog"/>
    <property name="cat" ref="cat"/>
</bean>
```

以上通过`ref`为`Dog`类型和`Cat`类型属性注入值的方式称为手动装配。

对`bean`类型的属性注入需要使用`ref`手动注入，他们的代码是相似的，有没有可能将这些相似的代码精简掉呢？这就需要使用`bean`的自动装配功能了。



**bean的自动装配有两种方式**，一种是在**xml中配置（组件扫描）**，另一种是在**Java代码中使用注解（比如@Autowired）**。

这两种方式具体使用哪一种需要视情况而定("it depends")，二者各有优缺点。使用注解更简洁，但会造成配置分散难以控制；使用XML配置的方式不需要在源代码上改动，但配置代码繁琐。

两种方式可以同时使用，要注意的是，**注解注入会在XML注入之前执行**，XML注入会覆盖`@Autowired`注解已经注入的内容。



## 6.2 使用xml自动装配



`xml`中的自动装配使用的参数为`autowire`，有以下5种类型，其中常用的是`byName`和`byType`：

* `byName`·：在应用上下文中自动查找和自己对象的`set`方法后的值对应的`bean id`。比如`setDog()`，会自动将`Dog`的首字母变为小写，查找`id`为`dog`的`bean`。

  > 只有命名符合驼峰命名规范的才会将首字母小写然后匹配，如果不符合规范，则会直接匹配，比如`setDOG()`，会去查找`id`为`DOG`的`bean`

  

  ```xml
  <!-- byName，通过set方法的方法名查找bean的id，为属性注入值 -->
  <bean id="cat" class="com.kang.pojo.Cat"/>
  <bean id="dog" class="com.kang.pojo.Dog"/>
  <bean id="person" class="com.kang.pojo.Person" autowire="byName">
      <property name="name" value="kang"/>
  </bean>
  ```

  这种方法要求bean的`id`唯一，并且bean应该和自动注入的属性的`set`方法的值（set方法名）一致。

  

* `byType`：在容器上下文中自动查找和自己对象属性类型相同的bean。

  ```xml
  <!-- 使用byType的方式，会自动查找和属性类型对应的bean并注入。 -->
  <bean id="cat" class="com.kang.pojo.Cat"/>
  <bean id="dog" class="com.kang.pojo.Dog"/>
  <bean id="person" class="com.kang.pojo.Person" autowire="byTpye">
      <property name="name" value="kang"/>
  </bean>
  ```

  这种方式要求所有bean类型必须唯一，并且该bean应该和自动注入的属性的类型一致。

  

* `constructor`：根据构造方法的参数的数据类型，进行byType模式的自动装配。

* `default`：由上级标签`<beans>`的`default-autowire`属性确定。

* `no`：默认情况。即不使用自动装配。



## 6.3 通过注解自动装配

JDK 5.0开始支持注解，Spring 2.5开始支持注解。使用注解注入不需要`set`方法

使用注解前，需要在配置文件中导入约束并配置注解支持：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
</beans>
```

和自动装配相关的注解有三个，分别是`@Autowired`、`@Qualifier`、`@Resource` 。其中`@Autowired`和`@Qualifier`是Spring中的注解，`@Resource` 是Java中的注解。



**1、`@Autowired`**

* 默认按类型匹配（`byType`），如果找不到则报错，如果有多个，则通过**属性名**找，如果通过名字也无法找到则报错。

  > 例如，一个接口如果有多个实现类，按仅按照类型查找会找到多个结果，Spring不知道使用哪一个，这时必须使用`byName`的方式，可以结合使用`@Autowired`和`@Qualifier`两种注解完成注入。也可以单独使用`@Resource`注解。

* 默认情况下要求依赖对象必须存在，如果要允许`null`值，可以设置这个注解的`required`属性为`false`，即`@Autowired(required = false)`

  > 使用@Nullable注解，也能表示属性值可以为null。

* 可以用于构造方法、set方法、普通方法、字段上。

* `@Autowired`注解进行注入的方式，是不通过`set`方法进行注入的，因此`set`方法可以省略。

`@Autowired`是三种注解中比较常用的。



**2、`@Qualifier`**

 按名称注入（`byName`），即在容器中查找和指定`value`值相同`id`的`bean`。



**3、`@Resource` **

是Java的注解。可以通过 `byName` 和 `byType`的方式注入， 默认先按 `byName`的方式进行匹配，如果匹配不到，再按 `byType`的方式进行匹配。



通过几个例子理解一下，这三个注解的用法。

**情况1、通过类型可以唯一确定bean，可以忽略名字，装配(注入)成功：**

`Person`类中的注解情况：

```java
public class Person {
    @Autowired
    private Cat cat;
    
    @Autowired
    private Dog dog;
    private String name;
    ...
}
```



`xml`中的`<bean>`:

```xml
<bean id="cat2" class="com.kang.pojo.Cat"/>
<bean id="dog2" class="com.kang.pojo.Dog"/>
<bean id="person" class="com.kang.pojo.Person"/>
```

上述情况下，可以通过类型唯一确定`bean`，因此可以注入成功。此时将`id`为`cat2`和`id`为`dog2`的bean分别注入到`Person`类的`cat`和`dog`属性中。



**情况2、查找出多个类型，但可以通过名称找到唯一的值，也能注入成功。**

`Person`：

```java
public class Person {
    @Autowired
    private Cat cat;
    
    @Autowired
    private Dog dog;
    private String name;
    ...
}
```



`xml`中的`<bean>`:

```xml
<!-- 将id为cat的bean注入到Person中，因为按名字查找，和属性名cat匹配 -->
<bean id="cat" class="com.kang.pojo.Cat"/> 
<bean id="cat2" class="com.kang.pojo.Cat"/>
<bean id="dog" class="com.kang.pojo.Dog"/>
<bean id="person" class="com.kang.pojo.Person"/>
```



观察上述情况，对于`Cat`类型，通过`byType`的方式可以找到两个`bean`，然后再通过`byName`的方式，查找和属性名(`Person`中的属性名为`cat`)相同的id，可以找到id为`cat`的`<bean>`，因此也可以注入成功。



**情况3、查找出多个类型，通过`byName`的方式也没找到，则注入失败。**

`Person`：

```java
public class Person {
    @Autowired
    private Cat cat;
    
    @Autowired
    private Dog dog;
    private String name;
    ...
}
```



`xml`中的`<bean>`:

```xml
<!-- 有两个Cat类型的bean，根据变量名cat找不到对应id的bean -->
<bean id="cat1" class="com.kang.pojo.Cat"/> 
<bean id="cat2" class="com.kang.pojo.Cat"/>
<bean id="dog" class="com.kang.pojo.Dog"/>
<bean id="person" class="com.kang.pojo.Person"/>
```

此时根据`byType`找到两个`Cat`类型的`bean`，然后根据变量名`cat`以`byName`的方式查找，仍然找不到，因此注入失败。

这种情况下，就需要使用`@Qualifier` 注解：

```java
//使用@Qualifier注解，指明查找的id名，即根据这个value在容器中查找bean
public class Person {
    @Autowired
    @Qualifier(value = "cat2")
    private Cat cat;
    
    @Autowired
    private Dog dog;
    private String name;
    ...
}
```

同时使用 `@Autowired`和`@Qualifier(value = "cat2")`两个注解，并指定按名称查找时的`id`，可以在容器中找到`id`为`cat2`的`bean`，因此可以注入成功，可以通过代码验证这一点：

```java
@Test
public void test2(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext2.xml");
    Person person = context.getBean("person", Person.class);
    System.out.println(person.toString());
    System.out.println(context.getBean("cat1")+"  "+context.getBean("cat2"));
}
/*运行结果
Person{cat=com.kang.pojo.Cat@2177849e, dog=com.kang.pojo.Dog@40cb8df7, name='kang'}
com.kang.pojo.Cat@13b13b5d  com.kang.pojo.Cat@2177849e
*/
```



可以看到，`person`实例中的`cat`属性被注入了值，并且这个对象确实是`@Qualifier`指定的`id`为`cat2`的`bean`。

`@Resource`注解可以使用`byName`和 `byType`的方式注入，优先使用`byName`的方式，具体使用案例不再赘述。



# 七、使用注解开发

使用注解开发，就是在Java代码中使用注解的方式，取代`xml`文件中配置的方式。**使用注解**和**XML配置**是Spring开发的两种主要的方式。

在Spring4之后，要使用注解开发，必须要保证已经导入了aop的包。

![确保导入aop包](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/spring-basis_2.png)



使用注解，需要在`applicationContext.xml`配置文件的头部添加注解对应的支持，使用`<contex>`指定需要使用的注解相关的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           https://www.springframework.org/schema/context/spring-context.xsd">

    <!--指定要扫描的包，这个包下的注解就会生效
        这个标签也包括了annotation-config的功能
        所以如果使用了这个标签，就可以不使用annotation-config了
    -->
    <context:component-scan base-package="com.kang"/>

    <!--
    使用context:annotation-config 可以隐式地自动向Spring容器注册4个BeanPostProcessor：
    AutowiredAnnotationBeanPostProcessor
    CommonAnnotationBeanPostProcessor
    PersistenceAnnotationBeanPostProcessor
    RequiredAnnotationBeanPostProcessor
    这样就可以使用@Resource 、@PostConstruct、@PreDestroy、
    @PersistenceContext、@Autowired、@Required等注解了，就可以实现自动注入。
    注册这4个 BeanPostProcessor的作用，就是为了系统能够识别相应的注解。
    -->
    <context:annotation-config/>

    <!--    <bean id="user" class="com.kang.pojo.User">-->
    <!--        <property name="name" value="kangst"/>-->
    <!--    </bean>-->

</beans>
```





## 7.1 注册bean



`@Componet`：组件，用于**类**上，说明这个类被Spring管理了，就是`bean`，等价于在`xml`配置文件中使用`<bean>`标签注册。默认的`id`为类名的小写。也可以显式指定`id`名。

```java
//@Component等价于<bean id="user" class="com.kang.pojo.User"/>
@Component
public class User {
    public String name;
}
```

`@Component`注解等价于`xml`配置文件中的：

```xml
<bean id="user" class="com.kang.pojo.User"/>
```





## 7.2 属性注入

使用`@Value`注解可以为属性注入值：

```java
@Component
public class User {
    //@Value用于给属性赋值。相当于<property name="name" value="Jarry"/>
    @Value("kangst")
    public String name;

    //@Value也可以用在set方法上
    @Value("Jarry")
    public void setName(String name) {
        this.name = name;
    }
}
```



`@Value`注解等价于`xml`配置文件中的：

```xml
<property name="name" value="Jarry"/>
```



## 7.3 @Component的衍生注解

`@Component`的几个衍生注解，比如在dao层、service层、controller层有各自的注解，在功能上和`@Component`是一样的。

这几个衍生注解可以解释为`@Component`的具体形式，**他们在Spring中的功能是相同的**，都是用于将其注解的类注册到Spring容器中，让容器托管。在不同的层使用不同的注解可以增加程序的可读性。

* `@Repository`：用于dao层，标注dao组件

```java
@Repository
public class UserDao {}
```



* `@Service`：用于service层

```java
@Service
public class UserService {}
```



* `@Controller`：用于controller层

```java
@Controller
public class UserController {}
```



## 7.4 自动装配

自动装配相关的注解，第六节已经详细介绍过了：

* `@Autowired`
* `@Qualifier`
* `@Resource` 

还有其他的注解，比如`@Nullable`注解标记的属性或字段允许为`null`。



## 7.5 作用域

使用`@Scope`注解可以指定当前bean的作用域，效果等价于配置文件中的`scope`属性。

```java
@Component
@Scope(value = "prototype")
public class User {}
```



## 7.6 小结

**XML文件配置和注解两种方式对比**

* XML适用性更好，适用于各种情况；便于维护。
* 注解只能在当前类使用；维护起来更复杂。



最佳实践：XML文件用来管理`bean`，注解只负责完成属性的注入。

使用注解时，需要注意要在配置文件中让注解生效，开启注解的支持。



# 八、基于Java的容器配置

## 8.1 使用Java配置类

上述使用注解的方式，仍然需要使用XML配置文件写一部分内容。这一节讲述如何**使用Java代码完全代替XML配置文件**，对Spring容器进行配置。

完全使用Java程序进行容器配置主要依赖于`JavaConfig`，其是Spring的一个子项目，在Spring之后它成为了核心功能。

使用Java类作为配置文件，实现XML配置文件的功能，完全取代了XML配置文件。

比如，创建一个配置类`MyConfig`:

```java
package com.kang.config;
import com.kang.pojo.User;

//@Configuration代表这是一个配置类，就和applicationContext.xml配置文件一样
//将当前类交给Spring容器托管，注册到容器中，因为它底层本质也是一个Component
@Configuration
//相当于配置文件中的<context:component-scan base-package="com.kang.pojo"/>
//@ComponentScan("com.kang.pojo")  
//引入另一个配置类
//@Import(MyConfig2.class)  
public class MyConfig {
    //@Bean就相当于配置文件中的<bean>标签，注册一个bean；
    //这个方法的名字就相当于bean标签中的id
    //方法的返回值就是bean标签中的class，表示要注册的类
    @Bean
    public User getUser(){
        return new User(); //返回要注入到容器中的对象
    }
}
```

配置类中使用注解替代XML文件的功能，比如：

* `@Configuration`：代表这是一个配置类，就和`applicationContext.xml`配置文件一样。同时这个注解表示将当前类交给Spring容器托管，注册到容器中，因为其底层也是`@Component`。

* `@Bean`：类似于`<bean>`标签，功能是在容器中注册一个bean，其id默认是方法名（可以指定别名），返回类型就是要注册的bean。

还有其他的一些注解：

* `@ComponentScan`：相当于配置文件中的`<context:component-scan base-package="xxx.xxx"/>`扫描包，使这个包下文件的注解生效。

* `@Import`：可以用于引入另一个配置类，也可以用来注册其他类的bean，相当于XML文件中的`<import>`标签。



有了配置类和bean以后，我们就可以实例化容器，获取注册好的bean对象：

```java
@Test
public void test1(){
    //如果完全使用配置类，只能通过AnnotationConfigApplicationContext来获取容器
    //通过配置类的class对象加载。
    ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
    //默认id为方法名
    User getUser = context.getBean("getUser", User.class);
    System.out.println(getUser.toString());
}
```

要注意的是，如果我们完全使用Java配置类，实例化容器用的是`AnnotationConfigApplicationContext`类。

`ApplicationContex`接口有很多实现类，`AnnotationConfigApplicationContext`也只是其中一种。



## 8.2 @Component和@Bean对比

**相同点**

二者都是注册bean。



**不同点**

* 作用对象不同：**`@Component` 注解作用于类，而`@Bean`注解作用于方法**。
* `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是在标有该注解的方法中定义产生这个 bean，`@Bean`告诉了Spring这是某个类的示例，当我们需要用它的时候容器会将其传递给我们。
* `@Bean` 注解比 `Component` 注解的自定义性更强，而且**很多地方我们只能通过 `@Bean` 注解来注册bean**。比如**当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现**。



只能使用 `@Bean`而不能使用 `Component` 的案例：

```java
@Bean
public OneService getService(int status) {
    switch (status)  {
        case 1:
            return new serviceImpl1();
        case 2:
            return new serviceImpl2();
        case 3:
            return new serviceImpl3();
    }
}
```





# 九、代理模式

Spring的AOP底层实现原理就是使用的**代理模式**。

使用代理模式能够在不改变原有代码的情况下，添加一些其他功能，符合**“开闭原则”，即“对扩展开放，对修改封闭”**。举个例子，比如说希望在当前程序的某个方法执行前后添加一个功能，直接修改源代码的做法是不提倡的，正确做法是使用一个代理类，将其功能代理过来然后添加新的功能。

这种**不改变原有代码，使用横向切入的开发方式，就是AOP（面向切面编程）的思想**。

代理模式的优势：

* 使被代理类的操作更加纯粹，不用关注一些额外功能。
* 额外功能交给代理类来实现，实现了业务的分工。
* 额外功能发生扩展时，方便集中管理。

## 9.1 静态代理

代理模式一般有以下三种角色：

**接口**：代理类和被代理类共同实现的接口。这样用户只需要对接口操作，不需要关心实现类是真实对象还是代理类对象。

> 虽然代理类不实现这个接口，使用组合的方式也能完成同样的功能，但是那样扩展性差。
>
> JDK接口实现的动态代理模式中，要求必须实现共同的接口，因为它就是代理的接口。
>
> Cglib实现的动态代理不需要实现接口，它是通过自动创建子类来实现代理的。

**被代理类**：实现接口，被代理类不直接和用户交互。

**代理类**：用于代理被代理类，供客户调用，完成被代理类的功能的同时，有一些扩展的功能。直接和用户交互。



静态代理的缺点：一个代理类只能代理一个接口，当被代理的接口增加的时候，会导致代理类翻倍。另外，如果接口中的功能修改了，会导致代理类也会修改。



## 9.2 动态代理

**什么是动态代理**

动态代理能够动态生成代理类，而不需要我们手动去写代理类。动态代理和静态代理的代理角色是一样的。



**动态代理的实现方式**

动态代理主要有**基于接口的动态代理**和**基于类的动态代理**两种方式：

* 基于接口：JDK的动态代理，代理类和被代理类实现共同的接口，动态代理返回的代理类类型就是这个接口类型。

* 基于类：Cglib。如果没有实现接口，则需要使用Cglib，生成被代理对象的子类来作为代理。

  > 实现接口的情况下也能用Cglib，只不过是代理的不是接口，而是使用子类来实现。

还有一种Java字节码实现：Javassist，是一个创建Java字节码的类库，可以直接编辑和生成Java生成的字节码。



**动态代理的优势**

动态代理除了具有传统代理模式的优势以外，还有其特有的优势：

* 动态代理类代理的同样是接口，但可以同时代理一个或多个接口，一般对应的是**一类**业务。
* 一个动态代理类可以代理多个类，前提是**这些类都是同一个接口的实现类**。也就是说，对于这些被代理类，代理类做的额外工作是相同的，这样这些被代理类都可以使用这一个代理类完成代理。



**基于接口的动态代理具体实现**

基于接口的动态代理是指**JDK动态代理，代理的是接口**。因此这种方式**要求代理类和被代理类必须实现同一个接口**。

JDK动态代理主要使用`Proxy`类和`InvocationHandler`两个接口。主要步骤是：

* 实现`InvocationHandler`接口和其中的`invoke()`方法，主要是用于动态地调用被代理类中的同名方法。这一步中需要被代理类的实例。
* 调用`Proxy`类的`newProxyInstance`方法，生成代理类对象，其中需要`InvocationHandler`实现类对象作为参数。这一步主要是用于动态生成代理类对象。

详情可以参考[反射的应用-动态代理](https://kangshitao.github.io/2021/04/18/java-note-1501/#%E4%B8%83%E3%80%81%E5%8F%8D%E5%B0%84%E7%9A%84%E5%BA%94%E7%94%A8%EF%BC%9A%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)。

下面是一种动态代理的实现：

```java
public class DynamicProxyTest {
    public static void main(String[] args) {
        //创建一个UserService实现类的对象，即被代理类对象
        UserService userService = new UserServiceImpl();
        //根据这个被代理类对象，生成一个代理类对象的实例
        //根据这里可以看出，被代理类必须也要实现接口，不然无法实现代理功能
        UserService proxyInstance = (UserService) ProxyFactory.getProxyInstance(userService);
        //调用代理类实例的方法，可以完成被代理类的功能和一些额外的方法。
        proxyInstance.addUser();
    }
}

//步骤1，需要实现InvocationHandler接口
class InvocationHandlerImpl implements InvocationHandler {
    private Object obj; //被代理类的对象

    public void setObj(Object obj) {
        this.obj = obj;
    }

    //当通过代理类的对象调用某个方法时，就会自动调用这个invoke()方法。
    //我们将代理类要执行的额外的方法写在里面，就可以完成代理的功能。
    @Override
    public Object invoke(Object proxy, Method method, 
                         Object[] args) throws Throwable {
        //proxy参数好像没用？
        System.out.println("调用方法之前，代理类执行一些额外方法");
        //调用被代理类对象的同名方法，args表示传入的参数，此返回值就是原方法的返回值
        Object result = method.invoke(obj, args);
        System.out.println("调用方法之后，代理类执行一些额外方法");
        return result;
    }
}

//步骤2，创建一个类，用来获取被代理类的实例
class ProxyFactory {
    //obj表示被代理类的对象
    //这里可以使用泛型，避免使用时类型强转。
    public static Object getProxyInstance(Object obj) {
        InvocationHandlerImpl handler = new InvocationHandlerImpl();
        handler.setObj(obj);
        //调用Proxy类的newProxyInstance方法获取被代理类的实例
        //需要将InvocationHandler实现类的对象作为参数
        return Proxy.newProxyInstance(
            obj.getClass().getClassLoader(),                  
            obj.getClass().getInterfaces(), 
            handler);
    }
}
/*运行结果
调用方法之前，代理类执行一些额外方法
执行UserServiceImpl类的addUser()方法
调用方法之后，代理类执行一些额外方法
*/
```



可以看到，当一个接口有多个实现类的时候，使用以上动态代理的方法，只要将被代理的对象传入，就可以自动生成一个代理类的对象，通过这个代理类对象完成代理功能。

> 如果使用了代理模式，因为要返回一个代理类对象，所以容器的`getBean()`方法的传入参数必须是共同实现的接口class对象。
>
> 如果没有使用代理模式，就是一般的IOC，则容器中返回的依旧是原生对象。

**动态代理代理的是接口，返回的代理类对象必须使用接口类型接收**，否则无法判断具体的类型，就不能调用相关方法。这也是代理类必须要和被代理类实现同一个接口的原因。





# 十、AOP

## 10.1 AOP介绍

### 10.1.1 简介

官方文档中关于AOP的介绍：[Aspect Oriented Programming with Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)

AOP(Aspect Oriented Programming)，面向切片编程，通过**预编译**方式（比如AspectJ）和**运行期动态代理**（比如Spring AOP）实现程序功能的统一维护的一种技术。利用AOP可以对业务逻辑各个部分进行隔离，使业务逻辑各部分之间的**耦合度降低**，提高程序的可重用性，同时提高开发效率。

AOP**能够将那些与业务无关却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码、降低模块间的耦合度**，并**有利于未来的可扩展性和可维护性**。

通俗点说，AOP提供声明式事务，允许用户自定义切面，**在不改变原来代码的情况下，增加新的功能**。

![AOP示意图](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/spring-basis_3.png)



### 10.1.2 相关概念

**AOP的相关概念，参考官方文档：[AOP Concepts](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-introduction-defn)**：

* **横切关注点**：跨越应用程序多个模块的方法或功能，和业务逻辑无关但需要关注的地方，比如安全（验证）、事务、日志、缓存等。

* **Aspect：切面**，是Advice和PointCut的集合，通知和切入点共同定义了切面的全部功能，即它是什么，在何时何处完成其功能。切面是横切关注点被模块化的**类**。

* **Advice：通知，增强**。即切面中要完成的工作，一般为切面类中的**方法**。共有五种类型。**定义了“切入什么”和“何时切入“**。

* **Target：目标**，相当于被代理类，即被通知的对象。

* **Proxy：代理**，相当于代理类，向target对象应用通知后创建的对象。

* **JointPoint：连接点**。指应用执行过程中能够插入切面的点，或者说能够应用通知的所有点，Spring AOP中的连接点指的是方法。

* **PointCut：切入点**。实际要切入的位置，**定义了在”何处“切入**。是一个具体确定的连接点。

* **Introduction：引入**。向现有的类中添加方法或属性。

* **Weaving：织入**。就是把切面连接到应用程序类型或者对象上，并创建代理对象的过程。

  > 在目标对象的生命周期里有多个点可以进行织入：
  >
  > 1、编译期织入：切面在目标类编译时期被织入。AspectJ采用的织入方式。
  >
  > 2、类加载期织入：目标类被引入之前增强该目标类的字节码。AspectJ5采用的方式
  >
  > 3、运行期织入：切面在应用运行期间的某个时刻被织入。Spring AOP采用这种方式，在织入切面的时候，AOP容器会为目标对象动态的创建代理对象。



**Advice的五种类型：**

* `Before advice`：前置通知。在连接点前面执行，前置通知不会影响连接点的执行，除非此处抛出异常。
* `After returning  advice`：正常返回通知。在连接点正常执行完成后执行，如果连接点抛出异常，则不会执行。
* `After throwing advice`： 异常返回通知。在连接点抛出异常后执行。
* `After (finally) advice`：返回通知。在连接点执行完成后执行，不论连接点有没有抛出异常，都会执行。
* `Around advice`：环绕通知。围绕在连接点前后，比如一个方法调用的前后。这是最强大的通知类型，能在方法调用前后自定义一些操作。环绕通知还需要负责决定是继续处理连接点（调用`ProceedingJoinPoint`的`proceed`方法）还是中断执行。 **环绕通知能够在前置通知之前，和返回通知之前执行一些操作**，并且能够控制连接点是否继续执行。



![AOP执行示意图](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/spring-basis_5.png)



## 10.2  AOP实现原理

String AOP的实现是**基于动态代理**的，如前面所述，如果被代理对象实现了某个接口，则会使用JDK的Proxy接口创建代理对象，实现动态代理；否则，如果被代理对象没有实现接口，此时必须使用Cglib来生成被代理对象的子类作为代理。

关于开启CGLIB代理的配置，可以参考官方文档：[Mixing Aspect Types](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-mixing-styles)

下图展示了两种代理模式的实现原理（[参考JavaGuide](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/Spring%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93?id=aop)）：

![JDK Proxy和CGLib代理](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/spring-basis_4.jpg)



## 10.3 Spring AOP和AspectJ AOP

AOP有多种实现框架，在Spring中主要是**Spring AOP实现**和**AspectJ AOP**两种实现方式。

AspectJ AOP也是一种AOP编程扩展框架，其内部使用BCEL框架完成其功能。

**Spring AOP基于代理模式，属于运行时增强；而AspectJ基于字节码操作(Bytecode Manipulation)，是编译时增强。**

Spring中集成了AspectJ，AspectJ 的AOP功能相比于Spring AOP功能更加强大，但Spring AOP使用起来更简单。



**关于Spring AOP和AspectJ的选择问题**

官方说明：[Choosing which AOP Declaration Style to Use](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-choosing)

* 如果切面较少，两者性能差异不大。如果切面太多时，AspectJ比Spring AOP快很多。
* 如果只需要在Spring bean上执行通知，建议使用Spring AOP即可
* 如果需要通知不受Spring 容器管理的对象，比如域对象，则需要使用AspectJ
* 如果希望通知连接点，而不是仅通知简单的方法（比如set、get方法等），则需要使用AspectJ



## 10.4 AOP在Spring 中的实现

参考官方文档[Aspect Oriented Programming with Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)

Spring中实现AOP需要导入AOP织入包：

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.6</version>
</dependency>
```

Spring中实现AOP可以通过三种方式，其中前两种都是Spring AOP，第三种是AspectJ AOP：

* 实现Spring提供的接口，比如`MethodBeforeAdvice`、`AfterReturningAdvice`等接口。
* 自定义切面类，然后在XML配置文件中使用`<aop:config>`标签。
* 使用注解，借助内部集成的AspectJ实现AOP。在切面类中使用`@Aspect `、`@Before`、`@After`、`@Around`等注解。



### 10.4.1 方式一：实现Spring接口

Spring中给定了一些相关的接口，用于定义通知，比如`MethodBeforeAdvice`、`AfterReturningAdvice`等接口。我们需要实现这些接口，完成通知的具体功能。

通过实现Spring接口实现AOP的方法，主要是使用`<aop:advisor>`标签，具体实现如下：

**1、定义接口的实现类，实现对象通知方法**

可以用一个实现类同时实现多个通知接口，也可以创建多个类各自实现一个接口。

这里创建一个`Log`类，在目标方法执行前后输出一些信息：

```java
public class Log implements MethodBeforeAdvice,AfterReturningAdvice{
    /**实现before方法
     * @param method 反射方法
     * @param args 方法的参数
     * @param target 方法调用的目标对象，可以为空
     */
    @Override
    public void before(Method method, Object[] args, Object target) {
        System.out.println("Before:"+target.getClass().getName()+
                           "类的"+method.getName()+"方法被执行");
    }

    //实现afterReturning方法
    @Override
    public void afterReturning(Object returnValue, Method method, 
                               Object[] args, Object target) {
        System.out.println("After:"+"执行了"+method.getName()+
                           ",返回结果为："+returnValue);
    }
}
```



**2、在配置文件中注册bean，然后配置AOP**：

```xml
<!-- 将Log类注册为bean -->
<bean id="log" class="com.kang.log.Log"/>
<!-- 配置AOP -->
<aop:config>
    <!--配置切入点-->
    <aop:pointcut id="pointcut" expression=
                  "execution(* com.kang.service.UserServiceImpl.*(..))"/>
    <!-- advisor，定义通知器，类似于切面，包括了通知和切点 -->
    <!-- 如果有多个实现类，就写多个标签 -->
    <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
</aop:config>
```

其中`advisor`表示定义一个通知器，通知器的概念类似于切面，包括了通知和切入点。

为了可读性，建议一个实现类对应一个通知接口，这样一个类就可以表示一个通知，结构清晰。

**3、配置完后，可以测试是否生效：**

```java
@Test
public void test1(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    // 动态代理是代理的接口，因此这里必须传入的是接口的class对象
    UserService userService = context.getBean("userService",UserService.class);
    userService.queryUser();
}
/*执行结果
Before:com.kang.service.UserServiceImpl类的queryUser方法被执行
执行queryUser()方法
After:执行了queryUser,返回结果为：null
*/
```



**补充：关于切点表达式**

切点表达式主要用于定位连接点。详细用法可以参考[切点表达式](https://www.jianshu.com/p/d99e3d0edc32)、[Spring AOP AspectJ 切入点表达式示例](https://www.kancloud.cn/apachecn/howtodoinjava-zh/1952644)

表达式中有三种通配符：

* `*`：匹配任何数量字符
* `..`：用在路径中表示任何数量子包，用在参数中则表示任何数量参数。
* `+`：匹配指定类型的子类型，仅能作为后缀放在类型模式后边。

格式如下：

```xml
//[]内的内容可以省略
execution([方法修饰符] 返回值类型 [包路径]方法名(参数类型列表)[throws 异常类型])
```

execution表达式主要由三部分组成

* 方法修饰符，比如`public`、`private`等；可以省略
* 返回值类型，可以用`*`表示所有类型。
* [包路径]方法名及其参数类型的列表。其中`*`表示所有方法或所有类，参数类型列表中填的是参数的类型。其中包路径可以省略，表示表示所有包下和这个方法同名的方法。

此外，在AspectJ中，切入点表达式可以通过`&&`、`||`、`!`来组合切入点表达式，由于在XML中使用`&&`需要使用转义字符来代替，因此Spring AOP 使用`and`、`or`、`not`来代替。



表达式举例：

1、全通配

```xml
execution(* *..*.*(..))
```

2、匹配所有目标类以xxx开头的方法，第一个*代表返回任意类型，参数类型为任意数量的任意类型。

```xml
execution(* xxx* (..))
```

3、匹配Service接口及其实现子类中的所有方法

```xml
execution(* com.xxx.Service.*(..))
```

4、匹配service包下的所有类的所有方法，**但不包括子包**

```xml
execution(* com.xxx.service.*(..))
```

5、匹配aop_part包下的所有类的所有方法，包括子包。

```xml
# 注意 （当".."出现在类名中时，后面必须跟" * ",表示包、子孙包下的所有类**）
execution(* com.xxx.service..*(..))
```

6、匹配所有方法名为add，且有两个参数，其中，第一个的类型为int 第二个参数是String

```xml
execution(* add(int, String))
```

7、匹配所有方法名为add，且至少含有一个参数，并且第一个参数为int的方法

```xml
execution(* add(int, ..))
```



### 10.4.2 方式二：自定义切面类

自定义切面主要借助于`<aop:aspect>`来实现。我们只需要在切面类中定义通知方法，不需要实现接口。

**1、定义切面类**

```java
//定义两个通知方法
public class MyAspect {
    public void before(){
        System.out.println("before：方法执行前");
    }
    public void after(){
        System.out.println("after：方法执行后");
    }
}
```

**2、在配置文件中注册bean，然后配置AOP**

```xml
<!--方式二：自定义切面-->
<!-- 将切面类注册为bean -->
<bean id="myAspect" class="com.kang.diy.MyAspect"/>
<!-- 配置AOP --> 
<aop:config>
    <aop:aspect ref="myAspect">
        <!--定义切入点-->
        <aop:pointcut id="pointcut" expression=
                      "execution(* com.kang.service.UserServiceImpl.*(..))"/>
        <!--定义通知，这里定义了before和after两种类型-->
        <aop:before method="before" pointcut-ref="pointcut"/>
        <aop:after method="after" pointcut-ref="pointcut"/>
    </aop:aspect>
</aop:config>
```

我们在配置文件中，使用`<aop:aspect>`定义切面类，然后定义切入点和通知的类型。



**3、测试是否生效：**

```java
@Test
public void test2(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserService userService = context.getBean("userService", UserService.class);
    userService.deleteUser();
}
/*运行结果
before：方法执行前
执行deleteUser()方法
after：方法执行后
*/
```



可以看到，这种方法和第一种方法比较相似，对比`<aop:aspect>`和`<aop:advisor>`：

* `aspect`和`advisor`都是定义切面，二者最终的原理基本上是一样的。 
* `aspect`定义切面时，只需要定义一般的bean，而`advisor`中引用的通知，必须实现相应的通知接口。
* `advisor`大多用于事务管理。



### 10.4.3 方式三：使用AspectJ的注解

前两种方法都是基于XML配置使用Spring AOP的实现，这种注解方式是使用AspectJ的注解实现AOP。

**1、创建类，并使用注解标记为切面类**

```java
@Aspect  //标注这个类是一个切面类
public class AnnotationAspect {
    //将当前方法标记为前置通知，并指明切入点
    @Before(value="execution(* com.kang.service.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("注解Before：方法执行前");
    }

    @After(value = "execution(* com.kang.service.UserServiceImpl.*(..))")
    public void after(){
        System.out.println("注解After：方法执行后");
    }

    /*
    环绕通知会在前置通知和返回通知之前执行一些东西，并且控制连接点是否继续执行
     */
    @Around(value = "execution(* com.kang.service.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint pjp){
        System.out.println("注解Around：方法执行前");
        try {
            // 执行目标方法，如果不调用此方法，则会中断执行目标方法。
            pjp.proceed();
            // pjp还可以获取目标方法的签名
            System.out.println(pjp.getSignature());
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("注解Around：方法执行后");
    }
}
```



**2、注册bean并开启AspectJ注解**

```xml
<bean id="annotationAspect" class="com.kang.anno.AnnotationAspect"/>
<!-- 使AspectJ注解生效，默认使用JDK接口动态代理 -->
<aop:aspectj-autoproxy/>
```



其中Spring的动态代理默认使用的是JDK动态代理。也可以通过参数设置为使用cglib：

```xml
<!-- proxy-target-class默认为false，表示使用JDK动态代理接口 -->
<aop:aspect-autoproxy proxy-target-class="true">
```



如果使用的是Java配置类，开启AspectJ需要使用`@EnableAspectJAutoProxy `注解：

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
}
```





**3、测试是否织入成功**

```java
@Test
public void test3(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserService userService = context.getBean("userService", UserService.class);
    userService.queryUser();
}
/*执行结果
注解Around：方法执行前
注解Before：方法执行前
query a user
void com.kang.service.UserService.queryUser()
注解Around：方法执行后
注解After：方法执行后
*/
```

使用注解的方式，如果每个通知的切入点相同，可以定义一个方法写切入点。



### 10.4.4 总结



选用XML配置还是AspectJ注解，需要视情况而定，官方说明：[@AspectJ or XML for Spring AOP?](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-ataspectj-or-xml)，官方文档提供了一个例子，只能使用AspectJ注解而无法使用XML的例子：

```java
@Pointcut("execution(* get*())")
public void propertyAccess() {}

@Pointcut("execution(org.xyz.Account+ *(..))")
public void operationReturningAnAccount() {}

@Pointcut("propertyAccess() && operationReturningAnAccount()")
public void accountPropertyAccess() {}
```

上述这种情况，只有前两种能用XML配置：

```xml
<aop:pointcut id="propertyAccess"
              expression="execution(* get*())"/>

<aop:pointcut id="operationReturningAnAccount"
              expression="execution(org.xyz.Account+ *(..))"/>
```

对于第三种这样结合使用过的方式，是XML无法完成的，这也是XML配置方式的一个缺陷。







#　十一、整合MyBatis

整合主要是将MyBatis中需要手动创建对象的地方，交给Spring容器去做。我们最后只需要通过容器获取实现类的对象，然后调用相关的方法即可。

一般来说Spring和MyBatis有三种整合方式：

* 使用`MapperScannerConfigurer`，它会查找类路径下的映射器并自动将它们创建为`MapperFactoryBean`。

* 使用`org.mybatis.spring.SqlSessionTemplate`类获取`SqlSession`实现类类对象，它是`SqlSession`接口的一个实现类

* 使用`org.mybatis.spring.support.SqlSessionDaoSupport`获取`SqlSession`实现类对象。相比于第二种方法，这中方法省略了在Spring容器中注册`SqlSessionTemplate`的步骤。

  > `org.apache.ibatis.session.SqlSession`类是MyBatis中和一个核心接口，它的实现类用于获取Mapper接口的实现类对象。

这里我们主要使用[`MyBatis-Spring`](http://mybatis.org/spring/zh/index.html)来整合，其采用了后两种方式。

首先我们需要在Maven配置中导入MyBatis-Spring依赖和spring-jdbc依赖：

```xml

<!-- spring整合mybatis需要用的依赖 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
</dependency>

<!-- Spring操作数据库，还需要一个spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.0.RELEASE</version>
</dependency>
```

我们需要准备一个POJO类，和一个对应的Mapper接口，以及对应的Mapper配置类。

`User`

```java
public class User {
    private int id;
    private String name;
    private String pwd;

    public User() {}
    ...
}
```

`userMapper`

```java
public interface UserMapper {
    public List<User> selectUser();
    ...
}
```

`UserMapper.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.kang.mapper.UserMapper">
    <select id="selectUser" resultType="User">
        select * from user
    </select>
</mapper>
```



目录结构如下：

![Spring-MyBatis整合目录](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/spring-basis_6.png)





## 11.1 方式一：使用SqlSessionTemplate

主要思路是将MyBatis中需要手动创建对象的地方交给Spring容器处理。

参考[mybatis-spring官方文档](http://mybatis.org/spring/zh/getting-started.html)，详细步骤如下：

**1、配置数据源**

```xml
<!-- DataSource 
 使用Spring的数据源替换Mybatis的dataSource配置
    这个数据源可以使用任意的数据源，这里使用的是Spring提供的JDBC.
    需要导入spring-jdbc依赖。
    -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>
```



**2、获取SqlSessionFactory对象**

```xml
<!-- 注册SqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref = "dataSource"/>
    <!-- 还可以绑定Mybatis配置文件和注册Mapper -->
    <property name="configLocation" value=
              "classpath:mybatis-config.xml"/>
    <property name="mapperLocations" value=
              "classpath:com/kang/mapper/UserMapper.xml"/>
</bean>
```

其中`dataSource`属性是必须的。

关于`SqlSessionFactoryBean`的详细介绍可以参考[SqlSessoinFactoryBean](http://mybatis.org/spring/zh/factorybean.html)

值得注意的是，`mapperLocations`也可以接受多个资源位置，即注册多个映射器，比如下面的配置表示会从类路径下加载所有在`sample.config.mappers`包和它的子包中的映射器`xml`文件。

```xml
<property name="mapperLocations" value="classpath*:sample/config/mappers/**/*.xml" />
```



**3、获取SqlSessionTemplate对象**

`SqlSessionTemplate`是`SqlSession`的一个实现类，可以无缝代替`SqlSession`

```xml
<!-- SqlSessionTemplate 就和SqlSession类似 -->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <!-- 只能使用构造器注入sqlSessionFactory，因为SqlSessionTemplate没有set方法 -->
    <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>
```



**4、给接口编写实现类**

为了将MyBatis中手动获取Mapper对象的过程由Spring IOC容器接管，我们需要定义一个Mapper实现类，然后将其注入到Spring容器：

```java
package com.kang.mapper;

public class UserMapperImpl implements UserMapper{
    //我们的所有操作，原来都是用sqlSession来执行，
    //现在mybatis-spring整合，使用SqlSessionTemplate代替SqlSession
    private SqlSessionTemplate sqlSession;

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public List<User> selectUser() {
        //获取UserMapper的一个实现类对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.selectUser();
    }
}
```

这个实现类相当于连接了MyBatis和Spring，将MyBatis获取Mapper实现类对象的步骤封装到一个类中，这个实现类虽然继承了接口，但它是借用MyBatis完成的功能，因此我们将这个实现类注册到Spring容器中。

之后从容器中获取这个实现类的对象就可以完成功能。



**5、将实现类注入到Spring中**

```xml
<bean id="userMapper" class="com.kang.mapper.UserMapperImpl">
    <property name="sqlSession" ref="sqlSession"/>
</bean>
```



**6、通过Spring容器获取实现类对象，调用方法，完成功能**

```java
@Test
public void selectUser() {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    //直接从容器中获取实现类对象，调用方法就可以完成功能。
    UserMapper userMapper = context.getBean("userMapper", UserMapper.class);
    List<User> users = userMapper.selectUser();
    for(User user:users){
        System.out.println(user);
    }
}
```



---



**汇总**

我们在Spring的配置文件中，可以实现对数据源的配置，也可以注册Mapper映射器，因此可以完全取代MyBatis配置文件，这里象征性地保留MyBatis配置文件`mybatis-config.xml`，里面保留了起别名的配置语句。

`mybatis-config.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="com.kang.pojo"/>
    </typeAliases>
</configuration>
```



我们可以将上述的获取数据源、获取`SqlSessionFactory`对象、获取`SqlSessionTemplate`对象这三个和数据库相关的配置语句单独放到一个配置文件`spring-dao.xml`中，在`applicationContext.xml`核心配置文件中用来注册bean，并负责将引入`spring-dao.xml`。

`spring-dao.xml`，用于获取SqlSessionTemplate对象：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 1、DataSource -->
    <bean>...</bean>
    <!-- 2、获取SqlSessionFactory对象 -->
    <bean>...</bean>
    <!-- 3、获取SqlSessionTemplate对象 -->
    <bean>...</bean>
</beans>
```

`applicationContext.xml`，引入`spring-dao.xml`，注册bean：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 引入spring-dao -->
    <import resource="spring-dao.xml"/>
    
    <!-- 注册bean，即userMapper实现类对象 -->
    <bean id="userMapper" class="com.kang.mapper.UserMapperImpl">
        <property name="sqlSession" ref="sqlSession"/>
    </bean>
</beans>
```



**测试**

通过上述配置以后，我们可以测试是否配置成功。

```java
@Test
public void selectUser() {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    //现在我们只需要从Spring容器中获取UserMapper实现类对象即可
    UserMapper userMapper = context.getBean("userMapper", UserMapper.class);
    List<User> users = userMapper.selectUser();
    for(User user:users){
        System.out.println(user);
    }
}
```





## 11.2 方式二：SqlSessionDaoSupport

第一种方式中，我们需要将`SqlSessionFactory`注入到`SqlSessionTemplate`类来获取`SqlSessionTemplate`对象。

现在我们将`UserMapperImpl`类继承`SqlSessionDaoSupport`类，就可以直接获得`SqlSessionTemplate`对象。

`UserMapperImpl`

```java
package com.kang.mapper;

public class UserMapperImpl extends SqlSessionDaoSupport implements UserMapper{
    @Override
    public List<User> selectUser() {
        //getSqlSession()方法会创建并返回一个SqlSessionTemplate对象
        SqlSession sqlSession = getSqlSession();
        return sqlSession.getMapper(UserMapper.class).selectUser();
    }
}
```

我们可以使用`getSqlSession()`方法，其内部为我们创建了一个`SqlSessionTemplate`对象并返回，因此我们不需要在Spring容器中注册`SqlSessionTemplate`类。

此时的`spring-dao.xml`省略了一个bean内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 1、DataSource -->
    <bean>...</bean>
    <!-- 2、获取SqlSessionFactory对象 -->
    <bean>...</bean>
</beans>
```

同时，我们在`applicationContext.xml`配置文件中注册`UserMapperImpl`的时候，需要将`SqlSessionFactory`依赖注入:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userMapperImpl" class="com.kang.mapper.UserMapperImpl">
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>
</beans>
```





# 十二、声明式事务

参考[Spring事务总结](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/Spring%E4%BA%8B%E5%8A%A1%E6%80%BB%E7%BB%93)

Spring中的事务有两种：

* 编程式事务：在代码中管理事务，即使用 `TransactionTemplate`类或者`TransactionManager`接口手动编码管理事务。（不推荐使用）
* 声明式事务：使用AOP，在配置文件中配置。

Spring中实现声明式事务同样也包括**XML配置**和**注解**两种方式，其中XML配置主要使用的时`<tx>`标签，注解主要使用的是`@Transactional`注解。

首先我们需要准备POJO类，对应的接口以及接口实现类。

`User`

```java
public class User {
    private int id;
    private String name;
    private String pwd;

    public User() {
    }
    ...
}
```

`UserMapper`

```java
public interface UserMapper {
    public int addUser(User user);
    public int deleteUser(int id);
    public int updateUser(int id);
    public List<User> querytUser();
}
```

`UserMapperImpl`内容和十一节用到的相似。



## 12.1 使用声明式事务

### 12.1.1 使用tx和aop命名空间实现声明式事务

前面说过，`advisor`多用于事务管理。我们可以结合使用`<tx>`和`<aop>`配置声明式事务。我们将事务配置语句写在`spring-dao.xml`文件中，内容如下。

`spring-dao.xml`

```xml
<!-- 配置声明式事务的步骤 -->
<!-- 1.开启事务处理功能，配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <constructor-arg ref="dataSource"/>
</bean>

<!-- 2.使用AOP织入 -->
<!-- 默认使用transactionManager -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <!-- 给切入点中指定的方法配置事务 -->
    <tx:attributes>
        <tx:method name="addUser" propagation="REQUIRED"/>
        <tx:method name="deleteUser" propagation="REQUIRED"/>
        <tx:method name="updateUser" propagation="REQUIRED"/>
        <tx:method name="queryUser" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>

<!-- 配置事务切入点和通知器 -->
<aop:config>
    <aop:pointcut id="txPointcut" expression="execution(* com.kang.mapper.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
</aop:config>
```

其中，`propagation`表示事务的传播行为，默认是`REQUIRED`，下文详细讲解，还可以根据需求配置其他属性。

`<tx:method>`中的`name`要注入事务的方法名，可以用`*`表示所有方法。

**注意：**为事务管理器指定的 `DataSource` **必须**和用来创建 `SqlSessionFactoryBean` 的是同一个数据源，否则事务管理器无法工作。

除了使用AOP的方式，还可以直接使用`JtaTransactionManager`，不用上面手动配置事务管理器和AOP织入等，直接配置事务管理器，[参考mybatis-spring官方-交由容器管理事务](http://mybatis.org/spring/zh/transactions.html#)：

```xml
<tx:jta-transaction-manager />
```

> JTA: Java Transaction API





### 12.1.2 使用@Transactional配置声明式事务

使用注解的方式配置声明式事务，主要步骤：

**1.开启事务处理功能，配置事务管理器**

**2. 开启Spring对注解事务的支持，使类中事务注解生效**

**3.在需要事务的方法上使用`@Transactional`注解即可**

示例，`spring-dao.xml`：

```xml
<!-- 1、开启事务处理功能，配置事务管理器 -->
<bean id="transactionManager" class=
      "org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <constructor-arg ref="dataSource"/>
</bean>

<!-- 2、开启Spring对注解事务的支持，指定事务管理器id -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

在实现类中的方法上使用`@Transactional`注解：

```java
public class UserMapperImpl implements UserMapper{
    private SqlSessionTemplate sqlSession;

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    //使用注解配置事务，还可以指定参数
    public List<User> queryUser() {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.queryUser();
    }
    ...
}
```

`@Transactional`的**工作原理**是基于AOP实现的，AOP 是使用动态代理实现的。如果目标对象实现了接口，默认情况下会采用 JDK 的动态代理，如果目标对象没有实现了接口，会使用 CGLIB 动态代理。



**`@Transactional`**的作用范围

* **方法** ：推荐将注解使用于方法上，不过**该注解只能应用到 public 方法上，否则不生效。**
* **类** ：如果这个注解使用在类上的话，表明该注解对该类中所有的 public 方法都生效。
* **接口** ：不推荐在接口上使用。因为**在接口上使用时只有基于接口的代理（比如JDK接口代理）才会生效**。因为注解是不能继承 的，这就意味着如果正在使用基于类（比如CGlib）的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装。

> @Inherited使注解能够继承，只能控制对类名上注解是否可以被继承。不能控制方法上的注解是否可以被继承。



**`@Transactional`**的常用参数

* `propagation`：事务的传播行为，默认值为REQUIRED
* `isolation`：事务的隔离级别，默认值采用DEFAULT
* `timeout`：事务的超时时间，默认值为-1，表示不会超时。如果超过改时间限制但事务还没有完成，则自动回滚事务。
* `readOnly`：指定事务是否是只读事务，默认值为false
* `rollbackFor`：用于指定能够触发事务回滚的异常类型，并且可以指定多个异常类型。默认为`RuntimeException.class`，表示在遇到`RuntimeException`异常时才回滚。



**`@Transactional`**的使用注意事项

* `@Transactional` 注解只有作用到 public 方法上事务才生效，不推荐在接口上使用；

* 避免同一个类中调用 `@Transactional` 注解的方法，这样会导致事务失效；

  > 也就是说，同一个类中，方法A调用使用了@Transactional注解的方法B，会导致方法B事务失效。这是由于Spring采用动态代理(AOP)实现对bean的管理和切片，它为我们的每个class生成一个代理对象。只有在代理对象之间进行调用时，可以触发切面逻辑。而在同一个class中，方法A调用方法B，调用的是原对象的方法，而不通过代理对象。所以Spring无法切到这次调用，也就无法通过注解保证事务性了。

* 正确的设置 `@Transactional` 的`rollbackFor`和`propagation`属性，否则事务可能会回滚失败.



## 12.2 事务属性

Spring中事务的常用属性包括以下几种。

### 12.2.1 事务传播行为

事务传播行为是为了解决业务层方法之间互相调用的事务问题。

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。不同的事务导致回滚时的行为也会不同。事务的传播行为就是解决这种问题的。

事务的传播行为共有以下7种：

支持当前事务的情况

* **`TransactionDefinition.PROPAGATION_REQUIRED`**(0)：默认。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
* **`TransactionDefinition.PROPAGATION_SUPPORTS`**(1)：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
* **`TransactionDefinition.PROPAGATION_MANDATORY`**(2)：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

不支持当前事务的情况
* **`TransactionDefinition.PROPAGATION_REQUIRES_NEW`**(3)：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
* **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**(4)：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
* **`TransactionDefinition.PROPAGATION_NEVER`**(5)：以非事务方式运行，如果当前存在事务，则抛出异常。

其他情况
* **`TransactionDefinition.PROPAGATION_NESTED`**(6)：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行（外部主事务回滚的话，子事务也会回滚，而内部子事务可以单独回滚而不影响外部主事务和其他子事务）；如果当前没有事务，则该取值等价于`REQUIRED`。



### 12.2.2 事务隔离级别

**TransactionDefinition 接口中定义了五个表示隔离级别的常量，**除了默认级别以外，其余四种和SQL中的四种隔离级别一一对应：

- **`TransactionDefinition.ISOLATION_DEFAULT`:** 使用后端数据库默认的隔离级别，Mysql 默认采用的 `REPEATABLE_READ`隔离级别，Oracle 默认采用的 `READ_COMMITTED`隔离级别.

- **`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**

- **`TransactionDefinition.ISOLATION_READ_COMMITTED`:** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**

- **`TransactionDefinition.ISOLATION_REPEATABLE_READ`:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**

- **`TransactionDefinition.ISOLATION_SERIALIZABLE`:** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

  

### 12.2.3 事务超时属性

事务超时，指一个事务所允许执行的最长时间。如果超过该时间限制但事务还没有完成，则自动回滚事务。

在 `TransactionDefinition` 中以 int 的值来表示超时时间，其单位是秒，默认值为-1，表示无限制。



### 12.2.4 事务只读属性

`readOnly`为只读属性，对于只有读取数据查询的事务，可以指定事务类型为 readonly，即只读事务。只读事务不涉及数据的修改，数据库会提供一些优化手段，适合用在有多条数据库查询操作的方法中。

MySQL 默认对每一个新建立的连接都启用了`autocommit`模式。在该模式下，每一个发送到 MySQL 服务器的`sql`语句都会在一个单独的事务中进行处理，执行结束后会自动提交事务，并开启一个新的事务。

如果给方法加上了`Transactional`注解的话，这个方法执行的**所有`sql`**会被放在一个事务中。如果声明了只读事务的话，**数据库就会去优化它的执行**，并不会带来其他的什么收益。

如果不加`Transactional`，每条`sql`会开启一个单独的事务，中间被其它事务改了数据，都会实时读取到最新值。会导致一个方法中的多次查询可能出现数据不一致的结果。

比如在一次执行多条查询语句，例如统计查询，报表查询的场景下，多条查询 SQL 必须保证整体的读一致性，否则，如前条 SQL 查询之后，后面的 SQL 查询之前，数据被改变了，则该次整体的统计查询将会出现读数据不一致的状态。



### 12.2.5 事务回滚规则

**事务的回滚规则定义了哪些异常会导致事务回滚而哪些不会。**

**默认情况下**，事务只有遇到运行期异常（`RuntimeException` 的子类）时才会回滚，`Error `也会导致事务回滚，在遇到检查型（`Checked`）异常时不会回滚。

可以自定义特定的异常类型，只需要为`rollbackFor`赋不同的值。



# 十三、其他问题

## 13.1 Spring的启动过程

* 创建`beanFactory`，加载xml配置文件。
* 解析配置文件转化`beanDefination`，获取到bean的所有属性、依赖及初始化用到的各类处理器等。
* 刷新`beanFactory`容器，初始化所有单例bean。
* 注册所有的单例bean并返回可用的容器，一般为扩展的`applicationContext`。  



## 13.2 Spring的BeanFactory和ApplicationContex容器

Bean 工厂(`com.springframework.beans.factory.BeanFactory`)是 Spring框架最核心的接口，它提供了高级IoC的配置机制。BeanFactory使管理不同类型的Java对象成为可能，`com.springframework.context.ApplicationContext`建立在 BeanFactory基础之上，提供了更多面向应用的功能，它提供了国际化支持和框架事件体系，更易于创建实际应用。我们一般称BeanFactory为IoC容器，而称`ApplicationContext`为应用上下文。但有时为了行文方便，我们也将`ApplicationContext`称为Spring容器。



二者的主要区别是，`BeanFactory`是延迟加载，比如说，如果Bean没有完全注入，BeanFacotry加载后，会在第一次调用getBean方法才会抛出异常；而ApplicationContext会在初始化的时候就加载并且检查，这样的好处是可以及时检查依赖是否完全注入。通常来说我们会选择使用ApplicationContext。

对于二者的用途，我们可以进行简单的划分：

* `BcanFactory`是 Spring框架的基础设施，面向Spring本身；
* `ApplicationContext`面向使用 Spring框架的开发者，几乎所有的应用场合都可以直接使用`ApplicationContext`而非底层的 `BeanFactory`。

## 13.3 Spring是如何解决循环依赖的

参考[Spring如何解决循环依赖，源码解析](https://blog.csdn.net/weixin_49592546/article/details/108050566)、[Spring如何解决循环依赖](https://www.zhihu.com/question/438247718)

**循环依赖**：bean对象直接互相依赖，比如A中依赖B，同时B中也依赖A，就构成了循环依赖，在注册Bean时就要考虑先后的问题。

Spring中的循环依赖有三种情况：

* **构造器注入形成的循环依赖。**也就是beanB需要在beanA的构造函数中完成初始化，beanA也需要在beanB的构造函数中完成初始化，这种情况的结果就是两个bean都不能完成初始化，循环依赖难以解决。

* **setter注入构成的循环依赖。**beanA需要在beanB的setter方法中完成初始化，beanB也需要在beanA的setter方法中完成初始化，**Spring设计的机制主要就是解决这种循环依赖**

* **prototype作用域bean的循环依赖。**这种循环依赖同样无法解决，因为spring不会缓存`prototype`作用域的bean，而Spring中循环依赖的解决正是通过缓存来实现的。



**Spring只能解决setter注入构成的依赖，使用三级缓存结构，提前曝光的思想：**

* `singletonObjects`：一级缓存，用于保存实例化、注入、初始化完成的bean实例，即**完全初始化好的bean**，即成品bean对象。从该缓存中取出的bean可以直接使用。

* `earlySingletonObjects`：二级缓存，用于保存实例化完成的bean实例，**尚未填充属性**，即半成品的bean对象。用于解决循环依赖。

* `singletonFactories`：三级缓存，用于**保存bean工厂对象**，实例化半成品bean并放到二级缓存，用于解决循环依赖。

  > 没有循环依赖的情况下，创建好完成品bean之后，才创建对应的代理。但是Spring并不知道有没有循环依赖，因此其选择了不提前创建代理对象，在出现循环依赖被其他对象注入时，才实时生成代理对象。
  >
  > 在对象外面包一层`ObjectFactory`，做到了提前曝光又不生成代理。在被注入时才使用`ObjectFactory.getObject`方法生成代理对象，并将生成好的代理对象放到二级缓存中。

具体步骤：

* 首先 A 完成初始化第一步并将自己提前曝光出来（通过` ObjectFactory` 将自己**提前曝光**，将其包装为`ObjectFactory`对象然后存放到三级缓存），在初始化的时候，发现自己依赖对象 B，此时就会去尝试 get(B)，这个时候发现 B 还没有被创建出来 。

* 然后创建 B，在 B 初始化的时候，同样发现自己依赖A，于是尝试 get(A)，这个时候由于 A 已经被包装为工厂类对象并被添加至缓存（三级缓存 ）中，所以可以通过 `ObjectFactory.getObject()` 方法来拿到 A 对象（A的半成品，同时也是代理对象，然后半成品A会被放到二级缓存），B拿到 A 对象后顺利完成初始化，然后将自己添加到一级缓存中。

* 回到 A，A 也可以拿到 B 对象，完成初始化，到这里整个链路就已经完成了初始化过程了。



> Spring是通过递归的方式获取目标bean及其所依赖的bean的
>
> Spring实例化一个bean的时候，是分两步进行的，首先实例化目标bean，然后为其注入属性。



另外，为什么需要三级缓存，而不是二级缓存，也就是bean构造完成后就生成代理对象？

因为没有出现循环依赖的情况下，Spring的初衷就是在Bean生命周期的最后一步完成代理，而不是实例化之后马上生成代理对象。

二级缓存便于存放半成品bean对象，二级缓存和三级缓存就是为了解决循环依赖问题才设置的，**三级缓存的实现提供了提前生成代理的口子，而不是直接生成代理，只有发生循环依赖执行getObject才会执行代理**，从而达到了**只有循环依赖发生时，才提前代理，而没有循环依赖，则代理方式不变，依然是初始化以后代理**的目的。[Spring循环依赖及三级缓存](https://blog.csdn.net/u012098021/article/details/107352463/)



## 13.4 Spring框架中用到了哪些设计模式

参考[Spring中都用到了那些设计模式?](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&token=255050878&lang=zh_CN#rd)

* **工厂模式**：比如Spring使用BeanFactory、ApplicationContext创建bean对象。
* **代理模式**：AOP的核心就是代理模式
* **单例模式**：Spring中的Bean默认就是单例模式
* **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
* **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
* **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
* **适配器模式** ：Adapter Pattern，将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。Spring AOP 的增强或通知(Advice)使用到了适配器模式，Spring MVC 中也是用到了适配器模式适配`Controller`。



---

参考：[JavaGuide-Spring常见问题总结](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/Spring%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93)

