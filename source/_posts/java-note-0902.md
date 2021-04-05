---
title: Java学习笔记09(2)-常用类之日期类
excerpt: Date类、Calendar类、SimpleDateFormat类、LocalDate、Instant、DateTimeFormatter
mathjax: true
date: 2021-04-04 19:34:14
tags: Java
categories: Java
keywords: Java
---

# 一、JDK8之前的日期时间API

JDK8.0之前，日期和时间的API主要有：

* `java.lang.System`类中的`public static long currentTimeMillis()`方法，用来返回当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差(毫秒数)，这个毫秒数称为时间戳。
* `java.util.Date` 和` java.sql.Date`类。`java.sql.Date`类是`java.util.Date`类的子类。
* `java.text.SimpleDateFormat`
* `Calendar`

## 1、Date类

`Date`类有两个，一般使用的是`java.util.Date`类，`java.sql.Date`类是数据库中使用的日期类。

`java.util.Date`类有两个构造器：

* `Date()`：创建一个对应当前时间的Date对象。
* `Date(long date)`：创建指定毫秒数的Date对象。

两个方法(大部分方法都过时了)：

* `toString()`：显示当前的年、月、日、时、分、秒。默认格式为 `EEE MMM dd HH:mm:ss zzz yyyy`，比如`Sat Feb 16 16:35:31 GMT+08:00 2019`
* `getTime()`：获取当前Date对象对应的毫秒数（时间戳）。



`java.sql.Date`类没有空参的构造器：

* `Date(long date)`：创建指定毫秒数的Date对象。
* `Date(int year, int month, int day)`，已过时。

将`java.util.Date`对象转换为`java.sql.Date`对象，具体见代码：

实现代码：

```java
public class DateTimeTest {
    @Test
    public void test2(){
        //构造器一：Date()：创建一个对应当前时间的Date对象
        Date date1 = new Date();
        System.out.println(date1.toString());//Sat Feb 16 16:35:31 GMT+08:00 2019
        System.out.println(date1.getTime());//1550306204104

        //构造器二：创建指定毫秒数的Date对象
        Date date2 = new Date(155030620410L);

        //创建java.sql.Date对象
        java.sql.Date date3 = new java.sql.Date(35235325345L);
        System.out.println(date3);//1971-02-13

        //如何将java.util.Date对象转换为java.sql.Date对象
        //情况一：
        //Date date4 = new java.sql.Date(2343243242323L);
        //java.sql.Date date5 = (java.sql.Date) date4;
        //情况二：
        Date date6 = new Date();
        java.sql.Date date7 = new java.sql.Date(date6.getTime());
    }
}
```



## 2、SimpleDateFormat

`java.text.SimpleDateFormat`是用来**格式化**和**解析**日期的具体类。

格式化：日期-->文本

解析：文本-->日期

格式化：

* `public SimpleDateFormat()`：根据默认的模式和语言环境创建对象。
* `public SimpleDateFormat(String pattern)`：用参数pattern指定的格式创建一个对象。
* `public String format(Date date)`：`SimpleDateFormat`类对象调用此方法来格式化Date类对象。

解析：

* `public Date parse(String source)`：从给定字符串的开始解析文本，生成一个Date类对象。

代码举例：

```java
public class DateTimeTest {
    @Test
    public void testSimpleDateFormat() throws ParseException {
        //实例化SimpleDateFormat:使用默认的构造器
        SimpleDateFormat sdf = new SimpleDateFormat();

        //格式化：日期 --->字符串
        Date date = new Date();  
        System.out.println(date); //格式化前：Sun Apr 04 20:51:54 CST 2021

        String format = sdf.format(date);
        System.out.println(format); // 格式化后：4/4/21 下午8:51

        //解析：格式化的逆过程，字符串 ---> 日期
        String str = "4/4/21 下午9:00"; //解析前
        Date date1 = sdf.parse(str);
        System.out.println(date1);  //解析后：Sun Apr 04 21:00:00 CST 2021
        //System.out.println(date1.toString()); //自动调用toString()方法
        

        //*************按照指定的方式格式化和解析：调用带参的构造器*****************
        SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        //格式化：2021-04-04 09:00:32
        System.out.println(sdf2.format(date)); 
        //解析：Sun Apr 04 09:00:32 CST 2021
        System.out.println(sdf2.parse(sdf2.format(date))); 
    }
```

> 解析的时候要求字符串必须和格式化时的格式相同(通过构造器参数体现)，否则抛异常。

## 3、Calendar

`Date`类不能转换时区，除了`toGMTString()`可以按`GMT+0:00`输出外，`Date`总是以当前计算机系统的默认时区为基础进行输出。此外，我们也很难对日期和时间进行加减，计算两个日期相差多少天，计算某个月第一个星期一的日期等。

`java.util.Calendar`类是一个抽象类，JDK 1.1版本引入，可以用于获取并设置年、月、日、时、分、秒，它和`Date`比，主要多了一个可以做简单的日期和时间运算的功能。

`Calendar`类实例化有两种方式：

* a.创建其子类`GregorianCalendar`的对象获取实例。
* b.调用其静态方法`getInstance()`获取实例。(通过getClass可知，类型还是`GregorianCalendar`)

`Calendar`类常用方法：

一个`Calendar`的实例是系统时间的抽象表示，通过`public int get(int field)`方法来取得想要的时间信息。参数`field`是`Calendar`中的静态变量，比如`YEAR`、`MONTH`、`DAY_OF_WEEK`、`HOUR_OF_DAY`、`MINUTE`、`SECOND`等。

* `public void set(int field,int value)`：将给定的字段设置为给定值。
* `public void add(int field,int amount)`：在给定的字段中添加或减去指定的时间。
* `public final Date getTime()`：日历类-->Date类
* `public final void setTime(Date date)`：Date类-->日历类

> 获取月份时：一月是0，二月是1，以此类推，12月是11
> 获取星期时：周日是1，周二是2，……周六是7

代码实现：

```java
public class calendarTest {
    @Test
    public void testCalendar(){
        System.out.println(new Date()); //当前时间：Sun Apr 04 21:40:07 CST 2021
        //1.实例化
        //方式一：创建其子类（GregorianCalendar）的对象
        //方式二：调用其静态方法getInstance()
        Calendar calendar = Calendar.getInstance();
        System.out.println(calendar.getClass()); //class java.util.GregorianCalendar

        //2.常用方法
        //get()
        int days = calendar.get(Calendar.DAY_OF_MONTH);
        System.out.println("今天是本月的第"+days+"天"); // 今天是本月的第4天

        //set()
        //calendar可变性
        calendar.set(Calendar.DAY_OF_MONTH,22); //设置calendar为本月的第22天
        days = calendar.get(Calendar.DAY_OF_MONTH);
        System.out.println(days);  // 22，此时的calendar是本月的第22天，即4月22日

        //add()
        calendar.add(Calendar.DAY_OF_MONTH,-3); //calendar减去3天
        days = calendar.get(Calendar.DAY_OF_MONTH);
        System.out.println(days); // 19，此时的calendar是本月的第19天，即4月19日

        //getTime():日历类---> Date
        Date date = calendar.getTime();
        System.out.println(date); // Mon Apr 19 21:40:07 CST 2021

        //setTime():Date ---> 日历类
        Date date1 = new Date();  //当前日期
        calendar.setTime(date1);
        days = calendar.get(Calendar.DAY_OF_MONTH);
        System.out.println(days);  //4, 今天是本月的第4天
    }
}
/*
以上程序输出结果：
Sun Apr 04 21:40:07 CST 2021
class java.util.GregorianCalendar
今天是本月的第4天
22
19
Mon Apr 19 21:40:07 CST 2021
4
*/
```



# 二、JDK8的新日期时间API

## 1、新日期API

`Calendar`类和`Date`类存在的问题：

* 可变性：像日期和时间这样的类应该是不可变的。
* 偏移性：Date中的年份是从1900开始的，而月份都从0开始。
* 格式化：格式化只对Date有用，Calendar则不行。

此外，它们也不是线程安全的；不能处理闰秒等。

JDK 8.0提供了`java.time`这个API，其包含了所有关于本地日期（`LocalDate`）、本地时间（`LocalTime`）、本地日期时间（`LocalDateTime`）、时区（`ZonedDateTime`）和持续时间（`Duration`）的类。

JDK 8.0新增了`toInstant()`方法，用于把Date转换成新的表示形式。这些新增的本地化时间日期API大大简化了日期时间和本地化的管理。

和旧的API相比，新API严格区分了时刻、本地日期、本地时间和带时区的日期时间，并且，对日期和时间进行运算更加方便。

此外，新API修正了旧API不合理的常量设计：

- Month的范围用1~12表示1月到12月；
- Week的范围用1~7表示周一到周日。

最后，新API的类型几乎全部是不变类型（和String类似），可以放心使用不必担心被修改。

新日期API包含：

* `java.time`：包含值对象的基础包。常用
* `java.time.chrono`：提供对不同的日历系统的访问。
* `java.time.format`：格式化和解析时间和日期。常用
* `java.time.temporal`：包括底层框架和扩展特性。
* `java.time.zone`：包含时区支持的类。

新日期API和旧日期API的对应关系：

* `java.util.Date`和`java.sql.Date`  对应(类似) `Instant`
* `SimpleDateFormat`  对应(类似)` DateTimeFormatter`
* `Calendar` 对应(类似) `LocalDate`、`LocalTime`、`LocalDateTime`

## 2、常用的三个类

其中三个比较重要的类，这三个类的实例是**不可变对象**：

* LocalDate代表IOS格式（yyyy-MM-dd）的日期,可以存储生日、纪念日等日期。
* LocalTime表示一个时间，而不是日期。
* LocalDateTime是用来表示日期和时间的，这是一个最常用的类之一。

这几个类常用的方法如下：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-0902_1.png'/>
</div>

实现代码参考：

```java
public class JDK8Time {
    @Test
    public void test1(){
        //now():获取当前的日期、时间、日期+时间
        LocalDate localDate = LocalDate.now();
        LocalTime localTime = LocalTime.now();
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDate); //2021-04-04
        System.out.println(localTime); //22:10:12.279529500
        System.out.println(localDateTime); //2021-04-04T22:10:12.279529500

        //of():设置指定的年、月、日、时、分、秒。没有偏移量
        LocalDateTime localDateTime1 = LocalDateTime.of(2020, 10, 6, 13, 23, 43);
        System.out.println(localDateTime1); //2020-10-06T13:23:43


        //getXxx()：获取相关的属性
        System.out.println(localDateTime.getDayOfMonth());  //4
        System.out.println(localDateTime.getDayOfWeek());  //SUNDAY
        System.out.println(localDateTime.getMonth());  //APRIL
        System.out.println(localDateTime.getMonthValue()); //4
        System.out.println(localDateTime.getMinute());  //10

        //体现不可变性
        //withXxx():设置相关的属性
        LocalDate localDate1 = localDate.withDayOfMonth(22); //返回新的对象
        System.out.println(localDate); //2021-04-04，原来的对象不会被修改
        System.out.println(localDate1); //2021-04-22

        //不可变性，不会改变原有对象
        LocalDateTime localDateTime3 = localDateTime.plusMonths(3); //加3个月
        System.out.println(localDateTime); //2021-04-04T22:10:12.279529500
        System.out.println(localDateTime3); //2021-07-04T22:10:12.279529500

        LocalDateTime localDateTime4 = localDateTime.minusDays(6); //减6天
        System.out.println(localDateTime); //2021-04-04T22:10:12.279529500
        System.out.println(localDateTime4); //2021-03-29T22:10:12.279529500
    }
}
```

## 3、Instant

`java.time.Instant`用来表示时间戳，它只是简单的表示自1970年1月1日0时0分0秒（UTC）开始的秒数。`java.time`包是基于纳秒计算的，`Instant`的精度可以达到纳秒级。

常用方法：

| 方法                                                  | 描述                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| `public static Instant now()`                         | 静态方法，返回默认UTC时区的Instant类的对象                   |
| `public static Instant ofEpochMilli(long epochMilli)` | 静态方法，返回在1970-01-01 00:00:00基础上加上指定毫秒数之后的`Instant`类的对象 |
| `public long toEpochMilli()`                          | 返回1970-01-01 00:00:00到当前时间的毫秒数，即为时间戳        |
| `public OffsetDateTime atOffset(ZoneOffset offet)`    | 结合即时的偏移来创建一个`OffsetDateTime`                     |

实现代码：

```java
public class JDK8Time {
    /*
   Instant的使用
   类似于 java.util.Date类
    */
    @Test
    public void test2(){
        //now():获取本初子午线对应的标准时间
        Instant instant = Instant.now();
        System.out.println(instant);//2021-04-05T03:02:58.898817900Z

        //添加时间的偏移量
        OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
        System.out.println(offsetDateTime);//2021-04-05T11:02:58.898817900+08:00

        //toEpochMilli():获取自1970年1月1日0时0分0秒（UTC）开始的毫秒数
        //相当于 Date类的getTime()
        long milli = instant.toEpochMilli();
        System.out.println(milli); //1617591778898

        //ofEpochMilli():通过给定的毫秒数，获取Instant实例
        //相当于Date(long millis)
        Instant instant1 = Instant.ofEpochMilli(1550475314878L);
        System.out.println(instant1); //2019-02-18T07:35:14.878Z
    }
}
```



## 4、DateTimeFormatter

使用`Date`对象时，用`SimpleDateFormat`进行格式化显示。使用新的`LocalDateTime`或`ZonedLocalDateTime`时，使用`java.time.format.DateTimeFormatter`进行格式化显示。

和`SimpleDateFormat`不同的是，`DateTimeFormatter`是不变对象，并且是线程安全的。因为`SimpleDateFormat`不是线程安全的，使用的时候，只能在方法内部创建新的局部变量。而`DateTimeFormatter`可以只创建一个实例，到处引用。

`DateTimeFormatter`类有三种格式化方法：

* 预定义的标准格式。如：`ISO_LOCAL_DATE_TIME`、`ISO_LOCAL_DATE`、`ISO_LOCAL_TIME`
* 本地化相关的格式。如：`ofLocalizedDateTime(FormatStyle.LONG)`
* **自定义的格式**。如：`ofPattern(“yyyy-MM-ddhh:mm:ss”)`，常用

同样地，`DateTimeFormatter`类也提供了相应的格式化和解析的函数：

| 方法                                                        | 描述                                                  |
| :---------------------------------------------------------- | :---------------------------------------------------- |
| `public static DateTimeFormatter ofPattern(String pattern)` | 静态方法，返回一个指定字符串格式的`DateTimeFormatter` |
| `public String format(Temporal Accessort)`                  | 格式化一个日期、时间，返回字符串                      |
| `public TemporalAccessor parse(CharSequence text)`          | 将指定格式的字符序列解析为一个日期、时间              |

代码使用：

```java
public class JDK8Time {
    @Test
   	public void test3(){
        //方式一：预定义的标准格式。如：ISO_LOCAL_DATE_TIME;
        //ISO_LOCAL_DATE;ISO_LOCAL_TIME
        DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
        //格式化:日期-->字符串
        LocalDateTime localDateTime = LocalDateTime.now();
        String str1 = formatter.format(localDateTime);
        System.out.println(localDateTime);
        System.out.println(str1);//2019-02-18T15:42:18.797

        //解析：字符串 -->日期
        TemporalAccessor parse = formatter.parse("2019-02-18T15:42:18.797");
        System.out.println(parse);
        
        //方式二：
        //本地化相关的格式。如：ofLocalizedDateTime()，适用于LocalDateTime
        //FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT :
        DateTimeFormatter formatter1 = DateTimeFormatter.
            ofLocalizedDateTime(FormatStyle.LONG);
        //格式化
        String str2 = formatter1.format(localDateTime);
        System.out.println(str2);//2019年2月18日 下午03时47分16秒

        //本地化相关的格式。如：ofLocalizedDate()，适用于LocalDate
        //FormatStyle.FULL / FormatStyle.LONG / FormatStyle.MEDIUM / 
        //FormatStyle.SHORT
        DateTimeFormatter formatter2 = DateTimeFormatter.
            ofLocalizedDate(FormatStyle.MEDIUM);
        //格式化
        String str3 = formatter2.format(LocalDate.now());
        System.out.println(str3);//2019-2-18


        //方式三：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)
        DateTimeFormatter formatter3 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
        //格式化
        String str4 = formatter3.format(LocalDateTime.now());
        System.out.println(str4);//2019-02-18 03:52:09

        //解析
        TemporalAccessor accessor = formatter3.parse("2019-02-18 03:52:09");
        System.out.println(accessor);
        //{HourOfAmPm=3, MinuteOfHour=52, MicroOfSecond=0, SecondOfMinute=9,
        //NanoOfSecond=0, MilliOfSecond=0},ISO resolved to 2019-02-18
    }
}
```

## 5、其他API

* `ZoneId`：该类中包含了所有的时区信息，一个时区的ID，如Europe/Paris
* ZonedDateTime`：一个在ISO-8601日历系统时区的日期时间，如2007-12-03T10:15:30+01:00Europe/Paris。其中每个时区都对应着ID，地区ID都为“{区域}/{城市}”的格式，例如：Asia/Shanghai等
* `Clock`：使用时区提供对当前即时、日期和时间的访问的时钟。
* 持续时间：`Duration`，用于计算两个“时间”间隔
* 日期间隔：`Period`，用于计算两个“日期”间隔
* `TemporalAdjuster` :时间校正器。有时我们可能需要获取例如：将日期调整到“下一个工作日”等操作。
* `TemporalAdjusters` :该类通过静态方法(`firstDayOfXxx()`/`lastDayOfXxx()`/`nextXxx()`)提供了大量的常用`TemporalAdjuster`的实现。

## 6、新旧日期转换

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-0902_02.jpg
'/>
</div>

