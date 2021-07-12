---
title: SpringBoot数据访问-SQL
excerpt: SpringBoot进行数据访问，整合MyBatis进行CRUD操作
mathjax: true
date: 2021-07-12 17:07:01
tags: ['SpringBoot2','Spring','MyBatis']
categories: SpringBoot2
keywords: Spring,SpringBoot2,SQL,MyBatis
---



# 一、DataSource

## 1.1 HikariDataSource数据源

如果未指定数据源，SpringBoot中默认使用HikariDataSource数据源。

使用其进行数据库的CRUD操作步骤：

**1、确保项目引入了JDBC场景**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

官方的JDBC启动器没有引入数据库驱动，我们需要根据需求，引入指定的数据库驱动，以MySQL为例：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.49</version>
</dependency>
```

我们要确保数据库版本和数据库驱动版本一致。SpringBoot2.5.2中的MySQL驱动版本是8，则需要数据库版本也是8以上。

如果想要修改驱动版本，有两种方式：

* 直接依赖引入具体版本（maven的就近依赖原则），比如上面的做法。
* 重新声明版本（maven的属性的就近优先原则），比如下面的做法：

```xml
<!-- 通过属性修改版本 -->
<properties>
    <java.version>1.8</java.version>
    <mysql.version>5.1.49</mysql.version>
</properties>
```



**2、分析自动配置**

分析SpringBoot为我们自动配置好的配置，通过分析源码可知，底层配置好了HikariDataSource连接池，以及事务管理器，分布式事务等。

底层还有一个`JdbcTemplate`组件。

连接池的配置可以使用`spring.datasource`配置。



**3、修改配置项**

在配置文件中，配置以下内容：

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springboot?useSSL=false&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

# 可以根据需求设置其他参数
  jdbc:
    template:
      query-timeout: 3
```



**4、测试**

```java
class Boot05WebAdminApplicationTests {
    @Autowired
    JdbcTemplate jdbcTemplate;

    @Test
    public void test() {
        Long aLong = jdbcTemplate.queryForObject(
            "select count(*) from books", Long.class);
       System.out.println(aLong);
    }
}
```





## 1.2 Druid数据源

Druid项目地址：https://github.com/alibaba/druid

整合第三方技术有两种方式：

* 自定义方式
* 使用其提供的starter

### 1.2.1 自定义方式

**1、引入依赖**

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.22</version>
</dependency>
```



**2、并创建数据源**

创建数据源，可以根据需求定义一些配置。可以使用传统XML的方式注册bean

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
    destroy-method="close">
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
```



也可以定义一个配置类，在配置类中使用`@Bean`注册组件：

```java
/*
手动配置数据源。在配置类中配置
 */
@Configuration
public class MyDataSourceConfig {
    @Bean
    //绑定配置文件。数据源的属性会跟配置文件的spring.datasource数据一一绑定
    @ConfigurationProperties("spring.datasource")
    public DataSource dataSource() throws SQLException {
        DruidDataSource druidDataSource = new DruidDataSource();
        //可以在这里配置，也可以直接读取配置文件中的配置，建议后者。
        //druidDataSource.setUrl();
        //druidDataSource.setUsername();
        //druidDataSource.setPassword();

        //可以加入或开启其他功能
        return druidDataSource;
    }
}
```



将账号密码等信息配置在配置文件中：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ssmbuild?useSSL=false&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```

**3、测试**

```java
@Controller
public class DataSourceController {
    @Autowired
    JdbcTemplate jdbcTemplate;

    @ResponseBody
    @GetMapping("/sql")
    public String druidTest(){
        Long aLong = jdbcTemplate.queryForObject("select count(*) from books", Long.class);
        return aLong.toString();
    }
}
```



### 1.2.2 使用官方starter

**1、引入starter**

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.17</version>
</dependency>
```

启动器为我们自动引入了数据源



**2、分析自动配置**

根据源码，分析官方已经配置好的配置，已经配置参数的使用。

可以使用`spring.datasource.druid`在配置文件中对druid进行各项的配置。



**3、配置示例**

可以参考：https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

    druid:
      aop-patterns: com.atguigu.admin.*  #监控SpringBean
      filters: stat,wall     # 底层开启功能，stat（sql监控），wall（防火墙）

      stat-view-servlet:   # 配置监控页功能
        enabled: true
        login-username: admin
        login-password: admin
        resetEnable: false

      web-stat-filter:  # 监控web
        enabled: true
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      filter:
        stat:    # 对上面filters里面的stat的详细配置
          slow-sql-millis: 1000
          logSlowSql: true
          enabled: true
        wall:
          enabled: true
          config:
            drop-table-allow: false
```





# 二、整合MyBatis

SpringBoot整合MyBatis的主要步骤如下：

* 导入MyBatis的官方starter
* 编写mapper接口，并使用@Mapper接口注册。
* 编写Service类
* 编写映射文件，绑定mapper接口
* 在总配置文件`application.yaml`中指定映射文件的位置，以及MyBatis配置文件的位置（建议直接将MyBatis的配置写在总配置文件中）。



## 2.1 配置模式

**1、引入stater**

首先引入官方starter，其为我们配置好了SqlSession

`pom.xml`

```xml
<!-- 引入MyBatis的starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
```



**2、编写mapper和service**

编写mapper接口和service层，所有的Mapper接口都需要使用`@Mapper`注解标注，表示注册当前接口为Mapper。



`BookMapper.java`

```java
@Mapper
public interface BookMapper {
    public Book getBookById(int id);
}
```

也可以在SpringBoot启动类上，使用`@MapperScan("com.kang.admin.mapper")` 注解，表示将指定包下的所有类统一注册为Mapper，这样其中的接口就可以不用单独使用@Mapper注解。



> @Mapper的功能类似于@Repository，都是用在DAO层，表示注册当前类。
>
> @Mapper不需要配置扫描地址，而@Repository需要配置扫描地址。
>
> SpringBoot中一般使用@Mapper注解。
>
> 源码对@Mapper的解释：Marker interface for MyBatis mappers



`BookService.java`

```java
@Service
public class BookService {
    @Autowired
    BookMapper bookMapper;
    public Book getBookById(int id){
        return bookMapper.getBookById(id);
    }
}
```



**3、编写sql映射器文件并绑定mapper接口**

`BookMapper.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kang.admin.mapper.BookMapper">
    <select id="getBookById" resultType="com.kang.admin.bean.Book">
        select * from ssmbuild.books where bookID=#{id}
    </select>

</mapper>
```



在`application.yml`中指定MyBatis的**映射器文件**和MyBatis**总配置文件**的位置。

`application.yml`

```yaml
# 配置数据源
...

# 配置MyBatis
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
```

`mybatis-config.xml`

```xml
mybatis配置文件
```

这里的路径，应该是编译后的配置文件路径，resources下的文件都会被直接编译到target目录下。



我们也可以在`application.yml`中的`mybatis.configuration`，对MyBatis进行配置，比如驼峰命名映射等，取代`mybatis-config.xml`配置文件。推荐这一种方式，尽量不写多余的配置文件。

比如：

```yaml
# 配置MyBatis
mybatis:
  # 将MyBatis配置写在这里，就不需要config-location这个配置了
  mapper-locations: classpath:mybatis/mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true  # 驼峰命名映射开启
```



## 2.2 注解模式

参考官方：https://github.com/mybatis/spring-boot-starter/wiki/Quick-Start

使用注解模式编写SQL语句，避免了为每个Mapper编写映射文件。

常用的注解：@Select、@Insert、@Update、@ Delete

具体使用方式可以参考：[MyBatis注解模式](https://kangshitao.github.io/2021/06/21/mybatis-basis/#%E5%9B%9B%E3%80%81%E6%B3%A8%E8%A7%A3)

案例：

```java
@Mapper
public interface CityMapper {
    @Select("select * from city where id=#{id}")
    public City getById(Long id);
}
```



## 2.3 混合模式

混合使用注解和配置文件两种方式。



```xml
<insert id="indert" useGenerateKeys="true" keyProperty="id">
	insert into city(`name`,`state`,`country`) values(#{name},#{state},#{country})
</insert>
```

`insert`语句中的`useGenerateKeys`属性，能够确保类似id这种自增主键插入数据库后，也能够返回到参数city中。

以上代码使用注解方式为：

```java
@Insert("insert into city(`name`,`state`,`country`) values(#{name},#{state},#{country})")
@Options(useGeneratedKeys = true,keyProperty = "id")
public void insert(City city);
```



最佳实践：

- 引入mybatis-starter
- 配置application.yaml中，指定mapper-location位置（将MyBatis配置文件单独配置）

- 编写Mapper接口并标注`@Mapper`注解
- 简单方法直接注解方式；复杂方法编写mapper.xml进行绑定映射。



# 三、整合MyBatis-Plus

## 3.1 什么是MyBatis-Plus

[MyBatis-Plus](https://mybatis.plus/guide/)是MyBatis的一个增强工具，在MyBatis的基础上只做增强不做改变，简化开发。



## 3.2 整合MyBatis-Plus

**1、引入starter**

```xml
<!-- 引入MyBatis-Plus的starter -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.1</version>
</dependency>
```



MyBatis-Plus的starter自动帮我们引入了JDBC、MyBatis、MyBatis-Spring等依赖包，因此如果使用MyBatis-Plus则不必单独这些依赖。



**2、分析starter，查看配置信息**

分析其自动配置类可知，可以通过`mybatis-plus`对其进行配置。

* 为我们配置好了SqlSessionFactory，数据源是容器中默认的数据源。
* 自动配置了`mapperLocations`，默认值为`classpath*:/mapper/**/*.xml`，即任意包的类路径（编译之后的路径）下的所有mapper文件夹中任意路径的xml文件，都作为映射文件。因此建议以后的映射文件放在`mapper`目录下面。
* `@Mapper`标注的接口，也会被自动扫描，不需要再配置。建议还是使用`@MapperScan`注解批量扫描。



**3、使用MyBatis-Plus**

MyBatis-Plus为我们提供了`BaseMapper`接口，其定义了大部分的增删查改操作。我们只需要在接口上继承`BaseMapper`即可，**不需要写任何方法，无需写映射文件**，使用的时候直接调用`BaseMapper`中的方法。



`UserMapper.java`接口

```java
public interface UserMapper extends BaseMapper<User> {}
```



如果我们要编写service层，是否要实现`BaseMapper`中的所有方法？MyBatis-Plus也为我们提供了一个service层的接口`IService`，这是顶级service，其定义了常用的service层方法。

并且，MyBatis-Plus还为我们提供了`IService`的实现类`ServiceImpl`，我们的实现类只需要继承它即可，同样也不需要重写方法。

`UserService.java`

```java
public interface UserService extends IService<User> {}
```



`UserServiceImpl.java`

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> 
    implements UserService {}
```



> 一般@Service\@Repository之类的注解使用在类上。@Autowired可以用在接口上

这样，我们就可以直接调用方法进行使用了。

使用MyBatis-Plus，为我们省去了配置映射文件，手动实现CRUD操作的步骤。





**其他说明**

`@TableName`注解和`@TableField`注解：

* `@TableName` ：可以用来指定要查找的表。因为MyBatis-Plus默认对**和Mapper接口名相同的表**进行操作。
* `TableField`：MyBatis-Plus默认要求实体类的属性必须都在表中存在，如果某个属性不在表中，可以使用这个注解进行标注。

```java
//指定要查找的表
@TableName("user")
public class User {
    @TableField(exist = false)
    private String userName;  //假设这个属性在表中不存在
}
```





## 3.3 分页功能

MyBatis-Plus为我们提供了[分页插件](https://mybatis.plus/guide/page.html)，我们可以使用它完成分页功能。

首先需要将`MybatisPlusInterceptor`这个组件设置参数，并注入容器。

```java
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join
        PaginationInnerInterceptor paginationInnerInterceptor = 
            new PaginationInnerInterceptor();
        paginationInnerInterceptor.setOverflow(true);
        paginationInnerInterceptor.setMaxLimit(100L);
        interceptor.addInnerInterceptor(paginationInnerInterceptor);
        return interceptor;
    }
}
```



然后在controller中获取`Page`类对象，实现分页功能。

```java
@Controller
public class TableController {
    @Autowired
    UserService userService;  //controller层调用service层

    //使用MyBatis-Plus中的Page对象，实现分页查询。
    @GetMapping("/dynamic_table")
    public String dynamic_table(@RequestParam(value = "pn",defaultValue = "1")
                                Integer pn, Model model) {

        Page<User> userPage = new Page<>(pn, 2); //指定当前页码和每页数量
        Page<User> page = userService.page(userPage, null);

        //可以获取page对象的各种参数，比如总页数，当前页之类的
        //long current = page.getCurrent();
        //long pages = page.getPages();
        //long total = page.getTotal();
        //List<User> records = page.getRecords();
        model.addAttribute("page",page);
        return "table/dynamic_table";
    }

    /**
        * 删除指定id的数据，然后重定向到当前页。
        * @param id 使用RESTFul风格传递，因此使用@PathVariable注解
        * @param pn 页码使用参数传递，因此使用@RequestParam绑定
        * @param ra RedirectAttributes类型用于给重定向时添加参数
        * @return
        */
    @GetMapping("/user/delete/{id}")
    public String deleteUser(@PathVariable("id") Long id,
                             @RequestParam(value="pn",defaultValue="1") Integer pn,
                             RedirectAttributes ra){

        //RedirectAttributes用于给重定向添加参数
        userService.removeById(id);
        ra.addAttribute("pn",pn);
        return "redirect:/dynamic_table";
    }
}

```



前端页面从`Page`对象中取数据，并分页显示：

![](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/springboot-sql_1.png)

