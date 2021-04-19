---
title: Java学习笔记16-Java8的其它新特征
excerpt: Lambda表达式，函数式接口，方法引用和构造器引用，Stream API，Optional类
mathjax: true
date: 2021-04-18 23:00:45
tags: Java
categories: Java
keywords: Java, Lambda表达式,函数式接口,方法引用,构造器引用,Stream API,Optional
---

# Java 8 简介

Java 8的特征：

* 速度更快
* 代码更少，因为新加入了Lambda表达式
* 强大的Stream API
* 便于并行
* 最大化减少空指针异常：Optional
* Nashorn引擎，允许在JVM上运行JS应用。

Java 8主要更新内容参考官方说明 [What's New in JDK 8](https://www.oracle.com/java/technologies/javase/8-whats-new.html)

新特性总结：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1601_1.png"/>

**并行流与串行流**
**并行流**就是把一个内容分成多个数据块，并用不同的线程分别处理每个数据块的流。相比较串行的流，并行的流可以很大程度上提高程序的执行效率。

Java 8中将并行进行了优化，可以很容易的对数据进行并行操作。Stream API可以使用parallel()与sequential()方法在并行流与顺序流之间进行切换。

# 一、Lambda表达式

Lambda表达式是JDK 8中新的语法，操作符为`->`，一个Lambda表达式由3部分构成：

`左侧->右侧`

其中操作符左侧指定Lambda表达式需要的参数列表，右侧指定了Lambda体，是抽象方法的实现逻辑，即Lambda表达式要执行的功能。

Lambda表达式的**本质是函数式接口的实例**，其作为接口的实例出现。

具体使用方式：

```java
//情况1:无参，无返回值
Runnable r = ()->{System.out.println("hello");};

//情况2，一个参数，无返回值
Consumer<String> con = (String str) ->{System.out.println(str);};

//情况3，数据类型可以省略，编译器可以推断，即类型推断
Consumer<String> con = (str) ->{System.out.println(str);};

//情况4，如果只有一个参数，参数的小括号可以省略
Consumer<String> con = str ->{System.out.println(str);};

//情况5，可以有两个及以上的参数，多条执行语句，有返回值
Comparator<Integer> com = (x,y)->{
    System.out.println("hello");
    return Integer.compare(x,y);
};

//情况6，只有一条语句时，{}和return可以省略
Comparator<Integer> com = (x,y)->Integer.compare(x,y);
```

总结：

* 左边：lambda形参列表的参数类型可以省略(类型推断)；如果lambda形参列表只有一个参数，其一对()也可以省略。

*    右边：lambda体应该使用一对`{}`包裹；如果lambda体只有一条执行语句（可能是return语句），省略`{}`和`return`。如果有返回值，则`return`和`{}`必须都有。因此`{}`和`return`要么都有，要么都没有。

# 二、函数式(Functional)接口

**函数式接口指的是只包含一个抽象方法的接口**。可以用注解`@FunctionalInterface`检查。

可以通过Lambda表达式创建函数式接口的对象，执行体就是实现抽象方法的代码。如果Lambda表达式抛出非运行时异常，该异常需要在目标接口的抽象方法上声明。

**Lambda表达式是一个函数式接口的实例**，只要一个对象是函数式接口的实例对象，就可以用Lambda表达式来表达。所以说**实现函数式接口的匿名实现类的表示方式都可以改成Lambda表达式**的形式。



`java.util.function`包下定义了Java8的丰富的函数式接口，其中有四种核心的函数式接口：

| 函数式接口                 | 参数类型 | 返回类型  | 用途                                                         |
| :------------------------- | :------: | :-------: | :----------------------------------------------------------- |
| 消费型接口`Consumer<T>`    |   `T`    |  `void`   | 对类型为`T`的对象应用操作，包含方法`void accept(T t)`，相当于消费者 |
| 供给型接口`Supplier<T>`    |    无    |    `T`    | 返回类型为`T`的对象，包含方法：`T get()`，相当于供给者，用户获取对象 |
| 函数型接口`Function<T, R>` |   `T`    |    `R`    | 对类型为`T`的对象应用操作，并返回结果是`R`类型的对象。包含方法：`R apply(T t)`，用于类型转换 |
| 断定型接口`Predicate<T>`   |   `T`    | `boolean` | 确定类型为`T`的对象是否满足某约束，并返回`boolean`值。包含方法：`boolean test(T t)`，用于判断 |

其他函数式接口：

| 函数式接口                                                   |           参数类型            |           返回类型            | 用途                                                         |
| :----------------------------------------------------------- | :---------------------------: | :---------------------------: | :----------------------------------------------------------- |
| `BiFunction<T,U,R>`                                          |           `T`, `U`            |              `R`              | 对类型为`T`,`U`参数应用操作，返回R类型的结果。包含方法为：`R apply(T t, U u);` |
| `UnaryOperator<T>`<br />(`Function`子接口)                   |              `T`              |              `T`              | 对类型为`T`的对象进行一元运算，并返回T类型的结果。包含方法为：`T apply(T t);` |
| `BinaryOperator<T>`<br />(`BiFunction`子接口)                |            `T`,`T`            |              `T`              | 对类型为`T`的对象进行二元运算，并返回`T`类型的结果。<br />包含方法为：`T apply(Tt 1,Tt 2);` |
| `BiConsumer<T,U>`                                            |           `T`, `U`            |            `void`             | 对类型为`T`,`U`参数应用操作。<br/>包含方法为：`void accept(T t,U u);` |
| `BiPredicate<T,U>`                                           |           `T`, `U`            |           `boolean`           | 包含方法为：`boolean test(T t,U u);`                         |
| `ToIntFunction<T>`<br />`ToLongFunction<T>`<br />`ToDoubleFunction<T>` |              `T`              | `int`<br/>`long`<br/>`double` | 分别计算`int`、`long`、`double`值的函数                      |
| `IntFunction<R>`<br />`LongFunction<R>`<br />`DoubleFunction<R>` | `int`<br/>`long`<br/>`double` |              `R`              | 参数分别为`int`、`long`、`double`类型的函数                  |

使用实例：

```java
public class LambdaTest {
    @Test
    public void test1(){
        //方法一，使用匿名实现类，重写accept方法。
        happyTime(500, new Consumer<Double>() {
            @Override
            public void accept(Double m) {
                System.out.println("价格为：" + m);
            }
        });
        System.out.println("********************");
        //方法二，使用Lambda表达式，执行体就是相当于重写accept方法。
        happyTime(400,money -> System.out.println("价格为：" + money));
    }
    //happyTime方法第二个参数是消费型接口类型的实例
    public void happyTime(double money, Consumer<Double> con){
        con.accept(money);
    }
}
```



# 三、方法引用与构造器引用

## 1、方法引用

当要传给Lambda体的操作，已经有实现的方法了，可以使用**方法引用**。

方法引用，本质上是Lambda表达式，可以认为是Lambda表达式的一个语法糖。所以**方法引用也是函数式接口的实例**。方法引用的操作符为`::`

方法引用有三种方式：

* 对象::实例方法名
* 类::静态方法名
* 类::实例方法名

对于前两种方式，要求接口中的**抽象方法的形参列表和返回值类型**必须和**方法引用的方法的形参列表和返回值类型**相同。

使用举例：

```java
/*
自定义Employee类。包含空参构造器和带参构造器。
其中getName()方法定义如下
public String getName() {return name;}
*/
public class MethodRefTest {
	// 情况一：对象 :: 实例方法
	/*例1：
	Consumer中的void accept(T t)
	PrintStream中的void println(T t)，二者都是返回某个类型的对象。
	*/
	@Test
	public void test1() {
		//Lambda表达式写法
		Consumer<String> con1 = str -> System.out.println(str);
		con1.accept("北京");
		//方法引用写法
		PrintStream ps = System.out;
		Consumer<String> con2 = ps::println;
		//Consumer<String> con2 = System.out::println;  //和上面相同
		con2.accept("beijing");
	}
	/*
	例2，
	Supplier中的T get()
	Employee类中的String getName()，二者都是返回某个类型的对象。
	*/
	@Test
	public void test2() {
		Employee emp = new Employee(1001,"Tom",23,5600);
		//Lambda表达式写法
		Supplier<String> sup1 = () -> emp.getName();
		System.out.println(sup1.get());
		//方法引用写法
		Supplier<String> sup2 = emp::getName;
		System.out.println(sup2.get());
	}

	// 情况二：类 :: 静态方法
	/*例1：
	Comparator中的int compare(T t1,T t2)
	Integer中的int compare(T t1,T t2)，二者实现方法相同
	*/
	@Test
	public void test3() {
		//Lambda表达式写法
		Comparator<Integer> com1 = (t1,t2) -> Integer.compare(t1,t2);
		System.out.println(com1.compare(12,21));
		//方法引用写法
		Comparator<Integer> com2 = Integer::compare;
		System.out.println(com2.compare(12,3));
	}
	/*例2：
	Function中的R apply(T t)
	Math中的Long round(Double d)
	*/
	@Test
	public void test4() {
		//匿名实现类写法
		Function<Double,Long> func = new Function<Double, Long>() {
			@Override
			public Long apply(Double d) {return Math.round(d);}
		};
		//Lambda表达式写法
		Function<Double,Long> func1 = d -> Math.round(d);
		System.out.println(func1.apply(12.3));
		//方法引用写法
		Function<Double,Long> func2 = Math::round;
		System.out.println(func2.apply(12.6));
	}

	// 情况三：类 :: 实例方法
	/*例1：
	Comparator中的int comapre(T t1,T t2)
	String中的int t1.compareTo(t2)
	*/
	@Test
	public void test5() {
		//Lambda表达式写法
		Comparator<String> com1 = (s1,s2) -> s1.compareTo(s2);
		System.out.println(com1.compare("abc","abd"));
		//方法引用写法
		Comparator<String> com2 = String :: compareTo;
		System.out.println(com2.compare("abd","abm"));
	}
	/*例2：
	BiPredicate中的boolean test(T t1, T t2);
	String中的boolean t1.equals(t2)
	*/
	@Test
	public void test6() {
		//Lambda表达式写法
		BiPredicate<String,String> pre1 = (s1,s2) -> s1.equals(s2);
		System.out.println(pre1.test("abc","abc"));
		//方法引用写法
		BiPredicate<String,String> pre2 = String :: equals;
		System.out.println(pre2.test("abc","abd"));
	}
	/*例3：
	Function中的R apply(T t)
	Employee中的String getName();
	*/
	@Test
	public void test7() {
		Employee employee = new Employee(1001, "Jerry", 23, 6000);
		//Lambda表达式写法
		Function<Employee,String> func1 = e -> e.getName();
		System.out.println(func1.apply(employee));
		//方法引用写法
		Function<Employee,String> func2 = Employee::getName;
		System.out.println(func2.apply(employee));
	}
}
```



## 2、构造器引用

格式：`ClassName::new`

和方法引用类似，要求函数式接口的**抽象方法的形参列表**和**构造器的形参列表**一致。

抽象方法的返回值类型即为构造器所属的类的类型。

代码实现：

```java
public class ConstructorRefTest {
    //构造器引用
    /*例1：
    Supplier中的T get()
    Employee的空参构造器：Employee()
    */
    @Test
    public void test1(){
        //匿名实现类写法
        Supplier<Employee> sup = new Supplier<Employee>() {
            @Override
            public Employee get() {return new Employee();}
        };
        //Lambda表达式写法
        Supplier<Employee>  sup1 = () -> new Employee();
        System.out.println(sup1.get());
        //构造器引用写法
        Supplier<Employee>  sup2 = Employee :: new;
        System.out.println(sup2.get());
    }
    /*例2：
    BiFunction中的R apply(T t,U u)
    Employee的带参构造器：Employee(int id, String name)
    */
    @Test
    public void test3(){
        //Lambda表达式写法
        BiFunction<Integer,String,Employee> func1 = 
            (id,name) -> new Employee(id,name);
        System.out.println(func1.apply(1001,"Tom"));
        //构造器引用写法
        BiFunction<Integer,String,Employee> func2 = Employee :: new;
        System.out.println(func2.apply(1002,"Tom"));
    }
}
```



## 3、数组引用

格式：`type[]::new`，其中type表示数组类型。

可以将数组看成一个特殊的类，写法与构造器引用一致。

代码实现：

```java
public class ArrayRefTest {
    //数组引用
    //Function中的R apply(T t)
    @Test
    public void test4(){
        //Lambda表达式写法
        Function<Integer,String[]> func1 = length -> new String[length];
        String[] arr1 = func1.apply(5);
        System.out.println(Arrays.toString(arr1));
        //数组引用写法
        Function<Integer,String[]> func2 = String[] :: new;
        String[] arr2 = func2.apply(10);
        System.out.println(Arrays.toString(arr2));
    }
}
```



# 四、强大的Stream API

## 1、Stream简介

**Stream API ( java.util.stream)**把真正的函数式编程风格引入到Java中。这是目前为止对Java类库最好的补充，Stream API可以极大提高Java程序员的生产力。

**Stream**是数据渠道，用于操作数据源(集合、数组等)所生成的元素序列。

**Stream API**可以对集合进行操作，可以执行非常复杂的**查找**、**过滤**和**映射数据**等操作，类似于使用SQL执行的数据库查询。
也可以使用Stream API来并行执行操作。简言之，Stream API提供了一种高效且易于使用的处理数据的方式。

**Stream和Collection集合的区别**：

`Collection`是一种静态的内存数据结构。主要面向内存，存储在内存中。

`Stream`是有关计算的。主要是面向CPU，通过CPU实现计算。

**Stream的特点**：

* Stream 自己不会存储元素。
* Stream 不会改变源对象，会返回一个持有结果的新Stream，类似于视图(Collections工具类会对集合本身进行修改，比如排序操作)。
* Stream 操作是延迟执行的，即会等到需要结果的时候才执行，执行终止操作的时候才执行中间操作。



## 2、使用Stream

想要使用Stream，需要执行以下三个步骤：

* 实例化`Stream`类(`java.util.stream.Stream`)，得到`Stream`类对象`stream`。
* 中间操作。中间操作链，对数据源的数据进行处理，比如排序，筛选，映射等。
* 终止操作。执行终止操作时才会执行中间操作，并产生结果。终止操作后，`stream`不能再被使用。

## 3、Stream实例化

Stream类有四种实例化方法：

* 通过集合。Java 8的`Collection`接口被扩展，提供了两个获取`Stream`实例的方法：
  * `default Stream<E> stream()`：返回一个顺序流
  * `default Stream<E> parallelStream() `:返回一个并行流
* 通过数组。`Arrays`工具类提供了`stream()`方法，用于获取数组流，对于不同的数据类型，提供了其重载方法，返回的类型也不同：
  * `public static <T> Stream<T> stream(T[] array)`：返回`Stream`实例。
  * `public static IntStream stream(int[] array)`：对于`int`型数组，返回`IntStream`实例。
  * `public static LongStream stream(long[] array)`：对于`long`型数组，返回`LongStream`实例。
  * `public static DoubleStream stream(double[] array)`：对于`double`型数组，返回`DoubleStream`实例。
* 通过`Stream`类的`of()`方法。
  * `public static<T> Stream<T> of(T...values)`：参数可以是单个数据，或者多个数据。如果直接将数组或者是集合作为参数，默认当作一个参数。参数不能为单个`null`，但是可以是多个`null`
* 通过`Stream`类静态方法创建无限流。创建无限流有两种方式：
  * `public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)`：迭代
  * `public static<T> Stream<T> generate(Supplier<T> s)`：生成

代码实现：

```java
public class StreamAPITest {
    //创建 Stream方式一：通过集合
    @Test
    public void test1(){
        List<Employee> employees = EmployeeData.getEmployees();
        //返回一个顺序流
        Stream<Employee> stream = employees.stream();
		
        //返回一个并行流
        Stream<Employee> parallelStream = employees.parallelStream();
    }

    //创建 Stream方式二：通过数组
    @Test
    public void test2(){
        //对于int、long、double类型数组，返回特定的stream
        int[] arr = new int[]{1,2,3,4,5,6};
        IntStream stream = Arrays.stream(arr);

        //传入对象数组
        Employee e1 = new Employee(1001,"Tom");
        Employee e2 = new Employee(1002,"Jerry");
        Employee[] arr1 = new Employee[]{e1,e2};
        Stream<Employee> stream1 = Arrays.stream(arr1);
    }
    //创建 Stream方式三：通过Stream的of()
    @Test
    public void test3(){
        //注意传入的参数个数
        List<Integer> list = new ArrayList<>();
        Stream<List<Integer>> list1 = Stream.of(list);
        Stream<Object> objectStream = Stream.of(list.toArray());

        //直接传入集合或数组，当成一个参数
        int[] array = new int[3];
        Stream<int[]> array1 = Stream.of(array);
        
        Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6);
    }
    //方式四：创建无限流
    @Test
    public void test4(){
        //迭代
        //遍历前10个偶数
        Stream.iterate(0, t -> t + 2).limit(10).forEach(System.out::println);
		
        //生成
        Stream.generate(Math::random).limit(10).forEach(System.out::println);
    }
}
```



## 4、Stream中间操作

有了`Stream`类的对象以后，可以执行中间操作。中间操作可以链式操作，只有遇到终止操作的时候，中间操作才会一次性执行，称为“惰性求值”。

中间操作分成以下3种：

①**筛选与切片**

| 方法                  | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| `filter(Predicate p)` | 接收Lambda表达式，从流中排除某些元素                         |
| `distinct()`          | 筛选，通过流所生成元素的`hashCode()`和`equals()`去除重复元素 |
| `limit(long maxSize)` | 截断流，使其元素不超过给定数量                               |
| `skip(long n)`        | 跳过元素，返回一个跳过前n个元素的流。<br />若流中元素不足n个，则返回一个空流。与`limit(n)`操作互补 |

②**映射**

| 方法                            | 描述                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| `map(Functionf)`                | 接收一个函数作为参数，该函数会被应用到每个元素上，<br />并将其映射成一个新的元素。 |
| `mapToDouble(ToDoubleFunction f)` | 接收一个函数作为参数，该函数会被应用到每个元素上，<br />产生一个新的`DoubleStream`。 |
| `mapToInt(ToIntFunction f)`     | 接收一个函数作为参数，该函数会被应用到每个元素上，<br />产生一个新的`IntStream`。 |
| `mapToLong(ToLongFunction f)`   | 接收一个函数作为参数，该函数会被应用到每个元素上，<br />产生一个新的`LongStream`。 |
| `flatMap(Function f)`           | 接收一个函数作为参数，将流中的每个值都换成另一个流，<br/>然后把所有流连接成一个流 |

③**排序**

| 方法                     | 描述                           |
| :----------------------- | :----------------------------- |
| `sorted()`               | 产生一个新流，按自然顺序排序   |
| `sorted(Comparator com)` | 产生一个新流，按比较器顺序排序 |



## 5、Stream终止操作

终止操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如：`List`、`Integer`，甚至是`void`。

流进行了终止操作后，不能再次被使用。

终止操作有以下几种：

| 方法                                       | 描述                                                         |
| :----------------------------------------- | :----------------------------------------------------------- |
| `count()`                                  | 返回流中元素总数                                             |
| `max(Comparator c)`                        | 返流中最大值                                                 |
| `min(Comparator)`                          | 返回流中最小值                                               |
| `forEach(Consumer c)`                      | 内部迭代<br />(使用`Collection`接口需要用户去做迭代，称为外部迭代) |
| `reduce(T iden,BinaryOperator b)`，   规约 | 可以将流中元素反复结合起来，得到一个值。<br />返回`T`        |
| `reduce(BinaryOperator b)`，   规约        | 可以将流中元素反复结合起来，得到一个值。<br />返回`Optional`<T> |
| `collect(Collector c)`，收集               | 将流转换为其他形式。<br />接收一个`Collector`接口的实现，用于给`Stream`中元素做汇总的方法 |

> map和reduce的连接通常称为map-reduce模式。



`Collector`接口（收集器）中方法的实现决定了如何对流执行收集的操作(如收集到`List`、`Set`、`Map`)。
另外，`Collectors`类提供了很多静态方法，可以方便地创建`Collector`实例，具体方法如下表：

| 方法                | 返回类型               | 作用                                                         |
| :------------------ | :--------------------- | :----------------------------------------------------------- |
| `toList`            | `List<T>`              | 把流中元素收集到`List`                                       |
| `toSet`             | `Set<T>`               | 把流中元素收集到`Set`                                        |
| `toCollection`      | `Collection<T>`        | 把流中元素收集到创建的集合                                   |
| `counting`          | `Long`                 | 计算流中元素的个数                                           |
| `summingInt`        | `Integer`              | 对流中元素的整数属性求和                                     |
| `averagingInt`      | `Double`               | 计算流中元素`Integer`属性的平均值                            |
| `summarizingInt`    | `IntSummaryStatistics` | 收集流中`Integer`属性的统计值。如：平均值                    |
| `joining`           | `String`               | 连接流中每个字符串                                           |
| `maxBy`             | `Optional<T>`          | 根据比较器选择最大值                                         |
| `minBy`             | `Optional<T>`          | 根据比较器选择最小值                                         |
| `reducing`          | 归约产生的类型         | 从一个作为累加器的初始值开始，利用`BinaryOperator`与流中元素逐个结合，从而归约成单个值 |
| `collectingAndThen` | 转换函数返回的类型     | 包裹另一个收集器，对其结果转换函数                           |
| `groupingBy`        | `Map<K, List<T>>`      | 根据某属性值对流分组，属性为`K`结果为`V`                     |
| `partitioningBy`    | `Map<Boolean,List<T>>` | 根据`true`或`false`进行分区                                  |



代码实现：

```java
/*
Employee的带参构造器声明：
public Employee(int id, String name, int age, double salary)
*/
class EmployeeData {
	public static List<Employee> getEmployees(){
		List<Employee> list = new ArrayList<>();
		list.add(new Employee(1001, "马化腾", 34, 6000.38));
		list.add(new Employee(1002, "马云", 12, 9876.12));
		list.add(new Employee(1003, "刘强东", 33, 3000.82));
		list.add(new Employee(1004, "雷军", 26, 7657.37));
		list.add(new Employee(1005, "李彦宏", 65, 5555.32));
		list.add(new Employee(1006, "比尔盖茨", 42, 9500.43));
		list.add(new Employee(1007, "任正非", 26, 4333.32));
		list.add(new Employee(1008, "扎克伯格", 35, 2500.32));
		return list;
	}
}

public class StreamAPITest2 {
    //1-匹配与查找
    @Test
    public void test1(){
        List<Employee> employees = EmployeeData.getEmployees();
        //练习：判断是否所有的员工的年龄都大于18
        boolean allMatch = employees.stream().allMatch(e -> e.getAge() > 18);
        System.out.println(allMatch);

        //练习：判断是否存在员工姓“雷”
        boolean noneMatch = employees.stream().
            noneMatch(e -> e.getName().startsWith("雷"));
        System.out.println(noneMatch);
        
        //返回第一个元素
        Optional<Employee> employee = employees.stream().findFirst();
        System.out.println(employee);
        
        //返回当前流中的任意元素
        Optional<Employee> employee1 = employees.parallelStream().findAny();
        System.out.println(employee1);
    }
    @Test
    public void test2(){
        List<Employee> employees = EmployeeData.getEmployees();
        // 返回流中元素的总个数
        long count = employees.stream().f
            ilter(e -> e.getSalary() > 5000).
            count();
        System.out.println(count);
		
        //返回流中最大值
        //练习：返回最高的工资
        Stream<Double> salaryStream = employees.stream().map(e -> e.getSalary());
        Optional<Double> maxSalary = salaryStream.max(Double::compare);
        System.out.println(maxSalary);
       
        //返回流中最小值
        //练习：返回最低工资的员工
        Optional<Employee> employee = employees.stream().
            min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
        System.out.println(employee);
        
        //内部迭代
        employees.stream().forEach(System.out::println);
        
        //使用集合的遍历操作
        employees.forEach(System.out::println);
    }
    
    //2-归约
    @Test
    public void test3(){
        //练习1：计算1-10的自然数的和
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        Integer sum = list.stream().reduce(0, Integer::sum);
        System.out.println(sum);

        //练习2：计算公司所有员工工资的总和
        List<Employee> employees = EmployeeData.getEmployees();
        
        Stream<Double> salaryStream = employees.stream().map(Employee::getSalary);
        Optional<Double> sumMoney = salaryStream.reduce((d1,d2) -> d1 + d2);
        System.out.println(sumMoney.get());
    }
    
    //3-收集
    @Test
    public void test4(){
        //练习1：查找工资大于6000的员工，结果返回为一个List或Set
        List<Employee> employees = EmployeeData.getEmployees();
       
        //返回List
        List<Employee> employeeList = employees.stream().
            filter(e -> e.getSalary() > 6000).
            collect(Collectors.toList());
        employeeList.forEach(System.out::println);
        
        //返回Set
        Set<Employee> employeeSet = employees.stream().
            filter(e -> e.getSalary() > 6000).
            collect(Collectors.toSet());
        employeeSet.forEach(System.out::println);
    }
}
```



# 五、Optional类

`Optional<T>`类(`java.util.Optional`)是一个容器类，它可以保存类型`T`的值，代表这个值存在。或者仅仅保存`null`，表示这个值不存在。是为了避免出现空指针异常而创建的。

> Optional容器类，存放的是对象

原来用`null`表示一个值不存在，现在`Optional`可以更好的表达这个概念。并且可以避免空指针异常。

`Optional`类提供了如下方法，利用这些方法，可以不用显式进行空值检测：
**创建Optional类对象的方法**：

* `Optional.of(T t)`：创建一个`Optional`实例，`t`必须非空；
* `Optional.empty()`：创建一个空的`Optional`实例
* `Optional.ofNullable(T t)`：`t`可以为`null`

**判断Optional容器中是否包含对象**：

* `boolean isPresent()`：判断是否包含对象
* `void ifPresent(Consumer<? super T> consumer)`：如果有值，就执行`Consumer`接口的实现代码，并且该值会作为参数传给它。

**获取Optional容器的对象**：

* `T get()`:如果调用对象包含值，返回该值，否则抛异常
* `T orElse(T other)`：如果有值则将其返回，否则返回指定的other对象。
* `T orElseGet(Supplier<? extends T> other)`：如果有值则将其返回，否则返回由`Supplier`接口实现类提供的对象。
* `T orElseThrow(Supplier<? extends X> exceptionSupplier)`：如果有值则将其返回，否则抛出由`Supplier`接口实现类提供的异常。

应用举例：

```java
class Girl{
    private String name;
    public Girl() {}
    public Girl(String name) {this.name = name;}
}
public class OptionalTest{
    public static void main(String[] args) {
        Girl girl = new Girl("Jack");
        //如果值为空，则返回指定的对象
        //值非空，返回非空值： Girl{name='Jack'}
        System.out.println(Optional.ofNullable(girl).
                           orElse(new Girl("Jessica")));
        //值为空，返回指定值：Girl{name='Tom'}
        System.out.println(Optional.ofNullable(null).
                           orElse(new Girl("Tom")));
    }
}
```

