---
title: SpringBoot2配置与使用
excerpt: 使用SprintBoot创建Web应用；使用SprintBoot整合MyBatis；
mathjax: true
date: 2021-07-10 17:14:03
tags: ['SpringBoot2','Spring']
categories: SpringBoot2
keywords: Spring,SpringBoot,SpringBoot2
---



# 一、SpringBoot介绍

## 1.1 SpringBoot

### 1.1.1 什么是SpringBoot

官方文档：[Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/index.html)

[SpringBoot介绍](https://snailclimb.gitee.io/springboot-guide/#/./docs/start/springboot-introduction)

Spring框架旨在简化J2EE企业应用程序的开发，而SpringBoot旨在简化Spring开发，使用SpringBoot能够快速创建出生产级别的Spring应用。

使用Spring框架虽然代码是轻量级的，但是其仍然需要大量的XML配置。SpringBoot解决了Spring项目配置文件繁多的问题。

SpringBoot是整合Spring技术栈的一站式框架。

SpringBoot是简化Spring技术栈的快速开发脚手架。



### 1.1.2 SpringBoot的优缺点

**优点**：

- 创建独立Spring应用

- Spring Boot 应用程序提供嵌入式 HTTP 服务器，如 Tomcat 和 Jetty，可以轻松地开发和测试 web 应用程序。

- 自动starter依赖，简化构建配置

- 自动配置Spring以及第三方功能

- 提供生产级别的监控、健康检查及外部化配置

- Spring Boot 不需要编写大量样板代码、XML 配置和注释。

  

**缺点**：

* 迭代快，需要时刻关注变化

- 封装太深，内部原理复杂，不容易精通



## 1.2 SpringBoot2

基于Java8的新特性，比如接口中允许定义默认方法（default方法不强制实现），Spring5的源码架构也重新设计，因此SpringBoot2相比于1版本也有很多升级，比如响应式编程。



SpringBoot2的系统要求，以2.5.2版本为例

* Maven 3.5+
* Tomcat 9.0，servlet3.1+
* JDK8+



# 二、SpringBoot2 快速入门

## 2.1 创建工程

可以在官网的https://start.spring.io/快速生成一个SpringBoot项目，生成以后下载到本地即可。

也可以使用IDEA自带的`Spring Initializr`创建，**`File->New->Project->Spring Initializr`**，配置好项目名之后，可以根据需求选择依赖，比如我们选择Spring Web依赖，这样创建的项目会自动在配置文件中导入选中的依赖。

![选择项目依赖](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot2-basis_1.png)



## 2.2 编写业务代码

创建完成后，`Spring Initializr`会自动为我们导入启动器依赖并生成目录结构：

![初始项目目录结构](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot2-basis_2.png)

其中templates文件存放JSP页面、thymeleaf模版等；static文件夹存放图片、css文件、js文件、欢迎页、网站图标等静态资源。



直接运行启动类`DemoApplication`就能运行项目：

`DemoApplicationTests`

```java
//注明这是一个SpringBoot应用
@SpringBootApplication
public class DemoApplicationTests {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplicationTests.class, args);
    }
}
```



我们可以新建一个controller进行测试，比如：

`com.example.demo.controller.HelloController.java`

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String handle01(){
        return "Hello, Spring Boot 2!";
    }
}
```

这样，在浏览器中输入hello请求，就会收到指定的返回值。一个web项目的demo就算搭建成功。

需要特别注意的是，**SpringBoot默认会自动扫描主程序所在的包及其下面所有的子包中的组件**，因此，我们需要将程序代码写在主程序所在的包及其子包内。

> 也可以改变扫描路径，比如@SpringBootApplication(scanBasePackages="com.kang")表示将扫描的包路径更改为com.kang



## 2.3 简化配置和部署

**简化配置**

SpringBoot默认的主配置文件是`application`可以是`properties`类型，也可以是`yaml`类型，可以同时使用多个配置文件，生效顺序根据情况判断。

对于SpringBoot简化配置的优势，主要体现在其所有的配置都可以只写在一个配置文件中，即主配置文件`application.properties`中，配置tomcat的端口号：

```properties
server.port=8888
```

SpringBoot中，用到的所有配置都可以只在这一个配置文件中修改,包括tomcat参数、mybatis、springmvc配置等。 SpringBoot为我们配置了一些默认值，可以根据需求修改。



**简化部署**

SpringBoot的简化部署主要体现在它为我们提供了maven插件，这个插件能够在maven打包时递归地将所有jar包和项目文件都打包进去。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```



# 三、SpringBoot配置

## 3.1 SpringBoot特点

### 3.1.1 依赖管理

**依赖管理**

SpringBoot中是父项目进行依赖管理，所有的SpringBoot项目都会引入这个父项目，这个项目主要是用于资源过滤和插件管理：

```xml
<!-- spring-boot-starter-parent是每个SpringBoot项目都要引入的父项目 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

这个父项目的又有一个父项目：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.5.2</version>
</parent>
```

这个`spring-boot-dependencies项目是真正管理SpringBoot应用里面所有依赖版本的地方，即SpringBoot的版本控制中心。其中**自动配置了常用的jar包的版本号（自动版本仲裁机制）**，因此**我们如果显式导入某个依赖，不需要手动写版本号**，除非SpringBoot依赖配置中没有这个包。

如果我们对于自动仲裁的版本号不满意，也可以手动指定版本号。首先需要查看`spring-boot-dependencies`里面规定依赖的版本用的 key，然后在当前项目里面重写配置。

比如要修改MySQL依赖的版本号，我们从底层依赖中查到定义其版本号的key为`mysql.version`，我们就可以在pom文件中手动修改其版本号：

```xml
<properties>
    <mysql.version>5.1.43</mysql.version>
</properties>
```





**启动器（starter）**

SpringBoot将所有的功能场景都抽取出来，做成一个个的**starter （启动器）**，就是在Maven的`pom.xml`配置文件中，类似`xxx-starter`的依赖，比如：

```xml
<!-- web场景启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

这种`starter`就称为xx的场景启动器：

* `spring-boot-starter-xxx` ： 官方提供的xxx场景启动器 。

*   `xxx-spring-boot-starter`： 第三方提供的简化开发的场景启动器。

* **只要引入starter，这个场景需要的所有常规依赖都自动引入 **

* 所有场景启动器最底层的依赖都是`spring-boot-starter`：

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.5.2</version>
  </dependency>
  ```

SpringBoot官方提供的所有场景，参考[Starters](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter) 

我们只需要在项目中引入这些starter即可，所有相关的依赖都会导入进来 ， 我们要用什么功能就导入什么样的场景启动器即可 ，也可以根据需求自定义自己的 starter。



### 3.1.2 自动配置

**自动配置依赖**

当我们引入某一个场景启动器的时候，SpringBoot会自动帮我们引入其需要的依赖，这是底层启动器自动引入的。

比如，我们引入web启动器，底层会自动帮我们引入Tomcat依赖、SpringMVC常用组件（比如视图解析器）、字符编码问题等。

**默认的包结构**

SpringBoot项目中，**主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来**，无需以前的包扫描配置。

如果想要改变扫描路径，使用`SpringBootApplication`的`scanBasePackages`属性，比如将扫描包的范围扩大为`com.example`：

```java
@SpringBootApplication(scanBasePackages="com.eaxmple")
```

或者单独使用`@ComponentScan `指定扫描路径。



## 3.2 主启动类

SpringBoot项目有一个主启动类，使用`@SpringBootApplication`标注：

```java
//表明这是一个Spring Boot应用
@SpringBootApplication
public class SpringbootApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

`@SpringBootApplication`是SpringBoot的基石，其标注在主程序类上，根据源码可知，其等价于下面三个注解：

* `@SpringBootConfiguration`：等价于`@Configuration`，表明这是一个SpringBoot项目的配置类。
* `@EnableAutoConfiguration`：开启自动配置功能。
* `@ComponentScan`：自动扫描并加载**给定的路径中**符合条件的组件，注入到IOC容器中。



## 3.3 容器功能

SpringBoot同样可以使用Spring的容器功能，下面介绍配置文件涉及到的常用注解。

### 3.3.1 组件添加

**1、`@Configuration`**

这个注解表示将当前类注册为配置类，可以代替Spring的配置文件进行配置，比如注册组件。

在配置类中，使用`@Bean`在方法上使用用于注册组件。

> 一种常用的做法是，在配置类的方法中使用@Bean，注册第三方bean。因为第三方组件只能通过这种方式注入。

`@Configuration`有两种模式：

* **Full模式**：`proxyBeanMethods = true`，保证每个@Bean方法被调用多少次返回的组件都是单实例的。默认情况就是Full模式。
* **Lite模式**：`proxyBeanMethods = false`，每次调用@Bean标注的方法方法，返回的组件都是新创建的，即每次获取的组件都是不同的。

> 组件依赖必须使用Full模式。
>
> Full模式每次都要检查容器中是否存在当前组件，因此效率较低。如果非必要情况下，建议使用Lite模式以提高效率。



**2、`@Bean`、`@Component`、`@Controller`、`@Service`、`@Repository`**

这几个注解的作用类似，都是注册组件：

* `@Bean`：**用在配置类的方法上**，方法的返回类型就是要注册组件。**如果注册第三方库中的类到IOC容器中，只能使用配置类+@Bean注解的方式**。
* `@Component`：用在类上，表示将当前类注册为组件。
* `@Controller`：@Component的衍生注解，功能类似，用在controller层
* `@Service`：@Component的衍生注解，功能类似，用在service层
* `@Repository`：@Component的衍生注解，功能类似，用在dao层



**3、`@ComponenScan`、`@Import`**

这两个注解也是用在配置类中。

`@ComponentScan`：自动扫描并加载**给定的路径中**符合条件的组件，注入到IOC容器中。

`@Import`：可以导入一个或多个组件，比如Bean或者配置类。

```java
//给容器中自动创建出这两个类型的组件，默认组件的名字就是全类名
@Import({User.class, DBHelper.class})
```

可以参考：[配置类注解](https://kangshitao.github.io/2021/06/27/spring-basis/#%E5%85%AB%E3%80%81%E5%9F%BA%E4%BA%8EJava%E7%9A%84%E5%AE%B9%E5%99%A8%E9%85%8D%E7%BD%AE)



**4、`@Conditional`**
`@Conditional`表示条件装配，只有满足一定条件，才会被注册。可以用在配置类上，也可以用在配置类的方法上。



as such, it is strongly recommended to use this condition on auto-configuration classes only. If a candidate bean may be created by another auto-configuration, make sure that the one using this condition runs after.

其衍生注解如下图：

![@Conditional及其衍生注解](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot2-basis_3.png)



比如在配置类上使用`@ConditionalOnMissingBean(name = "tom")`表示当容器中没有name为`tom`的组件时，配置类中的组件才会生效。



**关于Conditonal注解有时候不起作用的问题的讨论**

* 条件注解的条件是**根据当前已经加载的内容来判断**是否符合。

* Condition相关的处理是在**包扫描**的时候执行的，所以条件中的**判断前后顺序只跟包扫描的顺序有关**，而包扫描的顺序是根据包名和类名的字符排序，所以**配置类的解析无法保证bean注册的先后顺序**，如果在同一个配置类不同的@Bean注解的方法上使用条件注解可能会出现失效的情况。

  > 通过component-scan扫描的方式加载bean，在扫描范围内按照class的命名顺序加载。
  >
  > 在spring的xml文件中，bean的加载顺序按照书写顺序加载，如果不同组件之间有依赖关系，则先创建被依赖的组件。

* 对于`@ConfitionalOnBean`和`@ConditionalOnMissingBean`，官方推荐只在自动配置类中使用，因为自动配置类会在用户定义的bean被添加到容器之后才加载。



### 3.3.2 原生配置文件引入

使用`@ImportResource`注解，可以在配置类中引入一个或多个`.xml`配置文件，允许以配置文件的方式对组件进行注册。主要是为了将第三方的配置文件中的组件注入容器。

> @ImportResource引入的是资源文件；@Import引入的是组件。

`beans.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd 
                           http://www.springframework.org/schema/context 
                           https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="haha" class="com.atguigu.boot.bean.User">
        <property name="name" value="zhangsan"></property>
        <property name="age" value="18"></property>
    </bean>

    <bean id="hehe" class="com.atguigu.boot.bean.Pet">
        <property name="name" value="tomcat"></property>
    </bean>
</beans>
```



`MyConfig.java`

```java
@ImportResource("classpath:beans.xml")
@Configuration
public class MyConfig {}
```



可以在主程序中的IOC容器中，测试是否注入成功：

```java
public class Springboot01HelloworldApplication {
    public static void main(String[] args) {
        //可以根据返回的配置应用上下文对象，获取容器配置
        //返回IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(Springboot01HelloworldApplication.class, args);
        boolean haha = run.containsBean("haha");
        boolean hehe = run.containsBean("hehe");
        System.out.println("haha："+haha);//true
        System.out.println("hehe："+hehe);//true
    }
}
```





### 3.3.3 配置绑定

配置绑定，指的是将配置文件中的内容，封装到JavaBean中，以供随时使用。

使用原生的Java代码，解析配置文件比较麻烦，在SpringBoot中，我们可以使用注解自动将配置文件中的值绑定到组件中的属性上。

**1、@Value**

对于简单的属性注入，可以使用`@Value`注解。可以绑定配置文件中指定的属性到当前的属性。

`@Value`有三种赋值方式：

* 直接赋值，比如：`@Value("name")`
* 绑定配置文件中的属性，比如：`@Value("${person.name}")`，绑定配置文件中的`person.name`到当前属性。还可以指定默认值。
* 使用SpEL表达式赋值，比如：`@Value("#{'姓名'}")`，参考：[SpEL](https://blog.csdn.net/swordcenter/article/details/75239878)

```java
@Controller
public class HelloController {
    //使用@Value注解绑定主配置文件中的属性person.name
    //如果值为空，则指定一个默认值
    @Value("${person.name:default name}")
    private String name;
}
```



**2、@Component + @ConfigurationProperties**

如果一个类想要绑定配置文件，需要使用` @ConfigurationProperties`注解，用于指定和核心配置文件中什么属性绑定。

绑定配置文件的前提当前类必须是容器中的组件，因此可以结合使用`@Component`注解：

```java
//只有在容器中的组件，才会拥有SpringBoot提供的强大功能
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer price;
    ...//get、set方法等
}
```

在` @ConfigurationProperties`注解中使用`prefix`属性，指定要绑定的配置文件中的前缀，比如配置文件中的`mycar.price=10000`，当前组件中的`price`属性值就会被注入10000.



**3、@EnableConfigurationProperties 和 @ConfigurationProperties**

组件绑定配置文件的第二种方式是使用`@EnableConfigurationProperties` 和`@ConfigurationProperties`两个注解，其中`@EnableConfigurationProperties` 注解用在配置类中，有两个功能：开启指定类的配置绑定功能；将这个类注入到IOC容器。

比如：

`Car.java`

```java
//这里只需要指定要绑定的配置类中的属性
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer price;
    ...//get、set方法等
}
```

`MyConfig.java`

```java
//配置类
@Configuration  
//开启Car类的属性配置功能，并把Car注册到容器中
@EnableConfigurationProperties(Car.class)  
public class MyConfig {}
```



因为`@Component`不能用在第三方包中，所以如果`@ConfigurationProperties`在第三方包中，我们想对其进行配置，就必须使用`@EnableConfigurationProperties`这种方式。

**源码中的用法**

SpringBoot的底层源码中，常见的一种做法是在自动配置类中使用`@EnableConfigurationProperties`，并指定一个`xxxProperties`用于和配置文件中的值进行绑定。

比如：在`CacheAutoConfiguration.java`中

```java
@EnableConfigurationProperties(CacheProperties.class)
public class CacheAutoConfiguration {...}
```

`CacheProperties.java`绑定了配置文件:

```java
@ConfigurationProperties(prefix = "spring.cache")
public class CacheProperties {...}
```

这样，我们就能够在配置文件中使用`spring.cache`对cache相关属性进行配置。



## 3.4 自动配置

SpringBoot默认会在底层配置好所有的组件，自动装配所有能生效的类。我们可以在总配置文件`application.properties`中对需要修改的属性进行配置。

用户自己配置的参数，会以用户的优先。

自动配置的大概流程总结：

- SpringBoot先读取所有的自动配置类：`xxxxxAutoConfiguration`
- 每个自动配置类按照条件生效，默认都会绑定`xxxxProperties`类，这个类里面设置了默认的值。
- `xxxProperties`类和配置文件进行了绑定，可以通过配置文件对默认值进行设置。

- 生效的配置类就会给容器中装配很多组件。只要容器中有这些组件，相当于这些功能就有了

- 用户可以使用`@Bean`替换底层的组件。



可以通过底层源码，查看这个组件是获取的配置文件什么值。也可以通过官方文档查看如何进行配置。

**xxxxxAutoConfiguration ---> 组件  --->从 xxxxProperties里面取值  --->绑定 application.properties**



## 3.5 最佳实践

1、引入场景依赖，可以查看官方文档有哪些官方的场景启动器：[Starters](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)

2、如果有需求，可以查看自动配置了哪些功能，有两种方式：

* 通过源码分析
* 在配置文件中使用`debug=true`开启自动配置报告，Negative表示不生效，Positive表示生效。

3、判断配置参数是否需要修改：

* 参照官方文档，查看如何配置：[Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties)
* 或者分析源码，查看如何配置。

4、根据需求自定义或者替换组件。使用`@Component`或者`@Bean`注解。

5、也可以使用自定义器：xxxCustomizer



