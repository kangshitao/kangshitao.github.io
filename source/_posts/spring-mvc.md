---
title: SpringMVC总结
excerpt: SpringMVC工作原理，Controller，拦截器，RestFul，JSON，Ajax
mathjax: true
date: 2021-07-03 16:15:45
tags: ['SSM','SprinMVC']
categories: SSM框架
keywords: SpringMVC,Controller,view,JSON,Ajax
---



# 一、SpringMVC介绍

## 1.1 MVC



传统的三层架构：

* View：视图层，用于接收用户提交请求代码。

* Service：服务层，主要是系统的业务逻辑代码。

* Dao：持久层，直接操作数据库的代码。

  

MVC，即

* View：视图，主要负责显示数据和提交数据。比如JSP，HTML页面
* Model：模型，承载数据，并对用户提交请求进行计算的模块。包括数据承载（**bean**）和业务处理（**service**、**dao**）两部分。
* Controller：控制器，用于将用户请求转发给相应的Model进行处理，并处理Model的计算结果向用户提供响应。 

> MVC也演化出了其他的模式，比如MVP、MVVM等。

MVC架构的工作流程大致如下：

* 用户通过View页面向服务端发起请求。
* 服务端Controller控制器接受请求并进行解析，找到相应的Model对用户请求进行处理。
* Model处理后，将处理结果交给Controller。
* Controller接到结果以后发给View，将渲染后的页面响应给用户。



没有SpringMVC之前的web应用主要经历了两个阶段：

* **Model1时代**：主要有视图层和模型层，JSP将控制逻辑和表现逻辑混杂在一起，职责过重，不便于维护。
* **Model2时代**：项目分为模型（bean，dao)、视图（JSP）、控制（Servlet）三部分，但这种模式的抽象和封装程度远远不够，因此有了Struts、SpringMVC等MVC框架。



## 1.2 SpringMVC

[Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc)是Spring Framework的一部分，是基于Java实现MVC的轻量级Web框架。

SpringMVC功能强大，支持RESTful、数据验证、本地化、国际化、拦截器等功能。

SpringMVC中的`DispatcherServlet`用于控制所有请求，将请求分发到不同的处理器，而传统的项目中每个servlet只能处理一个请求。



# 二、SpringMVC的工作原理



SpringMVC主要是以`DispatcherServlet`（前端控制器、请求分发器）为核心。这个类继承于`HttpServlet`类，本质上是一个`servlet`。

SpringMVC的主要工作流程如下：

![SpringMVC工作原理](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/sprint-mvc_1.png)



1、用户发送请求，请求会被 `DispatcherServlet`接收。

2、`DispatcherServlet` 根据请求信息，调用 `HandlerMapping`来解析请求对应的 `Handler`。

3、解析到对应的 `Handler`（也叫做 `Controller` 控制器）后，交给由 `HandlerAdapter` 适配器处理。

4、`HandlerAdapter` 会根据 `Handler `来调用真正的处理器来处理请求，并处理相应的业务逻辑。

5、处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的视图（字符串表示的视图逻辑名，比如JSP页面名）。

> `ModelAndView`对象包括了`ModelMap`和`view`两个属性，`ModelMap`用于保存数据，比如请求查询得到的结果对象，`view`用于保存逻辑视图名。

6、`DispatcherServlet` 将逻辑`View`传递给`ViewResolver` 视图解析器，它会根据 逻辑`View` 查找实际的 `View`(即根据名称找到真正的页面)并创建视图，返回给`DispatcherServlet` 。

> 视图解析器根据设置好的前缀路径和后缀名，进行拼接，确定视图的位置，然后创建`View`接口的实现类对象，也就是视图对象，将其返回给请求分发器。

7、`DispaterServlet` 把视图解析器返回的 `View` 进行视图渲染，并将模型数据填充到`request`域。

> 视图就是html、jsp等页面。在底层源码中视图渲染会将`Model`里面的键值对数据全部写到`requestScope`中（前端可以通过EL表达式从域对象中获取数据），然后进行**请求转发**到指定的页面，最后将这个页面相应给用户。
>
> **也就是说，`ModelAndView`中的数据对象会被放在请求域中。**

8、最后，`DispaterServlet` 向用户响应结果。

> 将响应数据放到`Response`中返回给用户。



# 三、SpringMVC的使用

使用SpringMVC创建web项目，需要一个`springmvc-servlet.xml`配置文件用来配置处理器映射器、处理器适配器和视图解析器。然后在`web.xml`文件中配置请求分发器。

使用Maven框架，可以使用IDEA的Web模版创建web项目；

也可以先创建普通Maven模块，然后右键`Add Framework Support`，添加Web框架支持。这种方式创建web项目，需要在`Project Structure`->`Artifacts`中，在`WEB-INF`目录中创建`lib`文件夹，然后将用到的所有jar包添加进去。如下图：

![在项目发布目录手动添加jar包](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/spring-mvc_3.png)



创建完项目以后，基本的目录结构如下：

![SpringMVC项目的基础目录](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/sprint-mvc_2.png)



`DispatcherServlet`是SpringMVC的核心，其实现方式有两种，分别是**实现Controller接口**和**使用`@Controller`注解**。

无论是哪种方式，`web.xml`配置文件都是一样的，在其中需要配置请求分发器（前端控制器）。

`web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- 配置dispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 通过初始化参数，指定关联的Spring的xml文件位置 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <!-- 设置启动级别 -->
        <load-on-startup>1</load-on-startup>

    </servlet>
    <!-- 设置匹配的请求 -->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

其中的配置说明：

* `contextConfigLocation`表示装入配置文件，这个路径应该是Spring的总配置文件。如果不指定，默认加载`/WEB-INF/applicationContext.xml`，如果想自定义名称，则需要指定路径。
* `load-on-startup`：用于标记容器是否应该在web应用程序启动的时候就加载这个servlet（实例化并调用init()方法）。值必须是整数，表示servlet被加载的顺序。如果值为负数或者没有设置，则当这个servlet被请求时再加载。如果值为0或正整数，表示容器启动时就加载这个servlet，数值越小优先级越高，数值相同时按照声明顺序加载。

* `/`：匹配所有的请求，但不会匹配JSP、HTML等静态资源。

* ` /*`:匹配所有的请求，包括静态资源。这里不能使用`/*`，因为如果把JSP页面当作请求的话，如果视图解析器的后缀也是JSP，会陷入无限请求中导致请求失败。



## 3.1 方式一：实现Controller接口

其中实现接口的方式需要在Spring配置文件中注册bean，为了能够使映射器根据名字找到对应的`Handler`。

**1、配置`springmvc-servlet.xml`文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 映射器和适配器都是默认的，不注册也可以正常使用，这里为了流程清晰将其显式写出来 -->
    <!-- 配置处理器映射器 -->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
    
    <!-- 配置处理器适配器 -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!-- 配置视图解析器（必须），根据接收的视图名找到视图的真实位置并返回视图 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          id="internalResourceViewResolver">
        <!-- 前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!-- 后缀 -->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- BeanNameUrlHandlerMapping映射器会根据bean名字，找到对应的handler -->
    <!-- 注册 Handler，当请求为test的时候，就会执行HelloController -->
    <bean id="/test" class="com.kang.controller.HelloController"/>
</beans>
```

**说明**：

* `Controller`的`id`必须加`/`
* 视图解析器根据视图名，加上前缀和后缀定位视图。
* 这里的映射器和适配器都是默认的，不注册也可以正常使用，这里为了便于理解，将其显式写出来。



**2、创建控制器类，实现`Controller`接口**，在`java/com/kang/controller`目录下创建控制器类。

`HelloController`

```java
package com.kang.controller;

public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest,
                                      HttpServletResponse httpServletResponse) 
        throws Exception {
        ModelAndView mv = new ModelAndView();
        //Controller，需要返回一个ModelAndView对象
        //1、编写业务代码
        String result = "HelloSpringMVC";
        mv.addObject("msg",result); //将结果写到msg属性中
        //2、视图跳转，指定要跳转的jsp页面名字
        mv.setViewName("test"); 
        return mv;
    }
}
```



`mv.addObject("msg",result);`表示添加数据到`ModelAndView`对象，这个数据最终会被请求转发器添加到`requestScope`中。在前端页面就可以使用EL表达式从`requstScope`中取到数据：`${requestScope.msg}`

`mv.setViewName("test");`用于指定要请求转发的视图名字。

这个`ModelAndView`对象返回给请求分发器，然后经过视图解析器解析、视图渲染等步骤，最终将带有数据的视图写入`response`中响应给用户。



## 3.2 方式二：使用@Controller注解

使用注解不需要显式注册bean，而是在类中使用`@Controller`注解，自动将当前类注册为控制器类。

> `@Controller`效果和`@Component`注解类似，都表示将当前类交给Spring托管。
>
> 与之类似的还有`@Service`用于service层，`@Repository`用于dao层。



被`@Controller`注解的类中的所有方法（公有私有都包括），如果返回值是`String`，并且有具体的页面与之对应，就会被视图解析器解析。

> 如果想要方法的返回值不被视图解析器解析，可以使用在方法上使用`@ResponseBody`注解，表示当前方法的返回值直接写入到HTTP响应（response）体中，一般返回的是JSON或XML形式的数据，是前后端分离情况下常用的情况。详情参见下文JSON章节



使用注解时，配置文件和控制类的内容和方式一不同。

**1、配置`spring-mvc.xml`文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           https://www.springframework.org/schema/context/spring
                           context.xsd http://www.springframework.org/schema/mvc
                           https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.kuang.controller"/>
    <mvc:default-servlet-handler/>
    <mvc:annotation-driven/>

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          id="internalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```



**参数说明**：

`<context:component-scan>`

```
表示自动扫描包，让指定包下的注解生效，由IOC容器统一管理 
```



`<mvc:default-servlet-handler/>`

```
用于过滤静态资源，让SpringMVC不处理静态资源，比如.txt、.mp3、.jpg等。
配置这个处理器后，会在Spring MVC上下文中定义一个org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，
它像一个检查员，对进入DispatcherServlet的URL进行筛查，
如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理；
如果不是静态资源的请求，才由DispatcherServlet继续处理。
```



`<mvc:annotation-driven/>`

```
用于配置注解驱动。
一方面：
在spring中一般采用@RequestMapping注解来完成映射关系
要想使@RequestMapping注解生效，
必须向上下文中注册DefaultAnnotationHandlerMapping
和一个AnnotationMethodHandlerAdapter实例，
这两个实例分别在类级别和方法级别处理。
annotation-driven配置帮助我们自动完成上述两个实例的注入。

另一方面：
基础bean只能提供最基础的服务，其它的扩展功能，比如JSON、XML、Valid等等，
根据classpath有没有相关的依赖来决定要不要添加对应的bean或者属性，
annotation-driven的作用就是提供扩展功能。
```



**2、创建Controller类**

```java
@Controller
public class HelloController {
    @RequestMapping("/hello1")
    public String myController(Model model){
        model.addAttribute("msg","hello,MVC-Annocation");
        return "hello"; 
    }
}
```

我们自定义方法的返回值是视图名，相当于在Spring的XML文件中配置了`id`为`hello`的bean，用于映射器查找到对应的控制器。在方法中可以使用`Model`对象、`ModelMap`对象、`ModelAndView`对象三种方式向**请求域**中传递数据，也就是向前端页面传递数据，具体使用方法见下文。

**注解说明：**

`@Controller`：将当前类注册为Controller，类似于实现Controller接口。

`@RequestMapping`：表示映射`hello`请求到这个方法，也就是说当请求为`hello`时，会执行这个被其注解的方法。也可以用在类中，表示当前类中的每个方法的共同请求前缀，比如下面的案例，只有当请求为`/test/hello`时才会执行`myController`方法。

```java
//表示类中每个方法的请求前都要加路径/test
@RequestMapping("/test")
public class HelloController {
    @RequestMapping("/hello")
    public String myController(){}
}
```

`@RequestMapping`默认为任何请求方式，也可以在参数中指定请求方式，比如设置为`POST`请求方式：

```java
@RequestMapping(value = "/hello",method = RequestMethod.POST)
```



HTTP的几种请求方式参考[HTTP请求方法](https://kangshitao.github.io/2021/05/19/computer-network-review/#HTTP%E6%96%B9%E6%B3%95)



对于具体的请求方式，还可以使用其对于的注解，比如`@GetMapping`、`@PostMapping`

`@PutMapping`、`@DeleteMapping`等，其作用和`@RequestMapping`相同，不过是固定了具体的请求方式。



# 四、SpringMVC的结果跳转

使用`@Controller`方式的控制器，页面跳转可以使用视图解析器，也可以不使用视图解析器。



**1.使用视图解析器**

使用视图解析器的时候，只需要返回视图名字即可，或者返回带有视图名字属性的`ModelAndView`对象。

```java
//方式一：直接返回视图名给视图解析器
@Controller
public class HelloController implements Controller {
    @RequestMapping("/hello")
    public String myController(){
        //1.做一些数据处理
        //2.返回视图名
        return "test";
    }
}
//方式二：使用ModelAndView设置视图名
@Controller
public class HelloController implements Controller {
    @RequestMapping("/hello")
    public ModelAndView myController(){
        ModelAndView mv = new ModelAndView();
        //1.做一些数据处理
        //2.设置要跳转的视图名，并返回此对象
        mv.setViewName("test");
        return mv;
    }
}
```



以上两种方式都是使用视图解析器时的写法，都是将视图名传递给视图解析器。前提是SpringMVC的配置文件中注册了视图解析器。



**2.直接返回**

如果不使用视图解析器，也可以直接将视图返回。由于没有视图解析器的路径拼接，所以方法的返回值应该是真实不需要拼接的视图地址。

如果不指定前缀，默认表示请求转发。

> 不指定forward或redirect，则必须保证视图解析器关闭或注释掉，否则视图解析器仍会拼接结果。

如果指定了前缀，则会按照指定的方式转发或重定向，并且不会经过视图解析器（即使开启了视图解析器）。

```java
@Controller
public class ModelTest {
    @RequestMapping("/test2")
    public String  test2(){
        //return "/WEB-INF/jsp/hello.jsp";  //请求转发
        //return "forward:/WEB-INF/jsp/hello.jsp"; //请求转发
        return "redirect:/index.jsp"; //重定向,重定向的页面需要确保能够通过url访问
    }
}
```

这里要注意的是，如果是重定向，因为重定向是客户端重新发起请求，因此必须保证此页面可以通过URL直接访问。



**3.使用ServletAPI**

使用`HttpServletRequest`进行请求转发或者`HttpServletResponse`进行重定向。不需要视图解析器.

```java
@Controller
public class ModelTest1 {
    @RequestMapping("/test3")
    public void test3(HttpServletRequest request, 
                     HttpServletResponse response) 
        throws ServletException, IOException {
        request.setAttribute("msg","ModelTest1");
        request.getRequestDispatcher("/WEB-INF/jsp/hello.jsp").
            forward(request,response);
    }
}
```

可以使用`ServletAPI`进行手动请求转发或者重定向，同样不需要视图解析器。



# 五、SpringMVC中的数据处理



SpringMVC中的数据处理主要是两方面，一个是后端如何接收前端传来的数据，第二个是后端如何向前端传递数据。

> 在前端页面中，我们可以使用EL表达式从域对象中取出数据；通过表单等方式提交数据。



## 5.1 控制器接收数据

控制器对于前端传来的数据可以直接通过参数进行接收。可以分为以下三种情况。

以下案例以GET请求方式为例

**1、传递的参数名和方法的参数名一致**

url：`http://localhost:8080/SpringMVC_04/user/t1?name=spring`

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @PostMapping("/t1")
    public String acceptData(String name, Model model){
        //1.接收前端参数
        System.out.println(name);
        //2.做一些处理
        model.addAttribute("msg",name);
        //3.视图跳转
        return "hello";
    }
}
```

可以看到，请求的参数名称和方法的参数名称能够对应起来，因此方法中的`name`参数可以接收来自请求的数据，然后方法就可以对数据做一些操作。



**2、传递的参数名和方法的参数名不一致**

当传递的参数名和方法的参数名不一致的时候，需要使用`@RequestParam`注解指定对应关系。

url：`http://localhost:8080/SpringMVC_04/user/t1?username=spring`

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @GetMapping("/t1")
    public String acceptData(@RequestParam("username")String name, Model model){
        //1.接收前端参数
        System.out.println(name);
        //2.将返回的结果传递给前端
        model.addAttribute("msg",name);
        //3.视图跳转
        return "hello";
    }
}
```

`@RequestParam`是一个参数注解，用到参数前，表示将请求的参数名和方法参数名对应起来，注解中的值为请求的参数名。

比如上述代码的请求参数名是`username`，则注解中的值也为`username`，表示`name`这个参数将接收来自客户端请求中的`username`参数值。



**3、传递的是一个对象**

如果提交的表单是一个复杂的对象，则方法参数使用对象类型即可。

比如现在有一个`User`类定义如下：

```java
public class User {
    private int id;
    private String name;
    private int age;
}
```

url：`http://localhost:8080/SpringMVC_04/user/t1?name=kang&id=1&age=20`

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @GetMapping("/t2")
    public String test2(User user, Model model){
        //1.接收前端参数
        System.out.println(user);
        //2.处理数据
        model.addAttribute("msg","accept user");
        //3.视图跳转
        return "hello";
    }
```



这种方法需要保证请求参数的名称和对象属性名一致，如果名称不一致，则属性值为null。



## 5.2 控制器传递数据

前面提到过，控制器有三种方式向**请求域**中传递数据，即向前端页面传递数据。

这一过程对应于SpringMVC工作原理图中的第6、7步，即控制器返回`ModelAndView`。

### 1、使用Model

`Model`是一个接口。能够满足大部分传递数据的需求。比较简单，只适合用于存储数据。

使用`Model`传递数据：

```java
@Controller
public class MyController {
    @RequestMapping("/hello")
    public String test(Model model){
        //调用addAttribute方法写入数据
        model.addAttribute("msg","hello,SpringMVC");
        return "hello"; 
    }
}
```

需要注意的是，这种方法需要将`Model`写在参数中，使用时可以调用`addAttribute()`方法写入数据。



### 2、使用ModelMap

`ModelMap`是一个类，继承了`LinkedHashMap`，其除了拥有和`Model`相同的功能以外，还具有`LinkedHashMap`的所有功能。

使用`ModelMap`传递数据的用法和`Model`相同：

```java
@Controller
public class MyController {
    @RequestMapping("/hello")
    public String test(ModelMap modelMap){
        modelMap.addAttribute("msg","hello,SpringMVC");
        return "hello"; 
    }
}
```



注意：`ModelMap`和`Model`不是实现和被实现的关系，`ModelMap`可以看作一个增强版的`Model`。

`ModelMap`类的源码声明如下：

```java
public class ModelMap extends LinkedHashMap<String, Object> {
    public ModelMap() {}
    ...
}
```



### 3、使用ModelAndView

`ModerAndView`是一个类，其属性中包括了一个`Object`类型的`view`属性，也包括了一个`ModelMap`对象`model`。`model`是数据对象，`view`是视图，可以是视图名，也可以是视图对象。

除了在实现`Controller`接口重写方法时使用`ModelAndView`，还可以在自定义方法中使用。

```java
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public ModelAndView myController(){
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg","hello,SpringMVC"); //添加数据
        mv.setViewName("hello"); //指定视图名
        return mv;
    }
}
```

使用`ModelAndView`时，方法的返回值必须是`ModelAndView`类型，就是将`ModelAndView`对象传递给请求分发器。

`ModelAndView`源码：

```java
public class ModelAndView {
    @Nullable
    private Object view;  //视图名
    @Nullable
    private ModelMap model;  //保存数据
    @Nullable
    private HttpStatus status;
    private boolean cleared = false;

    public ModelAndView() {}
    //设置要跳转的视图名
    public void setViewName(@Nullable String viewName) {
        this.view = viewName;
    }
    //添加数据
    public ModelAndView addObject(String attributeName, 
                                  @Nullable Object attributeValue) {
        this.getModelMap().addAttribute(attributeName, attributeValue);
        return this;
    }
    ...
}
```

通过源码可以看出，`setViewName()`方法是设置视图名给`view`，`addObject()`方法是调用`ModelMap`类的方法添加数据。



**总结**

* 在使用`Model`和`ModelMap`的两种方式中，视图名是使用返回值或者手动重定向/转发来确定的，数据是通过`addAttribute()`方法添加的。

* 使用`ModelAndView`的方法中的视图名是通过`setViewName()`方法传递的，数据是通过`addObject()`方法添加的。
* 我们可以根据实际情况选用三种方式。



### 5.3 前端向后端传递数据时的乱码问题

前端向后端传递数据时，由于客户端浏览器编码方式的问题，传到后端时为乱码，此时可以使用SpringMVC提供的过滤器`CharacterEncodingFilter`，在`web.xml`文件中配置过滤器：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- 配置请求分发器 -->
    ...
    <!-- 配置SpringMVC提供的编码过滤器 -->
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!-- 设置编码方式为utf-8 -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    
    <filter-mapping>
        <filter-name>encoding</filter-name>
        <!-- /*也会过滤jsp页面 -->
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

`/*`过滤所有页面，包括静态资源。



# 六、JSON

## 6.1 JSON介绍

**JSON（JavaScript Object Notation, JS 对象标记）**是一种轻量级的数据交换格式。JSON采用一种完全独立于编程语言的文本格式来存储和表示数据。语法格式为：

* 对象保存为**键值对**，数据由逗号分隔；
* `{}`用来保存对象；
* `[]`用来保存数组。



比如，一个包括三个User对象的JSON：

```json
[{"name":"Tom","age":3,"sex":"man"},{"name":"Jerry","age":4,"sex":"man"},
{"name":"Rick","age":20,"sex":"man"}]
```



## 6.2 使用JSON传递数据

对于前后端分离的环境，前后端交互一般是传递的JSON数据。

常用的几个JSON处理工具包：

* [Gson](https://github.com/google/gson)：来自谷歌，使用广泛。
* [FastJson](https://github.com/alibaba/fastjson)：来自阿里巴巴，速度快。
* [Jackson](https://github.com/FasterXML/jackson)：适合数据量较大时使用。
* [Json-lib](http://json-lib.sourceforge.net/index.html)：已经不适合现在的需求。

参考[各种json工具包的比较](https://blog.csdn.net/jiyueqianxue/article/details/83377181)



### 6.2.1 JS中使用JSON

从JSON字符串转换为JavaScript对象：

```javascript
var obj = JSON.parse('{"a": "Hello", "b": "World"}');
//结果是 {a: 'Hello', b: 'World'}
```

从JavaScript 对象转换为JSON字符串：

```javascript
var json = JSON.stringify({a: 'Hello', b: 'World'});
//结果是 '{"a": "Hello", "b": "World"}'
```



### 6.2.2 SpringMVC中返回JSON数据

以Gson为例，使用时需要先创建Gson对象，然后调用方法，如下：

1、将User对象转化为JSON字符串，使用`toJson()`方法

```java
String str = new Gson().toJson(new User(1,"张三",24));
```



2、将JSON字符串转化为User对象，使用`fromJson()`方法

```java
User user = new Gson().fromJson(str,User.class);
```



在SpringMVC中使用Gson，需要先导入依赖。

`pom.xml`：

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>
```

定义`User`类：

```java
public class User {
    private int id;
    private String name;
    private int age;
    ...
}
```



我们需要控制器的方法直接返回数据，而不是返回视图，这时候就需要使用`@ResponseBody`注解了。或者直接将当前类注册为`RestController`。

*  `@ResponseBody`：可以用在**方法**上，也可以用在**类**上。用在方法上表示当前方法**返回值不会被视图解析器解析**，返回值会被直接写入到响应体（response body）中，通常为JSON或XML类型的数据。用在类上则对当前类所有方法生效。

* `@RestController`：只能用在**类**上，效果相当于同时使用 `@Controller` 和`@ResponseBody`，表示将当前类注册为控制器，且类中所有方法的返回值都返回的是数据对象，不会被视图解析器解析。

  > `@RestController`底层仍然是使用了 `@Controller` 和`@ResponseBody`。
  >
  > 更多对比，参考[@Controller和@RestController](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/Spring%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93?id=_3-restcontroller-vs-controller)

以下为组合使用 `@Controller` 和 `@ResponseBody`，单独使用`@ResponseBody`两种方式，二者效果相同。

1、使用 `@Controller` 和 `@ResponseBody`：

```java
@Controller
public class UserController {}
    @ResponseBody
    @RequestMapping(value = "/json")
    public String test(){
        User user = new User(1,"John",24);
        String str = new Gson().toJson(user);
        return str;
    }
}
```



2、使用`@RestController`：

```java
@RestController
public class UserController {
    @RequestMapping(value = "/json")
    public String test(){
        User user = new User(1,"John",24);
        String str = new Gson().toJson(user);
        return str;
    }
}
```

这样客户端可以从响应数据中接收数据。



## 6.3 使用HttpMessageConverter处理乱码

控制器返回JSON数据时，可能会由于编码不同而出现乱码问题，可以通过配置SpringMVC的`StringHttpMessageConverter`消息转换器来解决。

在spring的配置文件`springmvc-servlet.xml`文件的注解驱动中配置消息转换器：

```xml
<!-- annotation-driven的作用就是提供扩展功能 -->
<mvc:annotation-driven>
    <!-- StringHttpMessageConverter转换配置，解决乱码问题 -->
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.converter.
                     StringHttpMessageConverter">
            <constructor-arg value="UTF-8"/>
        </bean>
        <bean class="org.springframework.http.converter.json.
                     MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="org.springframework.http.converter.json.
                             Jackson2ObjectMapperFactoryBean">
                    <property name="failOnEmptyBeans" value="false"/>
                </bean>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```





# 七、RESTful

参考[理解RESTful架构](https://www.ruanyifeng.com/blog/2011/09/restful.html)

REST（Representational State Transfer），表现层状态转化：

* 每个URI代表一种资源
* 客户端和服务器之间传递这种资源的表现层
* 客户端通过四个HTTP请求方式，对服务器资源进行操作，实现“表现层状态转化”。
  * GET请求用来获取资源，对应查询操作
  * POST请求用来新建（或更新）资源，对应添加操作
  * PUT用来更新资源，对于修改操作
  * DELETE用来删除资源，对于删除操作



可以简单将RESTful理解为资源定位及资源操作的风格。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

RESTful风格下，请求和参数值都使用`/`分隔开，同样的请求地址可以根据不同的请求方式，实现不同的功能：

```
http://127.0.0.1/item/1 	查询,GET
http://127.0.0.1/item/1 	新增,POST
http://127.0.0.1/item/1 	更新,PUT
http://127.0.0.1/item/1 	删除,DELETE
```

比如：

```java
@Controller
public class RestFulController {
    @RequestMapping(value = "/add/{a}/{b}",method = RequestMethod.POST)
    public String test1(@PathVariable int a, @PathVariable int b, Model model){
        int res = a+b;
        model.addAttribute("msg","Result 1:"+res);
        return "restful";
    }
    //URL相同时，可以根据不同的请求调用不同的方法
    @GetMapping("/add/{a}/{b}")
    public String test2(@PathVariable int a, @PathVariable int b, Model model){
        int res = a*b;
        model.addAttribute("msg","Result 2:"+res);
        return "restful";
    }
}
```



`@PathVariable`用于将URI的变量和方法参数对应起来，名称要相同。

对于以上代码，URL为`http://localhost:8080/SpringMVC_04/add/3/4`时，如果是`get`请求，会调用第二个方法，结果为12；如果是`post`请求，调用第一个方法，结果为7。



RESTful风格的优势：

* 使路径变得更简洁。
* 获取参数更方便，框架会自动类型转换
* 通过路径变量的类型约束访问参数，如果类型不对应，会访问不到方法。





# 八、拦截器



SpringMVC中的**拦截器（Interceptor）**类似于servlet中的过滤器（Filter），不同的是：

* 过滤器在任何web工程都可以使用。拦截器属于SpringMVC框架，只能在SpringMVC框架工程中使用。
* 过滤器通过`/*`可以拦截所有要访问的资源。拦截器只会拦截访问的控制器方法。

拦截器和过滤器的区别：https://www.zhihu.com/question/30212464/answer/1786967139



自定义拦截器需要实现`HandlerInterceptor`接口，接口中的三个默认方法，可以根据需求进行重写。

在项目中使用自定义拦截器步骤如下：

**1、定义一个类实现`HandlerInterceptor`接口**

```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, Object handler) 
        throws Exception {
        System.out.println("=========处理前要执行的操作=========");
        return true; //true表示放行，执行下一个拦截器
    }

    /*
    postHandle和afterCompletion一般用于日志处理之类的后续工作
     */
    @Override
    public void postHandle(HttpServletRequest request, 
                           HttpServletResponse response, Object handler, 
                           ModelAndView modelAndView) throws Exception {
        System.out.println("=========处理后要执行的操作=========");
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response, 
                                Object handler, Exception ex) throws Exception {
        System.out.println("=========清理=========");
    }
}
```

`HandlerInterceptor`接口有三个方法：

* `preHandle()`：请求处理前执行，返回值为`true`表示放行，如果有的话执行下一个拦截器。返回值为`false`表示不放行。常用于身份验证等需求中。
* `postHandle()`：请求完成后执行，一般用于日志处理等后续操作。如果有多个拦截器，这个方法会按照声明顺序倒着执行。
* `afterCompletion()`：在请求分发器视图渲染之后执行，多用于资源清理。前提是`preHandle`返回`true`



**2、在SpringMVC配置文件中配置拦截器**

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <!-- 配置拦截所有请求的拦截器 -->
    <mvc:interceptor>
        <!-- /**表示这个请求下的所有请求，包括子请求 -->
        <mvc:mapping path="/**"/>
        <!-- 指定拦截器实现类 -->
        <bean class="com.kang.config.MyInterceptor"/>
    </mvc:interceptor>
    
    <!-- 配置一个只拦截user下所有请求的拦截器 -->
    <mvc:interceptor>
        <!-- /**表示这个请求下的所有请求，包括子请求 -->
        <mvc:mapping path="/user/**"/>
        <bean class="com.kang.config.LoginInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

可以配置多个拦截器，满足不同的需求。

`/**`：当前所有文件，递归包括所有子文件夹，即包括多级目录。即所有请求，包括子请求。

`/*`：当前所有文件，不包括子文件夹中的内容，即只包括一级目录。即只有当前请求下的请求，不包括子请求下的请求。



**使用情景举例**：使用拦截器确保用户登录才能进入某个页面。当用户访问某个页面的时候，在拦截器中验证是否登陆，如果登陆，则放行；如果未登录，则请求转发或重定向到登录界面。



# 九、Ajax

**AJAX（Asynchronous JavaScript and XML）**：异步的 JavaScript 和 XML，是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。

AJAX是一种**用于创建更好更快以及交互性更强的Web应用程序的技术。**比如搜索栏动态显示搜索建议；网页登陆时不用刷新网页就提示用户名和密码正确性。



**Demo：**在SpringMVC项目中，使用Ajax实现以下功能：

输入用户名后，动态检测用户名是否存在并提示；输入密码后，动态检测密码是否正确并提示。

**1、先定义一个控制器类`verifyUsernameAndPwd.java`：**

```java
/*
简单起见，用户名为“admin”，密码为“123456”，模拟从数据库中查询到的结果。
*/
@RestController
public class AjaxController {
    //登陆验证处理
    @RequestMapping("/login")
    public String login(String name,String pwd){
        String msg = "";
        if(name !=null){
            if("admin".equals(name)){
                msg="ok";
            }else{
                msg = "用户名不存在";
            }
        }
        if(pwd!=null){
            if("123456".equals(pwd)){
                msg="ok";
            }else{
                msg = "密码错误";
            }
        }
        return msg;
    }
}
```



**2、前端页面**

我们使用jquery提供的Ajax方法实现Ajax请求。使用前需要导入Jquery的jar包，或者使用CDN。

`login.jsp`：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>ajax</title>
        <!-- 需要将jquery包导入 -->
        <script src="${pageContext.request.contextPath}/statics/js/jquery-3.1.1.min.js"></script>
        <script>
            //用户名框失焦事件
            function verifyName(){
                //通过jquery中的post方法调用ajax方法
                $.post({
                    url:"${pageContext.request.contextPath}/login",
                    data:{'name':$("#name").val()},
                    success:function (data) {
                        //如果返回结果为ok,提示信息为绿色,否则为红色
                        if (data.toString()=='OK'){
                            $("#userInfo").css("color","green");
                        }else {
                            $("#userInfo").css("color","red");
                        }
                        $("#userInfo").html(data);
                    }
                });
            }
            //密码框失焦事件
            function verifyPwd(){
                $.post({
                    url:"${pageContext.request.contextPath}/login",
                    data:{'pwd':$("#pwd").val()},
                    success:function (data) {
                        if (data.toString()=='OK'){
                            $("#pwdInfo").css("color","green");
                        }else {
                            $("#pwdInfo").css("color","red");
                        }
                        $("#pwdInfo").html(data);
                    }
                });
            }

        </script>
    </head>
    <body>
        <p>
            用户名:<input type="text" id="name" onblur="verifyName()"/>
            <span id="userInfo"></span>
        </p>
        <p>
            密码:<input type="text" id="pwd" onblur="verifyPwd()"/>
            <span id="pwdInfo"></span>
        </p>
    </body>
</html>
```



上述JS代码中我们使用的`$.post`调用的是jquery中的`post`方法，`post`方法内部调用了`ajax`方法，我们也可以直接调用`ajax`方法，都可以完成Ajax请求。

Ajax方法需要关注的几个参数：

* `url`：请求地址。
* `type`：请求方式，比如get、post（1.9.0之后用method）。
* `data`：要发送的数据。
* `ontentType`：发送信息至服务器的内容编码类型。
* `success`：成功之后的回调函数。

在上述的Demo中，url就是`login`请求地址，`data`就是当前输入框里的内容，`success`回调函数的功能定义为显示提示信息，比如用户名是否存在、密码是否正确。



# 十、文件上传与下载

Spring MVC为文件上传提供了直接的支持，这种支持是通过即插即用的MultipartResolver实现的。

Spring MVC使用`Apache Commons FileUpload`实现了一个MultipartResolver的实现类`CommonsMultipartResolver`。因此，SpringMVC的文件上传还需要依赖Apache Commons FileUpload的组件。

导入文件上传需要的依赖：

```xml
<!--文件上传-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
```





## 10.1 文件上传

文件上传需要确保导入了`commons-fileupload`依赖。

**1、配置bean**

```xml
<!-- 配置CommonsMultipartResolver -->
<!--这个bean的id必须为：multipartResolver-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="defaultEncoding" value="utf-8"/>
    <!-- 根据需求确定是否配置上传文件大小限制，单位为字节(10485760=10M)-->
    <property name="maxUploadSize" value="10485760"/>
    <property name="maxInMemorySize" value="40960"/>
</bean>
```

其中`defaultEncoding`表示请求的编码格式，必须和JSP的`pageEncoding`属性一致，以便正确读取表单的内容，默认为`ISO-8859-1`.

`CommonsMultipartFile`类中的常用方法：

- `String getOriginalFilename()`：获取上传文件的原名
- `InputStream getInputStream()`：获取文件流
- `void transferTo(File dest)`：将上传文件保存到一个目录文件中



**2、前端页面**

```jsp
<!-- 上传文件的编码必须是multipart/form-data -->
<form action="/upload" enctype="multipart/form-data" method="post">
    <input type="file" name="file"/>
    <input type="submit" value="upload">
</form>
```



**3、控制器实现功能**

```java
@RestController
public class FileController {
    //@RequestParam("file") 将name=file控件得到的文件封装成CommonsMultipartFile对象
    //批量上传CommonsMultipartFile则为数组即可
    @RequestMapping("/upload")
    public String fileUpload(@RequestParam("file") CommonsMultipartFile file, 
                             HttpServletRequest request) throws IOException {

        //获取文件名 : file.getOriginalFilename();
        String uploadFileName = file.getOriginalFilename();

        //如果文件名为空，直接回到首页
        if ("".equals(uploadFileName)) {
            return "redirect:/index.jsp";
        }
        System.out.println("上传文件名 : " + uploadFileName);

        //上传路径保存设置
        String path = request.getServletContext().getRealPath("/upload");
        //如果路径不存在，创建一个
        File realPath = new File(path);
        if (!realPath.exists()) {
            realPath.mkdir();
        }
        System.out.println("上传文件保存地址：" + realPath);
        InputStream is = file.getInputStream(); //文件输入流
        OutputStream os = new FileOutputStream(
            new File(realPath, uploadFileName)); //文件输出流

        //读取写出
        int len = 0;
        byte[] buffer = new byte[1024];
        while ((len = is.read(buffer)) != -1) {
            os.write(buffer, 0, len);
            os.flush();
        }
        os.close();
        is.close();
        return "redirect:/index.jsp";
    }

    /*
     * 方式二：采用file.Transto 来保存上传的文件
     */
    @RequestMapping("/upload2")
    public String  fileUpload2(@RequestParam("file") CommonsMultipartFile file,
                               HttpServletRequest request) throws IOException {

        //上传路径保存设置
        String path = request.getServletContext().getRealPath("/upload");
        File realPath = new File(path);
        if (!realPath.exists()){
            realPath.mkdir();
        }
        //上传文件地址
        System.out.println("上传文件保存地址："+realPath);

        //通过CommonsMultipartFile的方法直接写文件
        file.transferTo(new File(realPath +"/"+ file.getOriginalFilename()));
        return "redirect:/index.jsp";
    }
}
```





## 10.2 文件下载

文件下载步骤：

* 设 response响应头

* 读取文件 -- InputStream

* 写出文件 -- OutputStream

* 执行操作
* 关闭流 （先开的后关）



**1、编写控制器类**

```java
@RestController
public class FileController {
    @RequestMapping(value="/download")
    public String downloads(HttpServletResponse response, 
                            HttpServletRequest request) throws Exception{
        //要下载的文件地址
        String  path = request.getServletContext().getRealPath("/upload");
        String  fileName = "test.txt";

        //1、设置response 响应头
        response.reset(); //设置页面不缓存,清空buffer
        response.setCharacterEncoding("UTF-8"); //字符编码
        response.setContentType("multipart/form-data"); //二进制传输数据
        //设置响应头
        response.setHeader("Content-Disposition",
                "attachment;fileName="+ URLEncoder.encode(fileName, "UTF-8"));

        File file = new File(path,fileName);
        //2、 读取文件--输入流
        InputStream input=new FileInputStream(file);
        //3、 写出文件--输出流
        OutputStream out = response.getOutputStream();

        byte[] buff =new byte[1024];
        int index=0;
        //4、执行 写出操作
        while((index= input.read(buff))!= -1){
            out.write(buff, 0, index);
            out.flush();
        }
        out.close();
        input.close();
        return null;
    }
}
```



**2、前端页面**

```jsp
<a href="/download">点击下载</a>
```



---

# 参考

1、[b站-SpringMVC](https://www.bilibili.com/video/BV1aE41167Tu)

2、[Spring常见问题](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/Spring%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93)