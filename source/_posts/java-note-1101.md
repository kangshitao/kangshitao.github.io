---
title: Java学习笔记11-Java集合
excerpt: Java集合框架，Collection，List，Set，Iterator，Map，Collections工具类
mathjax: true
date: 2021-04-09 22:11:30
tags: Java
categories: Java
keywords: Java,Java集合,HashMap,Collection,List,Set,Map
---

# 一、Java集合框架概述

## 1、概述

**Java集合**像一种容器，可以动态地把多个对象的引用放入容器中。

数组在存储方面的特点：

* 数组初始化以后长度确定。
* 声明的类型决定了元素初始化时的类型。

数组存储数据的缺点：

* 初始化以后长度就不可变，不便于扩展。
* 提供的属性和方法少，不便于添加删除、插入等操作，效率不高，无法直接获取元素个数。
* 数据有序、可以重复，存储数据的特点单一。

**Java集合类**可以用户存储数量不等的多个**对象**，还可用于保存具有映射关系的关联数组。

## 2、集合框架

Java 集合框架图：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1101_1.png'/>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1101_2.png'/>
</div>

Java集合框架主要有两个接口，`Collection`接口和`Map`接口：

`Collection`接口：单列集合，其包括以下几个接口

* `List`接口：存储**有序(指存储位置有序)、可重复**的数据
  * 常用实现类为`ArrayList`、`LinkedList`、`Vector`
* `Set`接口：存储**无序、不可重复**的数据
  * 常用实现类有`HashSet`、`LinkedHashSet`、`TreeSet`
* `Queue`接口：先进先出
  * 常用实现类有`LinkedList`、`PriorityQueue`

`Map`接口：**双列集合，用来存储成对的(key-value)数据**

* 常用实现类有`HashMap`、`LinkedHashMap`、`TreeMap`、`Hashtable`、`Properties`

> **无序性**指的是存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的哈希值决定的。

# 二、Collection接口方法

向`Collection`接口的实现类对象中添加数据时，要求此数据所在的类要重写`equals()`方法。

`Collection`接口定义了能够用于`List`/`Set`/`Queue`的方法。包括以下方法：

1、添加元素：`add(Object obj)`、`addAll(Collection coll)`
2、获取有效元素的个数：`int size()`
3、清空集合：`void clear()`
4、是否是空集合：`boolean isEmpty()`
5、是否包含某个元素：

* `boolean contains(Object obj)`：是通过元素的equals方法来判断是否是同一个对象
* `boolean containsAll(Collection c)`：也是调用元素的equals方法来比较。拿两个集合的元素挨个比较。

6、删除：

* `boolean remove(Object obj)`：通过元素的equals方法判断是否是要删除的那个元素。只会删除找到的第一个元素。
* `boolean removeAll(Collection coll)`：取当前集合的差集。

7、取两个集合的交集：`boolean retainAll(Collection c)`：把交集的结果存在当前集合中，不影响c本身

8、集合是否相等：`boolean equals(Object obj)`

9、转成对象数组：`Object[] toArray()`

> 数组转换为集合，调用数组的`asList()`方法：
>
> ```java
> List arr1 = Arrays.asList(new int[]{123, 456});
> System.out.println(arr1.size());//1，此时添加的是int[]数组对象引用，长度是1
> 
> List<Integer> arr2 = Arrays.asList(new Integer[]{123, 456});
> System.out.println(arr2.size());//2，添加的是Integer数组，有两个元素
> ```

10、获取对象哈希值：`hashCode()`

11、遍历：`iterator()`，返回迭代器对象，用于集合遍历



# 三、Iterator迭代器接口

`Collection`接口继承了`Iterator<E>`接口，该接口有`iterator()`方法，因此`Collection`的实现类都有这个方法，调用此方法返回一个实现了`Iterator`接口的对象，可以用于遍历：

```java
Collection coll = new ArrayList(); //以ArrayList为例
Iterator iterator = coll.iterator();  //返回一个迭代器对象
// Iterator<Integer> iterator = coll.iterator();  //Iterator是泛型类
while(iterator.hasNext()){
    Object obj = iterator.next();
    if(obj.equals("xxx")){
        iter.remove();
    }
}
```

Iterator接口的三个方法：

* `boolean hasNext()`：判断是否有下一个值
* `E next()`：指针下移，并返回下移以后指向的值，E是泛型
* `void remove()`：移除当前指向的值。执行remove操作后，需要执行next才能进行下一步操作

> 迭代器指针初始第一个元素之前，需要next()操作才指向第一个值。
>
> 集合对象每次调用`iterator()`方法都得到一个全新的迭代器对象，默认游标都在集合的第一个元素之前。



JDK 5.0新增了`foreach`循环，用于遍历集合、数组，foreach内部仍然使用了迭代器：

```java
/*
for (元素类型 局部变量：集合或数组对象){
	//方法体
}
*/
//遍历集合
ArrayList<Integer> a = new ArrayList<>();
for(Integer i : a){
    System.out.println(i);
}
//遍历数组
int[] array = new int[]{1,2,3};
for(int i:array){
    System.out.println(i);
}
```



# 四、Collection子接口一：List

## 1、List接口方法

除了从`Collection`接口继承的方法以外，`List`接口还新增了一些**根据索引操作集合元素**的方法：

* `void add(int index, Object ele)`:在index位置插入ele元素

* `boolean addAll(int index, Collection eles)`:从index位置开始将eles中的所有元素添加进来

* `Object get(int index)`:获取指定index位置的元素

* `int indexOf(Object obj)`:返回obj在集合中首次出现的位置

* `int lastIndexOf(Object obj)`:返回obj在当前集合中末次出现的位置

* `Object remove(int index)`:移除指定index位置的元素，并返回此元素

  > List接口继承了Collection中的方法，包括`remove(Object obj)`方法。`remove(2)`默认认为是索引`2`，`remove(new Integer(2))`才认为删除的是对象。

* `Object set(int index, Object ele)`:设置指定index位置的元素为ele

* `List subList(int fromIndex, int toIndex)`:返回从fromIndex到toIndex位置的子集合

## 2、实现类

`List`存储**有序的、可重复**的数据，常用实现类，三者的对比：

* `ArrayList`：List接口主要实现类，**适合频繁查找**；**线程不安全**，效率高；底层使用**数组**实现：`Object[] elementData`；扩容时默认为原来的1.5倍

* `LinkedList`：**适合频繁插入、删除**操作；**线程不安全**；底层使用**双向链表存储**；


* `Vector`：List接口的早期实现类，与ArrayList几乎相同；**线程安全**，因此效率低；底层同样使用数组实现；与ArrayList另一点不同是扩容时扩大为原来的2倍 (JDK15中，Vector和ArrayList扩容机制相同)。此外，Vector还有一个子类Stack。

## 3、ArrayList

`ArrayList`对象在JDK 7.0和JDK 8.0中的创建过程不同，JDK 7.0中创建`ArrayList`对象类似于单例模式的饿汉式，8.0中类似于单例模式的懒汉式，延迟了数组的创建，节省内存。

1. JDK 7.0中，创建`ArrayList`对象的过程：

   `ArrayList list = new ArrayList();`	初始化时底层创建**长度是10**的Object[]数组elementData
   `list.add(123);`	底层执行elementData[0] = new Integer(123);操作
   ...
   `list.add(11); `	如果此次的添加导致底层elementData数组容量不够，则扩容。
   默认情况下，扩容为原来的容量的`1.5`倍，同时需要将原有数组中的数据复制到新的数组中。

   结论：建议开发中使用带参的构造器：`ArrayList list = new ArrayList(int capacity)`，防止频繁扩容降低效率

2. JDK 8.0中，创建`ArrayList`对象过程：

   `ArrayList list = new ArrayList();`	底层Object[] elementData初始化为{}，并没有创建长度为10的数组

   `list.add(123);`	第一次调用add()时，底层才创建了**长度10**的数组，并将数据123添加到elementData[0]
   ...
   后续的添加和扩容操作与JDK 7 无异。

## 4、LinkedList

`LinkedList`类新增了其特有的方法，方便对双向链表操作：

* `void addFirst(Object obj)`
* `void addLast(Object obj)`
* `Object getFirst()`
* `Object getLast()`
* `Object removeFirst()`
* `Object removeLast()`

LinkedList创建对象的过程：

`LinkedList list = new LinkedList();`	内部声明了`Node`类型的first和last属性，默认值为null
`list.add(123);`	将123封装到Node中，创建了`Node`对象(创建链表节点，add方法用尾插法插入节点)。

其中，`LinkedList`的**双向链表**中的节点用`Node`表示，实现如下：

```java
public class LinkedList<E>...{
    ...
	private static class Node<E> { //内部类：Node，构造每个链表节点
        E item;
        Node<E> next;
        Node<E> prev;
        Node(Node<E> prev, E element, Node<E> next) { //构造器
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    ...
}
```



# 五、Collection子接口二：Set

## 1、Set概述

`Set`接口是`Collection`的子接口，Set接口没有提供额外的方法。

`Set`存储无序、不可重复的数据。添加相同的元素会添加失败。

`Set`根据`equals`判断两个对象是否相同。

`Set`接口的常用实现类有：

* `HashSet`：是Set接口的主要实现类；线程不安全；可以存储null值
  * `LinkedHashSet`：`HashSet`的子类；遍历其内部数据时按照添加的顺序遍历，对于频繁的遍历操作效率高于`HashSet`
* `TreeSet`：按照添加对象的制定属性进行排序。

向Set(主要指：`HashSet`、`LinkedHashSet`)中添加的数据，其所在的类一定要重写`hashCode()`和`equals()`。因为添加数据时要用到两个方法进行判断。

重写的`hashCode()`和`equals()`尽可能保持一致性：相等(equals返回true)的对象必须具有相等的散列码，不相等的对象尽量有不同的散列码。

> 对象中用作 equals() 方法比较的 Field，都应该用来计算 hashCode 值。

重写hashCode方法中，用到`31`这个数字的原因：

* 选择系数的时候要选择尽量大的系数。因为如果计算出来的hash地址越大，哈希冲突出现的次数越少，查找起来效率也会提高。（减少哈希冲突）
* 并且31只占用5bits,相乘造成数据溢出的概率较小。
* 31可以由`i*31== (i<<5)-1`来表示,现在很多虚拟机里面都有做相关优化。（提高算法效率）
* 31是一个素数，素数作用就是如果我用一个数字来乘以这个素数，那么最终出来的结果只能被素数本身和被乘数还有1来整除！(减少冲突)

> 哈希冲突指不同的哈希值(不同的数据)通过哈希函数得到的哈希值(索引位置)是相同的。解决哈希冲突的方法有：开放地址法、链式地址法(HashMap采用的方法)、建立公共溢出区、再哈希法。

## 2、HashSet

`HashSet`底层采用`HashMap`实现，Set就是HashMap中的Key，Value是统一定义的空对象，源代码中的Value：`private static final Object PRESENT = new Object();`

其底层原理和HashMap相同，使用了数组和链表（JDK8中加入了红黑树），具体见HashMap解析。

HashSet添加元素的过程，与HashMap相同，实现原理参考下文的[HashMap](#HashMap)。

## 3、TreeSet

`TreeSet`中添加的数据要求是相同类的对象，其可以根据**自然排序**和**定制排序**两种方式排序。

`TreeSet`中比较两个对象是否相同的标准是`compareTo()`/`compare()`返回0。

`TreeSet`用`TreeMap`实现，底层使用红黑树结构存储数据。

**红黑树**：

红黑树是一种自平衡的二叉查找树。红黑树牺牲部分平衡性，换取插入/删除操作时少量的旋转次数，但是搜索效率下降，总体性能优于AVL树（平衡二叉搜索树）。

红黑树定义：

*  结点是红色或黑色。
* 根节点是黑色。
* 所有叶子节点都是黑色(叶子节点是NIL节点，即空对象)
* 每个红色节点的两个子节点都是黑色。（从每个叶子到根的所有路径上不能有两个连续的红色结点）
* 从任意一个节点到其每个叶子节点的所有路径，包含相同数量的黑色节点。（保证了从根到叶子节点的最长路径不超过最短路径的两倍长，最多是两倍 ）



# 六、Map接口

## 1、概述

`Map`与`Collection`并列存在，用于保存具有**映射关系**的双列数据：`key-value`对。

`Map`结构的理解：

* `key`：使用`Set`存放key，无序，不可重复，key所在类必须重写`hashCode()`和`equals()`方法，因为添加数据时要用到，而Object类中的`hashCode()`没有方法体。
* `value`：用`Collection`存放，无序，可重复，value所在的类要重写`equals()`方法。
* `entry`：一个`key-value`构成一个`entry`，entry构成的集合是`Set`，是无序，不可重复的。
* 两个`key`相同需要保证：**equals方法返回true**并且**hashCode得到的值相等**：
  * 如果`a`和`b`相等，那么`a.equals(b)`一定为`true`，则`a.hashCode()`必须等于`b.hashCode()`；
  * 如果`a`和`b`不相等，那么`a.equals(b)`一定为`false`，则`a.hashCode()`和`b.hashCode()`尽量不要相等。

`Map`接口几个实现类的对比：

* `HashMap`：Map的主要实现类；**线程不安全**，效率高；允许存储null的key和value
  * `LinkedHashMap`：HashMap子类，在原有的HashMap基础上添加了一对指针，指向前一个元素和后一个元素，因此可以按照添加的顺序遍历。对于频繁遍历操作，效率高于HashMap。
* `TreeMap`：保证按照添加的key-value对进行排序，实现排序遍历；根据key进行自然排序或定制排序；底层使用红黑树
* `Hashtable`：原始的Map实现类；**线程安全**，效率低；不能存储null的key和value
  * `Properties`：Hashtable的子类，常用来处理配置文件，其key和value都是String类型。

## 2、Map接口方法

Map接口中定义的方法如下：

添加、删除、修改操作：

* `V put(K key,V value)`：将指定key-value添加到(或修改)当前map对象中，并返回value的值
* `void putAll(Map m)`:将m中的所有key-value对存放到当前map中
* `V remove(Object key)`：移除指定key的key-value对，并返回value
* `boolean remove(Object key, Object value)`：移除指定key和value的key-value对，并返回boolean类型
* `void clear()`：清空当前map中的所有数据

元素查询的操作：

* `V get(Object key)`：获取指定key对应的value
* `V getOrDefault(Object key, V defaultValue)`：如果指定的key值不存在，返回defaultValue
* `boolean containsKey(Object key)`：是否包含指定的key
* `boolean containsValue(Object value)`：是否包含指定的value
* `int size()`：返回map中key-value对的个数
* `boolean isEmpty()`：判断当前map是否为空
* `boolean equals(Object obj)`：判断当前map和参数对象obj是否相等

元视图操作的方法：

* `Set<K> keySet()`：返回所有key构成的Set集合

* `Collection<V> values()`：返回所有value构成的Collection集合

* `Set<Entry<K,V>> entrySet()`：返回所有key-value对构成的Set集合

## 3、HashMap

<span id="HashMap">`HashMap`</span>是Map接口最常用的实现类。在JDK 7.0版本和JDK 8.0版本中，`HashMap`的底层实现原理不同。

`HashMap`中的`Entry数组`，这个数组中可以存储元素的位置称为“`桶(bucket)`”，每个`bucket`都有指定的索引，系统可以根据索引快速访问该`bucket`里存储的元素。

**JDK 7.0中创建HashMap对象的过程**：

`HashMap map = new HashMap();` 实例化，底层创建了**长度是16**的一维数组**Entry[] table**。
`map.put(key1,value1);`执行put操作，首先，调用key1所在类的`hashCode()`和HashMap中的扰动方法`hash()`计算key1哈希值，此哈希值经过某种算法(`int index = (n - 1) & hash;`)计算以后，得到在Entry数组中的存放位置。

* 如果此位置上的数据为空，此时的key1-value1添加成功。 ---->情况1
* 如果此位置上的数据不为空(**哈希冲突/碰撞**。此位置上的存在一个或多个数据以链表形式存在),比较key1和已经存在的一个或多个数据的哈希值（这里比较的是经过`hash()`方法计算得出的哈希值）：
  * 如果key1的哈希值与已经存在的数据的哈希值都不相同，此时key1-value1添加成功。---->情况2
  * 如果key1的哈希值和已经存在的某一个数据(key2-value2)的哈希值相同，继续比较：调用key1所在类的equals(key2)方法：
    * 如果equals()返回false：此时key1-value1添加成功。---->情况3
    *  如果equals()返回true：使用value1替换value2（如果在HashSet中，这种情况会添加失败，但是在Map中则是进行value覆盖）。

对于添加成功的情况2和情况3而言：元素a 与已经存在指定索引位置上数据以链表的方式存储。

**关于HashMap的说明**

* `hashCode()`得到原始的哈希值，Java中又使用了`hash()`方法进行扰动，得到哈希值以后没有直接作为索引(直接将哈希值作为索引，数值太大)，而是使用`与`操作计算在数组中的索引，JDK7和8中的计算索引方法相同：`int index = (table.length - 1) & hash;`

* 扰动函数`hash()`的作用（源码注释：JDK7中为了防止低效的哈希函数，JDK8中是为了解决JDK7中只考虑低位的缺陷），JDK8中，改变了`hash()`的实现，将哈希值的高位和低位混合，加大低位的随机性，从而在获得数组索引时减少冲突（这种冲突是由于计算索引值引起的），因为只算低位的话，就算哈希值不同，也可能得到相同的索引值引起冲突。如果是`hashCode()`计算出不同数据的哈希值相同（哈希冲突），`hash()`方法是解决不了的。`hash()`的具体解析可以参考[链接](https://www.zhihu.com/question/20733617/answer/111577937)。

  ```java
  //JDK 7中的hash方法
  	final int hash(Object k) {
          int h = 0;
          if (useAltHashing) {
              if (k instanceof String) {
                  return sun.misc.Hashing.stringHash32((String) k);
              }
              h = hashSeed;
          }
          h ^= k.hashCode();
          h ^= (h >>> 20) ^ (h >>> 12);
          return h ^ (h >>> 7) ^ (h >>> 4);
      }}
  
  //JDK 8中的hash方法
  	static final int hash(Object key) {
          int h;
          return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
      }//将原始哈希值的高位和其本身异或，减少计算索引时出现的冲突。
  ```

  

* 哈希冲突指的是不同的数据映射到了相同的位置（不同数据的哈希值相同），在HashMap中出现冲突有两个原因，一个是`hashCode()`计算的哈希值相同引起的哈希冲突，另一个是不同哈希值但是经过索引计算映射到了同一个位置。JDK8中hashCode()得到的哈希值相同，则hash()得到的也相同，hashCode()得到的哈希值不同，那么hash()得到的值肯定也不同。

* 所以在添加数据时，如果此位置上已有数据（出现哈希冲突），需要判断哈希值（`hash()`返回值）是否相同，如果哈希值不同却出现在了同一个位置（情况2），是由于Java中计算索引的方法导致的，可以直接添加；如果哈希值相同（hashCode()计算出的哈希值相同引起哈希冲突），这时只能通过`equals()`方法比较是不是相同的数据，如果是相同的数据，则进行覆盖，否则就添加。

  

**JDK 8.0创建HashMap对象与JDK 7.0其他的不同**：

* new HashMap()初始化时，底层没有创建一个长度为16的数组，首次调用put()方法时才创建，这一点类比于ArrayList在两个版本的区别。

* JDK 8.0**底层数组是Node[]**，而非Entry[]，实际上**内部类Node**实现了**内部接口Entry**：

  ```java
  public class HashMap<K,V>...{
      //内部类Node
  	static class Node<K,V> implements Map.Entry<K,V> {
          final int hash;
          final K key;
          V value;
          Node<K,V> next; 
          //Node[]数组中每个位置存储一个Node对象，每个对象的next引用指向下一个元素
          //在JDK 7中，同样，每个Entry[]数组的每个位置存储一个Entry对象，有next指针
  
          Node(int hash, K key, V value, Node<K,V> next) {
              this.hash = hash;
              this.key = key;
              this.value = value;
              this.next = next;
          }
          ... //内部类中的其他方法，比如实现的getKey()、getValue()等
      }
      //内部接口 Entry
      interface Entry<K, V> {
          K getKey();
          V getValue();
          ...
      }
      ...//HashMap中的其他方法
  }
  ```

  

* JDK 7.0底层结构是**数组+链表**，JDK 8.0底层结构是**数组+链表+红黑树**，因此添加到已有元素的位置上时，需要判断是链表节点还是树节点。

* JDK 8.0中，当某个bucket的链表长度`>8`，并且数组长度（容量,capacity,table的长度）`>64`时，才将链表改为红黑树。如果仅是长度>8，数组容量不到64，会进行扩容。

* 如果映射关系被移除，下次resize方法时判断树的节点个数`<6`，会将树转为链表。这里为了避免树和链表频繁转换带来的效率损失，才使用6，而没有直接选择8。

* 形成链表时，和ArrayList相同（七上八下）。JDK 7.0中，冲突时使用头插法将新节点插入到链表，由于是线程不安全的，可能导致出现**环形链表(链表死循环)**。JDK 8.0使用尾插法解决了这一问题。



**HashMap的重要常量**：

* `DEFAULT_INITIAL_CAPACITY`：HashMap的默认容量，`16`
* `DEFAULT_LOAD_FACTOR`：HashMap的默认加载因子：`0.75`
* `threshold`：扩容的临界值 = 容量 x 填充因子，比如`16*0.75=12`
* `TREEIFY_THRESHOLD`：Bucket中链表长度大于改值时，就转化为红黑树：`8`
* `MIN_TREEIFY_CAPACITY`：桶中的Node被树化时最小的hash表容量：`64`
* `MAXIMUM_CAPACITY`：HashMap最大支持容量：$2^{30}$
* `table`：存储元素的数组，总是2的n次幂
* `entrySet`：存储所有entry元素的集合（Set）
* `size`：存储的键值对的数量
* `modCount`：HashMap扩容和结构改变的次数。

> 链表长度大于8并且容量大于64时，才变为红黑树，为什么是8？为什么容量要大于64？
>
> 答：阈值为8是出于时间和空间两方面权衡。哈希值离散性理想情况下，链表长度达到8的几率很小，bin中的节点分布频率服从泊松分布，链表长度大于8的几率小于千万分之一，此情况下如果仍达到了8个节点，说明节点数以后很可能还会继续增加，需要使用红黑树优化查找效率。另一方面，树节点占用空间是普通节点的2倍，如果阈值太小，频繁使用树结构会造成空间浪费。
>
> 关于容量大于64，源码中解释：
>
> ```java
> /*
> The smallest table capacity for which bins may be treeified.
> (Otherwise the table is resized if too many nodes in a bin.)
> Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts 
> between resizing and treeification thresholds.
> */
> ```
>
> 也就是说，为了避免扩容和树化阈值之间的冲突，至少需要4x8=32个bin，如果长度太小，树化以后，如果扩容可能还需要树化，降低了效率。底层数组容量一直是2的n次幂，要想保证32个位置有元素，考虑0.75负载因子，至少需要64的长度。



**HashMap的扩容机制**：

添加数据时，当**当前存放的值（size）超出临界值（threshold）且要存放的位置非空时**会扩容，默认的扩容方式是扩容为原来容量的2倍，然后重新计算旧数组中节点的存储位置并复制过去。由于是扩容为2倍，索引计算方式为哈希值和数组长度-1进行与操作，得到的新数组索引要么是原下标位置，要么是原下标+原数组大小。

JDK 7采用**头插法**将每个bucket上的链表依次取出并放到新数组指定的位置，结果是链表顺序会变反。并且在多线程的情况下，容易导致链表循环。想要线程安全，可以使用`ConcurrentHashMap`

JDK 8采用**尾插法**，解决了链表循环的问题。

**Java的HashMap解决哈希冲突的方法**：

* 使用链地址法，链接相同位置上的数据。
* 使用2次扰动函数（源码中的`hash()`函数），将`hashCode()`得到的哈希值的高位和低位混合，加大低位的随机性，使哈希值映射到数组索引更平均。
* JDK 8.0中又引入红黑树，进一步降低遍历的时间复杂度，使遍历更快



**LinkedHashMap**

`LinkedHashMap`是`HashMap`的子类，其在`HashMap`的存储基础上，使用双向链表记录添加元素的顺序。

与`LinkedHashSet`类似，`LinkedHashMap`可以维护`Map`的迭代顺序，迭代顺序与Key-value对插入顺序一致。

`LinkedHashMap`中的存储结构：

```java
public class LinkedHashMap<K,V> 
    extends HashMap<K,V>
    implements Map<K,V>
{
    ...
	static class Entry<K,V> extends HashMap.Node<K,V> {
        //Entry有两个额外的指针，分别指向上一个和下一个节点
        Entry<K,V> before, after; 
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    ...
}
```

JDK 8中HashMap添加数据的过程源码：

```java
public class HashMap<K,V>...{
    ...
	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果table为空，扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length; 
        //如果目标位置为空，创建新节点并添加
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //如果目标位置有值，进一步判断
        else {
            Node<K,V> e; K k;
            //如果目标位置的值，hash值和数据都相同，进行覆盖
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果目标位置的值和要添加的值不同，则添加。
            //判断是链表节点还是树节点，如果是树节点，调用putTreeVal
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //如果是链表节点，需要依次往下遍历
            else {
                //遍历当前链表
                for (int binCount = 0; ; ++binCount) {
                    //如果到达链表尾部，则新建链表节点，并不马上插入节点
                    //需要判断当前已有节点个数，是否需要树化
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //如果此时链表已经有了8个节点，调用treeifyBin树化
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            //此方法会额外判断table数组长度是否大于64，
                            //如果大于64才构造红黑树，否则执行扩容操作
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果找到和待添加的值相同的数据，则break
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    //如果既没有到达末尾，也没有找到相同数据，则将e赋给p，继续向后找
                    p = e;
                }
            }
            //进行value覆盖
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize(); //判断添加数据以后是否需要扩容
        afterNodeInsertion(evict);
        return null;
    }
    ...
}
```

关于HashMap的常见问题，可以参考[HashMap数据结构相关知识总结](https://www.cnblogs.com/Young111/p/11519952.html)

## 4、TreeMap

保证按照添加的`key-value`对进行排序，实现排序遍历。只考虑`key`的**自然排序或定制排序**。使用自然排序或定制排序的方法比较`key`的大小，而不是`equals()`方法。

底层使用红黑树结构。

## 5、Hashtable

**HashMap 和 Hashtable 的区别**

* HashMap 是线程不安全的，Hashtable 是线程安全的；
* 由于线程安全，所以 Hashtable 的效率比不上 HashMap；
* HashMap最多只允许一条记录的键为null，允许多条记录的值为null，而 Hashtable 不允许key和value的值为null；
* HashMap 默认初始化数组的大小为16，Hashtable 为 11，前者扩容时，扩大两倍，后者扩大两倍+1；
* HashMap 在hashCode基础上需要重新计算 hash 值，而 Hashtable 直接使用对象的 hashCode。

**Properties**

`Properties`是`Hashtable`的子类，用于处理属性文件。

由于属性文件里的`key`、`value`都是字符串类型，所以`Properties`里的`key`和`value`都是字符串类型

存取数据时，建议使用`setProperty(String key,String value)`方法和`getProperty(String key)`方法：

```java
Properties pros = new Properties();
pros.load(new FileInputStream("jdbc.properties"));
String user = pros.getProperty("user");
System.out.println(user);
```



# 七、Collections工具类

和操作数组的工具类`Arrays`类似，`Collections`是操作`Set`、`List`、`Map`等集合的工具类。

`Collections`提供了一系列静态方法，用于对集合元素的排序、查询和修改等操作：

* `reverse(List)`：反转 List 中元素的顺序
* `shuffle(List)`：对 List 集合元素进行随机排序
* `sort(List)`：根据元素的自然顺序对指定 List 集合元素按升序排序
* `sort(List，Comparator)`：根据指定的 Comparator 产生的顺序对 List 集合元素进行排序
* `swap(List，int i， int j)`：将指定 list 集合中的 i 处元素和 j 处元素进行交换
* `Object max(Collection)`：根据元素的自然顺序，返回给定集合中的最大元素
* `Object max(Collection，Comparator)`：根据 Comparator 指定的顺序，返回给定集合中的最大元素
* `Object min(Collection)`
* `Object min(Collection，Comparator)`
* `int frequency(Collection，Object)`：返回指定集合中指定元素的出现次数
* `void copy(List dest,List src)`：将src中的内容复制到dest中
* `boolean replaceAll(List list，Object oldVal，Object newVal)`：使用新值替换 List 对象的所有旧值

此外，`Collections `类还提供了多个` synchronizedXxx()` 方法，该方法可使将指定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题：

* `static <T> Collection<T> synchronizedCollection(Collection<T> c) `
* `static <T> Set<T> synchronizedSet(Set<T> s)`
* `static <T> List<T> synchronizedList(List<T> list)`
* `static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) `
* `static <K,V> NavigableMap<K,V> synchronizedNavigableMap(NavigableMap<K,V> m)`
* `static <T> NavigableSet<T> synchronizedNavigableSet(NavigableSet<T> s)`
* `static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m) `
* `static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s) `

