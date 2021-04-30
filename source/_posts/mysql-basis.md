---
title: MySQL数据库基础篇
excerpt: MySQL的使用，常用SQL语句，DDL、DML、DQL，事务，视图，存储结构，函数
mathjax: true
date: 2021-04-26 21:14:17
tags: ['数据库','MySQL','SQL']
categories: 数据库
keywords: MySQL,SQL,DQL,DML,DDL,事务,视图,存储结构,函数
---

# 一、数据库概述

## 1、使用数据库的优势

使用数据库有两个优势：

* 持久化数据到本地。
* 可以实现结构化查询，方便管理。

## 2、数据库相关概念

**DB(Database)**：数据库，保存一组有组织的数据的容器。

**DBMS(Database Management System)**：数据库管理系统，又称为数据库软件（产品），用于管理DB中的数据。

**SQL(Structured Query Language)**：结构化查询语言，用于和DBMS通信的语言，包括数据插入、查询、更新、删除，数据库模式创建和修改，以及数据访问控制。

## 3、SQL的语言分类

**DDL(Data Definition Language)**：数据定义语言。允许用户**定义**数据，包括创建（create）、删除（drop）、修改（alter）这些操作。通常，DDL由数据库管理员执行。

**DML(Data Manipulation Language**)：数据操作语言。DML为用户提供添加（insert）、删除（delete）、更新数据（update）的能力，这些是应用程序对数据库的日常操作。

**DQL(Data Query Language)**：数据查询语言，执行查询（select）操作。
**TCL(Transaction Control Language)**：事务控制语言，执行commit、rollback等操作。



# 二、关系数据库

## 1、什么是关系型数据库

**RDBMS(Relational Database Management System)**：关系型数据库管理系统，即基于关系模型的数据库。

**关系模型**把数据看作是一个二维表格，任何数据都可以通过行号和列号来唯一确定，它的数据模型看起来就是一个Excel表。

## 2、RDBMS存储数据的特点

RDBMS存储数据的特点：

* 数据以表格的形式出现，每个表都有唯一的名字，用于标识自己。
* 若干的表格组成数据库。
* 表本身具有一些特性，这些特性定义了数据在表中如何存储，类似java中 “类”的设计。
* 表中的**列**也称为**字段（Column）**。列类似于java 中的”属性”。
* 表中的**行**称为**记录（Record）**，每一行是一组相关的数据。行类似于java中的“对象”。



## 3、常见的关系型数据库

* 商用数据库，例如：[Oracle](https://www.oracle.com/)，[SQL Server](https://www.microsoft.com/sql-server/)，[DB2](https://www.ibm.com/db2/)等；
* 开源数据库，例如：[MySQL](https://www.mysql.com/)，[PostgreSQL](https://www.postgresql.org/)等；
* 桌面数据库，以微软[Access](https://products.office.com/access)为代表，适合桌面应用程序使用；
* 嵌入式数据库，以[Sqlite](https://sqlite.org/)为代表，适合手机应用和桌面程序。

## 4、NoSQL

NoSQL数据库，也就是非SQL的数据库，包括MongoDB、Cassandra、Dynamo等等，它们都不是关系数据库。SQL数据库从始至终从未被取代过，NoSQL的发展历程：

- 1970: NoSQL = We have no SQL
- 1980: NoSQL = Know SQL
- 2000: NoSQL = No SQL!
- 2005: NoSQL = Not only SQL
- 2013: NoSQL = No, SQL!

今天，SQL数据库仍然承担了各种应用程序的核心数据存储，而NoSQL数据库作为SQL数据库的补充，两者不再是二选一的问题，而是主从关系。

# 三、MySQL

## 1、MySQL介绍

MySQL是目前应用最广泛的开源关系数据库。MySQL最早是由瑞典的MySQL AB公司开发，该公司在2008年被SUN公司收购，SUN公司在2009年被Oracle公司收购，所以MySQL最终就变成了Oracle旗下的产品。

MySQL优势：

* 开源、免费、成本低
* 性能高、移植性好
* 体积小，便于安装

## 2、MySQL安装

MySQL属于c/s架构的软件，一般来讲只安装服务端。以下案例以 MySQL5.7 为例

下载[链接](https://dev.mysql.com/downloads/mysql/)

## 3、MySQL服务的启动和停止

Windows下，MySQL启动和停止有两种方式：

* 计算机—右击管理—服务，启动MySQL服务
* 通过管理员身份运行cmd：
  * net start 服务名（启动服务），例`net start mysql57`
  * net stop 服务名（停止服务），例`net stop mysql57`

## 4、MySQL服务的登录和退出

​	方式一：通过mysql自带的客户端（MySQL Line Client），只限于root用户

​	方式二：通过windows的命令提示符，以管理员身份打开

* 登录：`mysql [-h 主机名 -P 端口号] -u 用户名 -p密码`，`[]`里的内容表示可以省略

  > 其中`p`参数和密码直接不能有空格，其他参数和值之间空格可有可无，比如`-uroot`，表示root用户。
  >
  > 密码可以不添加值，直接回车，然后输入密码，不会显示出明文。
  >
  > 如果是本机，并且端口是3306，可以省略`-h`和`-P`参数，直接输入用户名和密码即可。

* 退出：`exit`

如下图：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-basis_1.png"/>



## 5、MySQL的常见命令



查看当前所有的数据库:

```mysql
show databases;
```

打开指定的库:

```mysql
use 库名;
```

查看当前库的所有表:

```mysql
show tables;
```

查看其它库的所有表：

```mysql
show tables from 库名;
```

查看表结构

```mysql
desc 表名;
```

查看服务器的版本

```mysql
# 方式一，在mysql服务端查看：
select version();
# 方式二，在cmd窗口查看：
mysql --version
或
mysql --V
```



## 6、MySQL的语法规范

MySQL中的命令有以下规范：

* **不区分大小写**，建议关键字大写，表名和列名小写
* 每条命令以`;`或`\g`结尾
* 每条命令根据需要，可以进行缩进或换行。关键字不能缩写或分行。
* 注释
  * 单行注释：`#注释文字`
  * 单行注释：`-- 注释文字`,(注意`--`后面有空格)
  * 多行注释：`/* 注释文字  */`



字符（建议加，有时必须加）和日期型要加引号，数值不需要加。**表的别名不需要加引号**



# 四、MySQL数据类型

MySQL支持所有标准SQL数值数据类型（SQL数据类型参考[链接](https://www.runoob.com/sql/sql-datatypes-general.html)）。作为SQL标准的扩展，MySQL也支持整数类型TINYINT、MEDIUMINT和BIGINT。详细参考[MySQL数据类型](https://www.runoob.com/mysql/mysql-data-types.html)

包括**数值型**、**字符型**、**日期型**等。

**数值型**

<table>
	<tr>
		<td><b></b></td>
		<td><b>类型</b></td>
        <td><b>字节</b></td>
        <td><b>说明</b></td>
	</tr>
	<tr>
        <td rowspan='5'>整型</td>
        <td>Tinyint</td>
        <td>1</td>
		<td>有符号：-128~127<br>
            无符号：0~255
        </td>
	</tr>
	<tr>
        <td>Smallint</td>
        <td>2</td>
		<td>有符号：-32768~32767<br>
            无符号：0~65535
        </td>
	</tr>
    <tr>
		<td>Mediumint</td>
        <td>3</td>
		<td>有符号：-8,388,608~8,388,607<br>
            无符号：0~16,777,215
        </td>
	</tr>
    <tr>
		<td>Int/Integer</td>
        <td>4</td>
		<td>有符号：-2,147,483,648~2,147,483,647<br>
            无符号：0~4,294,967,296
        </td>
	</tr>
    <tr>
		<td>Bigint</td>
        <td>8</td>
		<td>有符号：-2^63~2^63-1<br>
            无符号：0~2^64
        </td>
	</tr>
    <tr>
		<td rowspan='2'>浮点型小数</td>
        <td>float(M,D)</td>
        <td>4</td>
        <td>±1.75494351E-38 ~ ±3.402823466E+38</td>
	</tr>
    <tr>
        <td>double(M,D)</td>
        <td>8</td>
        <td>±2.2250738585072014E-308 ~ ±1.7976931348623157E+308</td>
    </tr>
      <tr>
		<td>定点型小数</td>
        <td>DEC(M,D)<br>
          	DECIMAL(M,D)
          </td>
        <td>M+2</td>
        <td>最大取值范围与double相同， 给定decimal的有效取值范围由M和D决定</td>
	</tr>
</table>


说明：

*  如果不设置无符号还是有符号，默认是有符号，如果想设置无符号，需要在类型后面添加`unsigned`关键字
*  如果插入的数值超出了整型的范围,会报out of range异常，并且插入临界值
*  如果不设置长度，会有默认的长度。长度不决定范围，长度代表了显示的最大宽度。可以选择在长度不够时，用0在左边填充，需要在类型后面添加`zerofill`。zerofill只支持正数（无符号）

*  定点型的精确度较高，如果要求插入数值的精度较高如货币运算等则考虑使用
*  M表示整数部位个数+小数部位个数的总长度。D表示小数部位长度。如果插入的数值超过范围，会报out of range异常，并插入临界值。
*  M和D都可以省略。如果是decimal，则M默认为10，D默认为0。
*  float和double，会根据插入的数值的精度来决定精度。




**字符型**

<table>
	<tr>
		<td><b>类型</b></td>
        <td><b>最多字符数</b></td>
        <td><b>描述</b></td>
	</tr>
	<tr>
        <td>char(M)</td>
        <td>M</td>
		<td>M为0~255之间的整数，固定长度的字符，比较耗费空间，效率高</td>
	</tr>
    <tr>
        <td>varchar(M)</td>
        <td>M</td>
		<td>M为0~65535之间的整数，可变长度的字符，比较节省空间，效率低</td>
	</tr>
</table>


M表示最大的字符个数，而不是存储空间。

M

其他字符型类型：

* `binary`和`varbinary`用于保存较短的二进制
* `enum`用于保存枚举类型，要求插入的值必须属于列表中指定的值之一。如果列表成员为`1~255`，则需要1个字节存储。如果列表成员为`255~65535`，则需要2个字节存储，最多为65535个成员。
* `set`用于保存集合。里面可以保存0~64个成员。一次可以选取多个成员。根据成员个数不同，存储所占的字节从1-8变化。
* `text`用于存放较长的文本
* `blob`用于存放较大的二进制

**日期型**

| 日期和时间类型 | 字节 | 最小值              | 最大值              |
| -------------- | ---- | ------------------- | ------------------- |
| `date`         | 4    | 1000-01-01          | 9999-12-31          |
| `datetime`     | 8    | 1000-01-01 00:00:00 | 9999-12-31 23:59:59 |
| `timestamp`    | 4    | 1970-01-01 00:00:00 | 2038-1-19某一时刻   |
| `time`         | 3    | -838:59:59          | 838:59:59           |
| `year`         | 1    | 1901                | 2155                |

其中字符型和日期型的常量值必须要用`''`或`""`包起来。数值型不需要。

**datetime和timestamp的对比**

* 二者都是保存日期和时间。

* datetime占用8字节，范围是1000-9999，不受时区的影响。
* timestamp占用4字节，范围为1970-2038，受时区的影响，其值会根据时区的变化而变化。即插入数据以后，如果修改时区，表中的时间戳也会改变为对应时区的值。



# 五、DQL语言

DQL为数据查询语言，执行查询（select）操作。

DQL的完整查询语句结构：

```mysql
select 字段,...				# 7
from 表1 [别名]				# 1
[连接类型 join 表2]...			# 2
[on 连接条件]				# 3
[where 筛选条件]			# 4
[group by 分组字段]			# 5
[having 分组后的筛选条件]		# 6
[order by 排序的字段或表达式]		# 8
[limit 偏移量(起始条目索引),条数]; 	# 9
```

以上各部分的执行顺序： 
`from`→`join`→`on`→`where`→`group by`(开始使用select中的别名，后面的语句都可以使用)-

→`AVG/SUM/MAX/MIN等分组函数`→`having`→`select`→`distinct`→`order by`→`limit`

以下案例使用的数据库：`myemployees`，其中有`departments`、`employees`、`job_grades`、`jobs`、`locations`共五张表。各自的包含的字段如下：

`departments`表：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-basis_3.png"/>

`employees`表：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-basis_2.png"/>

`job_grades`表：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-basis_4.png"/>

`jobs`表：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-basis_5.png"/>

`locations`表：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-basis_6.png"/>

## 1、基础查询

语法：

```mysql
SELECT 查询内容 
[FROM 表名];
```



> 类似于Java中的System.out.println(要打印的东西);
>
> 查询的结果是一个虚拟的表，不会改变原来的表格。

特点：
①通过select查询出的结果 ，是一个虚拟的表格，不是真实存在

②查询内容可以是**常量值**、**表达式**、**字段**、**函数**

示例：

```mysql
#1.查询单个字段
SELECT last_name FROM employees;

#2.查询多个字段
SELECT last_name,salary,email FROM employees;

#3.查询所有字段
SELECT 
    `employee_id`,
    `first_name`,
    `last_name`,
    `phone_number`,
    `last_name`,
    `job_id`,
    `phone_number`,
    `job_id`,
    `salary`,
    `commission_pct`,
    `manager_id`,
    `department_id`,
    `hiredate` 
FROM
    employees ;
#方式二，*号表示所有字段：  
SELECT * FROM employees;

#4.查询常量
SELECT 100;

#5.查询函数
SELECT VERSION();

#6.查询表达式
SELECT 100%98;
```

**别名**：如果要查询的字段有重名的情况，可以使用别名区分；同样，表也可以起别名。

定义别名有两种方式：

* 使用`as`:

  ```mysql
  SELECT 100%98 AS 结果;
  /*运行结果为：
  +------+
  | 结果 |
  +------+
  |    2 |
  +------+
  */
  SELECT last_name AS 姓,first_name AS 名 FROM employees;
  ```

* 使用空格，将`as`替换为空格即可：

  ```mysql
  SELECT last_name 姓,first_name 名 FROM employees;
  ```



**去重**：使用`distinct`对结果去重

```mysql
#案例：查询员工表中涉及到的所有的部门编号
SELECT DISTINCT department_id FROM employees;
```



**+号在MySQL中的作用**：

MySQL中的`+`号只有运算符的作用，其运算规则如下：

* 如果两个操作值都是数值型，做加法运算
* 如果一方为字符型，则尝试进行转换，转换成功就做加法运算，如果转换失败，则将字符值当作0
* 任何数和`null`做加法运算，结果都是`null`

```mysql
select 100+90;  # 结果为190
select '123'+90; # 结果为213
select 'john'+90; # 结果为90
select null+10;  # 结果为null
```



## 2、条件查询

语法：

```mysql
select 要查询的字段|表达式|常量值|函数
from 表
where 筛选条件;  # where后面加筛选条件
```

根据筛选条件的不同，可以分为以下几种：

① **条件表达式**

使用以下条件运算符：

|   操作符   | 含义                                                         |
| :--------: | :----------------------------------------------------------- |
|    `=`     | 等于，也可以作为赋值符号，为了便于区分，变量赋值尽量用`:=`符号。<br />不能用于判断`null`值 |
|    `>`     | 大于                                                         |
|   ` >=`    | 大于等于                                                     |
|    `<`     | 小于                                                         |
|    `<=`    | 小于等于                                                     |
| `<>`或`!=` | 不等于，两种写法均可，不能用于判断`null`值                   |

案例：

```mysql
#案例1：查询工资>12000的员工信息
SELECT * FROM employees WHERE salary>12000;

#案例2：查询部门编号不等于90号的员工名和部门编号
SELECT 
	last_name,
	department_id
FROM employees
WHERE department_id<>90;
```



②**逻辑表达式**

使用以下逻辑运算符：

| 操作符      | 含义 |
| ----------- | ---- |
| `and`或`&&` | 与   |
| `or`或$||$  | 或   |
| `not`或`!`  | 非   |

案例：

```mysql
#案例1：查询工资z在10000到20000之间的员工名、工资以及奖金
SELECT
	last_name,
	salary,
	commission_pct
FROM employees
WHERE salary>=10000 AND salary<=20000;

#案例2：查询部门编号不是在90到110之间，或者工资高于15000的员工信息
SELECT * FROM employees
WHERE NOT(department_id>=90 AND department_id<=110) OR salary>15000;
```



③**其他**

包括以下几种：

| 操作符                   | 含义                                                         |
| ------------------------ | :----------------------------------------------------------- |
| `like`                   | 模糊查询，`%`用于匹配**任意个**字符，包括0个，`_`用于匹配**任意一个**字符 |
| `between ... and ...`    | 判断是否在范围之间，相当于`>=`和`<=`                         |
| `in`                     | 使用`=`号判断                                                |
| `is null`，`is not null` | 用于判断是否为`null`                                         |



`like`  一般用于字符型数据，也可以查询数值型数据。使用通配符查询，默认`\`为转义字符，也可以使用`escape`自定义转移字符。

案例：

```mysql
#查询员工名中包含字符a的员工信息
SELECT * FROM employees
WHERE last_name LIKE '%a%';

#查询员工名中第二个字符为_的员工名
SELECT last_name FROM employees
WHERE last_name LIKE '_#_%' ESCAPE '#'; # 自定义#作为转义字符
```

`between and`语句包含两个临界值，且要求是合法的范围：

```mysql
#查询员工编号在100到120之间的员工信息
SELECT * FROM employees
WHERE employee_id BETWEEN 100 AND 120;
#等价于：
SELECT * FROM employees
WHERE employee_id >= 100 AND employee_id<=120;
```

`in`用于判断某字段的值是否属于**in列表**中的某一项，使用`=`号判断

* in列表的值类型必须一致或兼容
* in列表中不支持通配符

案例：

```mysql
#查询工种编号是IT_PROG、AD_VP、AD_PRES中其中一个的所有员工的姓名和工种编号
SELECT last_name, job_id FROM employees
WHERE job_id IN( 'IT_PROT' ,'AD_VP','AD_PRES');
#等价于
SELECT last_name, job_id FROM employees
WHERE job_id = 'IT_PROT' OR job_id = 'AD_VP' OR JOB_ID ='AD_PRES';
```

`is null`和`is not null`用于判断字段是否为`null`，且只能用于判断是否为`null`，不能用于判断是否等于某个数值：

```mysql
#查询没有奖金的员工名和奖金率
SELECT last_name,commission_pct 
FROM employees
WHERE commission_pct IS NULL;
```

`<=>`既可以判断是否等于某个数值，也能用于判断是否为`null`：

```mysql
#案例1：查询没有奖金的员工名和奖金率
SELECT last_name, commission_pct
FROM employees
WHERE commission_pct <=>NULL;

#案例2：查询工资为12000的员工名和工资
SELECT last_name, salary
FROM employees
WHERE salary <=> 12000;
```





## 3、排序查询

语法：

```mysql
select 查询列表
from 表名
[where 筛选条件]
order by 排序的字段|表达式|函数|别名 [asc|desc];  # 使用order by排序
```

> 默认asc，表示升序。也可以使用desc指定为降序
>
> order by的位置一般放在最后面（除limit外)

案例：

```mysql
#1、按单个字段排序
SELECT * FROM employees ORDER BY salary DESC;

#2、添加筛选条件再排序
#查询部门编号>=90的员工信息，并按员工编号降序
SELECT * FROM employees WHERE department_id>=90
ORDER BY employee_id DESC;

#3、按表达式排序
#查询员工信息 按年薪降序
SELECT *,salary*12*(1+IFNULL(commission_pct,0)) FROM employees
ORDER BY salary*12*(1+IFNULL(commission_pct,0)) DESC;

#4、按别名排序
#查询员工信息 按年薪升序
SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪
FROM employees
ORDER BY 年薪 ASC;

#5、按函数排序
#查询员工名，并且按名字的长度降序
SELECT LENGTH(last_name),last_name 
FROM employees
ORDER BY LENGTH(last_name) DESC;

#6、按多个字段排序
#案例：查询员工信息，要求先按工资降序，再按employee_id升序
SELECT * FROM employees
ORDER BY salary DESC,employee_id ASC;
```





## 4、常见函数

MySQL中的函数类似于java中的方法，将一组逻辑语句封装在方法体中，对外暴露方法名。

优势：提高代码重用性，隐藏了实现细节。

调用函数的语法：

```mysql
select 函数名(实参列表) [from 表名...]
```

函数根据作用对象的个数不同，可以分为**单行函数**和**分组函数**。

**单行函数**

* 字符函数

  * `length`：求参数的字节个数，比如一个汉字三个字节
  * `concat`：字符拼接，只要有一个为null，则结果为null
  * `substr`：截取子串。sql中的索引是从1开始的
  * `instr`：返回子串第一次出现的索引
  * `trim`：去除首尾指定的空格和字符
  * `upper`：转换成大写
  * `lower`：转换成小写
  * `lpad`：左填充
  * `rpad`：右填充
  * `replace(str,a,b)`：将str中的a替换成b

  案例：

  ```mysql
  #截取从指定索引处指定字符长度的字符(不是字节长度)
  SELECT SUBSTR('你好，MySQL',1,2) out_put;   # 你好
  
  SELECT LENGTH(TRIM('    张 翠 山    ')) AS out_put; # 11
  # trim只能去除首尾的空格，length求得是字节的个数
  ```

  

* 数学函数

  * `round`：四舍五入
  * `rand`：返回一个[0,1)之间的随机数
  * `ceil`：向上取整，返回>=该参数的最小整数
  * `floor`：向下取整，返回<=该参数的最大整数
  * `truncate`：截断（直接截断，不会四舍五入），保留小数点后的n位
  * `mod`：取余

  案例：

  ```mysql
  SELECT ROUND(-1.55); #-2
  SELECT CEIL(-1.02);  #-1
  SELECT FLOOR(-9.99);  #-10
  SELECT TRUNCATE(1.69999,1); # 1.6
  ```

  

* 日期函数

  * `now`：返回当前系统日期+时间
  * `curdate`：返回当前系统日期，不包含时间
  * `curtime`：返回当前时间，不包含日期
  * `year`：获取指定日期的年份
  * `month`：获取指定日期的月份
  * `monthname`：获取指定日期月份的英文
  * `day`：获取指定日期的日
  * `hour`：获取指定日期的小时
  * `minute`：获取指定日期的分钟
  * `second`：获取指定日期的秒
  * `str_to_date`：将字符通过指定的格式转换成日期
  * `date_format`：将日期转换成字符
  * `datediff`：返回两个日期相差的天数

  案例：

  ```mysql
  SELECT YEAR('1998-1-1') 年;  # 1998
  SELECT DATE_FORMAT(NOW(),'%y年%m月%d日') AS out_put;  # 21年04月25日
  ```

  

* <span id="refer2">流程控制函数</span>，包括`if`和`case`函数。这里作为函数，可以用在任何地方，包括begin end里面和外面，注意和后面的`if`结构和`case`结构区分。	

  * `if`函数：`if(条件，值1，值2)`，如果条件成立，返回值1，否则返回值2

  * `case`函数：作为函数，其有两种用法，格式如下：


```mysql
#用法1：
case 要判断的字段或表达式
when 常量1 then 值1
when 常量2 then 值2
...
else 值n
endl;
    
#用法2：
case 
when 条件1 then 值1
when 条件2 then 值2
...
else 值n
end;
```




> if/case结构作为函数时，可以应用在begin end结构中或外面，其返回结果必须是值；语句中不用加分号，最后结束才加分号。
>
> if/case作为流程控制结构时，只能用于begin end结构中，其返回的是执行语句，中间需要加分号，且end后面要加if/case。作为流程控制结构的语法参考最后一章流程控制。

案例：

```mysql
# if
# 例一
SELECT IF(10<5,'大','小');

# 例2
SELECT 
	last_name,
	commission_pct,
	IF(commission_pct IS NULL,'没奖金','有奖金') 备注
FROM employees;

# case
# 例1
SELECT salary 原始工资,department_id,
CASE department_id
WHEN 30 THEN salary*1.1
WHEN 40 THEN salary*1.2
WHEN 50 THEN salary*1.3
ELSE salary
END AS 新工资
FROM employees;

# 例2
SELECT salary,
CASE 
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS 工资级别
FROM employees;
```



* 其他

  * `version`：获取MySQL当前版本
  * `database`：获取当前数据库
  * `user`：获取当前用户
  * `if null(参数,指定值)`：如果参数值为`null`，则返回指定值，否则返回参数的值
  * `is null(参数)`：判断字段或表达式是否为`null`，如果是，返回1，否则返回0
  * `password("str")`：返回str的密码形式
  * `md5("str")`：返回str的MD5形式

  案例：

  ```mysql
  SELECT VERSION();
  SELECT DATABASE();
  SELECT USER();
  ```



**分组函数**

* `sum `：求和。一般用于处理数值
* `max` ：最大值
* `min`： 最小值
* `avg`：平均值。一般用于处理数值
* `count`：计数。参数可以是`字段`、`*`、`常量值（一般用1）`，`*`表示所有字段同时考虑，一般用来统计行数。常量值也能用来统计行数。

> 1.、以上五个分组函数都忽略`null`值，除了`count(*)`，因为主键一定不为空
> 2、max、min、count可以处理任何数据类型。
> 3、都可以搭配distinct使用，用于统计去重后的结果
>
> 4、MYISAM存储引擎下  ，`COUNT(*)`的效率高；INNODB存储引擎下，`COUNT(*)`和`COUNT(1)`的效率差不多，比COUNT(字段)要高一些。

和分组函数同时查询的字段，必须是group by后的字段，不然会出现错误结果。

案例：

```mysql
SELECT SUM(salary) FROM employees;
SELECT AVG(salary) FROM employees;
# 与distinct搭配使用
SELECT COUNT(DISTINCT salary),COUNT(salary) FROM employees;  # 57,107

# 和分组函数一同查询的字段有限制
# 下面这种，employee_id是无意义的，只有group by后的字段才能和分组函数一起查询
SELECT AVG(salary),employee_id  FROM employees;
```



## 5、分组查询

语法：

```mysql
select 查询列表
from 表
[where 筛选条件]
group by 分组的字段  # 使用group by进行分组
[having 条件语句];
```

说明：

1、和分组函数一同查询的字段必须是group by后出现的字段，确保意义正确

2、筛选分为两类：分组前筛选(where)和分组后(having)筛选。having后面一般跟的分组函数，表示对初步筛选后的结果再进行筛选。

3、分组可以按单个字段也可以按多个字段。

4、可以搭配排序使用。

5、having 和group by后，mysql支持别名，Oracle不支持。一般也不使用。

6、having后面的条件一般是分组函数，如果是一般的字段，必须是select中的字段（自己实验得出）。规范来说，having必须是在使用group by语句之后才能使用，如果不是在group by后面，having的作用和where的作用相同，但是不建议这么做，最好按照规范来做。

```mysql
/*
MySQL 5.7版本，出现以下结果
*/
SELECT salary FROM employees 
WHERE salary>15000; # 输出正确结果

SELECT last_name FROM employees
WHERE salary>15000; # 输出正确结果
#－－－－－－－－－－－－－－－－－－
SELECT salary FROM employees 
HAVING salary>15000; # 输出正确结果

#如果having后的字段，没在select中，会报错
SELECT last_name FROM employees 
HAVING salary>15000; # 显示语法错误，原因是‘having clause’中没有salary列
```



**`where`和`having`的区别：**

* where是分组前进行筛选，是对原始表筛选；having是在分组后筛选，是对分组后的结果再进行筛选，一般的筛选条件是分组函数。可以根据代码执行顺序进行推断。
* 在语句格式上，where写在group by 前面，having写在group by后面。

案例：

```mysql
#案例1：查询每个工种的员工平均工资
SELECT AVG(salary),job_id  #和分组函数同时查询的字段，是用于分组的字段
FROM employees
GROUP BY job_id;

#案例2：查询邮箱中包含a字符的每个部门的最高工资
SELECT MAX(salary),department_id
FROM employees
WHERE email LIKE '%a%'
GROUP BY department_id;

#案例3：查询哪个部门的员工个数>5
SELECT COUNT(*),department_id
FROM employees
GROUP BY department_id  # 先查询每个部门员工个数
HAVING COUNT(*)>5;  # 然后选出个数>5的

#案例4：查询每个工种有奖金的员工的最高工资>6000的工种编号和最高工资,按最高工资升序
SELECT job_id,MAX(salary) m
FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY job_id
HAVING m>6000
ORDER BY m;

#按多个字段分组
#案例5：查询每个查询每个工种每个部门的平均工资
SELECT department_id, job_id,AVG(salary) FROM employees
GROUP BY department_id,job_id; # 顺序可以颠倒
# 这两种写法等价，每个A，每个B和每个B，每个A结果相同。
SELECT department_id, job_id,AVG(salary) FROM employees
GROUP BY job_id,department_id;
```



## 6、多表连接查询

当查询的字段来自多个表时，就需要使用连接查询。

如果不使用连接条件，或者连接条件无效，则多个表会按照笛卡儿乘积的形式连接，即查询的结果为m*n的表。

解决这种现象就需要添加有效的连接条件。

连接方式按照功能分类：

* 内连接
  * 等值连接
  * 非等值连接
  * 自连接
* 外连接。
  * 左外连接，left join左边是主表
  * 右外连接，right join右边是主表
  * 全外连接
* 交叉连接

其中，SQL 92标准仅支持内连接，ORACLE和SQL Server支持一部分外连接（mysql不支持），SQL 99标准支持内连接+外连接+交叉连接(mysql不支持全外连接)。

下面以两个表连接为例，说明连接查询的语法格式。

**内连接**

等值连接

等值内连接的结果是多表的交集。n表连接，至少需要n-1个连接条件。


```mysql
# SQL 92语法
select 查询列表
from 表1 别名，表2 别名
where 表1.字段= 表2.字段   # 连接条件 
[and 筛选条件]
其他结构...

# SQL 99语法
select 查询列表
from 表1 别名
[inner] join 表2 别名
on 连接条件
其他结构...
# SQL 99语法将筛选条件和连接条件分离，便于阅读
```

非等值连接

```mysql
# SQL 92
select 查询列表
from 表1 别名，表2, 别名...
where 非等值连接条件
[and 筛选条件]
其他结构...

# SQL 99
select 查询列表
from 表1 别名
[inner] join 表2 别名
on 非等值连接条件
其他结构...
```

自连接

```mysql
# 自连接是指同一张表自己和自己连接。
#将同一张表看作两个表，起两个别名。
# SQL 92
select 查询列表
from 表 别名1,表 别名2
where 连接条件
[and 筛选条件]
其他结构...

# SQL 99语法
select 查询列表
from 表 别名1
[inner] join 表 别名2
on 连接条件
其他结构...
```

总结：对于内连接，SQL 92和SQL 99标准只有在连接表的方式和连接条件的位置不同，其余用法相同。SQL 99语法将连接条件写在`on`语句后面，与筛选条件分离，提高了易读性。

内连接案例：

```mysql
#案例1：查询有奖金的每个部门的部门名、部门的领导编号，以及该部门的最低工资
# SQL 92，等值连接
SELECT department_name,d.`manager_id`,MIN(salary)
FROM departments d,employees e
WHERE d.`department_id`=e.`department_id`
AND commission_pct IS NOT NULL
GROUP BY department_name,d.`manager_id`;

# SQL 99，等值连接
SELECT d.department_name,d.manager_id,MIN(salary)
FROM employees e 
JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`commission_pct` IS NOT NULL
GROUP BY e.`department_id`;

#案例2：查询员工的工资和工资级别
# SQL 92，非等值连接
SELECT salary,grade_level
FROM employees e,job_grades g
WHERE salary BETWEEN g.`lowest_sal` AND g.`highest_sal`;

# SQL 99，非等值连接
SELECT salary,grade_level
FROM employees e
JOIN job_grades g 
ON e.`salary` BETWEEN g.`lowest_sal` AND g.`highest_sal`;

#案例3：查询员工名和上级的名称
# SQL 92，自连接
SELECT e.last_name,m.last_name
FROM employees e,employees m
WHERE e.`manager_id`=m.`employee_id`;

# SQL 99，自连接
SELECT e.last_name,m.last_name
FROM employees e
JOIN employees m
ON e.`manager_id`= m.`employee_id`;
```



**外连接**

外连接的查询结果为主表中的所有记录，如果从表中有和它匹配的，则显示匹配的值，如果从表中没有和它匹配的，则从表的结果显示`null`。外连接查询结果=内连接结果+主表中有，从表没有的记录。

外连接的特性决定了其应用场景为一般**用于查询两个表交集以外的部分**。即用于查询一个表中有，另一个表没有的记录。

左外连接和右外连接交换两个表的顺序，可以实现同样的效果 。

全外连接=内连接的结果+表1有，表2没有+表2有，表1没有。

交叉连接可以省略连接条件，其结果是笛卡尔积。

MySQL中只有SQL 99支持外连接（但不支持全外连接），只介绍SQL 99，语法如下：

```mysql
# 语法：
select 查询内容
from 表1
left [outer]|right [outer]|full [outer]|cross join 表2 
on 连接条件
其他结构...;

#outer可以省略
```

外连接案例：

```mysql
#案例1：查询哪个部门没有员工
#左外
SELECT d.*,e.employee_id
FROM departments d
LEFT OUTER JOIN employees e
ON d.`department_id` = e.`department_id`
WHERE e.`employee_id` IS NULL;
 
#右外,调换两个表的顺序后使用右外连接，和上面写法等价 
SELECT d.*,e.employee_id
FROM employees e
RIGHT OUTER JOIN departments d
ON d.`department_id` = e.`department_id`
WHERE e.`employee_id` IS NULL;
```



**总结**

以集合关系来理解内连接和外连接：

<img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-basis_7.png"/>



## 7、子查询

嵌套在其他语句内部的select语句称为**子查询**或**内查询**。外面的语句可以是insert、update、delete、select等，一般为select语句较多。

如果外面的语句是select语句，则称其为**主查询**或**外查询**。

子查询根据位置的不同和结果集的不同可以有以下分类。

**按子查询出现的位置分类**：

* `select`后面：仅支持标量子查询
* `from`后面：表子查询
* `where`或`having`后面(常用)：标量子查询、列子查询、行子查询	
* `exists`后面（相关子查询）：均可。`exists`返回结果0或1，可以用`in`代替

**按结果集的行列数不同分类**：

* 标量子查询（结果集只有一行一列）
* 列子查询（结果集只有一列多行）
* 行子查询（结果集一行多列）
* 表子查询（又称嵌套子查询。结果集一般为多行多列,一或多都可）

**特点**：

* 子查询放在小括号内。
* 子查询一般放在条件的右侧。
* 子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果。
* 标量、单行子查询，一般搭配着**单行操作符**使用：`<`，` >`，`= `，`<=`，` >=`，`<>`
* 多行子查询，一般搭配着**多行操作符**使用：`in/not in`、`any|some`、`all`

`in/not in`表示等于/不等于列表中的**任意一个**，例：`xxx in ();`

`any|some`表示和子查询返回的**某一个值**进行比较（只要有一个满足即可），例：`xxx >any();`。大于`any|some`等价于大于最小值，小于`any|some`等价于小于最大值。`any|some`一般很少使用。

`all`表示和子查询返回的**所有值**进行比较。



**相关子查询和嵌套子查询对比**

**嵌套子查询**的执行**不依赖于外部的查询**。其执行过程为：

* 执行子查询，其结果不被显示，而是传递给外部查询，作为外部查询的条件使用。
* 执行外部查询，并显示整个结果。

**相关子查询**的执行**依赖于外部查询**。多数情况下是子查询的WHERE子句中引用了外部查询的表。其执行过程为：

* 从外层查询中取出一个元组，将元组相关列的值传给内层查询。
* 执行内层查询，得到子查询操作的值。
* 外查询根据子查询返回的结果或结果集得到满足条件的行。
* 然后外层查询取出下一个元组重复做步骤1-3，直到外层的元组全部处理完毕。 

比如下面的例子：

```mysql
# 嵌套子查询
# 查询工资大于平均工资的员工信息
/*
平均工资是子查询，其不依赖于外查询
*/
SELECT * FROM employees
WHERE salary>( 
	SELECT AVG(salary) FROM employees
);

# 相关子查询
# 查询每个部门的员工数量和部门信息
/*
可以使用子查询的方式，也可以使用外连接+分组查询的方式。
查询员工数量时，需要将子查询和父查询根据部门id关联起来，
先统计一个部门的数量，然后继续统计下一个部门数量,
子查询和父查询二者交替执行
*/
SELECT d.*,(
	SELECT COUNT(*) FROM employees e
	WHERE e.`department_id`=d.`department_id`
	) AS 'NUMBER'
FROM departments d;

# exist也是相关子查询。能用exist的，也能使用in代替
# 查询有员工的部门名
SELECT department_name
FROM departments d
WHERE EXISTS(
	SELECT *   
	FROM employees e
	WHERE d.`department_id`=e.`department_id`
);
```



子查询案例：

```mysql
# -------------用在where和having后使用子查询------------
#案例1：查询最低工资大于50号部门最低工资的 部门id 和 其最低工资
/*思路：
1.先查出50号部门的最低工资
2.然后查出每个部门的最低工资，
3.根据前两步的结果筛选出最低工资大于50号部门最低工资的部门信息
*/
SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id
HAVING MIN(salary)>(
	SELECT  MIN(salary)
	FROM employees
	WHERE department_id = 50
);

#案例2：查询其它工种中，比‘IT_PROG’工种任一工资低的员工的 员工号、工种和薪资
/*思路：
1.查询‘IT_PROG’部门的任一工资
2.查询员工信息，salary<1的结果
*/
SELECT employee_id,job_id,salary
FROM employees
WHERE salary<ANY(
	SELECT DISTINCT salary
	FROM employees
	WHERE job_id = 'IT_PROG'
) AND job_id<>'IT_PROG';

# 小于any，只要比最大值小就可以，因此可以不用any
SELECT employee_id,job_id,salary
FROM employees
WHERE salary<(
	SELECT MAX(salary) 
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';

# -------------用在select后使用子查询------------
#案例：查询每个部门的员工个数和部门信息
SELECT d.*,(
	SELECT COUNT(*)
	FROM employees e
	WHERE e.department_id = d.`department_id`
 ) 个数
 FROM departments d;
 
# -------------用在from后使用子查询------------
# from后面的子查询结果作为一张表，必须起别名
# 案例：查询每个部门的平均工资的工资等级
/*思路：
1.使用子查询，查询每个部门的平均工资
2.根据平均工资查找工资等级
*/
SELECT  ag_dep.*,g.`grade_level`
FROM (
	SELECT AVG(salary) ag,department_id
	FROM employees
	GROUP BY department_id
) ag_dep
INNER JOIN job_grades g
ON ag_dep.ag BETWEEN lowest_sal AND highest_sal;

# -------------用在exists后使用子查询------------
# 查询有员工的部门名
# 使用 in
SELECT department_name
FROM departments d
WHERE d.`department_id` IN(
	SELECT department_id
	FROM employees
);
# 使用外连接
SELECT DISTINCT department_name
FROM departments d
INNER JOIN employees e
ON d.`department_id`=e.`department_id`
WHERE e.`employee_id` IS NOT NULL;

# 使用exists
SELECT department_name
FROM departments d
WHERE EXISTS(
	SELECT *
	FROM employees e
	WHERE d.`department_id`=e.`department_id`
);
```



## 8、分页查询

如果想要显示查询结果中的一部分，就需要使用分页查询，使用`limit`限制返回的结果开始位置和条数。

分页查询通常用于实际web项目中，根据用户需求提交对应页数的查询结果。

语法：

```mysql
select 查询内容
from 表
[where 条件]
[group by 分组字段]
[having 条件]
[order by 排序字段]
limit [偏移量,] 条目数;  # 使用limit语句进行分页
```

说明：

* 偏移量从0开始，可以省略，默认为0，理解为索引-1。偏移量为0表示从第一条开始
* `limit`子句放在查询语句的最后。



如果每页显示条目数为`sizePage`，当前页数为`page`，则查询语句为：

```mysql
select * from 表
limit (page-1)*sizePage,sizePage;
```

案例：

```mysql
#案例1：查询第11条到第25条信息
SELECT * FROM  employees LIMIT 10,15;

#案例2：查询有奖金的员工中，工资较高的前10名员工信息
SELECT * FROM employees 
WHERE commission_pct IS NOT NULL 
ORDER BY salary DESC 
LIMIT 10;
```



**子查询经典案例，使用排序和分页组合求最值**

```mysql
# 1.查询工资最低的员工姓名和工资
SELECT last_name,salary
FROM employees
WHERE salary = ( # 先使用子查询查找最低工资
	SELECT MIN(salary) FROM employees
);

# 2.查询平均工资最低的部门信息
/*思路：
1.先查询每个部门的平均工资
2.筛选出平均工资中最低的部门
3.根据上一步的结果，查询出部门信息
*/
# 方式一：
SELECT * FROM departments d # d.根据id，查询部门信息
WHERE d.`department_id` = 
(
	SELECT department_id # c.找出平均工资为最低平均工资的部门id
	FROM employees
	GROUP BY department_id
	HAVING AVG(salary)=
    ( 
		SELECT MIN(ag.a_s) # b.找出最低的平均工资
		FROM( # a.查询出每个部门的平均工资和id
			SELECT AVG(salary) a_s,department_id 
            FROM employees
			GROUP BY department_id
			) AS ag
	)
);
# 方式二，使用limit，求出平均工资以后进行排序，就可以直接选出部门id
SELECT * FROM departments d
WHERE d.`department_id`= 
(
	SELECT department_id 
	FROM employees
	GROUP BY department_id
	ORDER BY AVG(salary)
	LIMIT 1   # 递增排序中的第一条就是要查询的部门
);

# 3.各个部门中 最高工资最低的部门的最低工资是多少
SELECT MIN(salary) FROM employees
WHERE department_id = ( # 子查询将排序和分页组合找出最值
	SELECT department_id 
    FROM employees
	GROUP BY department_id
	ORDER BY MAX(salary) 
	LIMIT 1
);

# 4.查询平均工资最高的部门的manager信息
SELECT last_name,department_id,email,salary
FROM employees
WHERE employee_id=
( # b,根据部门id找出管理者id
	SELECT manager_id FROM departments
	WHERE department_id = 
    (  # a,找出平均工资最高的部门id
		SELECT department_id FROM employees
		GROUP BY department_id
		ORDER BY AVG(salary) DESC
		LIMIT 1
	)
);
```



## 9、联合查询

如果要查询的结果**来自于多个表，且多个表没有直接的连接关系，但查询的信息一致时**，可以使用**联合查询**。联合查询使用`union`将多次的查询结果合并成一个结果。

语法：

```mysql
# 使用union将多个查询语句合并
select xxx
union [all]
select xxx
union [all]
.....
select xxx
```

说明：

* 要求多条查询语句的查询的**列数必须一致**
* 多条查询语句的查询的列的类型和含义尽量相同
* `union`默认去重，`union all`代表不去重



**`join`和`union`的区别**

* `join`用于外连接，是根据一定的**连接条件**将两张表连接，并生成连接后的结果表，连接条件是`on`后面的条件。连接包括左外连接、右外连接、全连接和交叉连接。
* `union`表示联合查询，是将两个查询结果合并在一起，不需要进行表的连接。`union`连接的查询语句查询的字段个数必须一致。`union`默认去重，可以使用`all`保留全部结果。

# 六、DML语言

DML指数据操作语言。

主要是对表格的添加（insert）、删除（delete）、修改（update）操作。

## 1、insert

向表格中插入数据有两种方式：

**方式一**：

```mysql
insert into 表名(字段1,字段2,...) values(值1,值2,...);
```

要求：

* 字段类型和值类型必须一致或兼容，字段和值的顺序可以和表中不一致，但是字段和值必须一一对应。
* 可以为空的字段，如果要插入`null`值，有两种方式：①字段和值都省略，此时值默认为`null`；②字段和值都写，值为`null`。
* 不可以为空的字段，必须插入值。
* 字段个数和值的个数必须一致。
* 字段可以省略，但默认所有字段，并且顺序和表中的存储顺序一致。

**方式二**：

```mysql
insert into 表名
set 列名1=值,列名2=值,...;
```

两种方式的对比：

* 方式一支持插入多行，方式二不支持：

  ```mysql
  INSERT INTO employees(employee_id,last_name,salary)
  values(123,'john','15000'),
  values(124,'jerry','12000'),
  values(125,'mickey','11000');
  ```

* 方式一支持子查询，方式二不支持：

  ```mysql
  # 查询标量
  INSERT INTO employees(employee_id,last_name,salary)
  SELECT 123,'john','15000';
  
  # 从表2中查询到的结果插入表1
  INSERT INTO employees(employee_id,last_name,salary)
  SELECT id,name,salary
  FROM employees WHERE id<3; 
  ```

  

## 2、update

**修改单表的记录**：

```mysql
update 表名 set 字段=新值,字段=新值...
[where 条件];

# 案例
# 将90号部门的员工工资修改为15000
update employees
set salary=15000
where department_id90;
```

**修改多表记录**：

```mysql
# SQL 92语法
update 表1 别名1,表2 别名2
set 字段=新值，字段=新值...
where 连接条件
[and 筛选条件];

# SQL 99语法，唯一不同的就是连接表的方式不同。其余相同
update 表1 别名
[inner]|left [outer]|right [outer] join 表2 别名
on 连接条件
set 列=值,...
[where 筛选条件];
```



## 3、delete

delete表示删除表中的一条或多条记录（一行或多行），有两种方式

**方式1，使用delete**

单表的删除：

```mysql
delete from 表名 
[where 筛选条件]
[limit 条目数]
```

一般会使用`where`筛选特定的记录删除，如果没有筛选条件，会删除所有的记录。

可以使用`limit`限定删除的条数，只能用一个参数，表示删除查询结果的前几条。

多表的删除（级联删除）：

```mysql
# 同样地，也分为两种标准的语句，仅仅是连接表的时候不同，其余语句完全相同。
# 以SQL 92为例
delete 别名1，别名2
from 表1 别名1，表2 别名2
where 连接条件
[and 筛选条件];
```



**方式2，使用truncate**：

```mysql
truncate table 表名;
```



`delete`和`truncate`两种方式删除记录的区别（详细对比可以参考[下文](#refer1)）：

* `truncate`不能加`where`条件，而`delete`可以加`where`条件，意味着`truncate`会删除表中的所有记录。
* `truncate`的效率稍高。
* `truncate` 删除带自增长的列的表后，如果再插入数据，数据从1开始，`delete` 删除带自增长列的表后，如果再插入数据，数据从上一次的断点处开始

* `truncate`删除没有返回值，`delete`删除有返回值
* `truncate`属于DDL语言，删除不能回滚，`delete`是DML语言，支持事务，可以回滚

# 七、DDL语句

DDL为数据定义语言。允许用户**定义**数据，包括创建（create）、删除（drop）、修改（alter）表和数据库，操作对象是**表和数据库**。通常，DDL由数据库管理员执行。

## 1、create

**创建数据库**：

```mysql
create database [if not exists] 库名;
```

**创建表**：

```mysql
# 语法：
create table [if not exists] 表名(
	列名 列的类型[(长度) 约束],
	列名 列的类型[(长度) 约束],
	列名 列的类型[(长度) 约束],
	...
	列名 列的类型[(长度) 约束]
);

# 例：创建名为stuinfo的表，包含的列和类型如下：
CREATE TABLE IF NOT EXISTS stuinfo(
	stuId INT,
	stuName VARCHAR(20),  # varchar类型必须有长度
	gender CHAR,
	bornDate DATETIME
);
```

创建库/表的时候，可以使用`if not exists`进行判断，如果要创建的库/表不存在，则创建，否则不会创建。不能创建名字相同的两个库/表。

**表的复制**

将`create`和`select`语句结合，可以实现表的复制：

```mysql
# 仅复制表的结构
create table 新表名 like 旧表

# 复制表的字段和数据
create table 新表名
select 字段列表 from 旧表 [where 筛选条件];

#仅仅复制某些字段
create table 新表名
select 字段列表 from 旧表 where false;
```



## 2、alter

修改库/表使用`alter`语句

修改数据库，一般很少对现有数据库进行修改：

```mysql
#已废弃.现在需要去文件路径中重命名
RENAME DATABASE books TO 新库名; 
# 更数据库字符集
ALTER DATABASE books CHARACTER SET gbk;
```

修改表：

修改表中的字段：

```mysql
# 添加新的列，必须指定类型
# 新列默认在表中的位置为最后，可以使用first或after参数指定位置
alter table 表名 add column 列名 类型 [约束] [first|after 列名];

# 删除某列（无法使用if exist进行判断）
alter table 表名 drop column 列名;

# 修改列名
alter table 表名 change column 旧列名 新列名 类型;

# 修改列的类型或约束
alter table 表名 modify column 列名 类型 [约束];
```

修改表名：

```mysql
# 格式：
alter table 表名 rename [to] 新表名;
# 案例：
ALTER TABLE author RENAME TO book_author;
```



## 3、drop

删除数据库：

```mysql
drop database [if exists] 库名;
```

删除表：

```mysql
drop table [if exists] 表名;
```

同样地，删除数据库/表的时候，可以使用`if exists`判断是否存在，避免报错。



# 八、约束和标识列

## 1、约束

约束指的是一种限制，用于限制表中的数据，为了保证表中的数据的准确和可靠性。

主要有以下六大约束：

* `NOT NULL`：非空，用于保证该字段的值不能为空。比如姓名、学号等
* `DEFAULT`：默认，用于保证该字段有默认值。比如性别
* `CHECK`：检查约束[mysql中不支持]。比如年龄、性别，检查年龄是否在某个区间内、性别是否是男或女。
* `PRIMARY KEY`：主键，用于保证该字段的值具有唯一性，并且非空。比如学号、员工编号等
* `UNIQUE`：唯一，用于保证该字段的值具有唯一性，可以为空（`unique key`可以插入多个`null`值，空值并不受唯一性约束）。比如座位号。
* `FOREIGN KEY`：外键，用于限制两个表的关系，用于保证该字段的值必须来自于主表的关联列的值在从表添加外键约束，用于引用主表中某列的值。比如学生表的专业编号（学生表为从表，专业表为主表），员工表的部门编号，员工表的工种编号。

其中主键、外键、唯一键都是`key`，会默认生成索引。

> **关于key**
>
> [MySQL文档](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)：key是索引的近义词。
>
> [参考1](https://sqlrelease.com/sql-server-tutorial/types-of-keys)：key是表中的字段，它们参与RDBMS系统中的以下活动
> a.表与表中数据之间的依赖关系由key建立
> b.标识数据的唯一性。
> c.使表的约束有效果, 进而能够保证数据都是有效的。
> d.有可能会提升数据库表的查询效率。
>
> key 包含两层意义和作用：
> 一是约束（偏重于约束和规范数据库的结构完整性）；
> 二是索引（辅助查询用）包括primary key (主键)、unique key(唯一键)、foreign key(外键) 等

约束可以在**创建表**和**修改表(添加数据前)**时添加。

添加约束有两类：

* 列级约束：六大约束语法上都支持，但外键约束没有效果。不可以起约束名。
* 表级约束：除了非空、默认，其他的都支持。可以起约束名（约束名对于主键没有效果，一直是primary）



**主键和唯一键对比**：

* 主键的值是唯一的，非空，一个表中最多只能有一个主键，允许多个字段组合在一起作为主键，但不推荐。
* 唯一键的值是唯一的，但是允许值为空，一个表中可以有多个唯一键，同样允许多个字段组合在一起作为唯一键，但不推荐。



**外键**：

* 要求在从表设置外键关系
* 从表的外键列的类型和主表的关联列的类型要求一致或兼容，名称无要求
* 主表的关联列必须是一个key（一般是主键或唯一）
* 插入数据时，先插入主表，再插入从表。删除数据时，先删除从表，再删除主表：

```mysql
#方式一：级联删除，删除主表数据时，将从表中相关的记录(整行)也删除
#添加外键时，在最后添加 on delete cascade。例如：
ALTER TABLE stuinfo 
ADD CONSTRAINT fk_stuinfo_major 
FOREIGN KEY(majorid) 
REFERENCES major(id) ON DELETE CASCADE; 

# 方式二：级联置空，删除主表数据时，将从表中相关的记录中对应的字段置为null
#添加外键时，在最后添加 on delete set null。例如：
ALTER TABLE stuinfo 
ADD CONSTRAINT fk_stuinfo_major 
FOREIGN KEY(majorid) 
REFERENCES major(id) ON DELETE SET NULL; 
```

**添加约束**

列级约束，直接在类型后添加约束名

```mysql
# 例如。
create table 表名(
    字段名 字段类型 not null,  # 非空
    字段名 字段类型 primary key,  # 主键
    字段名 字段类型 unique,  # 唯一键
    字段名 字段类型 default,  # 默认
    ...
);
```

表级约束，在字段都定义完后，再为指定的列添加约束

```mysql
# 语法：
[constraint 约束别名] 约束(字段名);
# 例如：
CREATE TABLE stuinfo(
	id INT,
	stuname VARCHAR(20),
	gender CHAR(1),
	seat INT,
	age INT,
	majorid INT,
    
	CONSTRAINT pk PRIMARY KEY(id),#主键
	CONSTRAINT uq UNIQUE(seat),#唯一键
	CONSTRAINT ck CHECK(gender ='男' OR gender  = '女'),#检查（MySQL不支持）
	CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id)#外键
);
```

如果多个字段组合作为主键或唯一键，将这几个字段都写在表级约束的括号中，逗号隔开。

表级约束和列级约束可以一起使用，定义外键时使用表级约束，其他约束使用列级约束即可。

查看表中的所有索引（主键、外键、唯一键）：

```mysql
SHOW INDEX FROM 表名;
```

除了在定义表时添加约束，也可以在修改表的时候添加约束：

```mysql
#1、添加列级约束
alter table 表名 modify column 字段名 字段类型 新约束;

#2、添加表级约束
alter table 表名 add [constraint 约束名] 约束类型(字段名) [外键相关];
```

**删除约束**

在修改表的语句中，也可以删除约束，一般的约束在类型后面不加约束就代表删除，对于`key`，则需要使用`drop`显式删除，具体如下：

```mysql
#1.删除非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20) NULL;

#2.删除默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT;

#3.删除主键
ALTER TABLE stuinfo DROP PRIMARY KEY;

#4.删除唯一键，使用删除索引的语法
ALTER TABLE stuinfo DROP INDEX seat;

#5.删除外键，需要先将对应的索引删除
ALTER TABLE stuinfo DROP INDEX fk_stuinfo_major;
ALTER TABLE stuinfo DROP FOREIGN KEY fk_stuinfo_major;
```

> MySQL中直接删除外键以后，外键依然存在，需要先将同名索引删除，然后再删除外键，这样查询时就不会显示外键了。
>
> 删除外键的时候，同名索引如果没被删，MYSQL认为外键仍然存在，MYSQL会在show keys命令里继续显示外键，当drop table时，MYSQL会提示Can't DROP 'xxx'; check that column/key exists"
> 如果再次想删除外键的时候，会报1091错误，提示外键名错误，因为实际上外键已经不存在了。因此MySQL中需要先将对应的索引删除，再删除外键。



## 2、标识列

**标识列**又称为**自增长列**，可以不用手动的插入值，系统提供默认的序列值，默认从1开始。

**格式**：在key的约束后面加上` auto_increment`即可。

**用法**：添加数据时，标识列可以填`null`，或者不赋值，系统都会自动在已有的基础上+1。也可以手动指定值。

**特点**：

* 标识列不是必须和主键搭配，但要求是一个key。
* 一个表最多只能有一个标识列。
* 标识列的类型只能是数值型，一般是int型。如果是非int型，自动赋值时是整数，可以手动插入小数。
* 可以通过`SHOW VARIABLES LIKE '%auto_increment%';` 查看标识列的步长和默认起始值.
* 标识列可以通过` SET auto_increment_increment=n`设置步长。mysql不支持设置起始值，但可以通过手动插入值，设置起始值。

**使用**：

```mysql
# 创建表时设置标识列
CREATE TABLE tab_identity(
	id INT,
	NAME FLOAT UNIQUE AUTO_INCREMENT,
	seat INT 
);

# 修改表时设置标识列
ALTER TABLE tab_identity MODIFY COLUMN id INT PRIMARY KEY AUTO_INCREMENT;

#三、修改表时删除标识列
ALTER TABLE tab_identity MODIFY COLUMN id INT;

```



# 九、数据库事务

**TCL（Transaction Control Language）**： 事务控制语言

## 1、事务概述

**事务**：一个或一组SQL语句（一般是DML语句）组成的一个**执行单元**，这个执行单元要么全部执行，要么全部不执行。

DDL语句不支持事务，可以认为DDL语句执行完自动提交，因此也无法回滚。

**事务的四大特性（ACID）**：

* **原子性**（**A**tomic）：组成事务的SQL语句要么都执行，要么都不执行（回滚）。
* **一致性**（**C**onsistent）：事务完成前后，所有数据的状态都是一致的。比如A账户只要减去了100，B账户则必定加上了100。
* **隔离性**（**I**solation）：多个事务同时操作相同数据库的同一个数据时，即多个事务并发执行，每个事务作出的修改必须与其他事务隔离。
* **持久性**（**D**uration）：事务一旦提交，对数据库数据的修改被持久化到本地存储。



**事务分类**：

* **隐式事务**：没有明显的开启和结束的标记，本身就是一个事务。对于单条SQL语句，数据库系统自动将其作为一个事务执行，这种事务被称为隐式事务。**隐式事务系统会自动提交**。
* **显式事务**：具有明显的开启和结束的标记，即手动开启事务。显式事务必须保证系统的自动提交功能被禁用，实际上，MySQL中显示开启事务时，自动提交是无效的，不需要手动设置。



## 2、事务的开启

一个完整的事务包括以下执行流程：开启事务→执行事务语句→提交或回滚。

事务的执行结果要么是提交，要么是回滚。

**步骤一：开启事务**

开启事务有两种方式，一种是关闭自动提交模式，隐式开启事务；另一种是显式开启事务，有两种显式开启事务的语法，具体如下：

方式一：**关闭自动提交模式**，之后遇到需要开启事务的sql时，会自动开启事务，相当于隐式开启事务:

```mysql
set autocommit=0; # 将自动提交模式关闭
执行语句
```

方式二：**显式开启事务**

```mysql
# 方式一：
start transaction;

#方式二：
begin;
```

**步骤二：执行语句**

在这一步，可以编写执行语句，包括insert、update、delete等语句。

DDL语言（create、drop、truncate等）是非事务的，可以理解为事务中的DDL语言执行完后会自动提交（禁用自动提交也没用），无法回滚，影响事务其他DML操作。

如果事务中掺杂了DDL语句，执行完DDL语句后会自动提交，导致之前的执行语句被提交，无法回滚，因此为了安全起见，尽量将DDL和DML完全分开，便于回滚。

**步骤三：结束事务**

事务的结束有两种情况：

* `commit [work]`：提交，表示事务的执行单元都执行了。
* `rollback [work]` ：回滚，表示事务的执行单元都没执行。

> 事务相关的其他关键字：
>
> `savepoint identifier`：设置一个名为identifier的保存点
>
> `release savepoint identifier`：删除一个事务的名为identifier的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；
>
> `rollback to identifier` ：把事务回滚到保存点。不算真正地结束事务，仍可以使用rollback将整个事务的修改撤销，因此执行了此语句后，也需要显式运行commit或rollback命令结束事务。

事务的使用案例：

```mysql
#开启事务，三种均可
#SET autocommit=0;
#START TRANSACTION;
begin;

#事务执行语句
UPDATE employees SET salary = 17200 WHERE username='K_ing';
UPDATE employees SET salary = 10000 WHERE username='Marry';

#结束事务，要么提交，要么回滚
ROLLBACK;
#commit;
```

<span id="refer1">**drop、delete、truncate三者的区别**</span>：

* drop用于删除整个表。
* delete可以使用筛选条件，删除表中的一行或多行记录。并且同时将该行的删除操作作为事务记录在日志中保存以便进行进行回滚操作。
* truncate table则一次性地从表中删除所有的数据，即所有行，不能使用筛选条件。并不把单独的删除操作记录记入日志保存，操作不能恢复。
* delete是DML语言，支持事务，delete操作可以会滚；而truncate和drop是ddl语言，不支持事务，因此无法回滚。
* truncate和delete只删除数据，drop则删除整个表，包括结构和数据。
* delete可以激活触发器。truncate不能激活触发器。
* 当表被truncate后，这个表和索引所占用的空间会**恢复到初始大小**，并且标识列也从1重新开始计数；delete操作不会减少表或索引所占用的空间，标识列还是从断点处计数。drop语句将表所占用的空间全释放掉。
* 速度上：drop>truncate>delete



## 3、事务的隔离级别

当多个事务同时操作同一个数据库的相同数据时，容易导致并发问题，包括以下问题：

**脏写**：一个事物修改了其他事务还没有提交的数据。事务B去修改了事务A修改过的值，但是此时事务A还没提交，所以事务A随时会回滚，导致事务B修改的值也没了。
**脏读**：一个事务读取到了其他事务未提交（一般指**数据修改**）的数据。事务B去查询了事务A修改过的数据，但是此时事务A还没提交，所以事务A随时会回滚，导致事务B再次查询就读不到刚才事务A修改的数据了。

>  脏读和脏写都是因为一个事务去更新或者查询了另外一个还没提交的事务更新过的数据。因为另外一个事务还没提交，所以它随时可能会回滚，那么必然导致你更新的数据就没了，或者你之前查询到的数据就没了，这就是脏写和脏读两种场景。

**不可重复读**：同一个事务中，多次读取到的数据不一致，即中间有其他事务提交了修改（一般指**数据修改**）。事务A读取一个字段后，事务B更新了该字段并提交了，导致A再次读取的时候和之前不一致了，即无法重复读取到相同的某个值。

**幻读**：一个事务多次读取数据时，中间有其他事务提交了更新操作（一般指**插入或删除**）的数据。比如同样的查询语句，第一次查询出n条，然后别的事务进行了插入或删除，第二次查询出m条，同样的查询语句得到的结果不一样，就像出现了幻觉。



可以通过设置**隔离级别**避免事务的并发问题，主要有以下四种隔离级别：

<table>
	<tr>
		<td width="23%">隔离级别</td>
		<td width="10%"><b>脏读</b></td>
        <td width="10%"><b>不可重复读</b></td>
        <td width="10%"><b>幻读</b></td>
        <td width="47%"><b>说明</b></td>
	</tr>
    <tr>
		<td>read uncommitted</td>
		<td>未解决</td>
        <td>未解决</td>
        <td>未解决</td>
        <td>允许事务读取未被其他事务提交的更改</td>
	</tr>
    <tr>
		<td>read committed</td>
		<td>解决</td>
        <td>未解决</td>
        <td>未解决</td>
        <td>只允许事务读取已经被其它事务提交的更改。<b>Oracle的默认隔离级别</b></td>
	</tr>
    <tr>
		<td>repeatable read</td>
		<td>解决</td>
        <td>解决</td>
        <td>未解决</td>
        <td>确保事务可以多次从一个字段中读取相同的值，事务持续期间，禁止其他事务对这个字段更新。<b>MySQL的默认隔离级别</b></td>
	</tr>
    <tr>
		<td>serializable</td>
		<td>解决</td>
        <td>解决</td>
        <td>解决</td>
        <td>确保事务可以多次从一个表中读取相同的行，事务持续期间，禁止其他事务对该表执行插入、更新和删除操作。可以避免所有并发问题，性能最差。</td>
	</tr>
</table>



上述四种隔离级别，MySQL全部支持，Oracle只支持`read committed`和`serializable`。



## 4、设置隔离级别

事务的隔离级别包括**全局级别**和**会话级别**：

* 全局级别：对后续新的所有会话连接有效，对已经存在的会话连接无效。
* 会话级别：对数据库会话连接的**后续新发起或当前未提交**的事务有效。如果没有session关键字的话，只对当前数据库会话连接的**后续新发起**的事务有效，当前未提交的事务，还是继续使用之前的事务隔离级别。

设置隔离级别的语法：

```mysql
# 设置全局隔离级别
set global transaction isolation level 隔离级别名;

# 设置当前会话连接的隔离级别
set [session] transaction isolation level 隔离级别名;
```

查看隔离级别：

```mysql
#查看当前会话隔离级别
select @@tx_isolation;
#查看全局隔离级别
select @@global.tx_isolation;
```



# 十、视图

## 1、概述

视图是MySQL 5.1的新特性。视图是一张**虚拟的表**，它的数据来自于表，通过执行时动态生成。视图的用法和表相同。视图只保存了SQL逻辑，没有保存查询结果，因此其几乎不占用物理空间，视图一般仅用于查询，仅仅少数情况下才能修改数据。
	

视图的优势：

* 提高SQL语句的重用性，简化了复杂的SQL操作，提高了效率
* 和表实现了分离，提高了安全性

## 2、创建视图

创建视图：

```mysql
create [or replace] view 视图名 as 查询语句;
# 使用or replace还可以用于视图的修改
```

使用案例：

```mysql
# 使用视图，查询各部门的平均工资级别

# 1.创建视图查看每个部门的平均工资
CREATE VIEW myview AS
SELECT AVG(salary) ag,department_id
FROM employees
GROUP BY department_id;

# 2.使用创建的视图查询
SELECT myview.`ag`,g.grade_level
FROM myview
JOIN job_grades g
ON myview.`ag` BETWEEN g.`lowest_sal` AND g.`highest_sal`;
```



## 3、删除视图

删除视图：

```mysql
drop view 视图名,视图名,...;  # 可以一次删除多个视图

#案例：删除myv1，myv2，myv3三个视图
DROP VIEW myv1,myv2,myv3;
```



## 4、查看视图

查看视图有两种方式：

```mysql
#方式一：
DESC 视图名;

#方式二：
SHOW CREATE VIEW 视图名;
```



## 5、更新视图

和修改表相似，更新视图包括修改视图本身，和修改视图中的内容两部分。

**修改视图本身**：

```mysql
#方式一：可用于修改和创建视图
create or replace view  视图名
as 查询语句;

# 方式二：
alter view 视图名
as 查询语句;
```



**修改视图的内容**

一般的视图只用来查询，对视图进行插入、删除、修改数据，如果能操作成功，则会对源表中的数据也会进行修改。但是绝大多数情况下无法对视图进行修改，而且也不建议对视图进行修改。

以下情况的视图无法更新：

* SQL语句包含`分组函数`、`distinct`、`group  by`、`having`、`union`或者`union all`的视图
* 常量视图
* `select`中包含子查询的视图
* 包含`join`的视图
* `from`一个不能更新的视图
* `where`子句的子查询引用了`from`子句中的表



# 十一、变量

变量包括：

* **系统变量**
  * 全局变量
  * 会话变量
* **自定义变量**
  * 用户变量
  * 局部变量

## 1、系统变量

**系统变量**：变量由系统定义，不是用户定义，属于服务器层面。必须拥有super权限才能修改系统变量。

系统变量分为**全局变量**和**会话变量**。全局变量需要添加global关键字，会话变量需要添加session关键字，如果不写，默认为会话级别。

**全局变量**

作用域：服务器每次启动将为所有的全局变量赋初始值，针对于所有会话（连接）有效，但不能跨重启。

查看全局变量：

```mysql
# 查看所有全局变量
SHOW GLOBAL VARIABLES;

# 查看满足条件的部分系统变量
SHOW GLOBAL VARIABLES LIKE 'xxx';

# 查看指定的系统变量的值，
SELECT @@global.变量名
```

设置全局变量：

```mysql
# 以autocommit为例
# 方式一：
SET @@global.变量名=值;
# 方式二：
SET GLOBAL 变量名=值;
```



**会话变量**

作用域：针对于当前会话（连接）有效

查看会话变量：

```mysql
# 查看所有会话变量
SHOW [SESSION] VARIABLES;

# 查看满足条件的部分会话变量
SHOW [SESSION] VARIABLES LIKE 'xxx';

# 查看指定的会话变量的值
#SELECT @@[session.]变量名;
# 例：
SELECT @@autocommit;
# 或写成
SELECT @@session.autocommit;
```

设置会话变量：

```mysql
SET @@session.变量名=值;
SET [SESSION] 变量名=值;
```



## 2、自定义变量

**自定义变量**是由用户自定义，而不是系统提供的。其分为**用户变量**和**局部变量**。

自定义变量的使用一般都有**声明、赋值、使用**（查看、比较、运算等）三个步骤。

**用户变量**

作用域：和会话变量的作用域相同，对于当前会话（连接）有效。在`begin end`里面和外面都可以使用。

声明用户变量，要求声明时必须初始化值。有两种赋值操作符：`=`和`:=`

```mysql
#方式一
SET @变量名=值;

#方式二
SET @变量名:=值;

#方式三
SELECT @变量名:=值;
```

为用户变量赋值：

```mysql
# 方式一，和初始化时相同：
SET @变量名=值;
SET @变量名:=值;
SELECT @变量名:=值;

# 方式二：
SELECT 字段 INTO @变量名 FROM 表;
```

查看用户变量：

```mysql
SELECT @变量名;
```



**局部变量**

作用域：仅在定义它的`begin end`块中有效。

声明局部变量必须在`begin end`块中的最前面部分。

局部变量一般不用加`@`符号，但是声明时需要指定类型，用户变量不需要指定类型。

声明局部变量：

```mysql
DECLARE 变量名 类型 [DEFAULT 值];
```

为局部变量赋值：

```mysql
#方式一，同样是三种：
SET 局部变量名=值;
SET 局部变量名:=值;
SELECT @局部变量名:=值;

#方式二：
SELECT 字段 INTO 局部变量名 FROM 表;
```

查看局部变量的值：

```mysql
SELECT 局部变量名;
```



# 十二、存储过程和函数

存储过程和函数都是事先经过编译 并存储在数据库中的一段SQL语句的集合。

优势：

* 提高了sql语句的重用性，减少了开发人员的压力
* 提高了数据处理的效率
* 减少数据在数据库和应用服务器之间的传输次数



## 1、存储过程

**创建存储过程**

语法：

```mysql
create procedure 存储过程名(参数模式 参数名 参数类型,...)
begin
	存储过程体(一条或多条合法的SQL语句);
end 结束符
```

参数可以有0个（比如只有插入语句）或多个。参数模式包括三种：

* `IN`：表示该参数可以作为输入，即该参数需要调用方传入值
* `OUT`：表示该参数可以作为输出，即该参数可以作为返回值
* `INOUT`：表示该参数既可以作为输入又可以作为输出，即该参数既需要传入值，又可以返回值

存储过程体如果只有一个SQL语句，可以省略`begin`和`end`，如果有多个SQL语句，**每条SQL语句必须需要使用`;`结尾**。

MySQL默认将`;`作为结束符，所以如果存储过程中有多个语句，但又希望将多个语句都执行，因此创建存储过程之前需要将修改默认的结束符。使用 `delimiter `重新设置结束符，保证过程体中的`;`被直接传递到服务器，而不会被客户端解释。例如，将`//`设置结束符的语句为：`delimiter //`

SQLyog 10.0，定义存储过程前必须要设置结束符，生成的存储过程会自动将结束符设置为`$$`，如果没有手动将结束符改为`;`，系统会自动添加`delimiter ;`语句，因此每次创建存储过程都要重新设置结束符。

手写存储过程，建议在开头设置结束符，在末尾将结束符重新设置为`;`



**调用存储过程**

```mysql
call 存储结构名(实参);
# 如果是in模式的参数，可以直接传入值
# 如果是out或inout模式的参数，必须预先定义变量，然后作为参数传入。
```



**删除存储过程**

存储过程不能修改，一般做法是将原来的删除，然后新建。

```mysql
# 每次只能删除一个，不能删除多个
DROP PROCEDURE [IF EXISTS] 存储过程名;
```



**查看存储过程**

```mysql
SHOW CREATE PROCEDURE 存储过程名;
```



例：创建存储过程，输入员工id，输出其管理者的id和名字

```mysql
/* 声明存储结构
*/
DELIMITER @@  # 设置结束符为@@
# 确保存储过程能正确创建，如果已经存在，删除原有的
DROP PROCEDURE IF EXISTS `myp`@@
CREATE PROCEDURE myp(IN id INT,
                     OUT managerId INT,
                     OUT managerName VARCHAR(20))
BEGIN
	# 将查询的结果值赋给输出变量，这里参数为局部变量
	SELECT m.`employee_id`,m.`last_name` INTO managerId,managerName
	FROM employees m 
	INNER JOIN employees e
	ON m.`employee_id`=e.`manager_id`
	WHERE e.`employee_id`=id;
END @@  # end后面要加设置的结束符
DELIMITER ;  # 将结束符改回分号

/*调用存储结构
in模式参数直接传入值，
out模式的参数要先声明，或者传入的时候指定名字
*/
CALL myp (110,@mid,@mname);  # 调用存储结构，传入参数
SELECT @mid,@mname;  # 查看结果值
```



## 2、函数

函数和存储过程相似，唯一的区别是，**存储过程可以有0个返回值**，也可以有多个返回，适合做批量插入、批量更新；而**函数有且仅有1个返回值**，适合做处理数据后返回一个结果的情况。

**创建函数**

```mysql
CREATE FUNCTION 函数名(参数名 参数类型,...) RETURNS 返回类型
BEGIN
	函数体;
	RETURN xx;  #必须有返回值
END 结束符
```

和存储过程相同，函数的参数也可是是0个或多个；结束符和存储过程的用法也相同；如果函数体只有一句，同样可以省略`begin end`语句。

**调用函数**

```mysql
SELECT 函数名(参数列表);  # 存储过程用的是CALL
```



**删除函数**

```mysql
# 同样每次只能删除一个
DROP FUNCTION [IF EXISTS] 函数名; 
```



**查看函数**

```mysql
SHOW CREATE FUNCTION 函数名;
```



函数使用案例：根据部门名，返回该部门的平均工资

```mysql
/*创建函数
*/
DELIMITER @@   # 先将分隔符设置为@@
DROP FUNCTION IF EXISTS `myf`@@
CREATE FUNCTION myf(deptName VARCHAR(20)) RETURNS DOUBLE
BEGIN
	#声明一个局部变量，用于接收查询结果，并返回
	DECLARE sal DOUBLE ;
	SELECT AVG(salary) INTO sal
	FROM employees e
	JOIN departments d ON e.department_id = d.department_id
	WHERE d.department_name=deptName;
	RETURN sal;  # 返回结果值
END @@
DELIMITER ;  # 将分隔符重新设置为分号

/*调用函数
*/
SELECT myf('IT');
```



# 十三、流程控制结构

流程控制结构包括以下三种：

* 顺序结构：程序从上到下依次执行
* 分支结构：程序从两条或多条路径中选择一条去执行，比如`if`、`case`结构
* 循环结构：程序在满足一定条件的基础上，重复执行一段代码，比如`while`、`loop`、`repeat`



## 1、分支结构

分支结构包括`if结构`和`case结构`，只能用于`begin end`中，要和`if函数`、`case函数`区分开，[流程控制函数](#refer2)既可以用在`begin end`里面，也可以用在外面。`if函数`适用于简单双分支，而`if结构`适用于区间判断的多分支，`case`适用于等值判断。

**if结构**

语法：

```mysql
if 条件1 then 语句1;
elseif 条件1 then 语句2;
...
else 语句n;
end if;
```



**case结构**

语法：

```mysql
# 方式一：类似于switch
case 表达式
when 值1 then 语句1;
when 值2 then 语句2;
...
else 语句n;
end case;

# 方式二：类似于多重if
case 
when 条件1 then 语句1;
when 条件2 then 语句2;
...
else 语句n;
end case;
```



**分支结构使用案例**

创建函数，实现传入成绩返回等级的功能。如果成绩>90，返回A，如果成绩>80，返回B，如果成绩>60，返回C，否则返回D。

```mysql
# 方式一：使用if结构
DELIMITER @@   # 先将分隔符设置为@@
DROP FUNCTION IF EXISTS `myf_if`@@
CREATE FUNCTION myf_if(score FLOAT) RETURNS CHAR
BEGIN
	# 设置局部变量保存结果
	DECLARE ch CHAR DEFAULT 'A';
	IF score>90 THEN SET ch='A';
	ELSEIF score>80 THEN SET ch='B';
	ELSEIF score>60 THEN SET ch='C';
	ELSE ch='D';
	END IF;
	RETURN ch;
END @@
DELIMITER ;
#调用函数
SELECT myf_if(87);

# 方式二：使用case结构
DELIMITER @@   # 先将分隔符设置为@@
DROP FUNCTION IF EXISTS `myf_case`@@
CREATE FUNCTION myf_case(score FLOAT) RETURNS CHAR
BEGIN 
	#设置局部变量保存结果
	DECLARE ch CHAR DEFAULT 'A';
	CASE 
	WHEN score>90 THEN SET ch='A';
	WHEN score>80 THEN SET ch='B';
	WHEN score>60 THEN SET ch='C';
	ELSE SET ch='D';
	END CASE;
	RETURN ch;
END @@
DELIMITER ;
#调用函数
SELECT myf_case(56);
```



## 2、循环结构

循环结构包括`while`，`loop`，`repeat`三种。

同样，循环结构只能在`begin end`中使用。

循环结构中，包括两个循环控制语句：

* `iterate`：类似于java中的continue，结束本次循环，进入下一次循环。
* `leave`：类似于java中的breek，跳出当前循环体。

**循环控制语句后面必须有循环结构的标签名**，也就是说，如果循环结构中使用了循环控制语句，此循环结构必须添加标签。

**while结构**


```mysql
[标签:] WHILE 循环条件 DO
	循环体
END WHILE [标签];
```



**loop结构**

`loop`结构没有循环条件，想要结束循环必须使用`leave`语句，`loop`结构可以用于模拟简单的死循环。

```mysql
[标签:] loop
	循环体;
end loop [标签];
```



**repeat结构**

```mysql
[标签:] repeat
	循环体;
until 结束循环条件
end repeat [标签];
```

使用循环结构的案例：

```mysql
# 使用存储结构和循环结构，实现批量插入，要求只插入偶数次的记录
DELIMITER @@   # 先将分隔符设置为@@
DROP PROCEDURE IF EXISTS test_while@@
CREATE PROCEDURE test_while(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 0;
	a: WHILE i<=insertCount DO
		SET i=i+1;
		IF MOD(i,2)!=0 THEN ITERATE a;
		END IF;
		INSERT INTO admin(username,`password`) 
		VALUES(CONCAT('xiaohua',i),'0000');
	END WHILE a;
END @@
DELIMITER ;

# 传入100，只会插入偶数时的记录
CALL test_while1(100);
```