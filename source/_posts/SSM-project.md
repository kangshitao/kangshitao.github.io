---
title: SSM框架整合
excerpt: Spring、SpringMVC、MyBatis整合案例
mathjax: true
date: 2021-07-04 19:31:36
tags: ['SSM','Spring','SpringMVC','MyBatis']
categories: SSM框架
keywords: SSM整合,Spring,SpringMVC,MyBatis
---



# 一、环境配置

整合SSM框架，实现对书籍的增删查改功能。

运行环境准备，数据库建表，Maven依赖包导入，基本结构搭建。

## 1、运行环境

* IDEA 2020.3
* MySQL  5.1.47
* TomCat 9.0.46
* Maven 3.8.1



## 2、数据库建表

创建数据库`ssmbuild`和表`books`：

```mysql
CREATE DATABASE `ssmbuild`;
USE `ssmbuild`;

DROP TABLE IF EXISTS `books`;

CREATE TABLE `books` (
    `bookID` INT(10) NOT NULL AUTO_INCREMENT COMMENT '书id',
    `bookName` VARCHAR(100) NOT NULL COMMENT '书名',
    `bookCounts` INT(11) NOT NULL COMMENT '数量',
    `detail` VARCHAR(200) NOT NULL COMMENT '描述',
    KEY `bookID` (`bookID`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT  INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`)VALUES
(1,'Java',1,'Java language'),
(2,'MySQL',10,'MySQL database'),
(3,'Linux',3,'Linux opereation system');
```



## 3、项目基本环境

1、在IDEA中新建Maven项目，并添加web支持（也可以使用web模版创建项目）。

> 如果先创建Maven项目，后添加的web支持，需要在在Project-Structure->Artifacts中给web项目在WEB-INF目录下新建lib目录，将所有jar包添加到lib目录。

2、导入相关的pom依赖。

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.kang</groupId>
    <artifactId>SSMbuild</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <!-- SSM整合需要的依赖
    junit，数据库驱动，连接池，servlet，jsp，mybatis，mybatis-spring，spring
    -->

    <dependencies>
        <!--Junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <!--数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!-- 数据库连接池,使用druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.22</version>
        </dependency>


        <!--Servlet - JSP -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!--Mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <!-- mybatis-spring，用于整合Spring和MyBatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.2</version>
        </dependency>

        <!--Spring-mvc和jdbc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>

        <!-- 配置log4j日志 -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
    </dependencies>



    <!-- 3、Maven资源过滤设置 -->
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>

</project>
```



3、Maven资源过滤设置，确保所有的配置文件都能够编译到out文件夹。

4、创建项目基本结构和配置文件。文件结构如图：

![SSM项目基本文件结构](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/SSM-project_1.png)

为了配置结构清晰，我们尽量将不同功能的配置文件单独写，使用引入的方式整合多个配置文件。

最后在Spring的总配置文件`applicationContext.xml`中引入其他三个Spring配置文件。





# 二、MyBatis相关

MyBatis相关的配置，主要是配置数据库连接的配置文件，以及Mapper映射配置文件。

**1、数据库配置文件**

`database.properties`

```properties
# 这里的名字必须加'jdbc.'前缀
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=false&useUnicode=true&characterEncoding=utf8
jdbc.username=root
jdbc.password=123456
```



**2、编写MyBatis核心配置文件**

`mybatis.config.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <settings>
        <!--日志工厂实现-->
        <setting name="logImpl" value="LOG4J"/>
    </settings>
    <!-- 为pojo包配置别名 -->
    <typeAliases>
        <package name="com.kang.pojo"/>
    </typeAliases>

    <!-- 配置数据源的语句交给spring,在spring-dao配置文件中 -->

    <!--Mapper映射，其实也可以交给spring配置-->
    <mappers>
        <mapper resource="com/kang/dao/BookMapper.xml"/>
    </mappers>
</configuration>
```



**3、编写数据库对应的实体类**

`com.kang.pojo.Books.java`

```java
package com.kang.pojo;

public class Books {
    private int bookID;
    private String bookName;
    private int bookCounts;
    private String detail;

    public Books() {
    }

    public Books(int bookID, String bookName, int bookCounts, String detail) {
        this.bookID = bookID;
        this.bookName = bookName;
        this.bookCounts = bookCounts;
        this.detail = detail;
    }

    public int getBookID() {
        return bookID;
    }

    public void setBookID(int bookID) {
        this.bookID = bookID;
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    public int getBookCounts() {
        return bookCounts;
    }

    public void setBookCounts(int bookCounts) {
        this.bookCounts = bookCounts;
    }

    public String getDetail() {
        return detail;
    }

    public void setDetail(String detail) {
        this.detail = detail;
    }

    @Override
    public String toString() {
        return "Books{" +
            "bookID=" + bookID +
            ", bookName='" + bookName + '\'' +
            ", bookCounts=" + bookCounts +
            ", detail='" + detail + '\'' +
            '}';
    }
}
```



**4、编写DAO层的Mapper接口**

DAO层定义基本的CRUD功能，供service层实现功能使用。

`com.kang.dao.BookMapper`

```java
package com.kang.dao;
import com.kang.pojo.Books;
import java.util.List;

public interface BookMapper {
    //添加一本书
    int addBook(Books books);

    //根据id删除一本书
    int deleteBookById(int id);  

    //修改一本书的信息
    int updateBook(Books books); 

    //根据id查询
    Books queryBookById(int id);  
 
    //查询全部
    List<Books> queryAllBooks();  

    //根据书名模糊查询
    List<Books> queryBookByName(String bookName); 
}
```



**5、编写Mapper对应的配置文件**

`BookMapper.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace
命名空间，用于绑定一个对应的DAO/Mapper接口,这里和UserMapper绑定。
这样相当于写了一个实现类来实现此接口。
-->
<mapper namespace="com.kang.dao.BookMapper">
    <insert id="addBook" parameterType="Books">
        insert into books(bookName, bookCounts, detail)
        values (#{bookName},#{bookCounts},#{detail})
    </insert>

    <delete id="deleteBookById" parameterType="int">
        delete from books where bookID=#{bookID}
    </delete>

    <update id="updateBook" parameterType="Books">
        update books set bookName=#{bookName},
        bookCounts=#{bookCounts},detail=#{detail}
        where bookID=#{bookID}
    </update>

    <select id="queryBookById" parameterType="int" resultType="Books">
        select bookID,bookName,bookCounts,detail from books 
        where bookID = #{bookID}
    </select>

    <select id="queryAllBooks" resultType="Books">
        select bookID,bookName,bookCounts,detail from books;
    </select>

    <!-- 根据书名模糊查询。使用concat拼接 -->
    <select id="queryBookByName" parameterType="String" resultType="Books">
        select bookID,bookName,bookCounts,detail from books 
        where bookName like concat('%',#{bookName},'%')
    </select>
    
</mapper>
```



**6、编写Service层的接口和实现类**

service根据用户的实际需求，调用DAO层方法实现功能。简单起见，功能和DAO层相同，只是简单的CRUD功能。

`com.kang.service.BookService`

```java
package com.kang.service;
import com.kang.pojo.Books;
import java.util.List;

public interface BookService {
    int addBook(Books books);

    int deleteBookById(int id);

    int updateBook(Books books);

    Books queryBookById(int id);

    List<Books> queryAllBooks();

    List<Books> queryBookByName(String bookName);
}
```



service接口的实现类`com.kang.service.BookServiceImpl`

```java
package com.kang.service;
import com.kang.dao.BookMapper;
import com.kang.pojo.Books;
import java.util.List;

//业务层调用DAO层完成功能
public class BookServiceImpl implements BookService{
    private BookMapper bookMapper;

    //提供一个set方法，用于依赖注入
    public void setBookMapper(BookMapper bookMapper) {
        this.bookMapper = bookMapper;
    }

    @Override
    public int addBook(Books books) {
        return bookMapper.addBook(books);
    }

    @Override
    public int deleteBookById(int id) {
        return bookMapper.deleteBookById(id);
    }

    @Override
    public int updateBook(Books books) {
        return bookMapper.updateBook(books);
    }

    @Override
    public Books queryBookById(int id) {
        return bookMapper.queryBookById(id);
    }

    @Override
    public List<Books> queryAllBooks() {
        return bookMapper.queryAllBooks();
    }

    @Override
    public List<Books> queryBookByName(String bookName) {
        return bookMapper.queryBookByName(bookName);
    }
}
```



# 三、Spring相关

接下来，需要在Spring中注册bean，并整合MyBatis。这里的数据源使用druid连接池。



**1、编写Spring整合MyBatis的相关配置文件**

`spring-dao.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd 
                           http://www.springframework.org/schema/context 
                           https://www.springframework.org/schema/context/spring-context.xsd">

	<!-- 此文件用于整合spring和mybatis，在spring中配置数据源。
 	这里选用第三方数据库连接池，而不是使用Spring原生的DriverManagerDataSource
	-->

    <!-- 1.关联数据库配置文件 -->

	<!-- property-placeholder表示用一个properties文件里的内容来替换spring配置文件
	里使用${}的变量定义，比如我们把对数据库的配置信息配置在别的properties文件里，
	然后这个文件中引入即可。
    -->
    <context:property-placeholder location="classpath:database.properties"/>

    <!-- 2.配置数据源（连接池）
    常用数据库连接池：dbcp\c3p0\druid\hikari
    这里的数据源使用druid数据库连接池
    -->
    <bean id = "dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <!-- 根据不同的数据库连接池，还可以设置其他的属性，比如关闭自动提交功能 -->
        <property name="defaultAutoCommit" value="false"/>
    </bean>

    <!-- 3.获取SqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>

    <!-- 4.获取SqlSession -->
    <!--使用MapperScannerConfigurer自动扫描包的方式获取SqlSession-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 设置要扫描的包 -->
        <property name="basePackage" value="com.kang.dao"/>
        <!-- 注入sqlSessionFactory -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

</beans>
```

关于Spring和MyBatis整合详情可以参考[Spring整合MyBatis](https://kangshitao.github.io/2021/06/27/spring-basis/#%E5%8D%81%E4%B8%80%E3%80%81%E6%95%B4%E5%90%88MyBatis)



**2、Spring整合service层**

为了配置结构清晰，我们将service层的相关配置单独列为一个配置文件。

`spring-service.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:comtext="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd 
                           http://www.springframework.org/schema/context 
                           https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 此文件主要用于配置声明式事务相关的内容 -->

    <!-- 扫描service下的包，使其注解生效 -->
    <comtext:component-scan base-package="com.kang.service"/>

    <!-- 2.将所有的业务类，注入到Spring，可以通过配置，或者注解实现 -->
    <bean id="bookServiceImpl" class="com.kang.service.BookServiceImpl">
        <!-- bookMapper是MapperScannerConfigurer帮我们自动创建的接口代理实现类 -->
        <property name="bookMapper" ref="bookMapper"/>
    </bean>

    <!-- 3.声明式事务配置 -->
    <bean id="transactionManager" 
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 4.根据需求配置是否注入AOP事务 -->
</beans>
```



# 四、SpringMVC相关

配置完Spring和MyBatis层，已经可以实现和数据库的连接了。最后是配置SpringMVC的相关配置。

## 4.1 配置SpringMVC

**1、配置web.xml**

`web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- 配置DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <!-- 这里引用的配置文件应该是总的applicationContext.xml文件 -->
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <!-- 启动级别 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- 配置过滤器，用于处理乱码 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- 为了安全起见，可以配置会话的有效期 -->
    <session-config>
        <!-- 比如设置会话有效期为15分钟 -->
        <session-timeout>15</session-timeout>
    </session-config>
</web-app>
```

web.xml配置文件主要是配置web相关的内容，比如请求分发器和过滤器。要注意的是，这里请求分发器关联的文件应该是总的Spring配置文件，在这里是`applicationContext.xml`配置文件。



**2、配置SpringMVC的配置文件**

`spring-mvc.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           https://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/mvc
                           https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 1.注解驱动 -->
    <mvc:annotation-driven/>
    <!-- 2.静态资源过滤 -->
    <mvc:default-servlet-handler/>
    <!-- 3.扫描包，使注解生效 -->
    <context:component-scan base-package="com.kang.controller"/>

    <!-- 4.配置视图解析器 -->
    <bean id="internalResourceViewResolver" 
          class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 指定视图解析器要匹配的前缀和后缀 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```



**最后，别忘了将spring配置文件整合到总的配置文件中**：

`applicationContext.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 引入其他的三个spring配置文件 -->
    <import resource="classpath:spring-service.xml"/>
    <import resource="classpath:spring-dao.xml"/>
    <import resource="classpath:spring-mvc.xml"/>
</beans>
```

至此，配置文件已经配置结束，可以编写测试类测试是否连接上数据库，以及各个方法是否正确。



## 4.2 阶段性测试

编写测试类，测试整体是否配置成功：

```java
import com.kang.pojo.Books;
import com.kang.service.BookService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class queryBookByIdTest {
    @Test
    public void test(){
        ApplicationContext context = new ClassPathXmlApplicationContext
            ("applicationContext.xml");
        BookService bookService = context.getBean("bookServiceImpl", 
                                                  BookService.class);
        Books books = bookService.queryBookById(3);
        System.out.println(books.toString());
    }
}
```



如果正确查询到结果，说明配置成功：

```java
Books{bookID=3, bookName='Linux', bookCounts=3, detail='Linux operation system'}
```

可以继续对其他方法进行测试。这里不再列举。



# 五、编写Controller和前端页面

经过上述一系列配置，已经可以在后端完成指定的功能。接下来就要和前端进行交互，即实现SpringMVC框架中的控制器。

控制层和前端页面交互，因此需要结合前端页面，实现功能。以下是简单的增删查改功能的实现。



## 5.1 编写Controller控制器

SpringMVC中的控制器用于处理请求 。controller层调用service层，处理请求。

`com.kang.controller.BookController`

```java
package com.kang.controller;

import com.kang.pojo.Books;
import com.kang.service.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

import java.awt.print.Book;
import java.util.List;

@Controller
@RequestMapping("/book")
public class BookController {
    //controller调用service层。设置自动装配
    @Autowired
    @Qualifier("bookServiceImpl")
    private BookService bookService;

    //查询全部书籍，并返回书籍展示页面视图
    @RequestMapping("/allBook")
    public String bookList(Model model){
        List<Books> books = bookService.queryAllBooks();
        model.addAttribute("bookList",books);
        return "allBook";
    }

    //跳转到查询页面
    @RequestMapping("/toAddBook")
    public String toAddBook(Model model){
        return "addBook";
    }

    //addBook
    @RequestMapping("/addBook")
    public String addBook(Books books){
        bookService.addBook(books);
        //添加完后，重定向到书籍展示页面
        return "redirect:/book/allBook"; 
    }

    //跳转到修改页面
    @RequestMapping("/toUpdateBook")
    public String toUpdateBook(int id,Model model){
        //通过前端传递过来的id查询出来对应的book信息，反馈到前端
        Books books = bookService.queryBookById(id);
        model.addAttribute("bookToUpdate",books);
        return "updateBook";
    }

    //updateBook
    @RequestMapping("/updateBook")
    public String updateBook(Books books){
        //前端表单的内容提交到这里，会根据name对应Books的属性
        bookService.updateBook(books);
        return "redirect:/book/allBook";
    }

    //deleteBook
    @RequestMapping("/deleteBook/{bookId}")
    //使用RESTfull风格传递数据
    public String deleteBook(@PathVariable("bookId") int id){
        bookService.deleteBookById(id);
        return "redirect:/book/allBook";
    }
    //搜索书籍
    @RequestMapping("/searchBookByName")
    public String searchBookByName(String queryBookName,Model model){
        List<Books> books = bookService.queryBookByName(queryBookName);
        //将查询到的结果重新写入到页面
        model.addAttribute("bookList",books);
        return "allBook";
    }
}
```



控制层的代码是根据客户端的功能一步步编写的，这里一次性列出了全部代码。

这里对基本的功能做简单的实现，仅供参考。



## 5.2 编写页面

### 5.2.1 首页

首页内容：

![首页](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/SSM-project_2.png)



`index.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>首页</title>
        <style>
            h3{
                width: 180px;
                height: 38px;
                margin:100px auto;
                text-align: center;
                line-height: 38px;
                background: deepskyblue;
                border-radius: 10px;
            }
            a{
                text-decoration: none;
                color: black;
                font-size: 18px;
            }
        </style>
    </head>
    <body>
        <h3>
            <a href="${pageContext.request.contextPath}/book/allBook">
                进入书籍展示页面
            </a>
        </h3>
    </body>
</html>
```

需要在`BookController`中编写`book/allBook`请求的代码。



### 5.2.2 书籍展示



展示页面内容：

![书籍展示页面](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/SSM-project_3.png)



`WEB-INF/jsp/allBook.jsp`

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>书籍展示页面</title>
        <%--使用BootStrap美化界面，使用CDN加速--%>
        <link href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">

    </head>
    <body>
        <!-- 使用BootStrap框架美化页面 -->
        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="page-header">
                        <h1>
                            <small>书籍列表----显示所有书籍</small>
                        </h1>
                    </div>
                </div>
            </div>

            <div class="row">
                <div class="col-md-4 column">
                    <%-- 跳转到toAddBook --%>
                    <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/toAddBook">新增书籍</a>
                </div>
                <div class="col-md-4 column"></div>
                <%-- 查询书籍 --%>
                <div class="form-inline">
                    <form action="${pageContext.request.contextPath}/book/searchBookByName" method="post" style="float: right">
                        <input type="text" name="queryBookName" class="form-control" placeholder="请输入要查询的书籍名称">
                        <input type="submit" value="查询" class="btn btn-primary">
                    </form>
                </div>
            </div>

            <div class="row clearfix">
                <div class="col-md-12 column">
                    <table class="table table-hover table-striped">
                        <thead>
                            <tr>
                                <th>书籍编号</th>
                                <th>书籍名称</th>
                                <th>书籍数量</th>
                                <th>书籍详情</th>
                                <th>操作</th>
                            </tr>
                        </thead>
                        <%-- 书籍需要从前端遍历出来 --%>
                        <tbody>
                            <c:forEach var="book" items="${bookList}">
                                <tr>
                                    <td>${book.bookID}</td>
                                    <td>${book.bookName}</td>
                                    <td>${book.bookCounts}</td>
                                    <td>${book.detail}</td>
                                    <td>
                                        <%-- 跳转到修改页面的时候，传递bookID参数 --%>
                                        <a href="${pageContext.request.contextPath}/book/toUpdateBook?id=${book.bookID}">修改</a>
                                        &nbsp;| &nbsp;
                                        <%-- 使用RESTFull风格跳转 --%>
                                        <a href="${pageContext.request.contextPath}/book/deleteBook/${book.bookID}">删除</a>
                                    </td>
                                </tr>
                            </c:forEach>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </body>
</html>
```

需要在`BookController`中编写处理`book/toAddBook`、`book/toUpdateBook`、`book/deleteBook`、`book/searchBookByName`请求的代码。



### 5.2.3 添加书籍

添加书籍页面：

![添加书籍页面](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/SSM-project_4.png)

`WEB-INF/jsp/addBook.jsp`

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>添加书籍页面</title>
        <%--使用BootStrap美化界面，使用CDN加速--%>
        <link href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">

    </head>
    <body>
        <!-- 使用BootStrap框架美化页面 -->
        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="page-header">
                        <h1>
                            <small>书籍列表----新增书籍</small>
                        </h1>
                    </div>
                </div>
            </div>

            <form action="${pageContext.request.contextPath}/book/addBook" method="post" >
                <div class="form-group">
                    <label for="bookname">书籍名称</label>
                    <input type="text" class="form-control" id="bookname" name="bookName" required>
                </div>
                <div class="form-group">
                    <label for="bookcount">书籍数量</label>
                    <input type="text" class="form-control" id="bookcount" name="bookCounts" required>
                </div>
                <div class="form-group">
                    <label for="bookdetail">书籍描述</label>
                    <input type="text" class="form-control" id="bookdetail" name="detail" required>
                </div>
                <div class="form-group">
                    <input type="submit" class="form-control" value="确认添加">
                </div>
            </form>
        </div>
    </body>
</html>
```



需要在`BookController`中编写处理`book/addBook`请求的代码。



### 5.2.4 修改书籍

修改书籍页面：

![修改书籍页面](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/SSM-project_5.png)



`WEB-INF/jsp/updateBook.jsp`

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>修改数据页面</title>
        <%--使用BootStrap美化界面，使用CDN加速--%>
        <link href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">

    </head>
    <body>
        <!-- 使用BootStrap框架美化页面 -->
        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="page-header">
                        <h1>
                            <small>修改书籍</small>
                        </h1>
                    </div>
                </div>
            </div>

            <form action="${pageContext.request.contextPath}/book/updateBook" method="post" >
                <%-- 设置一个隐藏域来传递id,用于更新内容 --%>
                <input type="hidden" name="bookID" value="${bookToUpdate.bookID}">

                <div class="form-group">
                    <label for="bookname">书籍名称</label>
                    <input type="text" class="form-control" id="bookname" name="bookName" value="${bookToUpdate.bookName}" required>
                </div>
                <div class="form-group">
                    <label for="bookcount">书籍数量</label>
                    <input type="text" class="form-control" id="bookcount" name="bookCounts" value="${bookToUpdate.bookCounts}" required>
                </div>
                <div class="form-group">
                    <label for="bookdetail">书籍描述</label>
                    <input type="text" class="form-control" id="bookdetail" name="detail" value="${bookToUpdate.detail}" required>
                </div>
                <div class="form-group">
                    <input type="submit" class="form-control" value="确认修改">
                </div>
            </form>
        </div>
    </body>
</html>
```



需要在`BookController`中编写处理`book/updateBook`请求的代码。



### 5.2.5 删除书籍

在书籍展示页面已经添加了删除选项：

```jsp
<a href="${pageContext.request.contextPath}/book/deleteBook/${book.bookID}">删除</a>
```

点击删除的时候，会向服务器发起请求，传递`bookID`给控制层，控制层接受请求并执行删除功能。

也就是下面的代码：

```java
@RequestMapping("/deleteBook/{bookId}")
//使用RESTfull风格传递数据
public String deleteBook(@PathVariable("bookId") int id){
    bookService.deleteBookById(id);
    //删除完后会重定向到书籍展示页面
    return "redirect:/book/allBook";
}
```



### 5.2.6 搜索功能

实现一个简单的根据书名模糊搜索的功能，书籍展示页面中，查询书籍的代码为：

```jsp
<div class="form-inline">
    <form action="${pageContext.request.contextPath}/book/searchBookByName" method="post" style="float: right">
        <input type="text" name="queryBookName" class="form-control" 
               placeholder="请输入要查询的书籍名称">
        <input type="submit" value="查询" class="btn btn-primary">
    </form>
</div>
```

对应地，我们应该在控制层写一个处理`book/searchBookByName`请求的代码，也就是下面的代码：

```java
//搜索书籍
@RequestMapping("/searchBookByName")
public String searchBookByName(
    @RequestParam("queryBookName")String queryBookName, Model model){
    List<Books> books = bookService.queryBookByName(queryBookName);
    //将查询到的结果重新写入到页面
    model.addAttribute("bookList",books);
    return "allBook";
}
```





# 六、总结

上述只是SSM框架整合的一个最简单的应用，只是基本的配置流程。代码参考：

实际开发中，可以根据需求添加相应的配置。比如：

* service层的事务管理功能
* 配置过滤器，解决前端向后端传递数据时的乱码问题。
* 配置消息转换器，处理控制器返回JSON数据时的乱码问题。

根据项目的实际需求，添加具体的功能。

其中的一些配置方式也有多种，并不是固定的。比如依赖注入的方式，MyBatis中获取` SqlSession`的方式等，也需要根据实际情况选取实现方式。



---

参考内容：[狂神说SSM框架系列连载](http://mp.weixin.qq.com/mp/homepage?__biz=Mzg2NTAzMTExNg==&hid=3&sn=456dc4d66f0726730757e319ffdaa23e&scene=18#wechat_redirect)