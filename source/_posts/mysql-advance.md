---
title: MySQL数据库高级篇-索引和性能优化
excerpt: MySQL索引的使用，索引优化，性能优化；MySQL锁机制，MVCC，主从复制。
mathjax: true
date: 2021-06-07 12:00:31
tags: ['数据库','MySQL','SQL']
categories: 数据库
keywords: MySQL,SQL,索引,性能优化,Expalin,查询日志,索引分析,锁,MVCC,主从复制
---

***



首先需要了解MySQL的逻辑架构：

![MySQL逻辑架构](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_1.png)







还可以参考这篇：[一条 SQL 语句在 MySQL 中如何执行的](https://snailclimb.gitee.io/javaguide/#/docs/database/一条sql语句在mysql中如何执行的)

什么原因会导致SQL执行速度变慢呢？

1、查询语句写的有问题。

2、索引失效。

3、由于设计缺陷或者不得已的需求，导致关联查询有太多join连接。

4、服务器各个参数设置问题。

如果想要提高SQL语句执行效率，就要从以上几个方面入手，其中索引是主要的关注点。

***



# 一、索引介绍

## 1.1 索引简介



**索引（Index）**是帮助MySQL高效获取数据的**数据结构**。除数据本身之外，数据库还维护着一个满足特定查找算法的数据结构，这些数据结构以某种方式指向数据，这样就可以在这些数据结构的基础上实现高级查找算法，这种数据结构就是**索引**。

索引的本质是数据结构。可以简单理解为**“排好序的快速查找数据结构”**。也就是说**索引不仅可以用于查找，也可以用于排序**。

常见的索引结构有：**B树（B+树是一种扩展的B树）**、**Hash**、**R-Tree**等

> B-Tree，即B树，B+Tree是一种扩展的B-Tree，读作B+树，B-Tree不是B-树。:-)



## 1.2 索引底层结构

索引的底层数据结构有**B树**、**哈希**、**R-Tree(空间数据索引)**等。

MySQL使用的**B树**结构作为索引的数据结构，具体来说是使用**B+树**的结构实现的。这里主要讨论MySQL数据库的InnoDB引擎的B+树结构。

使用`show index from 表名;`可以查看指定表的索引情况，其中`index_type`为`BTREE`说明MySQL确实使用的是B树实现的索引结构，更具体点来说是用的扩展的B树，即B+树。

![索引信息](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_2.png)





#### Hash

**哈希索引 （Hash index）**基于哈希表实现，只有精确匹配索引所有列的查询才有效#4。对于每一行数据，存储引擎都会对所有的索引列计算一个哈希码(hash code)，哈希码是一个较小的值，并且不同键值的行计算出来的哈希码也不一样。哈希索引将所有的哈希码存储在索引中，同时在哈希表中保存指向每个数据行的指针。

在MySQL中，只有Memory引擎显式支持哈希索引。这也是 Memory引擎表的默认索引类型，Memory引擎同时也支持B-Tree索引。值得一提的是，Memory引擎是支持非唯一哈希索引的，这在数据库世界里面是比较与众不同的。如果多个列的哈希值相同，索引会以链表的方式存放多个记录指针到同一个哈希条目中。

Hash索引最大的优点就是**能够在很短的时间内，根据 Hash 函数定位到数据所在的位置**，也就是说 Hash索引检索指定数据的时间复杂度可以接近 O(1)。

<font color='red'>MySQL 并没有使用 Hash 索引而是使用 B+树作为索引的数据结构的原因：</font>

* **Hash 冲突问题**。
* **Hash 索引不支持顺序和范围查询。**这是Hash索引最大的缺点*，假如我们要对表中的数据进行排序或者进行范围查询，那 Hash 索引就无效了，会导致全表扫描。

试想一种情况:

```mysql
SELECT * FROM tb1 WHERE id < 500;
```

Hash 索引是根据 hash 算法来定位的，在这种范围查询中，Hash索引无法处理的，而**B树**结构优势很大，直接遍历比500小的叶子节点即可。



#### B-Tree

**B-Tree**，即B树，**多路平衡查找树**，B+树是B树的一种变体。MySQL使用B树作为索引结构，但是没有直接使用B树的结构，而是使用B树的变体B+树，不同的存储引擎对于B+树的具体实现是不同的。

**B树和B+树结构对比**，参考自[[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)]：

- B树每个非叶子节点由n-1个key和n个指针组成，这n-1个key划分出了n个范围。这些key限定了其子节点中的最值。所有叶子节点具有相同的深度，等于树高h。

- B+树的每个非叶子节点的key和指针个数相等，即n个key，n个指针，key表示子节点元素中的最大（或最小）元素。

- B 树的所有节点既存放键(key)也存放数据(data)，而 B+树只有叶子节点存放 key 和 data，其他内节点只存放 key。

- B 树的叶子节点都是独立的，在MySQL中为了提高区间访问的性能，在经典B+树的叶子节点中额外添加了一条引用链，指向与它**相邻**的叶子节点。（是双向指针，可以用来正序排序和逆序排序）

  > 一般所说的B+树都是MySQL这种叶子节点带顺序访问指针的B+树。



B树和B+树的结构图如下，来自[知乎](https://zhuanlan.zhihu.com/p/149287061)：

![B-Tree结构图](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_3.jpg)

![B+树](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_4.jpg)



上图的B+树的结点值，表示子节点元素的最大值，也可以是表示子节点元素的最小值，两种方式均可。

对于B+树在MySQL中的实现索引的结构，可以参考下图：

![建立在B-Tree结构（从技术上来说是B+Tree）上的索引](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_5.png)





B-Tree对索引是**顺序组织存储**的，因此很适合查找范围数据，比如一个包含last_name、first_name和dob列的索引，其B树结构（使用B+树实现）如下：

![B-Tree(技术上来说是B+Tree)索引树中BUFF条目示例](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_6.png)





<font color='red'>MySQL使用的是B+树作为索引结构的实现，而没有直接使用B树，这是为什么呢？</font>

索引往往以索引文件的形式存储在磁盘上，这样索引的查找过程要产生磁盘I/O消耗，而I/O操作很耗时，I/O次数越多，查询的效率也就越低。因此索引的结构要尽量减少磁盘I/O操作次数。

* 对于查询单个值：B 树的检索的过程相当于对范围内的每个节点的关键字做二分查找，可能还没有到达叶子节点，检索就结束了，查询性能不稳定。而 **B+树的任何查找都是从根节点到叶子节点的过程，查询次数就是树的高度，检索效率稳定**。
* 对于查询连续数据：由于B树在非叶子节点中存储了数据，**导致查询连续数据时可能会带来更多额外I/O操作**，因为叶子节点的不连续，可能会多次I/O查找目标节点，因此查询时间很长。而**B+树只在叶子节点存储键和数据**，其他非叶子节点只存储了键（以及指向子树的指针），并且所有叶子节点可以通过指针相互连接，**减少了顺序遍历时产生的额外I/O操作**，从而大大提升了效率。因此**对于区间查询，B+树更简便实用**。



对于连续数据（范围）查找的举例，对于上图的B树和B+树，比如要查询[45,70]区间的所有节点：

**B树中的查找过程**： 

①加载根节点所在的页，发现59大于45；

②根据根节点指针加载左节点所在的页，发现44小于45；

③继续加载右节点所在的页，找到节点51；

④然后从51开始，进行中序遍历，重新加载44所在的节点，发现没有70，再向上加载根节点59，然后加载节点72，最后找到63。如果每次加载算一次随机I/O操作的话，一共需要7次I/O操作。如果重复加载不算次数，至少也需要5次I/O

**B+树中的过程**：加载[59,97]节点->加载[15,44,59]节点->加载[51,59]节点所在的页，利用叶子节点指针顺序遍历，加载[63,72]节点所在的页。最多需要4次I/O操作。减少了额外的I/O操作



#### R-Tree（空间数据索引）

以下内容来自《高性能MySQL 第3版》：

MyISAM表支持空间索引，可以用作地理数据存储。和B-Tree索引不同，这类索引无须前缀查询。空间索引会从所有维度来索引数据。查询时，可以有效地使用任意维度来组合查询。必须使用MySQL 的GIS相关函数如`MBRCONTAINS()`等来维护数据。MySQL的GIS支持并不完善，所以大部分人都不会使用这个特性。开源关系数据库系统中对GIS的解决方案做得比较好的是 PostgresQL的PostGIS。



## 1.3 MyISAM和InnoDB索引对比

不同的存储引擎对于B+树的实现方式不同，以MyISAM和InnoDB引擎为例。

**MyISAM存储引擎**

* 使用前缀压缩技术使得索引更小。
* MyISAM的索引文件和数据文件是分离的，属于**“非聚簇索引（或非聚集索引）”**。B+树叶子节点的 **data 域存放数据记录的地址**。在索引检索的时候，首先按照 B+树搜索算法搜索索引，如果指定的 Key 存在，则取出其 data 域的值，然后以 data 域的值为地址读取相应的数据记录。



**InnoDB存储引擎**

* InnoDB按照原数据格式进行存储索引。
* 而InnoDB则根据主键引用被索引的行。
* InnoDB 引擎中，表数据文件本身就是按B+树组织的一个索引文件，树的叶节点 data 域保存了完整的数据记录。这个索引的 key 是数据表的主键id，因此**InnoDB表数据文件本身就是主键索引。这被称为“聚簇索引（或聚集索引）”**，而其余的索引都作为**辅助索引**，**辅助索引的 data 域存储相应记录主键的值而不是地址**，这也是和 MyISAM 不同的地方。在根据主索引搜索时，直接找到 key 所在的节点即可取出数据。而根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。 因此，在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。



## 1.4 索引种类



从逻辑上，根据索引列是否是主键，索引可以分为**主键索引（primary key）**和**二级索引（辅助索引）**。

### 1.4.1主键索引

**数据表的主键列使用的就是主键索引。**

InnoDB的主键索引叶子节点存放的是数据本身，即<id,row>的形式，id是主键，可以通过id找到该行的全部列数据。

一张数据表只能有一个主键，并且主键不能为null，不能重复。

在 MySQL 的InnoDB引擎的表中，当没有显式地指定表的主键时，InnoDB会自动先检查表中是否有唯一索引的字段，如果有，则选择该字段为默认的主键，否则 InnoDB将会自动创建一个 6Byte 的自增主键。

<font color='red'>对于InnoDB来说，其主键索引是聚簇索引，data域存储数据本身。而MyISAM的索引属于非聚簇索引，因此其主键索引的叶子节点存放的是数据地址。</font>



### 1.4.2 二级索引

**二级索引又叫辅助索引，二级索引的叶子节点存储的数据是主键。即<index,id>的形式**。

**二级索引属于非聚集索引**。

通过二级索引，可以定位主键的位置，有需要的话再根据主键找当前行的数据。

唯一索引、普通索引、前缀索引等都属于二级索引：

* **唯一索引（Unique key）**：唯一索引也是一种约束（唯一键）。**唯一索引的属性列不能有重复的数据，允许数据为null**，一张表允许创建多个唯一索引。一般来说，唯一索引的目的是为了保证数据的唯一性，而不是为了查询效率。
* **普通索引（index）**：**普通索引的唯一作用就是为了快速查询数据，一张表允许创建多个普通索引，并允许数据重复和 NULL。**
* **前缀索引（Prefix）**：前缀索引只适用于字符串类型的数据。前缀索引是对文本的前几个字符创建索引，相比普通索引建立的数据更小， 因为只取前几个字符。
* **全文索引（Full Text)**：全文索引是一种特殊类型的索引，主要是为了检索大文本数据中的关键字的信息，是目前搜索引擎数据库使用的一种技术。Mysql5.6 之前只有 MYISAM 引擎支持全文索引，5.6 之后 InnoDB 也支持了全文索引。

其中，如果一个索引只包含单个列，这样的索引为**单值索引**，一张表可以有多个单列索引。

类似的，**复合索引**指的是一个索引包含多个列。



## 1.5 聚集索引和非聚集索引

从存储结构上，索引分为**聚集索引**和**非聚集索引**。

**聚集索引和非聚集索引决定了数据库的物理存储结构，即索引结构和数据是否一起存放，主键只是逻辑上的组织方式**，不同引擎的主键索引存储结构也不同。

### 1.5.1 聚集索引

**聚集索引是指索引结构和数据一起存放的索引**。InnoDB的主键索引属于聚集索引。

MySQL中InnoDB 引擎表的 `.ibd`文件就包含了该表的索引和数据，对于 InnoDB 引擎表来说，该表的索引(B+树)的每个非叶子节点存储索引，叶子节点存储索引和索引对应的**数据**。



#### 聚集索引的优点

聚集索引的查询速度非常的快，因为整个 B+树本身就是一棵多叉平衡排序树，叶子节点也都是有序的，定位到索引的节点，就相当于定位到了数据。



#### 聚集索引的缺点

* **依赖于有序的数据** ：因为 B+树是多路平衡树，如果索引的数据不是有序的，那么就需要在插入时排序。数据是整型的还好，如果是类似于字符串或 UUID 这种又长又难比较的数据，插入或查找的速度肯定比较慢。

* **更新代价大** ： 如果对索引列的数据修改时，那么对应的索引也将会被修改， 聚集索引的叶子节点存放着数据本身，修改代价较大， 所以对于主键索引来说，主键一般都是不可被修改的。



### 1.5.2 非聚集索引

**非聚集索引即索引结构和数据分开存放的索引。**

**二级索引属于非聚集索引。**

MySQL中MyISAM引擎的表的`.MYI`文件包含了表的索引， 该表的索引(B+树)的每个叶子非叶子节点存储索引， 叶子节点存储索引和索引对应**数据的指针**，指向`.MYD`文件的数据。

**非聚集索引的data域除了存储数据地址（指针）以外，还可以存放主键**，比如二级索引属于非聚簇索引，其叶子节点的data域存放的是主键，根据主键回表查询数据。



#### 非聚集索引的优点

由于非聚集索引的叶子节点不存放数据本身（叶子节点的data域是数据地址或主键），因此**非聚集索引的更新代价比聚集索引小**。



#### 非聚集索引的缺点

* 非聚集索引也依赖于有序的数据
* **可能会二次查询(回表查询)** ：这是非聚集索引最大的缺点。当查到索引对应的指针或主键后，可能还需要根据指针或主键再到数据文件或表中查询，也就是回表操作。



非聚集索引不一定会导致回表查询，比如**覆盖索引**情况下就不会回表查询：

如果要查询的字段正好建立了索引（无论是单值索引还是复合索引，只要索引字段包括要查询的字段就可以），就属于覆盖索引，不会回表查询：

```mysql
SELECT name FROM table WHERE name='john';
```

索引字段包括了`name`列，查询的时候，索引的key本身就是name，查到对应的name直接返回就行，无需回表查询数据。

对于MyISAM引擎中的主键索引，虽然其主键索引的叶子节点存放的是指针。但是如果 SQL 查的就是主键，也算是覆盖索引，不会导致回表查询：

```mysql
SELECT id FROM table WHERE id=1;
```



## 1.6 覆盖索引

如果一个索引**包含（或者说覆盖）所有需要查询的字段的值**，称为**“覆盖索引（Covering Index）”**。

> 就是说**select的数据列只从索引中就能够取得，不必读取数据行**，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，换句话说，查询列要被所建的索引覆盖。

比如在 InnoDB存储引擎中，如果不是主键索引，叶子节点存储的是key+主键。最终还是要“回表”，也就是要通过主键再查找一次，这样就会比较慢。覆盖索引就是要查询的列和索引是对应的，不做回表操作！

**覆盖索引即需要查询的字段正好是索引的字段，直接根据该索引就可以查到数据， 无需回表查询。**

例如：

* 如果需要查询name，name字段有索引，那么直接根据索引就可以查找到name的值，无需回表查询。



## 1.7 索引优缺点

### 1.7.1 索引的优点

* 索引可以加快数据的检索速度，提高检索效率，降低数据库的IO成本。
* 通过索引对数据进行排序，降低排序成本。
* 唯一性索引可以保证数据库表中每一行数据的唯一性。

### 1.7.2 索引的缺点

* 索引也是一张表，需要耗费物理空间。
* 创建索引和维护索引耗费时间长，对表中数据更改时，如果数据有索引，也要修改索引，会降低SQL执行效率。

使用索引不一定会提高性能。多数情况下，索引查询比全表扫描要快，如果数据量不大，索引带来的提升也不大。



## 1.8 索引的使用

### 1.8.1 适合使用索引的情况

创建的索引应该尽量符合以下条件，即适合建立索引的情况：

* **不为null的字段**：索引字段的数据应该尽量不为 NULL，因为对于数据为 NULL 的字段，数据库较难优化。如果字段频繁被查询，但又避免不了为 NULL，建议使用 0,1,true,false 这样语义较为清晰的短值或短字符作为替代。
* **频繁查询的字段**。
* **频繁作为查询条件的字段**：即经常出现在where条件里的字段。
* **频繁需要排序的字段**：因为索引是排好序的，这样查询的时候可以利用索引的排序，加快排序查询时间。
* **频繁用于和其他表关联的字段**：频繁用于和其他表连接的字段可能是外键列（不同于外键），对于频繁被连接查询的字段，建立索引可以提高多表连接查询的效率。
* **一般情况下考虑创建组合索引而不是单列索引**。索引是B+树结构，多个单列索引占用空间比一个组合索引的占用空间要大，且维护成本更高。
* **查询中统计或分组的字段适合建索引**。
* **考虑在字符串类型的字段上使用前缀索引代替普通索引**。前缀索引仅限于字符串类型，比普通索引占用更小的空间。



### 1.8.2 不适合使用索引的情况

以下情况不适合使用索引：

* 表记录太少。

* 频繁更新的字段，或者频繁增删改的表，不适合建立索引。因为数据更新也必须更新索引，维护索引的成本也不小，会导致更新表速度下降。

* 包含大量重复内容的字段不适合建立索引。索引的选择性越高，索引的效率越高。

  > 索引的选择性指索引列中不同值/记录总数的比值，比如100条数据，只有两种值，其选择性是0.5，如果有99种不同的值，选择性是0.99，选择性越接近于1，索引效率越高。

* where条件里面用不到的字段不创建索引。



## 1.9 索引创建和删除

#### 查看索引

```mysql
SHOW INDEX FROM 表名;
```



#### 创建索引

为了可读性，索引名一般命名为`idx_表名_列名`的格式，比如student表中的学号age的索引名：`idx_student_age`



**1、添加主键索引（primary key）**

```mysql
ALTER TABLE 表名 ADD PRIMARY KEY (列名);   # 指定列作为主键，会自动创建主键索引。
```



**2、添加唯一索引（UNIQUE）**

```mysql
ALTER TABLE 表名 ADD UNIQUE (列名);
```



**3、添加普通索引（INDEX）**

```mysql
# 写法1
ALTER TABLE 表名 ADD INDEX 索引名 (列名);

# 写法2
CREATE INDEX 索引名 ON 表名 (列名);
```



**4、添加全文索引（FULLTEXT）**

```mysql
ALTER TABLE 表名 ADD FULLTEXT (列名);
```



**5、添加复合索引**

```mysql
# 写法1
ALTER TABLE 表名 ADD INDEX 索引名 (列名1,列名2,...);

# 写法2
CREATE INDEX 索引名 ON 表名 (列名1,列名2,...);
```



#### 删除索引

**1、删除主键索引**

```mysql
ALTER TABLE stuinfo DROP PRIMARY KEY;
```



**2、删除一般索引**

```mysql
# 写法1
ALTER TABLE 表名 DROP INDEX 索引名;

# 写法2
DROP INDEX 索引名 ON 表名;
```



# 二、索引优化分析



## 2.1 性能分析

### 2.1.1 MySQL Query Optimizer

MySQL Query Optimizer是MySQL中专门负责优化SELECT语句的优化器，其主要功能是通过计算分析系统中收集到的统计信息，为客户端请求的Query提供它认为最优的执行计划（优化器认为最优的数据检索方式，不一定是开发人员认为是最优的，这部分最耗费时间)。

当客户端向MySQL请求一条Query，命令解析器模块完成请求分类，区别出是SELECT并转发给查询优化器，MySQL Query Optimizer首先会对整条Query进行优化，处理掉一些常量表达式的预算，直接换算成常量值。并对Query中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件、结构调整等。然后分析Query 中的 Hint 信息(如果有)，看显示Hint信息是否可以完全确定该Query 的执行计划。如果没有Hint 或Hint信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据 Query进行写相应的计算分析,然后再得出最后的执行计划。

### 2.1.2 MySQL常见瓶颈



* CPU：数据装入内存或从磁盘上读取数据的时候，可能会导致CPU饱和。
* IO：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候。
* 服务器硬件的性能瓶颈：可以通过`top`，`free`，`iostat`，`vmstat`指令查看系统的性能状态。



### 2.1.3 Explain

#### 简介

使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理SQL查询语句的。

Explain关键字用于分析查询语句或是表结构的性能瓶颈。官网介绍：[链接](https://dev.mysql.com/doc/refman/8.0/en/execution-plan-information.html)

使用方法为：

```mysql
EXPLAIN 查询语句;

# 举例：
EXPLAIN SELECT * FROM staffs WHERE name='july' AND age=20 AND pos='dev';
```



#### Explain的作用

* 可以得知表的读取顺序。
* 可以得知数据读取操作的操作类型
* **可以得知哪些索引可以使用、实际使用了哪些索引。**
* 可以得知表之间的引用。
* 可以得知每张表有多少行被优化器查询。
* **可以得知是否使用filesort或者temporary（临时表）等操作。**



#### Explain结果分析

执行Explain+SQL语句以后，输出的结果有以下几种类型：

![Explain结果类型](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_7.png)



每一行表示一张表的查询，包括实际存在的表和临时表（子查询重命名的表）。

其中各字段的含义如下。

<font color = 'red'>id</font>

表示select查询的序列号，表示查询中执行select子句或操作表的顺序。

* 如果id相同，则执行顺序从上往下依次执行，
* 如果id不同，id越大执行优先级越高，一般是最里层的子查询id最大，最先执行。
* 如果id既有相同的也有不同的，则可以认为相同id的为一组，按照前两条规则执行

例如：

![既有相同id，也有不同id的情况](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_8.png)



上图的例子中，可以看到，先执行id为2的语句，再依次往下执行id为1的语句。



<font color = 'red'>select_type</font>

表示查询的类型，主要是用于区别普通查询、联合查询、子查询等复杂查询。包括以下集中状态：

* **SIMPLE**：表示简单的select查询，查询中不包含子查询或UNION。
* **PRIMARY**：查询中若包含任何复杂的子部分，最外层的查询就会被标记为PRIMATY。
* **SUBQUERY**：子查询，出现在select或where列表中的子查询会被标记为SUBQUERY。
* **DERIVED**：在FROM列表中包含的子查询被标记为DERIVED（衍生），MySQL会递归执行这些子查询，把结果放在临时表里。
* **UNION**：出现在UNION之后的select语句就会被标记为UNION。
* **UNION RESULT**：从UNION表中获取结果的SELECT。

如图中的例子

![select_type举例](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_9.png)



<font color='red'>table</font>

表示这一行是关于哪张表的。

<font color='red'>type</font>

展示了查询使用了何种类型。

type的结果值从好到坏依次是：**system** > **const **> **eq_ref **> **ref **> fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > **range **> **index** > **ALL**。

其中常见的几种类型，从好到坏依次是：

* **system**：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，可以忽略不计。

* **const**：表示通过索引一次就找到了，const用于比较PRIMARY KEY和UNIQUE索引，因为只匹配一行数据，所以很快。比如where条件中是主键，MySQL能将该查询转换为一个常量。

  > 举例：`explain select * from t1 where id = 1;`

* **eq_ref**：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描。

  > 举例：`explain select * from t1,t2 where t1.id = t2.id;`，t2中只有一条数据与之对应，因此类型是eq_ref.

* **ref**：非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体。

  > 举例：`explain select * from t1 where col1='ac';`，其中t1中的索引为`idx_t1_clo1_col2`。t1中符合条件的数据有多条，因此类型为ref。

* **range**：只检索给定范围的行，使用一个索引来选择行。一般在where语句中出现了`between`、`<`、`>`、`in`等关键字的查询语句中会出现range类型。这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。

  > 举例：`explain select * from t1 where id between 30 and 60;`

* **index**：Full Index Scan，index和ALL的区别是index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。就是说虽然index和ALL都是读全表，但是index是从索引中读取的，而ALL是从硬盘中读取的。

  > 举例：`explain select id from t1;`，id为主键。

* **ALL**：Full Table Scan，遍历全表找到匹配的行。

  > 举例：`explain select * from t1 where column_without_index = ' ';`，对于没有索引的查询条件，需要扫描全表，效率最差。

**一般来说，得保证查询至少达到range级别，最好能达到ref级别。**



<font color='red'>possible_keys</font>

显示**可能**应用在这张表中的索引，一个或多个。

如果查询涉及的字段上存在索引，则该索引将被列出，但不一定被查询实际使用。



<font color='red'>key</font>

实际使用的索引。如果为NULL，表明没有用到索引。

如果查询中使用了覆盖索引，则该索引仅出现在key列表中，不会在possible_keys中出现。

比如下面的例子，对col1和col2建立了复合索引，查询内容为col1和col2，是覆盖索引。因此，possible_keys的值为NULL，索引`idx_col1_col2`只出现在key列表中。

![覆盖索引的例子](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_10.png)





<font color='red'>key_len</font>

表示索引中使用的字节数，可通过该列计算出查询中使用的索引长度（即使用了几列的索引）。

在不损失精确性的情况下，长度越短越好。key_len显示的值为索引字段的最大可能长度，**并非实际使用长度**，即key_len是根据表定义计算而得，不是通过表内检索出的。

如下图的例子，使用到一个列索引时key_len值为13字节，使用到两个索引列时，key_len的值为26字节。

![key_len例子](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_11.png)





<font color='red'>ref</font>

表示哪些列或常量被用于查找索引列上的值。

如图中的例子，第二行ref的shared.t2.col1表示使用t1中的索引查询时，使用到了t2的col1这一列，const表示使用到了常量，即常量'ac'。

![ref例子](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_12.png)





<font color='red'>rows</font>

根据表统计信息和索引选用情况，大致估算出找到所需记录需要读取的行数。

如下面的例子：

![rows优化案例](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_13.png)



第一条查询语句是没有索引时查询的情况，可以看到表t2需要读取的行数为640；第二条查询语句在t2上建立了索引，这时表t2使用到了索引，所需读取的行数从640优化到了142。



<font color='red'>Extra</font>

包含不适合在其它列中显示但十分重要的额外信息。

Extra包括以下几个值：

* <font color='red'>Using filesort</font>：说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL无法利用索引完成的排序操作称作”文件排序“。出现这种情况会严重拖慢效率。比如对col1和col2两列建立复合索引，但是select中的ORDER BY子句不是按照col1和col2的顺序排序，就会导致索引失效，并出现using filesort。
* <font color='red'>Using temporary</font>：说明使用了临时表保存中间结果，MySQL对查询结果排序时使用临时表。常见于排序ORDER BY和分组GROUP BY。这种情况比Using filesort还要糟糕。
* <font color='green'>Using index</font>：表示相应的select操作中使用了索引覆盖，避免了访问表的数据行，是我们希望看到的。其中，如果同时出现了using where，表示索引被用来根据索引键值查找数据（即根据索引值判断数据是否是符合条件的值）。如果没有同时出现using where，表示索引用来读取数据（仅读取索引值），而非执行查找动作。
* Using where：表明使用了where过滤。
* using join buffer：使用了连接缓存。
* impossible where：表明where子句的值总是false，不会有符合条件的数据。
* select tables optimized away：没有GROUP BY子句的情况下，基于索引优化MIN、MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。
* distinct：优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作。



**Explain例题**

![Explain例题](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_14.png)



对以上信息，分析如下：

![分析内容](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_15.png)





## 2.2 索引优化



### 2.2.1 索引分析

#### 单表案例

建表：

```mysql
CREATE TABLE IF NOT EXISTS article(
id INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
author_id INT(10) UNSIGNED NOT NULL,
category_id INT(10) UNSIGNED NOT NULL,
views INT(10) UNSIGNED NOT NULL,
comments INT(10) UNSIGNED NOT NULL,
title VARBINARY(255) NOT NULL,
content TEXT NOT NULL
);

INSERT INTO article(author_id,category_id,views,comments,title,content)
VALUES(1,1,1,1,'1','1'),
(2,2,2,2,'2','2'),
(1,1,3,3,'3','3');
```



![article表](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_16.png)



逐步优化过程：

**1、初始时，表格没有普通索引，执行下列语句**：

```mysql
EXPLAIN SELECT id,author_id FROM article 
WHERE category_id=1 AND comments>1 
ORDER BY views DESC\G
```

> 末尾的\G表示以键值对的形式显示内容，不需要加分号。

结果为：

```mysql
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: article
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 3
        Extra: Using where; Using filesort
1 row in set (0.00 sec)
```

可以看到type为`ALL`，表示全表扫描，并且出现了`Using filesort`。

接下来尝试对其优化。

**2、第一次优化：尝试对where条件和order by中设计的三个列都建立索引：**

```mysql
# 对三个列建立索引
CREATE INDEX idx_article_ccv ON article(category_id,comments,views);
# 再次执行SQL语句
EXPLAIN SELECT id,author_id FROM article 
WHERE category_id=1 AND comments>1 
ORDER BY views DESC\G
```

初步优化后，Explain语句运行结果为：

```mysql
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: article
         type: range
possible_keys: idx_article_ccv
          key: idx_article_ccv
      key_len: 8
          ref: NULL
         rows: 1
        Extra: Using index condition; Using filesort
1 row in set (0.00 sec)
```

可见，第二次运行用到了我们建立的索引，并且type变为了`range`，表示范围内查找，比`ALL`效率提升了，但是`Using filesort`还在。

为什么建立了索引，还出现了`Using filesort`呢？

* 根据B+树索引的结构分析可知，对category_id、comments、views三列建立复合索引，在B+树中是先根据category_id排序，category_id相同时，根据comments排序，views同理，这样就建立起一个排好序的索引B+树结构。

* 查询条件中，由于第二个条件是comments>1，是个范围，因此会导致其后面的列，即views列的索引失效。因为第二列是个范围，查询时需要遍历所有范围内的索引，因此第三列的索引用不到了，根据views排列时用不到索引就需要使用文件排序。

经过以上分析，可知，假如第二个条件也是常量，索引就不会失效，从而也不会导致`Using filesort`:

```mysql
EXPLAIN SELECT id,author_id FROM article 
WHERE category_id=1 AND comments=1 
ORDER BY views DESC\G
```

上述语句的结果为：

```mysql
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: article
         type: ref
possible_keys: idx_article_ccv
          key: idx_article_ccv
      key_len: 8
          ref: const,const
         rows: 1
        Extra: Using where
1 row in set (0.00 sec)
```

可以看到，type变为了`ref`，根据ref可知用到了两个常量，即前两个索引值都是固定的常量，因此根据索引的第三个列进行排列（正序倒序都可以）时，可以用到索引，因此没有了`Using filesort`。

如果针对第一次的SQL语句优化，可以考虑不对第二列建立索引。

**3、第二次优化，删除第一次建的索引，重新只对category_id和views两列建立索引**：

```mysql
# 删除第一次建立的索引
DROP INDEX idx_article_ccv ON article;
# 只对category_id和views建立索引
CREATE INDEX idx_article_cv ON article(category_id,views);
# 再次执行SQL语句
EXPLAIN SELECT id,author_id FROM article 
WHERE category_id=1 AND comments>1 
ORDER BY views DESC\G
```



这次的结果：

```mysql
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: article
         type: ref
possible_keys: idx_article_cv
          key: idx_article_cv
      key_len: 4
          ref: const
         rows: 2
        Extra: Using where
1 row in set (0.00 sec)
```

可以看到，这次用到索引的同时，没有了`Using filesort`，因为两列的索引都用到了。

**总结**：经过第一个案例可知，**如果索引遇到范围，则其后面列的索引全部失效。**



#### 两表案例

class表：

```mysql
mysql> select * from class;
+----+------+
| id | card |
+----+------+
|  1 |   18 |
|  2 |   18 |
|  3 |   16 |
|  4 |    7 |
|  5 |    5 |
|  6 |    3 |
|  7 |   19 |
|  8 |    5 |
|  9 |    7 |
| 10 |   20 |
| 11 |   20 |
| 12 |   19 |
| 13 |   16 |
| 14 |    3 |
| 15 |    6 |
| 16 |   19 |
| 17 |   16 |
| 18 |    3 |
| 19 |    5 |
| 20 |   17 |
+----+------+
20 rows in set (0.00 sec)
```

book表：

```mysql
mysql> select * from book;
+--------+------+
| bookid | card |
+--------+------+
|     11 |    1 |
|     14 |    3 |
|      8 |    4 |
|     15 |    4 |
|      6 |    5 |
|     18 |    7 |
|     19 |    7 |
|      7 |    8 |
|     17 |    9 |
|      1 |   10 |
|      4 |   10 |
|     16 |   13 |
|     20 |   13 |
|     10 |   15 |
|      3 |   16 |
|      2 |   17 |
|     13 |   17 |
|      9 |   18 |
|      5 |   19 |
|     12 |   20 |
+--------+------+
20 rows in set (0.00 sec)
```

**1、第一次查询，没有任何索引**：

```mysql
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card=book.card;
```

执行结果为：

```mysql
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: class
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 20
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: book
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 20
        Extra: Using where; Using join buffer (Block Nested Loop)
2 rows in set (0.00 sec)
```

可以看到，在两个表上的查询，type都是`ALL`，rows都是20。

**2、如果仅在左连接的右边的表中对应列建立索引：**

```mysql
# 在book表的card列上建立索引 Y，此时表中仅有一个普通索引Y
ALTER TABLE book ADD INDEX Y (card);
# 此时左连接的右表，type变为ref，且rows是1
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card=book.card;
```

执行结果为：

```mysql
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: class
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 20
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: book
         type: ref
possible_keys: Y
          key: Y
      key_len: 4
          ref: db01.class.card
         rows: 1
        Extra: Using index
2 rows in set (0.01 sec)
```

可以看到，在class表中的rows仍然是20，但是book表中的rows从20变为了1，且type也从ALL变为了ref，因为用到了建立的索引Y，说明索引Y起到了优化作用。因为有了索引，每次内循环从book表中查询时，只需要根据索引遍历一行数据即可，所以rows是1。

**3、如果仅在左连接的左边的表中对应列建立索引**

```mysql
#将上一步的索引删掉，然后对表class的card建立索引X
DROP INDEX Y ON book;
ALTER TABLE class ADD INDEX X (card);
# 此时book表中无索引，class表的card列有一个索引x
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card=book.card;
```

执行结果为:

```mysql
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: class
         type: index
possible_keys: NULL
          key: X
      key_len: 4
          ref: NULL
         rows: 20
        Extra: Using index
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: book
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 20
        Extra: Using where; Using join buffer (Block Nested Loop)
2 rows in set (0.00 sec)
```

可以看到，class表中虽然用到了索引，type是index，但是rows仍然是20，book表中没有索引因此也是20。

通过以上实验可知，对于左连接，如果对右表的对应列建立索引，优化效果更好。

同理，对于右连接，对左表的对应列建立索引，优化效果更好。

因此，**左连接的右表和右连接的左表是优化的重点。**



#### 三表案例

这次我们使用三表连接：

```mysql
# 对三个表的card列都建立索引
ALTER TABLE class ADD INDEX X(card);
ALTER TABLE book ADD INDEX Y(card);
ALTER TABLE phone ADD INDEX Z(card);
# 执行三表连接查询语句
EXPLAIN SELECT * FROM class 
LEFT JOIN book ON class.`card`=book.`card` 
LEFT JOIN phone ON book.`card`=phone.`card`;
```

执行结果：

![三表连接查询](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_17.png)



与两表案例类似，无论是否用到索引，最外层的表，即这里的class表，rows一直是20，而book和phone可以通过索引将row降到1。



#### 结论

1、通过单表案例可以总结出：**如果索引遇到范围，则其后面列的索引全部失效。**

2、根据两表和三表案例，可以总结出优化建议：

* 对于多表连接查询，优化的重点是内层的表，即优化被驱动表带来的效率提升较大。应**尽可能减少Join语句中嵌套循环的总次数**。（通过被驱动表建立索引，可以降低循环次数）。
* **永远要用小结果集驱动大结果集，即小表驱动大表**，就是说，要将小表作为驱动表，大表作为被驱动表。
* **在被驱动表上的Join的字段建立索引**，可以降低总循环次数，从而提升效率。
* 无法保证被驱动表的Join条件字段被索引且内存资源充足的前提下，不要太吝啬JoinBuffer的设置。

左连接中，左表是**驱动表**，右表是**被驱动表**。

右连接中，右表是**驱动表**，左表是**被驱动表**。

内连接时，MySQL自动选择数据量小的表作为驱动表，即小表驱动大表。



<font color='red'>为什么要用小表驱动大表？或者说为什么小表驱动大表效率较高？</font>

解答这个问题前，需要了解嵌套循环的效率对比。

嵌套循环的**连接次数**是取决于外层循环的，循环体的循环次数不变，如果连接次数越少，则效率越高。

比如：

```java
//循环1
for(int i=0;i<10;i++){
    for(int j=0;j<1000;j++){
        //循环体
    }
}
//循环2
for(int i=0;i<1000;i++){
    for(int j=0;j<10;j++){
        //循环体
    }
}
```

两个循环的循环体执行次数相同的，但是循环1的执行速度要比循环2快，因为循环1中，i和j初始化的初始化次数分别为1、10次，而循环2中i和j的初始化次数分别是1,、1000次，且循环1的连接次数是10，循环2的连接次数是1000，因此循环1速度快于循环2。[参考](https://blog.csdn.net/taohuaxinmu123/article/details/31419701)

**MySQL中的连接查询，正是类似于嵌套循环**。

以左查询为例，比如SQL语句：

```mysql
SELECT * FROM A LEFT JOIN B ON A.c=B.c;
```

以上连接查询语句可以理解为一个双重for循环，即：

```
for(循环次数为A的行数){
	select * from A where 第i行;
	for(循环次数不确定，直到找到B中符合条件的行位置){
		select * from B where B.c=A.c
	}
} 
```

可以看到，**驱动表相当于多重for循环的外层循环，被驱动表相当于内层循环。因此驱动表（外层循环）小于被驱动表（内层循环）时，效率较高**。



<font color='red'>为什么要在被驱动表建立索引？</font>

驱动表无论是否有索引，都要遍历所有行，而被驱动表则不是这样。从上述for循环的例子可知，对于驱动表A中的当前行，从**被驱动表B中查询的次数和是否用到了索引有关**。如果没有索引，则会遍历被驱动表B的所有行，找到所有符合当前A.c值的行；如果表B的c行建立了索引，则会根据索引，直接找到所有符合当前A.c值的行，不必遍历所有行。

上述案例中，建立索引之后的rows从20（没有索引，需要遍历所有行）降为了1（有了索引，只需根据索引进行查找）。



### 2.2.2 避免索引失效

从上一节的例子中可以看到，有时候虽然建立了索引，但是遇到了范围，导致了索引失效。有哪些情况会导致索引失效呢？

下面以staff表说明各种索引失效的情况：

![staff表格内容](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_18.png)



对staff表的`name`,`age`,`pos`三列建立复合索引`idx_staffs_nameAgePos`：

```mysql
ALTER TABLE staffs ADD INDEX idx_staffs_nameAgePos(name,age,pos);
```



#### 索引失效的情况

以下几种情况，会导致索引失效。

**1、违反最佳左前缀法则（或全值匹配，最左匹配），导致索引失效（或部分失效）**

如果where条件缺少了（即不在where条件中）索引中的某个列，则这个列后面的列都无法使用索引。如果是索引的第一列不在查询条件中，则会导致索引完全失效，如果是中间的某一列不在查询条件中，导致索引部分失效。

MySQL优化器会自动调整where条件的顺序，多个条件的书写顺序对于结果无影响。但还是推荐按照顺序写。

```mysql
# 以下四种情况，都正常使用到了索引。
EXPLAIN SELECT * FROM staffs WHERE name ='july';
EXPLAIN SELECT * FROM staffs WHERE name ='july' AND age=25;
EXPLAIN SELECT * FROM staffs WHERE name ='july' AND age=25 AND pos='dev';
# 正常使用索引。匹配顺序和书写顺序无关，只要调整顺序以后覆盖即可。
EXPLAIN SELECT * FROM staffs WHERE age=25 AND name ='july' AND pos='dev';

# 由于缺少了索引的第一列，因此索引完全失效
EXPLAIN SELECT * FROM staffs WHERE age=25 AND pos='dev';

/* 由于缺少了中间的age列，age列后面的pos列的索引失效,但name列的索引正常使用。
这种情况属于索引部分失效。
*/
EXPLAIN SELECT * FROM staffs WHERE name ='july' AND pos='dev';
```



**2、在索引上做操作（计算、函数、自动或手动类型转换），导致操作列和之后列索引失效**

如果在索引上做任何操作，都会导致操作列及其后面列的索引失效。

```mysql
# 以下操作，在name上取子串，导致索引全部失效，因为name是索引的第一列
EXPLAIN SELECT * FROM staffs WHERE SUBSTRING(name,1,4) ='july';

# age列的索引失效，只使用了name列的索引。
EXPLAIN SELECT * FROM staffs WHERE name='july' AND age+5=28;

# 在中间列age上操作，导致age和之后的pos列索引都失效，只使用了name列索引
EXPLAIN SELECT * FROM staffs 
WHERE name ='july' AND age+5=28 AND pos='dev';
```



**3、范围条件右边的列索引会失效**

如果某一索引列的查询条件是一个范围，会导致其后面的所有列的索引都失效。

```mysql
# 索引的中间列age，因为是范围查找，导致后面的pos列的索引失效。
# age列本身的索引不会失效，索引用于排序。
EXPLAIN SELECT * FROM staffs 
WHERE name='july' AND age>20 AND pos='dev';  # 用2个索引
```



**4、使用不等于号(`!=`或`<>`)时，导致索引全部失效。**

如果索引列的任一列使用了`!=`或`<>`，就会导致索引全部失效，造成全表扫描。

```mysql
# 索引全部失效
EXPLAIN SELECT * FROM staffs WHERE name <>'july';
# 索引全部失效
EXPLAIN SELECT * FROM staffs WHERE name ='july' AND age!=21;
# 索引全部失效
EXPLAIN SELECT * FROM staffs WHERE name ='z3' AND age=22 AND pos<>'dev';
```



**5、使用`is null`、`is not null`，导致当前列和之后列的索引失效**

如果索引列的某一列使用了`is null`或`is not null`，会导致当前列和其后面所有列的索引都失效。

```mysql
# 索引完全失效。
EXPLAIN SELECT * FROM staffs WHERE name IS NULL;

# 索引部分失效，只使用了name列的索引
EXPLAIN SELECT * FROM staffs WHERE name = 'july' AND age IS NOT NULL;

# 索引部分失效，只使用了name列的索引
EXPLAIN SELECT * FROM staffs 
WHERE name = 'july' AND age IS NOT NULL AND pos='dev';
```



**6、LIKE条件中，%放在开头，导致当前列和之后列的索引失效**

在使用了`LIKE`的条件语句中，如果将`%`号放在了开头位置，会导致当前列和之后所有列的索引失效。

```mysql
# 索引全部失效
EXPLAIN SELECT * FROM staffs WHERE name LIKE '%july'; 

# 索引全部失效
EXPLAIN SELECT * FROM staffs 
WHERE name  LIKE '%july' AND age = 25; # 索引失效

# 索引部分失效，只使用了name和age列的索引，pos列的索引失效。
EXPLAIN SELECT * FROM staffs 
WHERE name = 'july' AND age = 23 AND pos LIKE '%dev'; 
```

如果SQL语句中必须要将`%`写在开头，则可以使用覆盖索引避免索引失效：

```mysql
# 索引未失效
EXPLAIN SELECT name FROM staffs WHERE name LIKE '%july';

# 索引未失效
EXPLAIN SELECT name,pos,age FROM staffs WHERE name LIKE '%july%';

# 索引未失效
EXPLAIN SELECT name,pos,age FROM staffs 
WHERE name = 'july' AND age = 23 AND pos LIKE '%dev';

# 索引未失效,id是主键，主键索引为primary，自动创建。也是覆盖索引。
EXPLAIN SELECT id,name,pos,age FROM staffs 
WHERE name LIKE '%july%'; 

# 索引失效，不是索引覆盖，且%开头。
EXPLAIN SELECT name,add_time FROM staffs 
WHERE name LIKE '%july';
```



**7、字符类型的字段，查询时常量值不加单引号而触发类型转换，导致当前列和之后列的索引失效**

如果某一字段类型是字符类型，比如varchar类型，如果查询语句写的是整型数字（如果是字符串不写引号是语法错误），会触发自动类型转换（同第2条），导致当前列和之后所有列的索引失效。

```mysql
# 索引失效。name列是varchar型，2000会自动转换为varchar型，导致索引失效。
EXPLAIN SELECT * FROM staffs WHERE name=2000;

# 索引全部失效
EXPLAIN SELECT * FROM staffs WHERE name=2000 AND age=23; # 失效
```



**8、使用or时，如果其中一个列没有索引，导致索引全部失效**

使用or时，只要其中一个列没有索引，就会导致索引全部失效，其他字段有索引也不会使用。

因为如果其中的列没有索引，必然会导致一次全表扫描，如果带索引的列使用索引，那么总操作就是全表扫描+索引查找+合并，共三部分。于是干脆就不使用索引，直接全表扫描。

```mysql
# 使用全部索引
EXPLAIN SELECT name FROM staffs WHERE name = 'july' OR pos ='dev';

# 索引全部失效。因为add_time没有索引。
EXPLAIN SELECT name FROM staffs 
WHERE name = 'july' OR add_time = NOW();

# 正常使用索引，该使用几个使用几个。
# 因为有带索引的列，and条件，一定不会导致全部扫描，先根据索引查找，再判断即可。
EXPLAIN SELECT name FROM staffs 
WHERE name = 'july' AND add_time = NOW();
```



#### 索引优化总结

根据以上索引分析和索引失效的情况，可以总结出避免索引失效的优化建议：

* 遵守全值匹配（最左匹配）和最左前缀的规则。

* 尽量避免缺失索引首列或中间列，以防发生索引全部失效或部分失效的情况

* 尽量避免在索引列上计算，以防索引失效。

* 尽量使用覆盖索引，少用`*`查询，以避免因二次查询非索引列数据导致效率下降。

* LIKE条件中，尽量避免将`%`号写在开头。如果`%`必须写在开头，可以使用覆盖索引的方式避免索引失效。

  > 一般来说，%越往后，查询效率越高。

* 尽量避免使用`is null`、`is not null`、`!=`、`<>`和`or`，以免导致索引失效。

* 字符类型的单引号一定要写。



口诀：

```mysql
全值匹配我最爱，最左前缀要遵守;
带头大哥不能死，中间兄弟不能断;
索引列上少计算，范围之后全失效;
LIKE百分写最右，覆盖索引不写星;
不等空值还有or，索引失效要少用；
VAR引号不可丢，SQL高级也不难！
```



### 2.2.3 一般性建议

对于单键索引，尽量选择针对当前query过滤性更好的索引。

在选择组合索引的时候，当前Query中过滤性最好的字段在索引字段顺序中，位置越靠前越好。

在选择组合索引的时候，尽量选择可以能够包含当前query中的where字句中更多字段的索引。

尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的。



# 三、查询截取分析

SQL优化分析的大致流程：

1、慢查询日志的开启并捕获。

2、explain+慢SQL分析。

3、使用Show Profile查询SQL在MySQL服务器里面的执行细节和生命周期情况。

4、SQL数据库服务器的参数调优（一般是DBA来操作）。



## 3.1 查询优化

### 3.1.1 永远小表驱动大表

小表驱动大表是一个重要的优化原则，即小的数据集驱动大的数据集。

上文已经讨论过小表驱动大表在`join`时的效率情况，这里讨论的是关于`in`和`exist`两种子查询中，使用小表驱动大表的优化情况。

`in`和`exists`使用及对比：[子查询](https://kangshitao.github.io/2021/04/26/mysql-basis/#7%E3%80%81%E5%AD%90%E6%9F%A5%E8%AF%A2)

使用`in`关键字的子查询案例：

```mysql
select * from A where id in (select id from B);
/*等价于：
for (select id from B){
	for (select * from A where A.id = B.id)
}
*/
```



使用`exists`关键字的子查询案例：

```mysql
select * from A where exists (select 1 from B where B.id=A.id);
/*等价于
for(select * from A){
	for(select * from B where B.id=A.id)
}
*/
```

通过以上案例可知，当子查询的数据集（表B）小于主查询（表A）的数据集时，根据小表驱动大表原则，使用`in`关键字要优于`exists`，反之，主查询数据集小于子查询数据集时，用`exists`优于`in`。

这是由`exists`和`in`子查询的执行顺序决定的：

* 使用`in`的SQL语句，会先执行子查询语句，然后执行主查询，再判断。
* 使用`exists`的SQL语句，会先执行主查询，然后将主查询得到的数据，放到子查询中做条件验证，根据验证结果（`TRUE`或`FALSE`)来决定主查询的数据结果是否得以保留。

> 1、`exists`子句只返回`TRUE`或`FALSE`。子查询中可以是`SELECT *`，也可以是`SELECT 1`或其他，实际执行时会忽略 SELECT清单，因此SELECT后写什么都没有区别。
>
> 2、`exists`子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比，如果担忧效率问题，可进行实际检验以确定是否有效率问题。
>
> 3、`exists`子查询往往也可以用条件表达式、其他子查询或者`JOIN`来替代，何种最优需要具体问题具体分析。



### 3.1.2 ORDER BY关键字优化

<font color='red'>优化建议一：ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序。</font>

如果使用order by进行排序，则尽量保证在Explain结果的Extra列，出现`Using index`而不出现`Using filesort`。

案例：

```mysql
# 建表
CREATE TABLE tblA(
id INT PRIMARY KEY NOT NULL AUTO_INCREMENT,
 age INT,
 birth TIMESTAMP NOT NULL
);
INSERT INTO tblA(age,birth) VALUES(22,NOW());
INSERT INTO tblA(age,birth) VALUES(23,NOW());
INSERT INTO tblA(age,birth) VALUES(24,NOW());

# 在age和birth两列上建立复合索引。
CREATE INDEX idx_A_ageBirth ON tblA(age,birth);

/*表的内容如下:
+----+------+---------------------+
| id | age  | birth               |
+----+------+---------------------+
|  1 |   22 | 2021-06-02 22:22:43 |
|  2 |   23 | 2021-06-02 22:22:43 |
|  3 |   24 | 2021-06-02 22:22:43 |
+----+------+---------------------+
*/
```

在表`tblA`中，分析以下几个案例，只关注`Extra`列：

```mysql
# 1.Extra: Using where; Using index
EXPLAIN SELECT * FROM tblA WHERE age>20 ORDER BY age;

# 2.Extra: Using where; Using index
EXPLAIN SELECT * FROM tblA WHERE age>20 ORDER BY age,birth;

# 3.Extra: Using where; Using index; Using filesort
# 由于排序条件缺少复合索引的第一列，并且where中的age是个范围，
# 导致birth列的索引失效，MySQL不得不采用filesort方式排序
EXPLAIN SELECT * FROM tblA WHERE age>20 ORDER BY birth;

# 4.Extra: Using where; Using index
# 虽然排序条件缺少了age列，但是where中age是确定的，
# 这样就保证了根据birth排序时，将第一列当成常数，birth列可以正常使用索引
EXPLAIN SELECT * FROM tblA WHERE age=20 ORDER BY birth;

# 5.Extra: Using where; Using index; Using filesort
# 虽然根据索引的两个列排序，但是不是按照索引建立的顺序，
# 因此也会导致排序时索引失效，出现filesort
EXPLAIN SELECT * FROM tblA WHERE age>20 ORDER BY birth,age;

# 6.Extra: Using index; Using filesort
# 出现了firesort，因为不符合最左覆盖原则，age列是缺失的，导致索引失效
EXPLAIN SELECT * FROM tblA ORDER BY birth;

# 7.Extra: Using where; Using index; Using filesort
# 同样索引失效，因为没有age列
EXPLAIN SELECT * FROM tblA 
WHERE birth > '2021-05-29 16:24:38' ORDER BY birth;

# 8.Extra: Using where; Using index
# 索引可以使用，因此使用了索引排序
EXPLAIN SELECT * FROM tblA 
WHERE birth > '2021-05-29 16:24:38' ORDER BY age;

# 9.Extra: Using index; Using filesort
# 因为索引建立时，是根据从小到大排序的，复合索引也是如此，类似于基数排序。
# 复合索引的两个列排列规则不一致，会导致排序时索引失效，从而触发filesort
EXPLAIN SELECT * FROM tblA ORDER BY age ASC,birth DESC;

# 10.Extra: Using index
# 这两条都没有触发filesort，因为B+树叶子节点有指向左右叶子节点的指针
# 所以无论是从小到大还是从大到小，都可以借助索引排序，前提是多个列的排序规则要一致。
EXPLAIN SELECT * FROM tblA ORDER BY age DESC;
EXPLAIN SELECT * FROM tblA ORDER BY age DESC,birth DESC;
```



MySQL支持两种排序方式：Index和FileSort，对应于Using index和Using filesort。

Using index扫描索引本身就能完成排序，因此使用索引的效率比使用文件排序的效率要高。

当ORDER BY子句满足以下两种情况时，会使用Index方式排序：

* ORDER BY语句使用索引最左前列。
* 使用Where子句与ORDER BY子句条件列组合，满足了索引最左前列的要求，比如上述第4个例子。



<font color='red'>优化建议二：尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀条件</font>



ORDER BY之后的列如果没有建立索引，就会导致`Using filesort`。

ORDER BY之后的列虽然建立索引，但是没有符合最左前缀原则，同样会导致索引失效，触发`Using filesort`。

如果两个排序列排序规则不一致(比如一个降序一个升序)，也会导致索引失效，出现`Using filesort`



<font color='red'>优化建议三：MySQL的filesort算法的介绍和优化</font>

如果要排序的字段不在索引列上，那么MySQL使用filesort算法排序，filesort算法有两种，包括双路排序和单路排序：

* 双路排序：MySQL 4.1之前使用的是双路排序，即两次扫描磁盘，最终得到数据。具体来说，读取行指针和orderby 列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出。简言之，就是从磁盘取排序字段，在buffer中排序，然后根据排序好的字段再从磁盘中读取其他字段的数据。
* 单路排序：MySQL 4.1之后对双路排序改进。从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO,但是它会使用更多的空间。因为它把每一行都保存在内存中了。



> 虽然单路排序总体而言好于多路排序，但是其也有缺点：
>
> 单路排序比多路排序多占用很多排序缓存(sort_buffer)空间。因为单路排序排序时，把所有字段都取出，所以有可能取出数据的总大小超出了排序缓存的容量，导致每次只能取sort_buffer容量大小的数据进行排序（创建tmp文件，多路合并)，排完再取sort_buffer容量大小，重复这两步，从而导致多次IO，得不偿失。



**对filesort单路排序的优化策略：**

* 使用Order by时，只查询需要的字段，避免使用`select *'`。

  > 这点非常重要。因为：
  > 1、当Query的字段大小总和小于max_length_for_sort_data而且排序字段不是TEXTIBLOG类型时，会使用单路排序，否则会用多路排序算法。
  > 2、两种算法算法的数据都有可能超出sort_buffer的容量（超出之后，会创建tmp文件进行合并排序，导致多次IO），但是用单路排序算法的风险会更大一些，所以要提高sort_buffer_size。
  
* 尝试提高`sort_buffer_size`。

  > 无论那种算法，提高这个参数都会提高效率，要根据系统能力去提高，这个参数是针对每个进程的。

* 尝试提高`max_length_for_sort_data`。

  > 提高这个参数，会提高用单路排序的概率。但如果设置的太高，数据容量超出sort_buffer_size的概率就会增大，明显症状是提高磁盘I/O活动和处理器使用率变低。




### 3.1.3 GROUP BY关键字优化

GROUP BY的实质是先排序后分组，同样遵照索引建的最佳左前缀。

因此GRUOP BY的优化策略和ORDER BY一致，唯一不同的是，`where`高于`having`，**能写在where中的就不要写在having中。**



## 3.2 慢查询日志

### 3.2.1 慢查询日志介绍

MySQL的慢查询日志是MySQL提供的一种日志记录，用来记录在MySQL中响应时间超过阈值的语句，具体指运行时间大于`long_query_time`（默认是10秒）值的SQL语句，会被记录到慢查询日志中。

比如`long_query_time`的值为5时，执行时间大于5秒的SQL就会被记录到慢查询日志中。



### 3.2.2 慢查询日志的使用

默认情况下，MySQL数据库没有开启慢查询日志，需要手动开启。

如果不是调优需要，不建议开启慢查询日志，因为开启慢查询日志或带来一定的性能影响。

**查看慢查询日志是否开启：**

```mysql
SHOW VARIABLES LIKE '%slow_query_log%';
```



**开启慢查询日志：**

```mysql
SET GLOBAL slow_query_log=1;  # 1或TRUE都表示TRUE
# 只对当前数据库生效；MySQL重启后失效。
```



如果想要使开启状态永久生效，需要修改配置文件`my.cnf`(其他系统变量也是如此)：

修改`my.cnf`文件，在`mysqld`下增加或修改参数：

```shell
# 设置开启状态
slow_query_log=1;
# 设置慢查询日志文件路径。
# 如果没有指定，系统会默认给一个名为host_name-slow.log文件
slow_query_log_file=/var/lib/mysql/localhost-slow.log  
```

设置完以后，需要重启MySQL服务器。



**查看记录到慢查询日志中的阈值时间：**

```mysql
# 默认是10秒
SHOW VARIABLES LIKE '%long_query_time%';
```

同样地，可以使用`SET`命令修改，或者在`my.cnf`配置文件里修改。

注意：大于`long_query_time`才会记录到慢查询日志中，等于这个时间的SQL语句不会被记录。



**查询当前系统中有多少条慢查询记录：**

```mysql
SHOW GLOBAL STATUS LIKE '%Slow_queries%';
```



慢查询日志文件的内容举例：

![慢查询日志文件内容](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_19.png)



### 3.2.3 日志分析工具mysqldumpslow

实际生产环境中，可以使用日志分析工具mysqldumpslow，用来分析日志，查找、分析SQL。

在Linux服务器中使用`mysqldumpslow`命令可以使用mysqldumpslow工具的各种功能。



**查看帮助信息：**

````bash
mysqldumpslow --help
````



help信息的部分内容，详细参考[mysqldumpslow命令](https://www.linuxcool.com/mysqldumpslow)：

* -s：表示按照何种方式排序，其后可跟以下几个参数，表示不同的排序方式
  * aI：平均锁定时间
  * ar：平均返回记录数
  * at：平均查询时间
  * c：次数
  * l：锁定时间
  * r：查询行数
  * t：查询时间
* -r：反转排序顺序
* -t NUM：展示前NUM条查询
* -l：从总时间中不减去锁定时间
* -g：后面跟正则表达式，大小写不敏感



**实例参考**

得到返回记录集最多的10个SQL：

```bash
mysqldumpslow -s r -t 10 /usr/local/mysql/data/localhost-slow.log
```



得到访问次数最多的10个SQL：

```bash
mysqldumpslow -s c -t 10 /usr/local/mysql/data/localhost-slow.log
```



得到按照时间排序的前10条里面含有左连接的查询语句：

```bash
mysqldumpslow -s t -t 10 /usr/local/mysql/data/localhost-slow.log
```



## 3.3 批量数据脚本

案例：往表里插入1000万条数据。

**1、建表**

创建dept表：

```mysql
CREATE TABLE dept(
	id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
	deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
	dname VARCHAR(20) NOT NULL DEFAULT "",
	loc VARCHAR(13) NOT NULL DEFAULT ""
) ENGINE = INNODB DEFAULT CHARSET=GBK;
```

创建emp表：

```mysql
CREATE TABLE emp(
	id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
	empno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
	ename VARCHAR(20) NOT NULL DEFAULT "",
	job VARCHAR(9) NOT NULL DEFAULT "",
	mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
	hiredate DATE NOT NULL,
	sal DECIMAL(7,2) NOT NULL,
	comm DECIMAL(7,2) NOT NULL,
	deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0
) ENGINE = INNODB DEFAULT CHARSET=GBK;
```



**2、设置参数**

由于开启过慢查询日志，开启了bin-log，必须为我们的function指定一个参数，即使`log_bin_trust_function_creators`出于开启状态。

```mysql
SET GLOBAL log_bin_trust_function_creators =1;
```

同样地，如果想永久有效，必须将其写入`my.cnf`配置文件。



**3、创建函数，用于随机生成内容，保证每条数据都不同**

```mysql
# 功能为返回一个长度为n的随机字符串
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
	DECLARE chars_str VARCHAR(100) DEFAULT 	'abcdefghijk1mnopgrstuvwxyzABCDEFJHIJKIMNOPQRSTUVWXYZ';
 	DECLARE return_str VARCHAR(255) DEFAULT '';
 	DECLARE i INT DEFAULT 0;
 	WHILE i < n DO
 	# 随机选取一个字符
 	SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1)); 
 	SET i = i +1;
 END WHILE;
 RETURN return_str; #最终长度为n
END $$
DELIMITER ;

#用于随机产生部门编号
DELIMITER $$
CREATE FUNCTION rand_num() RETURNS INT(5)
BEGIN
	DECLARE i INT DEFAULT 0;
	SET i = FLOOR(100+RAND()*10);
	RETURN i;
END $$
DELIMITER ;
```



**4、创建存储过程，插入数据**

```mysql
# 创建向emp表中插入数据的存储过程
DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0; #关闭自动提交autocommit
 REPEAT
	SET i=i+ 1;
	INSERT INTO emp(empno, ename ,job ,mgr ,hiredate ,sal ,comm ,deptno)
	VALUES((START+i),rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num());
 UNTIL i = max_num
END REPEAT;
COMMIT;
END $$
DELIMITER ;

# 创建向dept表中插入数据的存储过程
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0; #关闭自动提交autocommit
 REPEAT
	SET i=i+ 1;
	INSERT INTO dept(deptno, dname,loc)
	VALUES((START+i),rand_string(10),rand_string(8));
 UNTIL i = max_num
END REPEAT;
COMMIT;
END $$
DELIMITER ;
```



**5、调用存储过程**

```mysql
# 向dept表批量插入10条部门数据
CALL insert_dept(100,10);  

# 向emp表批量插入50万条数据
CALL insert_emp(100001,500000);
```





## 3.4 Show Profile

### 3.4.1 Show Profile介绍

Show Profile是MySQL提供的可以用来分析当前会话中语句执行的资源消耗情况的工具，可以用于SQL的调优的测量。

官网介绍：https://dev.mysql.com/doc/refman/8.0/en/show-profile.html



### 3.4.2 Show Profile使用

使用步骤：

1、查看是否支持

```mysql
show variables like 'profiling';
```



2、开启Show Profile

```mysql
set profiling = 1;
```



3、运行SQL

4、查看结果

```mysql
show profiles;
```



结果中可以查看profile中保存的查询记录的运行时间和SQL语句。

![profile中保存的结果](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_20.png)





5、诊断SQL：`show profile [option,...] for query SQL语句id号;`

其中可选的参数有：

* ALL：显示所有的开销信息。
* BLOCK IO：显示块IO相关开销。
* CONTEXT SWITCHES：上下文切换相关开销。
* CPU：显示CPU相关开销信息。
* IPC：显示发送和接收相关开销信息
* MEMORY：显示内存相关开销信息
* PAGE FAULTS：显示页面错误相关开销信息
* SOURCE：显示和Source_function，Source_file，Source_line相关的开销信息
* SWAPS：显示交换次数相关开销的信息



使用诊断指令，展示具体SQL的执行过程以及每个步骤消耗的资源信息。比如查询id为15的那条SQL的cpu和block io的使用情况：`show profile cpu,block io for query 15;`，结果如下：

![查询第15个语句的资源消耗情况](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_21.png)





>  日常开发需要注意的结论：
>
>  1、converting HEAP to MyISAM：查询结果太大，内存不够用了
>
>  2、Creating tmp table：创建临时表，复制数据到临时表，用完再删除。
>
>  3、Copying to tmp table on disk：把内存中临时表复制到磁盘。**危险信号**。



## 3.5 全局查询日志

 **永远不要在生产环境开启此功能。**

同样的，有两种开启方式，如果想要永久生效，必须写在配置文件中。

### 3.5.1 编码启用

开启全局查询日志：

```mysql
set global general_log=1;
```



设置文件输出格式：

```mysql
# 比如，将输出格式设置为TABLE类型
set global log_output='TABLE';
```

开启并设置输出为TABLE类型以后，所有的SQL语句，都会记录到MySQL库里的general_log表，可以使用以下命令查看：

```mysql
select * from mysql.`general_log`;
```



### 3.5.2 配置启用

在MySQL的配置文件`my.cnf`中，设置如下：

```bash
# 开启
general_log=1
# 记录日志文件的路径
general_log_file=/path/logfile
# 输出格式
log_output=FILE
```

如果将输出格式设置为FILE，则记录会保存到指定的文件目录，默认路径和慢查询日志相同，默认文件名为`localhost.log`，比如MySQL 5.6中，`/usr/local/mysql/data/localhost.log`。




# 四、MySQL锁机制

## 4.1 锁的定义和分类

### 4.1.1 定义

锁是计算机协调多个进程或线程并发访问某一资源的机制。

### 4.1.2 锁分类

1、MySQL中的锁，根据对数据的操作粒度，分为**表锁（表级锁，table-level locking）**和**行锁（行级锁，row-level locking）**：

* **表锁**：对当前操作的整张表加锁
* **行锁**：只针对当前操作的行进行加锁。



***

扩展：

MyISAM引擎仅支持表锁。InnoDB引擎支持表锁和行锁，默认采用行锁。

InnoDB引擎的行锁实现算法有三种：

* **Record lock：记录锁**，单个行记录上的锁。列必须是**唯一索引或者主键列**，否则加的锁会变成临键锁。查询语句必须为`=`，不能是`>`或`<`，否则也会变为临键锁。
* **Gap lock：间隙锁**，锁定一个范围，但不包括记录（表中有的边界）本身。基于非唯一索引。比如查询区间为[1,5]，则间隙锁会锁定(1,5)。防止间隙中被其他事务插入数据。
* **Next-key lock：临键锁**，record+gap锁的组合，锁定一个范围和记录本身。临键锁只在**非唯一索引列**。临键锁是加锁的基本单位。锁定的是**左开右闭**区间，即(a,b]。如果最后一个值不满足条件（即表中没有），则临键锁退化为间隙锁(a,b)。临键锁解决幻读问题。

MySQL只有在可重复读的隔离级别下才有间隙锁和临键锁

***



2、根据对数据操作的类型(读/写)，分为**读锁**和**写锁**：

* **读锁（共享锁）**：针对同一份数据，多个读操作可以同时进行而不会互相影响。
* **写锁（排它锁）**：当前写操作没有完成前，会阻断其他写锁和读锁。



## 4.2 表锁

### 4.2.1 特点

MySQL中锁定**粒度最大**的一种锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，**加锁快，不会出现死锁**。由于锁定粒度大，导致触发锁冲突的概率最高，并发度最低。



### 4.2.2 案例分析

#### 例一：添加表级读锁

假设目前有一个表`mylock`，两个会话（或者是事务）：`session 1`和`sessoin 2`，会话1表示加锁的会话，会话2代表其他会话。

在session 1种对mylock表添加表级锁，读锁：

```mysql
lock table mylock read;
```

> 一个会话同时只能对一个表加锁

加锁后，**对于读操作**：

session 1只可以查询该表记录，但不能查询其他没有被session 1锁定的表。session 2既可以查询表`mylock`，也可以查询其他表。

**对于写操作**：

session 1无法对当前表写入或更新数据，也无法对其他表进行写入或更新操作，会提示错误。session 2如果对当前表插入或更新，会一直处于阻塞状态，直到session 1释放锁，session 2的操作才会执行；对于其他表，session 2正常对他们进行操作。

释放锁的指令：

```mysql
# 释放当前会话持有的锁
unlock tables;
```



总结：当前会话对某个表添加表级读锁后，只能对其添加表级读锁的表进行读操作，不能写，也不能对没被它锁定的表进行任何操作。其他会话只有对这个表写操作时会阻塞，但可以读，同时对其他表的操作不会受影响。





#### 例二：添加表级写锁

session 1对`mylock`表添加写锁：

```mysql
lock tables mylock write;
```

此时session 1只能对当前表（加写锁的表）进行读和写操作，不能对其他表进行任何操作。

session 2对`mylock`表进行读写操作都会进入阻塞状态，直到锁释放才会执行。对其他表的操作不受影响。



释放锁：

```mysql
unlock tables;
```



#### 结论

MyISAM引擎在执行查询语句前，会自动给涉及的所有表加读锁，在执行增删改操作前，会自动给涉及的表加写锁。MySQL的表级锁有两种模式：

* 表共享读锁（Table Read Lock）
* 表独占写锁（Table Write Lock）



根据上述案例，可以总结出两个结论：

* 对MyISAM表的读操作(加读锁），不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求（包括自己也不能写）。只有当读锁释放后，才会执行其它进程的写操作。
* 对MyISAM表的写操作(加写锁)，会阻塞其他进程对同一表的读和写操作，只有当写锁释放后，才会执行其它进程的读写操作。



### 4.2.3 表锁分析

查看哪些表被加锁：

```mysql
show open tables;
```



查看表相关状态：

```mysql
show status like'table%';
```

其结果中，变量`Table_locks_immediate`表示产生表级锁定的次数（即可以立即获取锁的查询次数，每立即获取锁，值加1）。

`Table_lock_waited`表示出现表级锁争用而发生等待的次数（即不能立即获取锁的次数，每等待一次，锁值+1)，此值高则说明存在着较严重的表级锁争用情况。



此外，MyISAM引擎的读写锁调度是写优先，这也是MyISAM不适合做写为主表的引擎的原因，因为添加写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。因此，**要保证MyISAM引擎的表偏读**，避免堵塞。



## 4.3 行锁

### 4.3.1 特点

MySQL中锁定**粒度最小** 的一种锁，只针对当前操作的行进行加锁。**行级锁能大大减少数据库操作的冲突**。由于锁定粒度小，因此并发度高。但加锁的开销也最大，加锁慢，容易出现死锁。

### 4.3.2 案例分析

以下案例的前提当前表使用的是InnoDB引擎，且使用行锁。



#### 1、行锁基本案例

情景一：会话1对某一行进行修改操作，但还未提交，此时会话2如果要对同一行进行操作，会被阻塞，直到会话1提交以后，会话2才会被执行。

情景二：会话1对某一行进行修改操作，同时会话2对另一行进行操作，二者互不影响。



#### 2、无索引行锁升级为表锁

如果会话1的操作导致索引失效，或者没有使用索引，会导致行锁变为表锁，导致会话1操作时，会话2不能进行写操作（比如update和insert），处于阻塞状态。

> 和MyISAM的表锁不同的是，这里的其他会话虽然不能进行写操作，但是可以进行读操作。MyISAM中的表锁，其他会话对于此表既不能读也不能写。



例如，表`test_innodb_lock`中的a和b两列都有单独的索引，且a的类型是`INT`，b的类型是`VARCHAR`，此时会话1对其中的一行进行更新，由于自动类型转换，导致了索引失效，从而行锁变为表锁。

```mysql
# 会话1进行更新操作，未提交：
update test_innodb_lock set a=44 where b=4000;

# 会话2进行写操作,即使不是同一行，也会阻塞，因为行锁升级为了表锁
update test_innodb_lock set a=5 where b='5000';
```





#### 3、间隙锁的危害

关于InnoDB引擎的行锁，有三种实现算法，分别为记录锁、间隙锁、临键锁。根据4.1.2节的介绍，间隙锁锁定的是一个范围，不包括边界值。

间隙锁锁住的是索引记录中的间隔。间隙锁的主要目的，是为了防止其他事务在间隔中插入数据。

如果一个事务使用范围查找，会锁定一个范围，具体范围是什么，需要根据表内容和索引确定。

间隙锁的致命弱点就是，锁定一个范围后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据，在某些场景下，可能会对性能造成很大的危害。



关于间隙锁的详细讲解，可以参考[链接1](https://mp.weixin.qq.com/s/y_f2qrZvZe_F4_HPnwVjOw)、[链接2](https://blog.guitar-coder.cn/MySql%E9%94%81-%E9%97%B4%E9%9A%99%E9%94%81%E5%92%8C%E4%B8%B4%E9%94%AE%E9%94%81.html)



#### 4、如何锁定某一行

如果想要单独锁定某一行，可以在事务中，在语句后面添加`for update`:

```mysql
begin;  # 开启事务，也可以使用set autocommit=0;
# 假如想要单独锁定a=8的这一行,在执行语句后面加for update
select * from test_innodb_lock where a=8 for update;
commit;
```



#### 5、案例结论

InnoDB存储引擎实现了行锁，虽然在实现行锁所带来的性能损耗可能比表锁更高，但是在整体并发处理能力方面要远远优于MyISAM的表锁的。当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势。
但是，Innodb的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让Innodb的整体性能表现不仅不能比MyISAM高，甚至可能会更差。



### 4.3.3 行锁分析

使用以下指令查看行锁争夺相关的状态信息：

```mysql
show status like 'innodb_row_lock%';
```

其结果中，各个状态说明如下：

* `Innodb_row_lock_current_waits`：表示**当前**正在等待锁定的数量。
* `Innodb_row_lock_time`：表示从系统启动到现在，**等待的总时间**。
* `Innodb_row_lock_time_avg`：表示每次**等待所花的平均时间**。
* `Innodb_row_lock_time_max`：表示从系统启动到现在，等待最长的一次时间。
* `Innodb_row_lock_waits`：表示系统启动到现在，**总共等待的次数**。



如果等待次数很高，且每次平均等待时长也很高的时候，就说明系统可能存在问题，需要对系统进行分析，然后优化。



### 4.3.4 优化建议

1、尽可能让所有数据检索都通过索引完成，避免无索引行为导致行锁升级为表锁。

2、合理设计索引，尽量缩小锁的范围。

3、尽可能减少检索条件，避免间隙锁。

4、尽量控制事务大小，减少锁定的资源量和时间。

5、尽可能低级别事务隔离。



## 4.4 页锁

开销和加锁时间介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般。



# 五、MVCC

以下内容参考[链接1](https://juejin.cn/post/6871046354018238472#heading-0)、[链接2](https://zhuanlan.zhihu.com/p/133480167)

## 5.1 MVCC介绍

**MVCC(Mutil-Version Concurrency Control)，多版本并发控制**。它使得大部分支持行锁的事务引擎，不再单纯的使用行锁来进行数据库的并发控制，取而代之的是**把数据库的行锁与行的多个版本结合起来**，只需要很小的开销，就可以实现非锁定读，从而大大提高数据库系统的并发性能。可以将MVCC看成是乐观锁的一种实现。

MVCC主要是对于InnoDB引擎来说的。



## 5.2 相关概念



**乐观锁（Optimistic Lock）**：总是假设最好的情况，假设一般情况下不会造成冲突，所以取数据时不会上锁。在提交更新时，才会判断是否有冲突。**乐观锁适用于多读的情况，可以提高吞吐量。**

乐观锁的实现方式主要有两种：CAS（比较并替换）算法、版本号控制。

比如，Java中的`util.concurrent.atomic`包下面的原子变量类就是使用了CAS实现的。

**悲观锁（Pessimistic Lock）**：总是假设最坏的情况。对数据操作之前，总是认为会发生冲突，因此会上锁，其他线程如果想操作数据就会进入阻塞状态。悲观锁具有强烈的独占和排他性，共享锁（读锁）和排它锁（写锁）是悲观锁的两种情况。

悲观锁的实现：数据库中的锁机制，比如表锁、行锁等，都是在操作之前先上锁；Java中的`synchronized`的实现、`ReentrantLock`的实现。

参考[乐观锁和悲观锁](https://zhuanlan.zhihu.com/p/40211594)、[什么是乐观锁、悲观锁](https://www.jianshu.com/p/d2ac26ca6525)



**快照读**：快照读的实现是基于**多版本**并发控制，即MVCC的，快照读读取到的数据不一定是最新的数据，好比对某一个版本生成了一个快照，这个快照一旦生成了其中的数据就不会发生改变。**MVCC是“维持一个数据的多个版本，使读写操作没有冲突”的一个抽象概念，这个概念的具体实现就是快照读。**

比如不加锁的select操作。

**当前读**：总是读取当前最新版本的数据，会对当前读的数据进行加锁。是悲观锁的一种操作。

比如共享锁、排它锁。



## 5.3 解决的问题

**数据库并发场景**

- `读-读`：不存在任何问题，也不需要并发控制。
- `读-写`：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读，幻读，不可重复读。
- `写-写`：有线程安全问题，可能会存在更新丢失问题，比如第一类更新丢失，第二类更新丢失。



**MVCC解决并发哪些问题？**

MVCC是用来解决读写冲突的无锁并发控制，就是为事务分配`单向增长`的`时间戳`。为每个数据修改保存一个`版本`，版本与事务时间戳`相关联`。

读操作`只读取`该事务`开始前`的`数据库快照`。

**解决问题如下：**

- `并发读-写时`：可以做到读操作不阻塞写操作，同时写操作也不会阻塞读操作。
- 解决`脏读`、`幻读`、`不可重复读`等事务隔离问题，但不能解决上面的`写-写 更新丢失`问题。

**因此有了下面提高并发性能的`组合拳`：**

- `MVCC + 悲观锁`：MVCC解决读写冲突，悲观锁解决写写冲突
- `MVCC + 乐观锁`：MVCC解决读写冲突，乐观锁解决写写冲突




## 5.4 MVCC的实现原理

MVCC的实现原理主要是**版本链**、**undo日志**、**ReadView**来实现的。

### 5.4.1 版本链

数据库每行数据中，除了可见数据，还包括了几个隐藏字段，分别是：

* `db_trx_id`：6B，最近修改事务id，记录创建或最后一次修改这条记录的事务id

* `db_roll_pointer`：7B，回滚指针，指向这条记录的上一个版本

* `db_row_id`：6B，隐含的自增id，如果数据表没有主键，InnoDB会自动以db_row_id生成一个聚簇索引。

  > 还有一个删除flag字段，删除或更新字段时，只是删除flag改变了。



每次对数据库记录进行改动，都会记录一条`undo日志`，每条undo日志都有一个`roll_pointer`属性，可以将这些undo日志连起来串成一个链表，对该记录每次更新后，都会将旧值放到一条undo日志中，算是该记录的一个旧版本，随着更新次数的增多，所有的版本都会被`roll_pointer`属性连接成一个链表，这个链表称之为`版本链`，**版本链的头节点就是当前记录最新的值**。

**每个版本中还包含生成该版本时对应的事务id，用于Read View中判断版本可见性**。

### 5.4.2 undo日志

**undo log**主要用于记录数据被修改之前的日志，在表信息被修改之前会先把数据拷贝到`undo log`中。

事务进行回滚时，可以通过`undo log`里的日志进行数据还原。

**undo日志的作用：**

- 保证事务进行回滚时的`原子性和一致性`，当事务进行回滚的时候可以用`undo log`的数据进行恢复。
- 用于MVCC`快照读`的数据，在MVCC多版本控制中，通过读取`undo log`的`历史版本数据`可以实现`不同事务版本号`都拥有自己`独立的快照数据版本`。

> InnoDB中，用undo日志保证原子性和一致性，redo日志保证持久性，锁机制保证隔离性。



**undo日志的分类**

undo日志主要有两种：

* insert undo log：代表事务在insert新记录时产生的undo log , 只在事务回滚时需要，并且在事务提交后可以被立即丢弃。

* update undo log（主要）：事务在进行update或delete时产生的undo log ; **不仅在事务回滚时需要，在快照读时也需要**；不能随便删除，只有在快速读或事务回滚不涉及该日志时，对应的日志才会被purge线程统一清除。



### 5.4.3 Read View（读视图）

事务进行`快照读`操作的时候生成`读视图`(Read View)，在该事务执行的快照读的那一刻，会生成数据库系统当前的一个`快照`。

每个事务开启时，都会被分配一个ID，这个ID是递增的，越新的事务ID值越大。其中没有commit的事务称为**活跃事务**。

Read View记录并维护着系统**当前活跃事务的id**，是系统中当前不应该被**本事务**看到的**其他事务id列表**。

<font color='red'>Read View主要是用来做可见性判断</font>，当某个事务执行快照读的时候，对该记录创建一个Read View读视图，根据这个读视图，**用来判断当前事务能够看到哪个版本的数据**，可能是当前最新版本的数据，也可能是该行记录的undo log里的某个版本的数据。



**Read View的几个属性**

* `trx_ids`：当前系统活跃（未提交）事务版本号（id）集合
* `low_limit_id`：创建当前读视图时的当前**系统最大事务版本号+1**
* `up_limit_id`：创建当前读视图时，系统**活跃事务的最小版本号**。
* `creator_trx_id`：创建当前读视图的事务版本号。



**Read View可见性判断**

对于某条记录，其对于当前事务的可见性根据以下条件逐步进行判断。

**1、`db_trx_id`<`up_limit_id` || `db_trx_id`==`creator_trx_id`**

显示内容。包括两种情况。

第一种，如果当前数据的最近修改事务id，小于读视图中最小活跃事务id，说明**该数据肯定是当前事务开启之前就存在**了，可以显示给当前事务。

第二种，如果当前数据的最近修改事务id，就是创建当前读视图的事务，说明这个数据就是当前事务自己创建的，自己创建的数据当然可以显示。



**2、`db_trx_id` >= `low_limit_id`**

不显示。

如果当前数据的最近修改事务id，比生成读视图时候的最大事务id还大，说明这个数据是创建当前读视图之后才生成的，所以此数据不显示。

如果不满足此条件，则进行下一步判断。



**3、`db_trx_id`是否在`trx_ids`（活跃事务）中**

* 不在其中：显示。如果当前数据的最近修改事务id，不在创建读视图时的活跃事务中，说明创建读视图的时候，这个事务已经提交了（因此不是活跃事务），则此数据可以显示。
* 存在其中：不显示。说明创建读视图的时候，这个事务还在活跃，处于未提交状态，因此不予显示。



## 5.5 MVCC和事务隔离级别

根据MVCC的实现原理，可以知道，Read View用于**RC（Read Commited，读提交）**和**RR（Repeatable Read，可重复读）**这两种隔离级别的实现，解决了**脏读**和**不可重复读**的问题。



**解决脏读和不可重复读**

在不同的隔离级别下，MVCC中Read View的创建是不同的。

对于`RC`隔离级别：**每个快照读都会生成并获取最新的Read View**，因此只能解决脏读问题，不能解决不可重复读的问题。

对于`RR`隔离级别：**对于同一个事务来说，只有第一个快照读才会创建Read View**，之后的快照读获取的都是同一个Read View，**不会重复生成**，因此其他事务的修改对于当前事务来说是不可见的，所以每次查询的结果都是一样的，解决了不可重复读的问题。



**解决幻读问题**

* 快照读：对于快照读来说，直接用MVCC控制就可以，不需要加锁。根据MVCC中的规则进行增删查改操作，就可以避免幻读。
* 当前读：对于当前读来说，使用next-key(临键锁)来控制，避免幻读。



# 六、主从复制

参考[[MySQL高级](七) MySQL主从复制及读写分离实战](https://blog.csdn.net/why15732625998/article/details/80463041)

## 6.1 复制的基本原理



主从复制的基本原理：slave机器会从master机器读取`binlog`文件来进行数据同步，主要有三步：

* master将改变记录到二进制文件（`binlog`）中，这些记录过程叫做二进制日志事件（binary log events）；
* slave将master的二进制文件中的内容（二进制日志事件）拷贝到自己的中继日志（`relay log`）；
* salve重做中继日志中的事件，将改变应用到自己的数据库中，MySQL复制是异步的且串行化的。



> MySQL 的二进制文件，即常说的`binlog`文件，是MySQL执行改动产生的二进制日志文件，其主要作用有两个：
>
> 1、Replication（*主从数据库）*：在master端开启`binlog`文件后，log会记录所有数据库的改动，然后slave端获取这个日志文件内容就可以在slave端进行同样的操作。
>
> 2、备份（*数据恢复* ）：在某个时间点a做了一次备份，然后利用二进制日志文件记录从这个时间点a后的所有数据库的改动，然后下一次还原的时候，利用时间点a的备份文件和这个二进制日志文件，就可以将数据还原。



## 6.2 复制的基本原则

主从复制的基本原则：

* 每个slave只有一个master

* 每个slave只有一个唯一的服务器ID

* 每个master可以有多个slave

  

## 6.3 复制的最大问题

延时问题是主从复制最大的问题。



## 6.4 一主一从常见配置

以一个Windows主机，一个Linux从机为例。

**要求：**

①MySQL版本尽量一致，且后台以服务运行。版本不同容易导致兼容问题。

②主从都配置在`mysqld`节点下

**步骤：**

**一、主机修改`my.ini`配置文件（Windows中是ini后缀，linux中是cnf后缀）**

假设本地安装路径为`D:/MySQLServer5.7`

* 设置主服务器唯一ID（必须）

  ```bash
  server-id=1
  ```

  

* 启用二进制日志（必须）

  ```bash
  log-bin=本地路径/data/mysqlbin
  # 即
  log-bin=D:/MySQLServer5.7/data/mysqlbin
  ```

  

* 启用错误日志（可选）

  ```bash
  log-err=本地路径/data/mysqlerr
  ```

* 根目录（可选）

  ```bash
  basedir="D:/MySQLServer5.7"
  ```

* 临时目录（可选）

  ```bash
  tmpdir="D:/MySQLServer5.7"
  ```

* 数据目录（可选）

  ```bash
  datadir="D:/MySQLServer5.7/Data/"
  ```

* 设置`read-only=0`，意为主机可以读写。

* 设置不要复制的数据库（可选）

  ```bash
  # 比如设置mysql这个数据库不复制
  binlog-ignore-db=mysql
  ```

* 设置需要复制的数据库（可选）

  ```bash
  # 比如需要复制student数据库
  binlog-do-db=student
  ```



**二、从机修改`my.cnf`配置文件**

* 设置从服务器唯一ID（必须）

  ```bash
  # 需要和主机id号不同
  service-id=2
  ```

  

* 启用二进制文件（可选）

  ```bash
  log-bin=mysql-bin
  ```

  

修改完配置文件后，主机和从机必须重启MySQL服务。



**三、主机和从机都关闭防火墙**

* CentOS 7关闭防火墙

  ```bash
  systemctl stop firewalld.service
  ```

* Windows关闭防火墙

  ```bash
  # cmd窗口
  # 查看防火墙状态
  netsh advfirewall show allprofiles
  
  # 关闭防火墙
  netsh advfirewall set allprofiles state off
  ```

  

**四、主机上建立账户并授权slave**

* 授权给指定ip的从机账户名和密码：

  ```mysql
  # 比如从机ip为192.168.168.168，给从机建立的账户名为kang，密码为123456
  # 192.168.168.168从机就可以根据此用户名和密码进行复制。
  # replication slave表示给从机赋予复制的权限，*.*表示所有数据库。
  # ip如果为%则表示任意ip都可以使用此账号密码登陆
  grant replication slave on *.* to 'kang'@'192.168.168.168' identified by '123456';
  ```

* 刷新一下权限，使权限立即生效：`flush privileges;`

* 查询master状态

  ```mysql
  show master status;
  # 记录下File和Position的值，slave需要从File文件的Position位置开始复制。
  ```

  执行完此步骤后，不要操作主机MySQL，防止主服务器发生变化。

  > 重置master二进制文件：`reset master;`，会删除所有binlog文件，慎用。

**五、从机配置需要复制的主机信息**

* 配置账户名和File和Position：

  ```mysql
  # 需要指定主机ip，账户名和密码就是主机授权的账户名和密码
  # File和Position是从File文件的Positon位置开始复制
  change master to master_host='主机ip',
  master_user='kang',
  master_password='123456',
  master_log_file='File名字',
  master_log_pos=Position数字;
  
  # 如果之前做过同步，需要停止同步：stop slave;
  ```

* 启动从机的复制功能：`start slave;`

* 显示状态：

  ```mysql
  show slave status;
  # 可以在指令后面使用\G代替分号，表示按照键值对的方式显示内容
  ```

  如果`Slave_IO_Running: Yes`，`Slave_SQL_Running: Yes`说明配置成功。



配置成功后，主机的新建库、新建表、插入数据等相关操作就会被从机复制下来，从而实现数据同步。

如果需要同步的时主机中已有，但从机中没有的数据库，需要先手动将此数据库备份到从机，才能完成此数据库的同步。

使用`mysqldump`指令可以导出数据。



**六、停止从主机复制数据的功能**

```mysql
stop slave;
```



## 6.5 故障排除

常见故障排除方法：

**1、网络问题。**

确保主机和从机可以互相ping通，才能够保证主从复制正常使用。

**2、检查账号密码**

仔细检查账号密码是否有误。

**3、防火墙**

主机和从机的防火墙都要关闭。

**4、MySQL文件配置问题**

检查文件配置，确保主机开启二进制日志文件，确保主机和从机的`server-id`不同

**5、检查语法问题**

**6、检查账号权限**

在主机服务器中，检查授权账号的权限和登陆主机限制。

```mysql
# host限制了此账号在哪个主机上可以登录。如果是%表示任何主机。
select user,host from mysql.user;
```



**遇到的问题**

**问题1、问题描述**

配置完后，发现`Slave_IO_Running`状态一直处于`Connecting`状态。说明一直在连接主机数据库。

尝试手动使用授权的账号密码登陆主机数据库，正常情况下应该能够登录成功，结果登陆不成功，因此导致了从机的`Slave_IO_Running`状态一直处于`Connecting`状态。如下：

![从机无法使用授权账号登陆到主机](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_22.png)



![Slave_IO_Running](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_23.png)





**解决方法：**

ERROR 1130错误，说明在当前ip下，此账号没有权限登录到主机的MySQL数据库。在主机中可以查看用户以及允许登陆的主机：

```mysql
# 查看用户和host信息
select user,host from mysql.user;
```

可以看到，slave账号允许登陆的主机ip，我使用的是Vmware虚拟机CentOS7作为从机，Windows作为主机，网络模式是NAT模式，虽然从机ip确实是`192.168.198.198`，主机和从机也能互相ping通，但是就不允许登陆。可能是和NAT模式有关？（没搞清楚为什么会这样，按理说应该可以登陆。。。）如果换成桥接模式是不是就可以了？（以后有空再试吧）

使用`update`语句将`slave`账号的host改为`%`后，就可以登录了，`Slave_IO_Running`变为正常。

![主机服务器查看用户信息](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/mysql-advance_24.png)





**问题2、问题描述**

上个问题解决了以后，看起来没问题了，但是数据只能同步执行一次。主机创建一个数据库以后，从机可以同步创建，但是主机再进行操作，从机就不继续同步了，用`show slave status`查看状态，`Read_Master_Log_Pos`一直和主机状态中的保持同步，但是就不同步执行主机相应命令:-(

可能是因为MySQL数据库版本不同？主机用的MySQL 5.7，从机用的MySQL 5.6，不知道是不是因为版本的问题。等有空再将版本统一一下，重新配置一遍试试。



# 七、其他内容补充

## 7.1 数据库范式

第一范式（1NF）：属性不可分割，即表中的字段不能再划分为多个字段

第二范式（2NF）：满足1NF的同时，必须有主键，且其他字段都依赖于主键（或者是候选码），保证唯一性。即每个非主属性完全依赖于候选码。

第三范式（3NF）：每个非主属性既不传递依赖于主码，也不部分依赖于主码。即属性不依赖于其他非主属性，直接依赖于主键。主要是用于避免冗余，同一个属性不能存在于多个表中，即一个表中的属性不能依赖于其他表的主键，否则这个属性在多个表中都有，就出现了冗余。



# 参考链接

1. [MySQL数据库优化](https://www.bilibili.com/video/BV1KW411u7vy)
2. [JavaGuide-MySQL数据库索引总结](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B4%A2%E5%BC%95)
3. [一条 SQL 语句在 MySQL 中如何执行的](https://snailclimb.gitee.io/javaguide/#/docs/database/一条sql语句在mysql中如何执行的)
4. [为什么 MySQL 使用 B+树](https://zhuanlan.zhihu.com/p/110375475)
5. [B+树看这一篇就够了](https://zhuanlan.zhihu.com/p/149287061)
6. [MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)
7. 《高性能 MySQL第3版》