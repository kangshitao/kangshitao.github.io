---
title: SpringBoot2-配置与原理深入解析
excerpt: SpringBoot2的自动配置原理、启动流程
mathjax: true
date: 2021-07-12 21:29:42
tags: ['SpringBoot2','Spring']
categories: SpringBoot2
keywords: Spring,SpringBoot2,Profile
---



# 一、SpringBoot自动配置原理

SpringBoot中的自动配置原理，需要从`@SpringBootApplication`这个注解出发。

这个注解相当于以下三个注解：

* `@SpringBootConfiguration`：等价于`@Configuration`，表明这是一个SpringBoot项目的配置类。
* `@EnableAutoConfiguration`：开启自动配置功能。
* `@ComponentScan`：自动扫描并加载**给定的路径中**符合条件的组件，注入到IOC容器中。

其中自动配置的原理，就和`@EnableAutoConfiguration`有关。

下面着重对其进行分析。

`@EnableAutoConfiguration`的源码：

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

可以看到，`@EnableAutoConfiguration`是由`@AutoConfigurationPackage`和`@Import`两个注解组成的。

其中`@AutoConfigurationPackage`会自动导入主程序所在包及其子包的一系列组件。即**自动导入包机制。**

`@Import`主要是按需加载自动配置类，将生效的自动配置类加载到容器。

下面对这两个注解详细分析。



## 1.1 @AutoConfigurationPackage自动导包

`@AutoConfigurationPackage`注解是用来自动导入包的，下面深入解析其是如何导入的。

进入其源码：

```java
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {}
```

可以看到，`AutoConfigurationPackage`为我们自动引入了一个注册器。继续进入`AutoConfigurationPackages.Registrar`：

![内部类Registrar的声明](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_1.png)



`Registrar`是`AutoConfigurationPackages`的一个内部类，其中有两个方法，主要关注第一个方法。

对`registerBeanDefinitions()`这个方法进行分析

* `metadata`参数：注解的元信息。这里指的是`@AutoConfigurationPackage`这个注解，而这个注解相当于直接配置在了启动类上，因此从元信息中获取包名，就是启动类所在的目录名。比如当前启动类在`com/kang/root`目录下，则`new PackageImports(metadata).getPackageNames()`的值为`com.kang.root`。
* `register()`：将指定目录下的包的所有组件注册进来。

由此，我们可以得知，**`@AutoConfigurationPackage`注解，利用`Registrar`给容器中导入一系列组件，将指定的一个包下的所有组件导入进来，这个包就是主程序所在的包。**

因此我们可以解释，为什么SpringBoot扫描包的路径就是主程序所在包及其子包。



## 1.2 @Import 按需加载配置类



`@EnableAutoConfiguration`第二个重要的注解是`@Import`，其引入了`AutoConfigurationImportSelector`，这个类的继承体系如下：

![AutoConfigurationImportSelector的继承体系](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_2.png)

可以看到其实现了`ImportSelector`接口，并且实现了这个接口的`selectImports`方法，这个方法主要用于**获取所有符合条件的全限定类名，这些类需要被加载到IOC容器中**。

`AutoConfigurationImportSelector`类：

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    //判断自动装配开关是否打开
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    //获取所有需要装配的bean
    AutoConfigurationEntry autoConfigurationEntry =
        getAutoConfigurationEntry(annotationMetadata);
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```

接下来进入`getAutoConfigurationEntry()`这个方法，查看是如何获取需要装配的bean的。

这个方法的源码如下：

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(
    AnnotationMetadata annotationMetadata) {
    //1.判断自动装配开关是否打开
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    //2.
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    //3.扫描spring.factories下需要自动装配的所有类
    List<String> configurations = getCandidateConfigurations(annotationMetadata,
                                                             attributes);
    //4.下面是过滤出真正需要加载的类。
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = getConfigurationClassFilter().filter(configurations);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

代码解析如下：

1、判断自动装配开关是否打开，即`spring.boot.enableautoconfiguration`的值是否为true.

2、获取`@EnableAutoConfiguration`注解的所有属性，即`exclude`和`excludeName`。

3、`getCandidateConfigurations()`方法，从`META-INF/spring.factories`中获取所有需要自动装配的配置类。

* 内部调用`SpringFactoriesLoader.loadFactoryNames()`，然后调用`loadSpringFactories()`方法。然后进入这个方法。

  * 可以看到，这个方法加载了`META-INF/spring.factories`这个配置文件，默认扫描当前系统里面所有的`META-INF/spring.factories`位置的文件。

  * ![](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_3.png)

  * 当前版本一共131个需要自动装配的类，没有第三方starter时，他们都是来自`spring-boot-autoconfigure-2.5.2`包里的`META-INF/spring.factories`文件。这个自动配置包里面规定了SpringBoot自动配置的一些属性，其中的`EnableAutoConfiguration`属性就是所有要自动装配的类，如图。

    ![META-INF/spring.factories里配置的需要自动装配的类](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_4.png)



因此，如果我们想要实现一个starter，就必须在`META-INF/spring.factories`文件中配置这样的配置项，指定要加载的自动配置包。

这一步中将自动配置类的所有需要加载的包都读取过来了，那么这些包必须全部加载吗？答案是否定的。

顺着源码继续往下。

4、**按需开启自动配置项**。虽然读取了所有需要自动加载的类，但是这些类生效都是有条件的，比如需要导入相关的包，这个类才能被加载。因此最终只加载生效的部分类。

> 配置类中使用@Condition条件装配，判断当前类是否生效。



## 1.3 总结

关于自动配置：

- SpringBoot先读取所有的自动配置类：`xxxxxAutoConfiguration`
- 每个自动配置类按照条件生效，默认都会绑定`xxxxProperties`类，这个类里面设置了默认的值。
- `xxxProperties`类和配置文件进行了绑定，可以通过配置文件对默认值进行设置。

- 生效的配置类就会给容器中装配很多组件。只要容器中有这些组件，相当于这些功能就有了



如果我们需要的包没有被starter，则需要手动导入。

我们可以在配置文件中对配置项的值进行修改。





# 二、Profile功能

为了方便多环境适配，SpringBoot简化了profile功能。

我们可以配置多套环境，比如测试环境和生产环境，可以方便的在不同的环境中切换。

比如同时配置多个配置文件，表示多个配置环境，根据需求使用不同的配置：

```
application.yaml     总配置文件
application-test.yaml  test环境
application-prod.yaml  prod环境
...
```



## 2.1 application-profile功能



- 默认配置文件 `application.yaml`任何时候都会加载
- 指定环境配置文件，格式： application-{env}.yaml，比如application-prod.yaml文件，代表`prod`环境。

- 激活指定环境：
  - 方式一：使用配置文件激活。比如在默认配置文件中使用`spring.profiles.active=test`激活test环境。

  - 方式二：命令行激活。对于已经打包的程序，命令行也能够**指定环境并修改配置文件的任意值**。比如使用命令行激活prod环境，并指定参数值

    ```shell
    # 这时，命令行的参数值会生效
    java -jar xxx.jar --spring.profiles.active=prod  --person.name=haha
    ```

- 默认配置与环境配置同时生效
- 对于同名配置项，指定的环境配置生效。



## 2.2 @Profile条件装配功能



@Profile可以用在类上或者方法上，表示只有在指定的环境下才生效。

比如`@Profile("test")`标注，表示这个类/方法只有在test环境下才生效。

比如：

`person.java`

```java
@Component
@ConfigurationProperties("person")
public interface Person {}
```

`Boss.java`

```java
//这个类只在prod环境下注册
@Profile("prod")  
@Component
public class Boss implements Person{
    private String name;
    private Integer age;

    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
    public Integer getAge() {return age;}
    public void setAge(Integer age) {this.age = age;}
}
```



`Worker.java`

```java
//这个类只在test环境下注册
@Profile("test")
@Component
public class Worker implements Person{
    private String name;
    private Integer age;

    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
    public Integer getAge() {return age;}
    public void setAge(Integer age) {this.age = age;}
}
```

然后定义一个`controller`类，`HelloController.java`

```java
@RestController
public class HelloController {
    @Autowired
    Person person;

    @GetMapping("/")
    public String  hello(){
        return person.getClass().getName();
    }
}
```

这样，如果在`test`环境下，访问主页就会返回`com.kang.boot.bean.Worker`，如果在`prob`环境下，返回值为`com.kang.boot.bean.Boss`。



## 2.3 profile分组

可以使用`spring.profiles.group`对配置文件进行分组，然后同时激活同一个组的多个配置文件。比如：

`application.properties`

```properties
# 激活myprod组
spring.profiles.active=myprod

# 定义两个配置组，myprod和mytest
spring.profiles.group.myprod[0]=prod
spring.profiles.group.myprod[1]=prod2
spring.profiles.group.mytest[0]=test
```



# 三、配置加载优先级



参考官方文档[Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#boot-features-external-config)

后加载的会覆盖前面加载的。

根据官方文档，SpringBoot的配置文件可以来自于以下方面，按照从上往下的顺序，其中后加载的会覆盖前面的：

* Default properties (specified by setting `SpringApplication.setDefaultProperties`).
* `@PropertySource` annotations on your `@Configuration` classes. Please note that such property sources are not added to the `Environment` until the application context is being refreshed. This is too late to configure certain properties such as `logging.*` and `spring.main.*` which are read before refresh begins.
* **Config data (such as`application.properties` files)**
* A `RandomValuePropertySource` that has properties only in `random.*`.
* OS environment variables.
* Java System properties (`System.getProperties()`).
* JNDI attributes from `java:comp/env`.
* `ServletContext` init parameters.
* `ServletConfig` init parameters.
* Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property).
* **Command line arguments.**
* `properties` attribute on your tests. Available on `@SpringBootTest` and the [test annotations for testing a particular slice of your application](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests).
* `@TestPropertySource` annotations on your tests.
* [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-devtools-globalsettings) in the `$HOME/.config/spring-boot` directory when devtools is active.

加粗是比较常用的两种。



## 3.1 外部配置源

常用：Java属性文件、YAML文件、环境变量、命令行参数；



## 3.2 配置文件查找位置

配置文件的查找顺序：

(1) classpath 根路径（Java文件夹和resource文件夹都属于根路径，编译后都位于target根目录下）

(2) classpath 根路径下config目录

(3) jar包当前目录

(4) jar包当前目录的config目录

(5) /config子目录的直接子目录

## 3.3 配置文件加载顺序：

1. 　当前jar包内部的application.properties和application.yml
2. 　当前jar包内部的application-{profile}.properties 和 application-{profile}.yml

1. 　引用的外部jar包的application.properties和application.yml
2. 　引用的外部jar包的application-{profile}.properties 和 application-{profile}.yml

## 3.4 总结

指定环境优先，外部优先，后面的可以覆盖前面的同名配置项。



# 四、自定义starter

## 4.1 starter启动原理

starter的启动流程：

**1、指定要加载的自动配置类**

在引入starter后，会加载指定的自动配置类。

SpringBoot的`autoconfigure`包中，`META-INF`下的`spring.factories`文件，使用了 `EnableAutoConfiguration` 的值指定了项目启动时要加载的自动配置类。如图：

![SpringBoot的spring.factories](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_5.png)



**2、自动配置类中指定要加载的类`xxxProperties`**

一般来说，自动配置类用于自动注册组件，并使用指定的配置类，进行属性值的注入。

以`org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration`为例，加载这个自动配置类，这个配置类的声明如下：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureAfter({ CouchbaseDataAutoConfiguration.class, 
                     HazelcastAutoConfiguration.class,
                     HibernateJpaAutoConfiguration.class,
                     RedisAutoConfiguration.class })
@Import({ CacheConfigurationImportSelector.class, CacheManagerEntityManagerFactoryDependsOnPostProcessor.class })
public class CacheAutoConfiguration {...}
```

这个自动配置类使用`@EnableConfigurationProperties`注解，指定了要加载的配置类`CacheProperties`。

其余的注解则规定了这个自动配置类的生效条件，比如`@ConditionalOnBean`、`@ConditionalOnMissingBean`等注解。例如：`@ConditionalOnBean(CacheAspectSupport.class)`表示当前容器中有`CacheAspectSupport`这个类的组件时，当前的自动配置类才生效。



**3、xxxProperties中绑定配置文件中的配置属性**

这个类中规定了默认值，自动配置组件的值就是从这里取。并且这个类和配置文件绑定，我们可以通过配置文件对默认值进行修改。

```java
@ConfigurationProperties(prefix = "spring.cache")
public class CacheProperties {...}
```

比如`CacheProperties`这个配置类，使用`@ConfigurationProperties(prefix = "spring.cache")`注解，将配置文件中`spring.cache`开头的配置项的值，绑定到当前类中对应的属性。

这样，我们在总配置文件中使用`spring.cache.xxx=xxx`，就可以对cache相关的内容进行配置。



## 4.2 自定义starter

仿照官方的starter启动流程，我们可以创建自定义的starter。

**1、创建starter和starter-autoconfigure项目**

自定义starter需要创建一个starter项目和一个对这个starter自动配置的项目。

如图：

![](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_6.png)

自定义的启动器一般是xxx-spring-boot-starter的命名格式，对其自动配置的包一般是xxx-spring-boot-starter-autoconfigure格式。

在启动器中引入自动配置包的依赖，如图：

![启动器中引入自动配置包的依赖](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_7.png)

这样，我们使用的时候，只需要引入starter就可以使用，它就会自动为我们导入自动配置的依赖包。

我们可以把需要的配置，都在自动配置包中引入依赖。

**2、在`spring.factories`中指定要加载的自动配置类**

为了保证我们的自动配置类能自动加载，需要在自动配置包的`META-INF/spring.factories`文件中，指定项目启动时要加载的自动配置类：

![](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_8.png)



`spring.factories`中的内容如下：

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.kang.hello.auto.HelloServiceAutoConfiguration
```

指定项目启动时，会加载`com.kang.hello.auto.HelloServiceAutoConfiguration`这个配置类。



**3、编写xxxAutoConfiguration和xxxProperties和功能类**

配置类`HelloProperties`，对属性值和配置文件中的值进行绑定

```java
//绑定配置文件中的前缀
@ConfigurationProperties(prefix = "kang.hello")
public class HelloProperties {
    private String prefix;
    private String suffix;

    public String getPrefix() {return prefix;}
    public void setPrefix(String prefix) {this.prefix = prefix;}
    public String getSuffix() {return suffix;}
    public void setSuffix(String suffix) {this.suffix = suffix;}
}
```

这样，项目中在配置文件中使用`kang.hello`就可以对`prefix`和`suffix`进行配置。



自动配置类`HelloServiceAutoConfiguration`，用于注册组件：

```java
@Configuration
@EnableConfigurationProperties(HelloProperties.class)
public class HelloServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean(HelloService.class)
    public HelloService helloService(){
        return new HelloService();
    }
}
```

当容器中没有`HelloService`组件时，自动配置类才会注册`HelloService`组件，并根据`HelloProperties`中的绑定的值，对`HelloProperties`中的属性值进行注入。

`HelloService`中定义了自定义启动器要封装的功能方法。

```java
/*
默认不要放在容器中
 */
public class HelloService {
    @Autowired  //自动装配配置类
    HelloProperties helloProperties;
    
    //定义一个方法，在输入参数中添加前缀和后缀
    public String sayHello(String userName){
        String prefix = helloProperties.getPrefix();
        String suffix = helloProperties.getSuffix();
        return prefix+","+userName+","+suffix;
    }
}
```



**4、将 starter和starter-autoconfigure项目进行打包安装**

将自动配置包使用Maven进行install，然后对starter进行install。

这样，我们的自定义启动器就算创建好了，别的项目直接引入自定义的starter依赖就可以使用其中提供的方法了。



**5、测试自定义starter**

新建一个项目，引入自定义的starter依赖：

`pom.xml`

```xml
<dependency>
    <groupId>com.kang</groupId>
    <artifactId>kang-hello-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```



然后在配置类中，使用`kang.hello`对两个参数进行配置

`application.properties`

```properties
kang.hello.prefix = hello
kang.hello.suffix = bye
```



编写方法，调用starter中定义好的方法完成功能：

`HelloController`

```java
@RestController
public class HelloController {
    //引入了自定义的启动器，可以使用其提供的方法。
    @Autowired
    HelloService helloService;

    @GetMapping("/hello")
    public String sayHello(){
        String str = helloService.sayHello("KANG");
        return str;
    }
}
```

这样，在浏览器中发送`hello`请求，返回值为：

```
hello,KANG,bye
```

测试成功，说明自定义的启动器没有问题。

上面使用的是自动装配的`HelloService`，如果我们不希望使用提供的原方法，就可以自定义`HelloService`，这样，自动配置类不会自动注册`HelloService`，项目会使用我们自己的`HelloService`。实现了SpringBoot中的**自定义优先**的情况。

因此，如果我们在使用starter的过程中，如果现有的方法不能满足我们的需求，我们就可以根据源码找到其方法进行重写，这样项目就会使用我们重写之后的方法。

比如我们定义一个配置类，并自己新建一个`HelloService`对象，通过调用方法实现需求，这时底层使用的就是我们创建的这个对象：

```java
@Configuration
public class MyConfig {
    @Bean
    public HelloService helloService(){
        HelloService helloService = new HelloService();
        //对helloService进行自定义的操作，比如setXxx()
        return helloService;
    }
}
```





# 五、SpringBoot启动原理

## 5.1 SpringBoot启动过程

进入`run`方法：

![run方法代码](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-advance_9.png)



可以看到主要有两步，创建`SpringApplication`和运行`SpringApplication`.

**1、创建`SpringApplication`**

进入`new SpringApplication`方法，创建`SpringApplication`，应用创建的过程，简单来说就是先将一些组件读取到应用中。

`SpringApplication.java`的构造器方法：

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    //加载一些信息
    this.resourceLoader = resourceLoader;
    //判断是否有主配置类信息
    Assert.notNull(primarySources, "PrimarySources must not be null");
    //保存主配置类信息  class com.kang.boot.Springboot09HelloTestApplication
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    //判断应用类型   Servlet
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    //去spring.factories查找初始启动引导器  null
    this.bootstrapRegistryInitializers = getBootstrapRegistryInitializersFromSpringFactories();
    //去spring.factories查找ApplicationContextInitializer  找到n个
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    //去spring.factories查找监听器  找到n个
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //查找主程序  class com.kang.boot.Springboot09HelloTestApplication
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

主要步骤总结如下：

- 用于保存一些信息。
- 判定当前应用的类型。内部使用了ClassUtils工具类。判断结果为Servlet类型

- `bootstrapRegistryInitializers`：初始启动引导器注册初始化，其内部通过`SpringFactoriesLoader`去`spring.factories`文件中找**`org.springframework.boot.Bootstrapper`**，即查看是否有初始的启动引导器。
- 去`spring.factories`文件找 **`ApplicationContextInitializer`**，即`ApplicationContext`初始化器
  - 结果保存到：List<ApplicationContextInitializer<?>> initializers
- 去`spring.factories`找**` ApplicationListener`，即应用监听器**
  - 将结果保存到：List<ApplicationListener<?>> listeners
- 查找主程序，找到第一个main方法就是主程序。



**2、运行 `SpringApplication`**

创建完`SpringApplication`之后，执行`run()`方法。

`SpringApplication.java`的`run()`方法：

```java
public ConfigurableApplicationContext run(String... args) {
    //stopWatch用于监听整个应用程序的启动和停止
    StopWatch stopWatch = new StopWatch();
    //启动stopWatch，记录应用的启动时间
    stopWatch.start();
    //创建引导上下文
    DefaultBootstrapContext bootstrapContext = createBootstrapContext();
    ConfigurableApplicationContext context = null;
    //让当前应用进入Headless模式 java.awt.headless
    configureHeadlessProperty();
    //获取所有的运行监听器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    //starting方法调用了所有监听器的starting方法
    listeners.starting(bootstrapContext, this.mainApplicationClass);
    try {
        //保存命令行参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        //准备环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners, 
                                                                 bootstrapContext, 
                                                                 applicationArguments);
        configureIgnoreBeanInfo(environment);
        //打印banner，即运行程序的SPRING的ASCII码图案
        Banner printedBanner = printBanner(environment);
        //创建IOC容器
        context = createApplicationContext();
        //容器保存当前项目的startup信息
        context.setApplicationStartup(this.applicationStartup);
        //准备IOC容器的基本信息
        prepareContext(bootstrapContext, context, environment, listeners, 
                       applicationArguments, printedBanner);
        //刷新IOC容器
        refreshContext(context);
        //刷新之后的工作
        afterRefresh(context, applicationArguments);
        //关闭stopWatch，计算出应用启动花费的时间
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), 
                                                                        stopWatch);
        }
        //通知所有的监听器的started方法
        listeners.started(context);
        //调用所有runners
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        //如果有异常，会调用监听器的failed()方法
        handleRunFailure(context, ex, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        //调用监听器的running方法
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, null);
        throw new IllegalStateException(ex);
    }
    //最后返回容器
    return context;
}
```

`run()`方法的主要步骤如下：

- 创建`StopWatch`对象并启动，其记录了应用的启动时间。

- 创建引导上下文（Context环境）---`createBootstrapContext()`
  - 获取到所有之前的 bootstrappers， 依次执行 intitialize() ，来完成对引导启动器上下文环境设置

- 让当前应用进入`headless`模式，即`java.awt.headless`(自力更生模式)

- 获取所有`RunListener`（运行监听器），为了方便所有Listener进行事件感知。
  - 去`spring.factories`文件中找**`SpringApplicationRunListener`， **比如`EventPublishRunListenr`

- 遍历所有的`SpringApplicationRunListener `调用其` starting()`方法；相当于通知所有感兴趣系统正在启动过程的监听器，项目正在 `starting`。

- 保存命令行参数---`ApplicationArguments`
- 准备环境---`prepareEnvironment()`;
  - 返回或者创建基础环境信息对象---`StandardServletEnvironment`
  - 配置环境信息对象。
    - 读取所有的配置源的配置属性值。
  - 绑定环境信息
  - 监听器调用 `environmentPrepared()`，通知所有的监听器当前环境准备完成。

- **创建IOC容器** ---`createApplicationContext()`
  - 根据项目类型创建容器。

    ```java
    try {
        switch (webApplicationType) {
            case SERVLET:
                return new AnnotationConfigServletWebServerApplicationContext();
            case REACTIVE:
                return new AnnotationConfigReactiveWebServerApplicationContext();
            default:
                return new AnnotationConfigApplicationContext();
        }
    }
    ```

    比如当前类型为servlet，则创建 `AnnotationConfigServletWebServerApplicationContext`

- **准备 IOC容器的基本信息**  ---`prepareContext()`

  - 保存环境信息
  - IOC容器的后置处理流程。
  - 应用初始化器--`applyInitializers`；
    - 遍历所有的 `ApplicationContextInitializer `。就是创建`SpringApplication`时找到的那些初始化器。然后调用它们的` initialize`方法，来对ioc容器进行初始化扩展功能。
  - 遍历所有的 listener，即第一步中查找到的监听器，内部调用每个监听器的 `contextPrepared()`，表示IOC容器准备完成。
  - 这个方法最后，所有的监听器调用` contextLoaded()`，即通知所有的监听器IOC容器加载完毕。

- **刷新IOC容器。**refreshContext
  
  - 创建容器中的所有组件（即Spring的运行原理）
  
- 容器刷新完成后工作 --- `afterRefresh()`
- 调用`started`方法，调用所有监听器的`started`方法，即通知所有监听器start结束。

- 调用所有runners --- `callRunners()`
  - 获取容器中的**`ApplicationRunner`**
  - 获取容器中的**`CommandLineRunner`**
  - 合并所有runner并且按照@Order进行排序
  - 遍历所有的runner，调用 run方法

- 如果以上有异常，调用Listener 的`failed()`方法，表示启动失败了。

- 调用所有监听器的` running`方法 ---`listeners.running(context);`，通知所有的监听器，容器正在运行。
  - 如果`running `有问题，继续通知` failed()` 。调用所有 Listener 的`failed()`，通知所有的监听器。



## 5.2 Application Events and Listeners

SpringApplication有独特的[事件和监听器](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-events-and-listeners)，根据SpringBoot的启动过程，我们可以使用他们在启动过程中做一些额外的事情：

* **ApplicationContextInitializer**
* **ApplicationListener**
* **SpringApplicationRunListener**：在IOC容器的各个阶段做一些事情，一共有七种状态，表示IOC容器的不同阶段，不同阶段的容器具备的功能也是不同的。根据这个特征可以定制化一些功能。

实现代码举例：

`MyApplicationContextInitializer`

```java
public class MyApplicationContextInitializer implements ApplicationContextInitializer {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        //do something
        System.out.println("MyApplicationContextInitializer...initialize...");
    }
}
```

`MyApplicationListener`

```java
public class MyApplicationListener implements ApplicationListener {
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        //do something
        System.out.println("MyApplicationListener...onApplicationEvent...");
    }
}

```

`MySpringApplicationRunListener`

```java
public class MySpringApplicationRunListener implements SpringApplicationRunListener {

    private SpringApplication application;
    //注意：这里需要一个有参构造器
    public MySpringApplicationRunListener(SpringApplication application,
                                          String [] args){
        this.application = application;

    }

    @Override
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        System.out.println("MySpringApplicationRunListener...starting...");
    }

    @Override
    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext,
                                    ConfigurableEnvironment environment) {
        System.out.println("MySpringApplicationRunListener...environmentPrepared...");
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener...contextPrepared...");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener...contextLoaded...");
    }

    @Override
    public void started(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener...started...");
    }

    @Override
    public void running(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener...running...");
    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("MySpringApplicationRunListener...failed...");
    }
}

```



此外，由于`ApplicationContextInitializer`,`ApplicationListener`,`SpringApplicationRunListener`都需要在`spring.factories`文件中找，因此我们需要在这个文件中配置他们：

`resources/META-INF/spring.factories`

```xml
# Initializers
org.springframework.context.ApplicationContextInitializer=\
  com.kang.boot.listener.MyApplicationContextInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
  com.kang.boot.listener.MyApplicationListener

org.springframework.boot.SpringApplicationRunListener=\
  com.kang.boot.listener.MySpringApplicationRunListener
```



## 5.3 ApplicationRunner和CommandLineRunner

`ApplicationRunner`和`CommandLineRunner`都是在IOC容器中获取的，因此我们需要将它们注册为组件。

`MyApplicationRunner`

```java
@Component
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("MyApplicationRunner...run");
    }
}
```

`MyCommandLineRunner`

```java
/*
应用一启动，做一个一次性的事情
 */
@Component
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("MyCommandLineRunner...run...");
    }
}
```



这样，启动主程序，可以看到运行结果，和我们分析的SpringBoot启动过程是一样的：

```java
MyApplicationListener...onApplicationEvent...
MySpringApplicationRunListener...starting...
MyApplicationListener...onApplicationEvent...
MySpringApplicationRunListener...environmentPrepared...

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.2)

MyApplicationContextInitializer...initialize...
MyApplicationListener...onApplicationEvent...
MySpringApplicationRunListener...contextPrepared...
Starting Springboot09HelloTestApplication
No active profile set, falling back to default profiles: default
MyApplicationListener...onApplicationEvent...
MySpringApplicationRunListener...contextLoaded...
Tomcat initialized with port(s): 8080 (http)
Starting service [Tomcat]
Starting Servlet engine: [Apache Tomcat/9.0.48]
Initializing Spring embedded WebApplicationContext
Root WebApplicationContext: initialization completed in 2013 ms
Tomcat started on port(s): 8080 (http) with context path ''
MyApplicationListener...onApplicationEvent...
MyApplicationListener...onApplicationEvent...
Started Springboot09HelloTestApplication in 3.645 seconds (JVM running for 7.097)
MyApplicationListener...onApplicationEvent...
MyApplicationListener...onApplicationEvent...
MySpringApplicationRunListener...started...
MyApplicationRunner...run
MyCommandLineRunner...run...
MyApplicationListener...onApplicationEvent...
MyApplicationListener...onApplicationEvent...
MySpringApplicationRunListener...running...
```
