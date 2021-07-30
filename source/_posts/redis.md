---
title: Redis基础
excerpt: Redis数据库的数据类型、事务、配置、主从配置等
mathjax: true
date: 2021-07-30 17:16:56
tags: ['Redis','SQL','数据库']
categories: 数据库
keywords: NoSQL,Redis,SQL,数据库
---



# 一、Redis入门

Redis（Remote Dictionary Server）远程字典服务。基于**键值对**的NoSQL数据库

Redis教程参考：[Redis中文教程](https://www.redis.com.cn/tutorial.html)

Redis命令参考：[Redis命令](http://www.redis.cn/commands.html)

## 1.1 Redis安装

获取安装包：

```bash
wget http://download.redis.io/releases/redis-5.0.8.tar.gz
```



一般我们将安装的第三方文件放到`usr/local`目录下。

因此将其解压到`usr/local/redis`目录下，作为redis的根目录。

解压到指定目录：

```bash
tar xzvf redis-5.0.8.tar.gz /usr/local/redis
```

进入`redis`目录进行安装：

```bash
cd redis
# 因为Redis是c语言编写的，需要gcc编译器的支持，因此需要保证系统中已安装了gcc
# 编译
make
```

编译后，在`redis`目录下会生成一个`src`目录，里面包含了`redis-server`服务端和`redis-cli`客户端，已经可以启动redis服务了。

![编译后的目录](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/redis_1.png)

通常，我们还会执行`make install`进行安装，这样可以方便管理：

```bash
# 进入src目录
cd src
# 安装，并指定安装的位置，在这个位置下会生成一个bin目录
make install PREFIX=/usr/local/redis
```



安装完后，指定的`/usr/local/redis`目录下会生成一个`bin`目录，这个目录里面包含了常用的几个可执行文件，我们将`redis/redis.conf`配置文件复制一份到`redis/bin`这个目录，作为我们自己的配置文件。

```bash
cp redis.conf ./bin/myredis.conf
```

![bin目录](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/redis_2.png)

这样，我们以后只需要在`redis/bin`这个目录下进行操作就可以了。

redis服务默认是前台执行的，将配置文件中的`daemonize`值修改为`yes`，就可以将redis启动为后台进程。

在`bin`目录下启动redis服务端：

```bash
# 在bin目录下，根据指定的配置文件启动,如果不指定，会根据默认的配置文件启动
./redis-server myredis.conf
```

启动成功。



## 1.2 Redis启动



Redis的默认端口号是6379。

启动服务端，在安装目录下：

```bash
# 使用默认的配置文件启动
./redis-server 
# 也可以使用指定的配置文件启动
./redis-server redis-config.conf
```

启动客户端连接服务端：

```bash
./redis-cli -p 6379
```

如果是服务端就是本机，则端口号和服务器主机地址可以不用写

在客户端中，可以使用`exit`命令退出连接，使用`shutdown`命令关闭服务端。

Redis命令不区分大小写。

redis默认有16个数据库，`select`命令选择指定的数据库

```bash
# 选择第0号数据库
select 0
```



清空当前数据库：

```bash
flushdb
```

清空所有数据库：

```bash
flushall
```

删除当前数据库的某个key

```bash
del mykey
```





Redis是单线程的，基于内存的，没有使用多线程，redis6.0之后引入了多线程，主要是为了提高网络IO读写性能。

Redis基于内存，因此其瓶颈主要受限于内存和网络。



## 1.3 Redis速度快的原因

1、**基于内存**

Redis的绝大部分请求是纯粹的内存操作，非常快速。



2、**数据结构简单**

Redis的数据结构简单，并且数据操作也简单，Redis中的数据结构是专门采用C语言进行设计的



3、**采用单线程**

Redis使用单线程模型，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不需要考虑锁的问题以及加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗。

因为**Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈是内存的大小和网络带宽**，因此可以采用单线程的方式。

Redis在4.0版本加入了多线程支持，主要的目的是针对大键值对删除操作的命令，使用这些命令就会使用主处理之外的其他线程来“异步处理”。

Redis 6.0正式引入了多线程，目的是为了提高网络IO读写性能。Redis6.0虽然引入了多线程，但是只在网络数据的读写这类耗时操作上使用，执行命令仍然是单线程顺序执行。



4、**使用多路I/O复用模型**

Redis虽然是单线程，但采用**非阻塞的IO多路复用程序**来监听来自客户端的大量连接。

> 非阻塞IO可以在等待期间做其他事情；阻塞IO等待期间什么也不能做。



5、**使用底层模型不同**

Redis自己构建了VM 机制 ，因为一般的系统调用系统函数，会浪费一定的时间去移动和请求；



# 二、数据类型

Redis中有五种基本数据类型：String、List、Set、Hash、sorted set（Zset）

另外有三种特殊的数据类型：geospatial、hyperloglog、bitmaps



## 2.1 五大数据类型

### 2.1.1 基础操作

查看所有的key：

```bash
keys *
```

设置一个key:

```bash
set name Rick   # 设置key为name，value为Rick
```

获取key的值：

```bash
get name   # 获取名为name的这个key的值
```

判断key是否存在：

```bash
exists name  # 判断name这个key是否存在
```

查看key的类型：

```bash
type name  # 查看name这个key的类型
```

移动指定key到其他数据库：

```bash
move name 1  # 将当前数据库中的name这个键值对移动到1号数据库中
```

设置key的过期时间：

```bash
expire name 10  # 将name这个key设置为10秒后过期，10s后就会被销毁
```

查看某个key的剩余存活时间

```bash
ttl name  # 查看name的剩余存活时间,-1表示永久有效
```



### 2.1.2 String

`append`追加字符串

```bash
set key1 v1  # 设置一个key
append key1 hello  # 向key1的值后面追加一个字符串hello，结果为v1hello
```

如果当前key不存在，会新建一个，相当于`set`命令

`strlen` 查看字符串长度

```bash
strlen key1   # key1为v1时，结果为2
```

`incr`将key的值+1，前提是key的值必须是整数：

```bash
set key1 10  # 设置一个key
incr key1  # 将key1的值+1，结果是11
```

`incrby`：将key的值加上指定的值：

```bash
set key1 10 # 设置key1的值为10
incrby key1 5 # 将key1的值+5，结果是15
```

`decr`将key的值减一。

`decrby`将key的值减指定的值。

`getrange`获取指定范围的字符：

```bash
set key1 hello  
getrange key1 0 3  # 获取key1的值的0-3范围的字符，结果是hell
getrange key1 0 -1 # hello，-1表示全部字符串
```

`setrange`替换指定位置开始的字符串：

```bash
set key1 hello
setrange key1 3 world  # helworld，将索引从3开始的字符开始往后替换
setrange key1 1 hh # hhhworld，字符从指定位置向后覆盖
```

`setex`，set with expire，即设置一个带过期时间的key：

```bash
setex key1 10 hello   # 设置key1的值为hello，且10秒后过期
```

`setnx`，set if not exist，如果当前key不存在，就创建，否则创建失败（而普通的`set`指令会替换值）：

```bash
set key1 hello # 当前有一个key1
setnx key1 world # 返回值为0，表示设置失败，因为key1已经存在了
setnx key2 world # 返回值为1，key2不存在的时候，会新建成功
```

`mset`和`mget`同时设置和获取多个值

```bash
mset k1 v1 k2 v2 k3 v3
mget k1 k2 k3 # 结果为v1 v2 v3
```

`msetnx`如果指定的key都不存在才创建，只要有一个已经存在，就创建失败，这是一个原子性的指令。

---



使用`set`/`mset`命令创建对象，可以有两种方式，一种是保存JSON格式的字符串，另一种则是以键值对的方式：

```bash
# 方式一：JSON
set user:1 {name:Rick,age:20}

# 方式二：完全使用键值对
mset user:1:name Rick user:1:age:20
```

方式一保存了一条数据，key为`user:1`，value为`{name:Rick,age:20}`的数据；

方式二保存了两条数据，key为`user:1:name`，value为`Rick`；第二条数据的key为`user:1:age`，value为`20`。

两种方式都能够用来表示一个user对象，使用key来区分不同的对象。第二种方式的key是一个巧妙的设计。

---

`getset`先获取值，再设置值，如果key不存在，则会新建一个key，并把值赋进去：

```bash
getset key1 value1 # 如果key1不存在，则会返回nil
getset key1 value2 # 再次执行，此时可以获取到上一条指令设置的value1
```



String类型的使用场景，value值除了是字符串以外，还可以是数字型的字符串，可以用来计算。

应用场景：

* 计数器
* 记录阅读量、浏览量等



### 2.1.3 List

list，列表，可以当作栈、队列、阻塞队列来使用，本质是一个**双向链表**。具体用法参考：[List](https://www.redis.com.cn/redis-lists.html)

`list`的指令一般是以`l`和`r`开头的。`l`可以理解为list或left，r理解为right。

list中的值，索引是从最新的值为0开始算的，类似于栈顶元素的索引值为0。

`lpush`向一个list中添加一个值或多个值，从左边添加，即从头部添加：

```bash
# 向list1这个list中添加3个数据1、2、3、4、5
lpush list1 1
lpush list1 2
lpush list1 3
lpush list1 4 5
# 此时的list1值为5，4,3,2,1
```

`lindex`通过索引获取列表中的元素：

```bash
lindex list1 0  # 值为5
```

此时数据1、2、3、4、5在list1中的索引为4、3、2、1、0

`lrange`取出list数据中指定范围的值：

```bash
lrange list1 0 -1  # 获取list1的所有值
```

 `rpush`在列表中添加一个或多个值，从尾部添加：

```bash
rpush list1 0 # 此时的list1值为5,4,3,2,1,0
```

`lpop`从头部移除一个数据

`rpop`从尾部移除一个数据

`blpop`移除并获取列表的第一个元素

`brpop`移除并获取列表的最后一个元素

`llen`获取列表长度

`lrem`移除指定的元素：

```bash
# 从key中，移除count个值为value的元素
lrem key count value  
```

* count>0：从头到尾移除count个值为value的元素
* count<0：从尾到头移除count个置为value的元素
* count=0：移除所有值为value的元素。

`ltrim`将一个list截取为指定的元素，list的值会改变。

`rpoplpush`移除列表最后一个元素，并将其添加到指定的列表

`lset`通过索引设置列表指定索引元素的值。前提是list和这个索引位置必须存在。

`linsert`在列表的指定的值前或后插入元素

**应用场景**

* 发布或订阅
* 消息队列、慢查询



### 2.1.4 Set

set，集合，其中的值无序、不能重复。

set的指令以`s`开头。

| 指令                                                         | 操作                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------ |
| [sadd](https://www.redis.com.cn/commands/sadd.html)          | 向集合添加一个或多个成员                               |
| [scard](https://www.redis.com.cn/commands/scard.html)        | 获取集合的成员数                                       |
| [sdiff](https://www.redis.com.cn/commands/sdiff.html)        | 返回给定所有集合的差集，返回值是第一个集合中独有的元素 |
| [sdiffstore](https://www.redis.com.cn/commands/sdiffstore.html) | 返回给定所有集合的差集并存储在指定集合中               |
| [sinter](https://www.redis.com.cn/commands/sinter.html)      | 返回给定所有集合的交集                                 |
| [sinterstore](https://www.redis.com.cn/commands/sinterstore.html) | 返回给定所有集合的交集并存储在指定集合中               |
| [sismember](https://www.redis.com.cn/commands/sismember.html) | 判断 member 元素是否是集合 key 的成员                  |
| [smembers](https://www.redis.com.cn/commands/smembers.html)  | 返回集合中的所有成员                                   |
| [smove](https://www.redis.com.cn/commands/smove.html)        | 将 member 元素从 source 集合移动到指定的集合           |
| [spop](https://www.redis.com.cn/commands/spop.html)          | 移除并返回集合中的一个**随机**元素                     |
| [srandmember](https://www.redis.com.cn/commands/srandmember.html) | 返回集合中一个或多个随机数                             |
| [srem](https://www.redis.com.cn/commands/srem.html)          | 移除集合中一个或多个成员                               |
| [sunion](https://www.redis.com.cn/commands/sunion.html)      | 返回所有给定集合的并集                                 |
| [sunionstore](https://www.redis.com.cn/commands/sunionstore.html) | 所有给定集合的并集存储在指定集合中                     |
| [sscan](https://www.redis.com.cn/commands/sscan.html)        | 迭代集合中的元素                                       |

**应用场景**

* 需要存放的数据不能重复，以及需要获取多个数据源交集和并集的情况，比如共同关注，共同粉丝等。



### 2.1.5 Hash

hash类似于HashMap，即key的值是一个键-值对（域-值对），即`field`和`value`的映射。特别**适合存放对象。**

```bash
hset user:1 name Rick age 20 # 设置user:1这个hash的name为Rick，age为20
hget user:1 name  # 结果为Rick
```

hash的指令以`h`开头，表示其操作是对hash类型的数据操作的。

| 命令           | 描述                                            |
| -------------- | ----------------------------------------------- |
| `hdel`         | 删除一个或多个哈希表字段                        |
| `hexists`      | 查看哈希表key中，指定的字段是否存在             |
| `hget`         | 获取存储在哈希表中指定字段的值                  |
| `hgetall`      | 获取在哈希表中指定key的所有字段和值             |
| `hset`         | 设置哈希表中field的值                           |
| `hsetnx`       | 只有在字段filed不存在时，设置哈希表字段的值     |
| `hmget`        | 获取所有给定字段的值                            |
| `hmset`        | 同时将多个filed-value对设置到哈希表中           |
| `hkeys`        | 获取所有哈希表中的字段                          |
| `hvals`        | 获取哈希表中所有值                              |
| `hincrby`      | 为哈希表key中指定字段的整数值加上增量           |
| `hincrbyfloat` | 为哈希表key中指定字段的浮点数值加上增量         |
| `hlen`         | 获取哈希表中字段的数量                          |
| `hscan`        | 迭代哈希表中的键值对                            |
| `hstrlen`      | 返回哈希表中与给定域field相关联的值的字符串长度 |

**应用场景**

常用于存储对象数据



### 2.1.6 Zset

Zset，即sorted set，有序集合。与set相比，Zset增加了一个权重参数score，使得集合中的元素能够按照score进行有序排列，还可以通过score的范围来获取元素的列表。

| 命令                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [zadd](https://www.redis.com.cn/commands/zadd.html)          | 向有序集合添加一个或多个成员，或者更新已存在成员的分数       |
| [zcard](https://www.redis.com.cn/commands/zcard.html)        | 获取有序集合的成员数                                         |
| [zcount](https://www.redis.com.cn/commands/zcount.html)      | 计算在有序集合中指定区间分数的成员数                         |
| [zincrby](https://www.redis.com.cn/commands/zincrby.html)    | 有序集合中对指定成员的分数加上增量                           |
| [zinterstore](https://www.redis.com.cn/commands/zinterstore.html) | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中 |
| [zlexcount](https://www.redis.com.cn/commands/zlexcount.html) | 在有序集合中计算指定字典区间内成员数量                       |
| [zrange](https://www.redis.com.cn/commands/zrange.html)      | 通过索引区间返回有序集合成指定区间内的成员                   |
| [zrangebylex](https://www.redis.com.cn/commands/zrangebylex.html) | 通过字典区间返回有序集合的成员                               |
| [zrangebyscore](https://www.redis.com.cn/commands/zrangebyscore.html) | 通过分数返回有序集合指定区间内的成员，默认是闭区间，-inf为负无穷，+inf正无穷 |
| [zrank](https://www.redis.com.cn/commands/zrank.html)        | 返回有序集合中指定成员的索引                                 |
| [zrem](https://www.redis.com.cn/commands/zrem.html)          | 移除有序集合中的一个或多个成员                               |
| [zremrangebylex](https://www.redis.com.cn/commands/zremrangebylex.html) | 移除有序集合中给定的字典区间的所有成员                       |
| [zremrangebyrank](https://www.redis.com.cn/commands/zremrangebyrank.html) | 移除有序集合中给定的排名区间的所有成员                       |
| [zremrangebyscore](https://www.redis.com.cn/commands/zremrangebyscore.html) | 移除有序集合中给定的分数区间的所有成员                       |
| [zrevrange](https://www.redis.com.cn/commands/zrevrange.html) | 返回有序集中指定区间内的成员，通过索引，分数从高到低         |
| [zrevrangebyscore](https://www.redis.com.cn/commands/zrevrangebyscore.html) | 返回有序集中指定分数区间内的成员，分数从高到低排序           |
| [zrevrank](https://www.redis.com.cn/commands/zrevrank.html)  | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| [zscore](https://www.redis.com.cn/commands/zscore.html)      | 返回有序集中，成员的分数值                                   |
| [zunionstore](https://www.redis.com.cn/commands/zunionstore.html) | 计算一个或多个有序集的并集，并存储在新的 key 中              |
| [zscan](https://www.redis.com.cn/commands/zscan.html)        | 迭代有序集合中的元素（包括元素成员和元素分值）               |

**应用场景**

常用于需要对数据根据权重进行排序的场景，比如成绩排名、直播间礼物排行榜、热度排行榜等



## 2.2 三种特殊数据类型

### 2.2.1 geospatial

Redis GEO 主要用于存储地理位置信息，并对存储的信息进行操作，是 Redis 3.2新增的类型。

Redis GEO 操作方法有：

- `geoadd`：添加地理位置的坐标。
- `geopos`：获取地理位置的坐标。
- `geodist`：计算两个位置之间的距离。
- `georadius`：根据给定的经纬度坐标为中心，获取指定半径范围内的地理位置集合。
- `georadiusbymember`：根据储存在位置集合里面的某个地点，获取指定半径范围内的地理位置集合。
- `geohash`：返回一个或多个位置对象的 geohash 值。

使用举例：

`geoadd key 经度 纬度 member(即地点名)`

```bash
geoadd city 116.40 39.90 beijing  # key为city，成员为beijing，经纬度数据为116.4 39.90
geoadd city 121.47 31.23 shanghai
geoadd city 120.16 30.24 hangzhou
geoadd city 114.05 22.52 shenzhen
geoadd city 106.50 29.53 chongqing
```

`geopos`获取地理位置的坐标。

```bash
geopos city beijing
# 输出结果为：
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
```

`geodist key member1 member2 [unit]` 可以指定单位，比如m、km

```bash
# 计算beijing和shanghai之间的直线距离，默认是m为单位
geodist city beijing shanghai km  
"1067.3788"
```



geospatial底层使用Zset实现，因此可以使用Zset的部分指令，比如查看所有成员，或者移除成员：

```bash
127.0.0.1:6379> type city  # 查看geospatial的数据类型
zset  # geospatial的类型是zset
zrange city 0 -1  # 查看city里面的所有成员
zrem city hangzhou  # 移除city里面的hangzhou这个成员
```

**应用场景**

比如查找附近的人、查找指定地点周围的地点等。





### 2.2.2 hyperloglog

HyperLogLog是用来做基数统计的算法，它的优势是在输入元素的数量或者体积非常大时，计算基数所需的空间总是固定的、并且是很小的。

HyperLogLog**只会根据输入元素来计算基数，不会存储输入元素本身**，所以它不能像集合那样返回输入的各个元素。

> 基数，即集合中不重复的元素，比如{1,2,3,4,4,5,2,3}，其基数为{1,2,3,4,5}

使用HyperLogLog是由计算基数值是有误差的，但误差率小于1%。



常用命令：

| 命令      | 描述                                        |
| --------- | ------------------------------------------- |
| `pfadd`   | 添加指定元素到HyperLogLog中                 |
| `pfcount` | 返回给定HyperLogLog的基数估算值，误差小于1% |
| `pfmerge` | 将多个HyperLogLog合并为一个HyperLogLog      |



**应用场景**

比如计算网站的UV（独立访客，访问用户数），前提是允许有误差，否则只能用set。



### 2.2.3 bitmaps

Bitmaps使用二进制位来记录数据，只有0和1，比较适合表示某个元素的两种状态。

常用命令：

| 命令       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `setbit`   | 设置数据，返回值为之前位的值，默认为0。                      |
| `getbit`   | 获取某个位上的值                                             |
| `bitcount` | 计算并返回1的个数                                            |
| `bitop`    | 对一个或多个key进行位运算（and、or、not、xor），将结果保存到另一个key中 |
| `bitpos`   | 查找指定范围内，0或1出现的第一个位置。                       |

使用案例：

`setbit key offset value`，将key上指定offset上的值设置为value（0或1），offset可以用来表示用户id或日期。

```bash
# 设置day的7个值，表示七天的签到状态，1表示签到，0表示未签到
# 即1101011
setbit day 0 1
setbit day 1 1
setbit day 2 0
setbit day 3 1
setbit day 4 0
setbit day 5 1
setbit day 6 1

getbit day 5  #获取offset为5的值，结果为1
bitcount day  # 统计day中1的个数，结果为5，表示有5个1
bitpos day 1  # 查找day中1第一次出现的位置，默认是所有位，可以指定范围

# day1 1101011
# day2 1001100
bitop and day3 day1 day2 # 将day1和day2的与运算结果保存到day3中，结果为1001000
bitop or day4 day1 day2  # 将day1和day2的或运算结果保存到day4中，结果为1101111
```



**常用场景**

用户是否活跃、用户签到状态等。



# 三、Redis事务



## 3.1 Redis执行事务

<font color=red>Redis的单条命令是原子性的，但是Redis中的事务不保证原子性。因此没有隔离级别的概念</font>

Redis事务不支持回滚，因而不满足原子性，并且不满足持久性。Redis事务就是将多个命令请求打包，按顺序执行，并且中途不会被打断。

> 只有当发生语法错误(语法错误在命令队列时无法检测到)，对keys赋予了一个类型错误的数据时，Redis命令才会执行失败, 这些都是程序性错误，这类错误在开发的过程中就能够发现并解决掉，几乎不会出现在生产环境。因此不需要回滚机制
>
> 由于不需要回滚，这使得Redis内部更加简单，而且运行速度更快。

Redis的事务**本质**：一组命令的集合。一个事务中的所有命令都会被序列化，在事务的执行过程中，会按照顺序执行。

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送`EXEC`命令前被放入队列缓存。
- 收到`EXEC`命令后进入事务执行，**事务中任意命令执行失败，其余的命令依然被执行**。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

Redis事务从开始到执行的过程：

- 开始事务：`multi`
- 命令入队，即批量操作的命令，以队列FIFO的顺序执行
- 执行事务：`exec`

可以使用`discard`命令取消一个事务。



使用案例：

```bash
> multi 	# 开启事务
OK
> set k1 v1  # 命令入队
QUEUED
> set k2 v2
QUEUED
> set k3 v3
QUEUED
> get k2
QUEUED
> exec     # 执行事务，会按照入队顺序依次执行命令
1) OK
2) OK
3) OK
4) "v2"
===================
> multi		#开启事务
OK
> set k4 v4  # 命令入队
QUEUED
> discard  # 取消事务
OK  
> get k4   # 事务被取消，命令没有被执行，因此k4为空
(nil)
```



如果事务中的命令本身有语法错误，比如命令拼写错误，则事务不会执行，即事务中所有命令都不会执行；

如果某条命令执行失败，比如对字符串进行`incr`操作，则其他命令仍然会被执行，执行错误的命令会抛出异常。



## 3.2 watch



`watch`用于监听一个或多个`key`，如果被监听的`key`在事务执行之前被其他命令（比如其他线程）改动，则事务不会执行，直接返回失败。（即乐观锁的机制）

`unwatch`取消`watch`命令对所有`key`的监视。

事务执行完成后，成功或失败，都会自动解锁。

 



# 四、Jedis

Jedis是Redis官方推荐的Java连接开发工具，即使用Java操作Redis的一个中间件。



**1、导入jedis依赖**

导入jedis依赖

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>3.3.0</version>
    </dependency>
</dependencies>
```

**2、编写代码测试**

```java
public class TestPing {
    public static void main(String[] args) {
        //1.new Jedis对象;通过主机名和端口号连接
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        //jedis.auth("123456"); //如果是远程连接，需要验证密码
        
        //jedis中的所有命令就是redis的基础指令
        //测试连接，如果输出PONG，说明连接成功
        System.out.println(jedis.ping());
        jedis.close(); //关闭连接
    }
}
```



如果是远程的Redis，需要进行以下配置：

* 将redis配置文件的`bind 127.0.0.1`注释掉，确保外部主机可以访问。

* `protected-mode`如果为`true`，则需要设置访问redis的密码`requirepass`或`bind`指定ip，外部网络才可以访问；如果关闭保护模式，则外部网络可以直接访问。

  > 安全起见，建议开启`protected-mode`，设置访问密码，并修改默认的端口号。

* 确保服务器的防火墙关闭，`systemctl status firewalld.service`查看防火墙状态。

* 重启redis服务。

* 如果设置了密码，需要在Java代码中使用`auth()`方法验证密码，才能使用Redis数据库。



**3、连接成功后，可以使用jedis对象进行操作**

方法名和操作Redis的指令名是相同的，比如：

```
jedis.keys("*");
等价于
> keys *
```





**在Java代码中使用Redis事务**

```java
public class TestTransaction {
    public static void main(String[] args) {
        //1.new Jedis对象
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.auth("123456"); //验证密码
        Transaction multi = jedis.multi();//开启事务
        try{
            multi.set("user1","v1");
            multi.set("user2","v2");
            //int i = 1/0;  //模拟一个java中的异常
            multi.exec();  //执行事务
        }catch (Exception e){
            multi.discard();  //在exec()方法执行之前出现了Java代码异常，会放弃事务
            e.printStackTrace(); 
        }finally {
            jedis.close(); //关闭连接
        }
    }
}
```

这里`catch`中的异常，只能是捕获Java代码的异常，如果是Redis中的异常不会被捕获。比如，如果对字符串进行`incr`操作，Java代码并不会抛出异常，只是底层这个Redis语句会返回执行异常，事务中的其他命令仍然会执行。



# 五、SpringBoot整合Redis



## 5.1 环境配置

在SpringBoot中使用Redis，主要是通过[Spring Data Redis](https://spring.io/projects/spring-data-redis)

在SpringBoot项目中，可以使用模版自动引入启动器，或者手动引入Spring Data Redis启动器：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



在SpringBoot 2.x版本，底层使用的是lettuce，而不再是Jedis：

* Jedis：直连，多个线程操作时是不安全的；如果要避免不安全的情况，需要使用jedis pool连接池。更像BIO模式
* lettuce：使用netty，实例可以在多个线程中共享，不存在线程不安全的情况。可以减少线程数据，更新NIO模式。



根据自动配置类，找到Redis自动配置的源码，并且根据配置类`RedisProperties`，可以找到配置Redis的前缀为`spring.redis`

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

    //可以自定义redisTemplate来替换这个默认的
    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")  
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
        throws UnknownHostException {
        //默认的template没有过多的设置；redis对象都是需要序列化的
        //两个类型都是Object类型，需要根据需求强制类型转换，比如<String,Object>
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    //由于String类型是最常使用的类型，所以单独定义了一个bean
    @Bean
    @ConditionalOnMissingBean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
        throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```



在导入依赖后，可以进行配置，然后测试

`application.properties`

```properties
# 配置redis
spring.redis.host=127.0.0.1
spring.redis.port=6379
# 如果有密码，要配置密码
spring.redis.password=123456
```

测试：

```java
@SpringBootTest
class Redis02SprintbootApplicationTests {

    //注入redisTemplate
    @Autowired
    private RedisTemplate redisTemplate;
    @Test
    void contextLoads() {
        /*redisTemplate调用不同的方法操作不同的数据类型
        opsForValue 操作字符串，类似String
        opsForList 操作List 类似List
        常用的方法，可以直接调用方法，比如multi()方法
         */
//        redisTemplate.opsForValue().set("k1","v1");
//        redisTemplate.opsForList().leftPush("k2","v2");
//        redisTemplate.multi();
        /*
        获取连接对象
        */
//        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
//        connection.flushDb();
        redisTemplate.opsForValue().set("mykey","test");
        System.out.println(redisTemplate.opsForValue().get("mykey"));
    }
}
```



## 5.2 自定义RedisTemplate

根据Redis自动配置类，我们也可以自定义`RedisTemplate`，实际开发过程中都是使用自定义的`RedisTemplate`：

```java
/*
定义一个配置类，编写自定义的redisTemplate
*/
@Configuration
public class RedisConfig { 
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
        throws UnknownHostException {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        //底层默认的是JDK序列化，我们可以配置自定义的序列化方式
        //具体序列化方式根据实际情况确定
        //template.setKeySerializer();

        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```



此外，通常会创建工具类，使用工具类调用`RedisTemplate`的方法，这样更简洁。

```java
//自定义Redis工具类，将原生的方法封装到这个工具类。
@Component
public class RedisUtil {
    //使用自定义的配置类
    @Autowired
    private RedisTemplate redisTemplate;

    //调用原生的方法，创建工具类方法。这样可以更简洁地调用方法
    /**
     * 获取指定key的值
     */
    public Object get(String key){
        //工具类中功能调用底层方法
        return key==null ? null:redisTemplate.opsForValue().get(key);
    }
}
```



然后，我们就可以使用我们定义的工具类实现功能：

```java
@SpringBootTest
class Redis02SprintbootApplicationTests {
    //创建好工具类以后，进行装配
    @Autowired
    private RedisUtil redisUtil;

    @Test
    void test(){
        //调用工具类提供的get方法，更简洁
        System.out.println(redisUtil.get("t1"));
        //功能上相当于直接调用底层方法
        System.out.println(redisTemplate.opsForValue().get("t1"));
    }
}
```



# 六、 Redis配置文件

Redis安装完以后，配置文件为`redis.conf`，对其中的内容进行解析。

Redis启动，是通过配置文件来启动的。

配置文件主要有以下部分：

**1、UNITS**

配置文件对单位的大小写不敏感，比如1GB和1gb相同。

**2、INCLUDES**

可以包含其他配置文件。如：

```bash
include /path/to/local.conf
include /path/to/other.conf
```



**3、NETWORK**

可以使用bind来绑定ip，比如：

```bash
# 绑定指定的ip地址
bind 127.0.0.1  
```

保护模式：

```bash
# yes表示保护模式开启
protected-mode yes
```

端口设置：

```bash
port 6379
```



**4、GENERAL**

是否以守护进程方式启动

```bash
# 默认是no
# yes表示以守护进程方式运行，即后台运行
daemonize yes
```

配置文件的pid文件保存位置，如果redis以后台方式运行，就需要指定pid文件：

```bash
pidfile /var/run/redis_6379.pid
```

日志：

```propertiesbash
# 有四种日志级别，默认是notice，适用于生产环境
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice
```

生成的日志文件名：

```bash
# 如果为空，则为标准的输出
logfile ""
```

默认的数据库个数：

```bash
# 默认16个数据库，从0-15
databases 16
```

是否显示启动时的logo：

```bash
# 默认为开启状态
always-show-logo yes
```



**5、SNAPSHOTTING**

用于设置持久化`rdb`的配置。

持久化：在规定的时间内，执行了多少次操作，就会持久化到文件`.rdb`

Redis是内存数据库，如果没有持久化操作，就会断电即失。

设置持久化时间：

```bash
# 如果下面时间n内，至少m个key被修改，就会进行持久化操作
# In the example below the behaviour will be to save:
# after 900 sec (15 min) if at least 1 key changed
# after 300 sec (5 min) if at least 10 keys changed
# after 60 sec if at least 10000 keys changed
save 900 1
save 300 10
save 60 10000
# 可以根据需求设置
```

rdb文件名称：

```bash
# The filename where to dump the DB
dbfilename dump.rdb
```



如果bgsave操作失败，是否停止写入硬盘（持久化）：

```bash
# 默认是yes，即bgsave出错时，停止写入
stop-writes-on-bgsave-error yes
```

是否压缩rdb文件：

```bash
# 这个操作需要消耗一定的CPU资源
rdbcompression yes
```

是否校验rdb文件：

```bash
# 保存rdb文件的时候，进行错误校验检查
rdbchecksum yes
```

持久化（rdb文件）保存的目录：

```bash
#　默认是当前目录，即rdb文件保存的目录
dir ./
```



**6、REPLICATION**

replication是主从复制相关的配置



**7、SECURITY**

安全相关的配置

设置密码：

```bash
# 设置数据库密码为123456
requirepass 123456
```

也可以在命令行设置：

```bash
127.0.0.1:6379> config set requirepass "123456"
```



设置完密码后，需要使用`auth`进行密码验证才能使用数据库：

```bash
127.0.0.1:6379> auth 123456
```



**8、CLIENTS**

客户端配置。

限制最大客户端连接数：

```bash
# 设置最大连接数为10000
maxclients 10000
```



**9、MEMORY MANAGEMENT**

Redis的内存配置。

设置最大内存：

```bash
# 设置redis的最大内存容量
maxmemory <bytes>
```

内存满时的处理策略：

```bash
# 默认是noeviction
maxmemory-policy noeviction
# 一共有以下几种策略：
# volatile-lru -> Evict using approximated LRU among the keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key among the ones with an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
```



**10、APPEND ONLY MODE**

持久化`aof`配置。

默认不开启aof，而是使用rdb方式持久化：

```bash
# 默认使用rdb方式持久化，大部分情况下，rdb方式就够用了
appendonly no
```

aof持久化文件的名字：

```bash
# rdb文件的后缀名就是rdb
appendfilename "appendonly.aof"
```

同步策略：

```bash
# appendfsync always  # 每次修改都会同步，消耗性能
appendfsync everysec  # 每秒同步一次，可能会丢失最后一秒的数据
# appendfsync no      # 不执行同步，此时操作系统自己同步数据，速度最快
```



# 七、数据持久化

Redis是一个内存数据库，内存的特征是断电即失，因此Redis的持久化操作是将数据写入硬盘进行持久化。

Redis中的持久化操作有两种方式：**RDB（Redis DataBase）**、**AOF（Append Only File）**



## 7.1 RDB持久化

**RDB(Redis DataBase)方式**也叫**快照(snapshotting)**方式，因为这种方式是通过生成快照进行持久化的。Redis默认的持久化方式就是rdb方式。

RDB方式是通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。Redis 创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本（Redis 主从结构，主要用来提高 Redis 性能），还可以将快照留在原地以便重启服务器的时候使用。

Redis会单独创建 ( fork )一个子进程来进行持久化，会**先将数据写入到一个临时文件中，待持久化过程结束后，再用这个临时文件替换上次持久化好的文件**（原子性系统调用rename重命名）。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。**RDB的缺点是最后一次持久化后的数据可能丢失。**

> 如果最后一次修改，还未来得及进行rdb持久化就宕机，则最后一次持久化的数据就会丢失。

redis配置文件中的`SNAPSHOTTING`部分，就是关于rdb持久化的配置。其中的`save`设置：

```bash
save 900 1  # 表示如果900秒后，至少有一个key发生了改变，就进行持久化。
save 300 10
save 60 10000
```

RDB持久化一般通过`bgsave`操作创建快照，创建的文件名默认为`dump.rdb`

> RDB持久化有三种保存数据的机制：save、bgsave、自动化。
>
> save方式会阻塞Redis服务器，执行save命令期间，不能执行其他命令，直到RDB过程完成位置。
>
> bgsave（background save），常用的方式，会使用fork子进程的方式
>
> 自动化：通过配置文件完成。

生成`.rdb`文件的情况：

* 满足`save`指定的规则
* 执行`flushall`命令
* 退出Redis，也会生成`.rdb`文件

开发中要对`dump.rdb`文件进行备份，防止误删`.rdb`文件导致数据无法恢复。



**恢复rdb文件**

只需要将rdb文件放入Redis的启动目录就可以，Redis启动的时候会自动检查`dump.rdb`，恢复其中的数据。

查看需要存放的位置：

```bash
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/redis/bin"
```



**RDB方式的优缺点**

优点

* 适合大规模的数据恢复。

缺点：

* 需要一定的时间间隔来生成快照，如果没有到时间间隔系统宕机，则最后一段时间内的数据未被持久化导致数据丢失。
* fork进程的时候，会占用一定的内存空间。fork操作会对主进程进行阻塞（只是fork时会阻塞，fork完成后的子进程操作不会影响主进程），影响主进程读写。
* RDB是对完整的数据进行持久化，如果数据量较大，fork操作会消耗比较大的资源。





## 7.2 AOF持久化

AOF（Append Only File），只追加文件。

RDB方式是的持久化数据库所有数据，而AOF方式是追加日志。AOF是将所有命令都记录下来，恢复的时候把这个文件记录的命令再执行一遍。

与RDB方式相比，**AOF的实时性更好**，开启 AOF 持久化后**每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件**。Redis也会fork一个子进程用于追加日志文件，重启后，会读取AOF文件根据指令**重新构建数据。**

> 只记录写操作，不记录读操作。
>
> 只追加文件，不改写文件

AOF的配置在Redis配置文件的`APPEND ONLY MODE`部分。AOF 文件的保存位置和 RDB 文件的位置相同，都是通过`dir`参数设置的，默认的文件名是`appendonly.aof`。AOF默认是关闭的，需要手动开启。

Redis提供了三种AOF同步策略：

```bash
# appendfsync always  # 每次修改都会同步，消耗性能
appendfsync everysec  # 每秒同步一次，但可能会丢失最后一秒的数据
# appendfsync no      # 不执行同步，此时操作系统自己同步数据，速度最快
```

兼顾性能和数据，可以选用`everysec`方式，每秒同步一次，这样最多只会丢失一秒的数据。



`appendonly.aof`文件的内容是记录的写入指令，可以被人为破坏。如果`.aof`文件有错误，会导致Redis启动失败。可以使用`redis-check-aof`对其进行修复：

```bash
# 执行redis-check-aof修复aof文件
./redis-check-aof --fix appendonly.aof
```



**重写规则**

由于AOF是增量持久化方式，因此AOF文件会不断增大，这时候需要使用重写对其进行瘦身。

```bash
no-appendfsync-on-rewrite no
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
# 当文件大小大于当前文件的100%且文件大小至少是64MB时，才会触发重写机制。
```

AOF 重写可以产生一个新的 AOF 文件，这个新的 AOF 文件和原有的 AOF 文件所保存的数据库状态一样，但体积更小。AOF是通过读取数据库中的键值对来实现的，程序无须对现有 AOF 文件进行任何读入、分析或者写入操作。

> **AOF Rewrite** 虽然是“压缩”AOF文件的过程，但并非采用“基于原AOF文件”来重写或压缩，而是采取了类似RDB快照的方式：基于Copy On Write，全量遍历内存中数据，然后逐个序列到AOF文件中。因此AOF rewrite能够正确反应当前内存数据的状态。

**AOF持久化流程**

在执行 BGREWRITEAOF 命令时，Redis 服务器会维护一个 AOF 重写缓冲区，该缓冲区会在子进程创建新 AOF 文件期间，记录服务器执行的所有写命令。当子进程完成创建新 AOF 文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新 AOF 文件的末尾，使得新旧两个 AOF 文件所保存的数据库状态一致。最后，服务器用新的 AOF 文件替换旧的 AOF 文件，以此来完成 AOF 文件重写操作.



**AOF的优缺点**

优点：

* 可以每次修改都同步、每秒同步一次、从不同步三种同步策略。数据完整性从高变低，而效率从低变高。

缺点：

* 相对于RDB方式来说，AOF恢复数据慢，因为其要重新执行一遍指令重构数据。
* AOF是文本文件，体积较大，需要不断进行AOF重写，进行瘦身。





**Redis 4.0 对于持久化机制的优化**

Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（默认关闭，可以通过配置项 `aof-use-rdb-preamble` 开启）。

如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。缺点是 AOF 里面的 RDB 部分是压缩格式不再是 AOF 格式，可读性较差。



# 八、数据删除与淘汰

## 8.1 数据过期时间

Redis是内存数据库，而内存是有限的，如果缓存中的所有数据都一直保存的话，很容易导致内存溢出。因此需要对数据设置过期时间。此外，一些场景中的数据本身就只需要存活一段时间，比如验证码，通过设置过期时间，可以省略多余的判断操作。

**设置过期时间**

string类型有自带的创建带有过期时间的指令：

```bash
setex key 60 value  # 设置key为60s后过期
```

通用的设置过期时间的指令是`expire`，使用`persist`指令可以移除一个键的过期时间：

```bash
expire name 10  # 将name这个key设置为10秒后过期，10s后就会被销毁
```

查看某个key的剩余存活时间

```bash
ttl name  # 查看name的剩余存活时间,-1表示永久有效
```



**Redis是如何判断数据是否过期的？**

[Redis如何判断数据是否过期](https://snailclimb.gitee.io/javaguide/#/docs/database/Redis/redis-all?id=_11-redis-%e6%98%af%e5%a6%82%e4%bd%95%e5%88%a4%e6%96%ad%e6%95%b0%e6%8d%ae%e6%98%af%e5%90%a6%e8%bf%87%e6%9c%9f%e7%9a%84%e5%91%a2%ef%bc%9f)

Redis 通过**过期字典**（可以看作是 hash 表）来保存数据过期的时间。过期字典的键指向Redis中的某个 key，过期字典的值是一个 long long 类型的整数，这个整数保存了 key 所指向的数据库键的过期时间（毫秒精度的 UNIX 时间戳）。

过期字典是存储在`redisDb`这个结构里的：

```c
typedef struct redisDb {
    ...
    dict *dict;     // 数据库键字典,保存着数据库中所有键值对
    dict *expires   // 过期字典,保存着键的过期时间
    ...
} redisDb;
```



## 8.2 过期数据删除

对于过期的数据，Redis有三种删除策略：立即删除、惰性删除、定期删除。

**1、立即删除**

在设置键的过期时间时，创建一个回调事件，当过期时间达到时，由时间处理器自动执行键的删除操作。

优点：

* 立即删除能保证内存中数据的最大新鲜度，因为它**保证过期键值会在过期后马上被删除，其所占用的内存也会随之释放**。

缺点：

* **立即删除对cpu是最不友好的**。因为删除操作会占用cpu的时间，如果刚好碰上了cpu很忙的时候，比如正在做交集或排序等计算的时候，就会给cpu造成额外的压力。

redis事件处理器对时间事件的处理方式是无序链表，查找一个key的时间复杂度为O(n)，所以并不适合用来处理大量的时间事件。



**2、惰性删除** 

只会在取出 key 的时候才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。

优点：

* key被使用的时候才会检查是否过期。**对 CPU 最友好。**

缺点：

* **浪费内存**。比如某些数据在过期后可能很长一段时间内不会被访问（比如日志数据），但是这段时间dict字典和expires字典都要保存这个键值的信息，会浪费很多空间，对于依赖内存大小的Redis来说这个确定是致命的。



**3、定期删除** ： 每隔一段时间抽取一批 key 执行删除过期 key 操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

优点：

* 对CPU比较友好，优势介于立即删除和惰性删除之间。
* 另一方面，减少了因惰性删除带来的内存浪费。

Redis 采用的是 **定期删除+惰性/懒汉式删除** 。

但是，仅仅通过给 key 设置过期时间还是有问题的。因为还是可能存在定期删除和惰性删除漏掉了很多过期 key 的情况。这样就导致大量过期 key 堆积在内存里，同样会导致OOM。解决方法就是通过Redis的内存淘汰机制。



## 8.3 内存淘汰机制

当内存满的时候，Redis可以通过内存淘汰策略，将选中的数据淘汰。Redis一共有8种淘汰策略：

* **volatile-lru**：从**设置过期时间的数据集中**挑选出最近最少使用（LRU）的数据淘汰。
* **allkeys-lru** ：从数据集中挑选出最近最少使用（LRU）的数据淘汰。
* **volatile-lfu** ：从**设置过期时间的数据集中**挑选出最近最不常使用（LFU）的数据淘汰。
* **allkeys-lfu** ：从的数据集中挑选出最近最不常使用的数据淘汰。
* **volatile-random** ：从**设置过期时间的数据集中**随机挑选数据淘汰。
* **allkeys-random**：从数据集中随机挑选数据淘汰。
* **volatile-ttl**：从**设置过期时间的数据集中**挑选最近要过期的数据淘汰。
* **noeviction** ：禁止淘汰数据，当内存不足以容纳新写入数据时，新写入操作会报错。

> 其中LFU是Redis 4.0版本新增的淘汰策略。



# 九、Redis发布订阅

## 9.1 介绍

Redis的发布订阅（pub/sub）是一种消息通信模式：**发送者**（pub）发送消息，**订阅者**（sub）接收消息。传递消息的通道称为**频道**（channel）。

订阅者可以订阅任意数量的频道。

当发送者通过PUBLISH命令将新消息发送给频道时，这个消息就会被发送给订阅这个频道的n个订阅者。

发布订阅常用的命令：

| 命令         | 描述                               |
| :----------- | :--------------------------------- |
| psubscribe   | 订阅一个或多个符合给定模式的频道。 |
| pubsub       | 查看订阅与发布系统状态。           |
| publish      | 将信息发送到指定的频道。           |
| punsubscribe | 退订所有给定模式的频道。           |
| subscribe    | 订阅给定的一个或多个频道的信息。   |
| unsubscribe  | 指退订给定的频道。                 |



**使用案例**

订阅者订阅频道：

```bash
127.0.0.1:6379> subscribe pubTest
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "pubTest"
3) (integer) 1
```

发布者往频道内发布消息：

```bash
127.0.0.1:6379> publish pubTest "hello,world"
(integer) 1  # 表示有1个客户端收到
```



此时的订阅者能够收到消息：

```bash
127.0.0.1:6379> subscribe pubTest
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "pubTest"
3) (integer) 1
1) "message"  # 收到message
2) "pubTest"  # 来自频道pubTest
3) "hello,world"  # 消息内容为"hello,world"
```



Redis中的订阅功能最明显的用途就是用作**实时消息系统**，比如普通的即时聊天，群聊等功能。



## 9.2 pub/sub原理

**subscribe命令**

通过`SUBSCRIBE`命令订阅某频道后，redis-server里会维护一个字典，字典的键代表channel，字典的是一个链表，链表中保存了所有订阅这个channel的客户端。`SUBSCRIBE`命令的关键，就是将客户端添加到给定 channel 的订阅链表中。

**publish命令**

通过`PUBLISH`命令向订阅者发送消息，redis-server会使用给定的频道作为键，在它所维护的 channel 字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。



# 十、Redis主从复制

## 10.1 介绍

主从复制：将一台Redis服务器的数据，复制到其他的Redis服务器，前者称为主节点（master/leader），后者称为从节点（slaver/follower)。

数据的复制是单向的，只能由主节点到从节点。master以写为主，slaver以读为主。默认情况下每个服务器都是主节点，一个主节点可以由多个从节点，而一个从节点只能有一个主节点。

**主从复制的作用**：

* 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
* 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
* 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
* 高可用（集群）基石：主从复制是哨兵和集群能够实施的基础，是Redis高可用的基础。



## 10.2 环境配置

查看当前库信息：

```bash
# 使用info命令查看信息
127.0.0.1:6379> info replication 
# Replication
role:master  # 角色：master
connected_slaves:0  # 从机个数：0
master_replid:0fae2ec2d0ed428e4a08a46f126d426e70f45601
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

在一台服务器上通过创建多个配置文件，模拟多个Redis主机：

* 修改端口号
* 修改pid文件名
* 修改日志文件名
* 修改`dump.rdb`文件名



在单机上，通过不同端口，模拟一主二从的配置。

![两个配置文件，用来启动不同端口的Redis服务](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/redis_3.png)

![多个Redis服务](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/redis_4.png)

Redis主从复制遵循**配从不配主**的原则，只对从机进行配置。

在从机客户端使用`slaveof`命令指定其主机地址：

```bash
# 设置其从机地址
127.0.0.1:6380> slaveof 127.0.0.1 6379
# 查看此时的状态，已经变为了从机
127.0.0.1:6380> info replication
# Replication
role:slave  # 当前角色：从机
master_host:127.0.0.1   # 主机地址
master_port:6379
master_link_status:up  # 主机连接状态
master_last_io_seconds_ago:2
master_sync_in_progress:0
slave_repl_offset:42
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:e93e4bf39bf9973a62074e0d6a8e40f4babc71ed
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:42
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:29
repl_backlog_histlen:14
```

> Redis 5.0之前的版本使用`slaveof`，5.0之后的版本可以使用`replicaof`命令。

使用命令行配置从机的方式是一次性的，重启服务就失效了。如果要永久生效，需要在配置文件中配置`replicaof`选项，这样每次启动时会自动被设置为从机，这种方式是常用的方式：

```bash
# 配置主机地址和端口号
replicaof 127.0.0.1 6379
```



如果主机设置了密码，会导致`master_link_status`这一项结果为`down`，需要在从机的配置文件中设置`masterauth`添加主机密码：

```bash
# 指定主机密码
masterauth 123456
```



两个从机都设置好后，我们可以在主机客户端看到已连接的从机：

```bash
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2  # 有两个从机，然后是从机的详细信息
slave0:ip=127.0.0.1,port=6381,state=online,offset=42,lag=0
slave1:ip=127.0.0.1,port=6380,state=online,offset=42,lag=1
master_replid:e93e4bf39bf9973a62074e0d6a8e40f4babc71ed
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:42
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:42
```

简单的主从复制模型就配置好了。



## 10.3 主从复制测试



主机中的所有信息和数据，都会自动被从机保存。

默认情况下，从机是只读复制模式，即从机只能读，不能写。

在只读复制模式下，从机写入数据，会失败：

```bash
127.0.0.1:6380> set t1 v1
(error) READONLY You can't write against a read only replica.
```



**测试1、主机写入数据，从机会自动保存数据，可以读取**

**测试2、如果主机宕机再重连以后，从机同样能够保存新写入的数据**

**测试3、如果从机宕机，如果重启后仍然是之前主机的从机，那么仍然可以获取到主机的数据**

如果从机使用配置文件配置的主机地址，那么从机重启后仍然是从机，仍然可以获取到主机的数据。



## 10.4 复制原理

从机和主机的数据会保持一致，无论是哪一方宕机，重启后仍然能够同步数据，其复制原理如下：

* Slave启动成功连接到Master后会发送一个`sync`同步命令。
* Master接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。

复制方式有两种：

* **全量复制：**而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
* **增量复制：**Master继续将新的所有收集到的修改命令依次传给slave，完成同步



每次重新连接master，都会自动执行一次完全同步（全量复制）。因此即使从机宕机，重连以后也能同步到主机的数据。



## 10.5 链路模式

上述模式是一主机多从机的模式，主机和从机是一对多的关系。

还有一种主从复制是链路模式，主机和从机是一对一的关系，每个从机又有其自己的从机，层层相连。中间的从机既是主机，又是从机。

最顶端的主机写数据，后面的所有从机都能同步到数据。



# 十一、哨兵模式

哨兵模式实现了自动主从切换。

## 11.1 哨兵的作用

如果主机宕机，需要选用其中一个从机当作主机。使用`slaveof no one`命令，使当前从机变回主机。然后其他从机就可以连接到这个从机。在没有哨兵模式之前，上述操作是手动完成的。

Redis从2.8版本提供了Sentinel架构（哨兵模式）自动解决上述问题，哨兵模式能够自动剩余的从机中选出一个当作主机。

**哨兵模式**是一种特殊的模式，Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。**

实际情况中， 我们通常会设置多个哨兵，多个哨兵之间互相监督，防止单一哨兵出现问题时无法及时主从切换。



**多哨兵模式，基本原理如下：**

正常情况下，哨兵会向master发送心跳ping来确认Master是否存活。

假设主服务器宕机，master会不回应PONG或者回复错误值，哨兵1先检测到这个结果，系统并不会马上进行`failover`过程，仅仅是哨兵1主观的认为主服务器不可用，即**主观下线**（Subjectively Down，SDOWN）。

当其他哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行**failover（故障转移）**操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，即**客观下线**（Objectively Down，ODOWN）。

> 客观下线才是真正的进行主从切换。



## 11.2 创建哨兵

以一个哨兵为例，每一个哨兵都需要一个自己的配置文件：

`sentinel.conf`

```bash
#格式：sentinel monitor <option_name> <ip> <redis-port> <quorum>
sentinel monitor T1 127.0.0.1 6379 1
```

上述是最基础的哨兵配置。监控的master的名字叫做T1（自定义），地址为127.0.0.1:6379，行尾最后的数字1表示在sentinel集群中，至少有多少个哨兵认为masters挂掉了，才能真正认为该master不可用了，开始选举从机作为主机。



如果主机设置了密码，客户端和从机在连接时都需要提供密码。master通过`requirepass`设置自身的密码，slave通过`masterauth`来设置访问master时的密码。客户端需要`auth`提供密码。

但是当使用了sentinel时，由于一个master可能会变成一个slave，一个slave也可能会变成master，所以需要同时设置上述两个配置项，即同时配置`requirepass`和`masterauth`，并且哨兵需要连接master和slave，也必须设置参数：`sentinel auth-pass <master_name> xxxxx`。



配置好哨兵的配置文件以后，就可以执行`redis-sentinel`，并指定配置文件，启动指定的哨兵进程:

![启动哨兵](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/redis_5.png)



手动将主机`shutdown`掉，哨兵检测到主机宕机，会投票选出新的主机节点：

![执行了主从转换](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/redis_6.png)



一个一主二从一哨兵的架构就搭建好了，通常情况下，我们不会使用单个哨兵，而是使用多个哨兵组成哨兵集群。如果是哨兵集群，则需要配置多个哨兵配置文件，并且每个哨兵都需要配置自己的端口。详细的哨兵配置，参考[Redis哨兵机制](https://www.cnblogs.com/happydreamzjl/p/11322937.html)

如果主机重新连接回来，只能归并到新的主机下，当作新的主机的从机。



关于哨兵的底层原理，可以参考[哨兵选举底层原理](https://www.cnblogs.com/myd620/p/7811156.html)



## 11.3 哨兵模式总结

**优缺点**

优点：

* 基于主从复制原理，用于主从复制的优点。
* 主从可以切换，故障可以转移，系统的可用性更好。
* 哨兵模式就是主从模式的升级，从手动主从切换到自动主从切换。

缺点：

* redis较难支持在线扩容，如果集群容量到达上限，在线扩容很复杂。
* 实现哨兵集群的配置选择项很多，不容易配置。



# 十二、缓存穿透、缓存击穿、缓存雪崩

## 12.1 缓存穿透

**缓存穿透的概念**

用户查询一个redis缓存中没有的数据，并且数据库中也没有，因此每次都会到持久层中查询数据库，因为数据库中没有，最终查询失败。当用户很多，这种不存在的数据查询量很大的时候，都去请求持久层，给持久层数据库造成了很大的压力，导致失去了缓存保护后端数据库的意义，相当于穿透了缓存。



**解决方法**

**1、缓存空对象**

如果不存在此key，则在缓存中为这个key设置一个空值，这样每次查询就会直接从缓冲中返回空值。

但是这种方法存在两个问题：

* 值为`null`不代表不占用内存空间，空值做了缓存，意味着缓存层中存了更多的键，需要更多的内存空间，因此会造成一定程度上的空间浪费。比较有效的方法是针对这类数据**设置一个较短的过期时间，让其自动剔除**。
* **缓存和数据库的数据会有一段时间数据不一致**，这可能会对业务有一定影响。例如过期时间设置为5分钟，如果此时数据库中添加了这个数据，那此段时间就会出现缓存和数据库中数据不一致的情况，此时可以利用消息系统或者其他方式清除掉缓存层中的空对象。

**2、布隆过滤器**

在访问缓存和持久层之前，将存在的key用布隆过滤器提前保存起来，做第一层拦截，当收到一个对key请求时先用布隆过滤器验证是key否存在，如果存在才会进入缓存、持久层。否则就直接返回。

布隆过滤器是一个bit向量或者说bit数组，其基本原理是事先将所有可能查询的key映射到这个数组，是使用多个不同的哈希函数对插入的对象进行hash操作并生成多个哈希值，将每个生成的哈希值指向的bit位置置为1。当用户查询的时候，会查找多个哈希值对于位置的值是否为1，如果这多个位置上的值都是1，说明这个值**可能存在**，**否则一定不存在**。

> 布隆过滤器只能判断可能存在，并不能确定一定会存在，但是能够判断一定不存在的情况。
>
> 参考：[详解布隆过滤器原理](https://zhuanlan.zhihu.com/p/43263751)

![布隆过滤器](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/redis_7.jpg)





## 12.2 缓存击穿

**缓存击穿的概念**

与缓存穿透不同的是，缓存击穿指的是**某一时刻某一个热点数据失效**，对应的大量请求涌入数据库，造成数据库压力过大。

比如微博某一热点数据设置了过期时间，导致过期后到重新创建的这一小段时间内热点数据短暂失效，在这段时间内大量对于这一数据的查询请求会直接访问数据库，导致数据库压力过大，好比缓存被击穿。



**解决方案**

**1、设置热点数据永不过期**

从缓存层面来看，没有设置过期时间，所以不会出现热点 key过期后产生的问题。但是会存在数据不一致的情况。

**2、分布式互斥锁**

使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大。



## 12.3 缓存雪崩

**缓存雪崩概念**

缓存雪崩指的是**缓存在同一时间大面积的失效，后面的请求都直接落到了数据库上**，造成数据库短时间内承受大量请求。就像雪崩一样。

造成缓存雪崩有两种情况：一种是因为缓存模块本身的故障，比如宕机，导致原本应该访问缓存的请求都去访问了数据库；另一种造成缓存雪崩的情况是大量热点数据在某一时刻大面积失效，导致对应的请求落到了数据库上。



**解决方案**

针对Redis服务不可用的情况（情况一）：

* **使用redis集群**。采用Redis集群，避免单机出现问题整个缓存服务都不可用。（比如异地多活)
* **限流降级**。限流，避免同时处理大量的请求，设置最高的QPS阈值，当请求数超过这个阈值后，就不再调用后续资源。降级，指的是服务器压力剧增的情况下，根据业务情况，对一些服务和页面进行有策略的降级，以此释放服务器资源保证核心任务的正常运行。

针对Redis服务可用的情况（情况二）：

* **设置不同的失效时间**，比如随机设置缓存的失效时间。或者设置缓存永不失效。
* **数据预热**。在正式部署之前，先将可能的数据预先访问一遍，使那些可能大量访问的数据预先加载到缓存中。



# Reference

[Redis教程](https://www.bilibili.com/video/BV1S54y1R7SB)
