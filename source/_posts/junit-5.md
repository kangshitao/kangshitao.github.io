---
title: Junit5与单元测试
excerpt: Junit5的使用
mathjax: true
date: 2021-07-10 11:30:06
tags: Junit5
categories: Tools
keywords: JUnit5,单元测试
---



# 一、JUnit5 介绍

## 1.1 介绍

Junit5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

**JUnit Platform**：Junit Platform是在JVM上启动测试框架的基础，不仅支持Junit自制的测试引擎，其他测试引擎也都可以接入。

**JUnit Jupiter**：JUnit Jupiter提供了JUnit5的新的编程模型，是JUnit5新特性的**核心**。内部包含了一个**测试引擎**，用于在Junit Platform上运行。

**JUnit Vintage**：由于JUint已经发展多年，为了照顾老的项目，JUnit Vintage提供了兼容JUnit4.x,Junit3.x的测试引擎。

SpringBoot2.4以上版本移除了JUnit Vintage的依赖，如果需要兼容JUnit4，需要自行引入依赖。



## 1.2 JUnit5常用注解

与JUnit4的差异：[Annotations](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)

JUnit5的@Test：

```
import org.junit.jupiter.api.Test;
```

JUnit4的@Test

```
import org.junit.Test;
```



SpringBoot中使用JUnit5，只需要在测试类上使用`@SpringBootTest`注解，使用这个注解以后，测试类就具有SpringBoot的功能，比如自动装配、事务功能等。包括`@Autowired`、`@Transactional`等。

```java
//测试类使用@SpringBootTest标注
@SpringBootTest
class Springboot05WebAdminApplicationTest {
    @Autowired
    UserMapper userMapper;
    
    //测试方法使用@Test标注
    @Test
    public void userMapperTest(){
    }
}
```



> JUnit4中实现上述功能则需要@SpringBootTest+@RunWith(SpringRunner.class)



- **@Test **:表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- **@ParameterizedTest **：表示方法是参数化测试

- **@RepeatedTest **：表示方法可重复执行
- **@DisplayName**：为测试类或者测试方法设置展示名称

- **@BeforeEach **：表示在每个单元测试之前执行
- **@AfterEach **：表示在每个单元测试之后执行

- **@BeforeAll **：表示在所有单元测试之前执行；测试方法必须是静态的，或者测试类标注了@TestInstance(Lifecycle.PER_CLASS)
- **@AfterAll **：表示在所有单元测试之后执行；测试方法必须是静态的，或者测试类标注了@TestInstance(Lifecycle.PER_CLASS)

- **@Tag **：表示单元测试类别，类似于JUnit4中的@Categories
- **@Disabled **：表示测试类或测试方法不执行，类似于JUnit4中的@Ignore

- **@Timeout **：表示测试方法运行如果超过了指定时间将会返回错误
- **@ExtendWith**：为测试类或测试方法提供扩展类引用



# 二、断言

断言是测试方法中的核心，用来对测试需要满足的条件进行验证。

如果一个方法有多个断言，如果前边的断言失败，后边的代码都不能执行。

## 2.1 简单断言

用来对单个值进行简单的验证：

| 方法            | 说明                                 |
| --------------- | ------------------------------------ |
| assertEquals    | 判断两个对象或两个原始类型是否相等   |
| assertNotEquals | 判断两个对象或两个原始类型是否不相等 |
| assertSame      | 判断两个对象引用是否指向同一个对象   |
| assertNotSame   | 判断两个对象引用是否指向不同的对象   |
| assertTrue      | 判断给定的布尔值是否为 true          |
| assertFalse     | 判断给定的布尔值是否为 false         |
| assertNull      | 判断给定的对象引用是否为 null        |
| assertNotNull   | 判断给定的对象引用是否不为 null      |

```java
@Test
@DisplayName("simple assertion")
public void simple() {
    assertEquals(3, 1 + 2, "simple math");
    assertNotEquals(3, 1 + 1);

    assertNotSame(new Object(), new Object());
    Object obj = new Object();
    assertSame(obj, obj);

    assertFalse(1 > 2);
    assertTrue(1 < 2);

    assertNull(null);
    assertNotNull(new Object());
}
```



## 2.2 数组断言

使用`assertArrayEquals`方法来判断两个对象或原始类型的数组是否相等：

```java
@Test
@DisplayName("array assertion")
public void array() {
    assertArrayEquals(new int[]{1, 2}, new int[] {1, 2});
}
```



## 2.3 组合断言

`assertAll`方法接受多个`org.junit.jupiter.api.Executable`函数式接口的实例作为要验证的断言，可以通过 lambda表达式很容易的提供这些断言:

```java
@Test
@DisplayName("assert all")
public void all() {
    assertAll("Math",
              () -> assertEquals(2, 1 + 1),
              () -> assertTrue(1 > 0)
             );
}
```



## 2.4 异常断言

JUnit5提供了一种新的异常断言方式`Assertions.assertThrows()`，配合函数式编程就可以进行使用：

```java
@Test
@DisplayName("异常测试")
public void exceptionTest() {
    ArithmeticException exception = Assertions.assertThrows(
        //扔出断言异常
        ArithmeticException.class, () -> System.out.println(1 % 0));

}
```



## 2.5 超时断言

Junit5还提供了`Assertions.assertTimeout()` 为测试方法设置了超时时间

```java
@Test
@DisplayName("超时测试")
public void timeoutTest() {
    //如果测试方法时间超过1s将会异常
    Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(500));
}
```



## 2.6 快速失败

通过`fail`方法直接使得测试失败：

```java
@Test
@DisplayName("fail")
public void shouldFail() {
    fail("This should fail");
}
```





# 三、 前置条件

JUnit 5 中的前置条件（assumptions）类似于断言，不同之处在于**不满足的断言会使得测试方法失败**，而**不满足的前置条件只会使得测试方法的执行终止**。

前置条件可以看成是测试方法执行的前提，当该前提不满足时，就没有继续执行的必要。

```java
@DisplayName("前置条件")
public class AssumptionsTest {
    private final String environment = "DEV";

    @Test
    @DisplayName("simple")
    public void simpleAssume() {
        assumeTrue(Objects.equals(this.environment, "DEV"));
        assumeFalse(() -> Objects.equals(this.environment, "PROD"));
    }

    @Test
    @DisplayName("assume then do")
    public void assumeThenDo() {
        assumingThat(
            Objects.equals(this.environment, "DEV"),
            () -> System.out.println("In DEV")
        );
    }
}
```

`assumeTrue`和`assumFalse`确保给定的条件为`true`或`false`，不满足条件会使得测试执行终止。

`assumingThat`的参数是表示条件的布尔值和对应的`Executable`接口的实现对象。只有条件满足时，Executable 对象才会被执行；当条件不满足时，测试执行并不会终止。



# 四、嵌套测试

JUnit5可以通过 Java 中的内部类和`@Nested`注解实现嵌套测试

嵌套测试情况下，外层的Test不能驱动内层的`@BeforeEach`、`@AfterAll`之类的方法提前/之后运行。

而内层的Test可以驱动外层的`@BeforeEach`、`@AfterAll`之类的方法。也就是说各个层是从外向内运行的，Before、After之类的方法也只是在本层生效。

```java
@DisplayName("A stack")
class TestingAStackDemo {
    Stack<Object> stack;
    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {
        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```







# 五、参数化测试

参数化测试是JUnit5很重要的一个新特性，它使得用不同的参数多次运行测试成为了可能，也为我们的单元测试带来许多便利。

利用`@ValueSource`等注解，指定入参，我们将可以使用不同的参数进行多次单元测试，而不需要每新增一个参数就新增一个单元测试，省去了很多冗余代码。

**@ValueSource**: 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型

**@NullSource**: 表示为参数化测试提供一个null的入参

**@EnumSource**: 表示为参数化测试提供一个枚举入参

**@CsvFileSource**：表示读取指定CSV文件内容作为参数化测试入参

**@MethodSource**：表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流)



参数化测试可以**支持外部的各类入参**。如CSV、YML、JSON 文件甚至方法的返回值也可以作为入参。只需要去实现`ArgumentsProvider`接口，任何外部文件都可以作为它的入参。

```java
@ParameterizedTest
@ValueSource(strings = {"one", "two", "three"})
@DisplayName("参数化测试1")
public void parameterizedTest1(String string) {
    System.out.println(string);
    Assertions.assertTrue(StringUtils.isNotBlank(string));
}


@ParameterizedTest
@MethodSource("method")    //指定方法名
@DisplayName("方法来源参数")
public void testWithExplicitLocalMethodSource(String name) {
    System.out.println(name);
    Assertions.assertNotNull(name);
}

static Stream<String> method() {
    return Stream.of("apple", "banana");
}
```



# 六、从JUnit4到JUnit5

在进行迁移的时候需要注意如下的变化：

- 注解在`org.junit.jupiter.api`包中，断言在`org.junit.jupiter.api.Assertions`类中，前置条件在 `org.junit.jupiter.api.Assumptions`类中。
- 把`@Before` 和`@After` 替换成`@BeforeEach` 和`@AfterEach`。

- 把`@BeforeClass` 和`@AfterClass` 替换成`@BeforeAll` 和`@AfterAll`。
- 把`@Ignore` 替换成`@Disabled`。

- 把`@Category`替换成`@Tag`。
- 把`@RunWith`、`@Rule` 和`@ClassRule` 替换成`@ExtendWith`。



参考官方文档：[Migrating from JUnit 4](https://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4)



---

参考链接：https://www.yuque.com/atguigu/springboot/ksndgx

