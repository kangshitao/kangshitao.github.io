---
title: Java学习笔记12-泛型
excerpt: 泛型的使用，自定义泛型，泛型通配符
mathjax: true
date: 2021-04-11 12:30:29
tags: Java
categories: Java
keywords: Java,泛型
---

# 一、泛型的引入

JDK 5.0引入了**泛型(Genetic)**。JDK 5.0之前的集合容器类在声明阶段无法确定容器内存的什么类型数据，JDK 5.0中将元素类型设计为一个参数，这个类型参数就是泛型。例如`Collection<E>`、`List<E>`、`ArrayList<E>`中的`<E>`就是类型参数，即泛型。

**定义**：泛型就是允许在定义类、接口时通过一个标识表示类中某个属性的类型或者是某个方法的返回值及参数类型。这个类型参数将在使用时（例如继承或实现这个接口、用这个类型声明变量、创建对象时）确定（即传入实际的类型参数，也称为类型实参）。

如果没有泛型，任何类型都可以添加到集合中，类型不安全，且读取出来的对象需要强制转换，可能会有`ClassCastException`异常。使用泛型可以在编译时就检查，只有指定类型才可以添加到集合中。

# 二、在集合中使用泛型



集合接口或集合类在jdk5.0时都修改为带泛型的结构。

在实例化集合类时，可以指明具体的泛型类型。指明完以后，在集合类或接口中凡是定义类或接口时，内部结构（比如：方法、构造器、属性等）使用到类的泛型的位置，都指定为实例化的泛型类型。比如：`add(E e)`，实例化以后：`add(Integer e)`

注意点：

* **泛型的类型必须是类，不能是基本数据类型。需要用到基本数据类型的位置，用包装类替换**。
* 如果实例化时没有指明泛型的类型，默认类型为`java.lang.Object`类型。
* 以`List`为例，`List`实际上表示**持有任何Object类型的原生List**，而`List<?>`表示**具有某种特定类型的非原生List，只是我们不知道哪种类型是什么**

集合中使用泛型的例子：

```java
public class GenericTest {
    //在集合中使用泛型,以ArrayList为例
    @Test
    public void test2(){
        //指定泛型类型，只能添加Integer类型
        ArrayList<Integer> list =  new ArrayList<Integer>();
        //也可以写成以下形式，JDK 7.0新增的类型推断功能
        //ArrayList<Integer> list =  new ArrayList<>();
        list.add(78);
        list.add(87);
        //编译时，就会进行类型检查，保证数据的安全
        //添加的类型不是Integer时，编译时就会报错
        //list.add("Tom"); 
        
        //迭代器中使用泛型
    	Interator<Integer> iterator = list.iterator();
        while(iterator.hasNext()){
            //不需要判断类型，一定是Integer
            int num = iterator.next();
            System.out.println(num);
        }
    }
}
```

泛型的嵌套使用：

```java
public class GenericTest {
    //在集合中使用泛型的情况：以HashMap为例
    @Test
    public void test3(){
		//Map<String,Integer> map = new HashMap<String,Integer>();
        //类型推断
        Map<String,Integer> map = new HashMap<>();
        map.put("Tom",87);
        map.put("Jerry",87);
        map.put("Jack",67);
        
        //泛型的嵌套
        Set<Map.Entry<String,Integer>> entry = map.entrySet();
        Iterator<Map.Entry<String, Integer>> iterator = entry.iterator();
        while(iterator.hasNext()){
            Map.Entry<String, Integer> e = iterator.next();
            String key = e.getKey();
            Integer value = e.getValue();
            System.out.println(key + "=" + value);
        }
    }
}
```



# 三、自定义泛型结构

可以在类、接口以及方法上使用泛型，分别为泛型类、泛型接口、泛型方法。

## 1、泛型类与泛型接口

* 泛型类可以有多个参数，此时应将多个参数一起放在尖括号内，比如`class Test<E1,E2,E3>`。

* 泛型类的构造器不能有尖括号结构，和一般类的构造器一样。

* 类实例化以后，操作原来泛型位置的结构必须与指定的泛型类型一致。

* 泛型不同的引用不能相互赋值，比如`List<Object> list = new ArrayList<String>();`是错误的，提示无法类型转换。

* 泛型如果不指定，将被擦除，泛型对应的类型均按照`Object`处理，但不等价于`Object`。建议如果使用泛型就一直使用泛型，如果不使用就都不使用。泛型擦除后，编译不会进行类型检查。

  ```java
  public class Test {
      public static void main(String[] args) {
          //使用时：类似于Object，但不等同于Object
          ArrayList list = new ArrayList();
          list.add(33);
          test(list); //泛型擦除，编译不会类型检查
          
          //一旦指定Object，编译会类型检查，必须按照Object处理
          ArrayList<Object> list2 = new ArrayList<Object>();
          //test(list2); //会报错，类型必须相同
      }
      public static void test(ArrayList<Integer> list) { }
  }
  ```

* 如果泛型结构是接口或抽象类，则不可创建泛型类的对象。

* 在类/接口上声明的泛型，在本类或本接口中即代表某种类型，可以作为**非静态属性的类型**、**非静态方法的参数类型**、**非静态方法的返回值类型**。但在**静态方法中不能使用类的泛型**，因为类的泛型是在实例化时指定的，静态方法不需要通过对象调用。

* **异常类不能是泛型的**，try-catch中不能用泛型。

* 不能实例化泛型，即不能`new E[]`，可以先声明`Object`类型的数组，然后强制转换：`E[] elements = (E[])new Object[capacity];`参考`ArrayList`源码中的声明`Object[] elementData;`，而非泛型参数类型数组。

  ```java
  public class Order<T> {
      public Order(){
          T[] arr = new T[10]; //错误，编译不通过
          T[] arr = (T[]) new Object[10]; //正确，编译通过
      }
  }
  ```

* 父类有泛型，子类可以选择**指定泛型类型**或者**保留泛型**：
  
  * 子类不保留父类的泛型：按需实现
    * 如果没有类型，则进行擦除
    * 指定具体类型
  * 子类保留父类的泛型：泛型子类。泛型子类在实例化时可以指定泛型类型
    * 全部保留
    * 部分保留
  
  > 结论：子类必须在保留或者指定二者之间做出选择。此外，子类除了指定或保留父类的泛型，还可以额外增加自己的泛型。
  
  比如下面的例子：
  
  ```java
  class Father<T1,T2>{}
  //子类不保留父类泛型
  class Son1 extends Father{} //1.没有类型，擦除
  //2.指定类具体类型。这时的Son2类不是泛型子类，只是个普通的类。
  class Son2 extends Father<Integer,String>{} 
  
  //子类保留父类泛型
  //Son3和Son4都是泛型子类，因为它们都有泛型
  class Son3<T> extends Father<T1,T2>{} //1.全部保留。子类可以有自己的泛型
  class Son4 extends Father<Integer,T2>{} //2.部分保留
  ```
  
  

## 2、泛型方法

方法中也可以使用泛型，泛型方法中可以定义泛型参数，此时参数的类型就是传入数据的类型。

泛型方法的泛型与类的泛型无关，即泛型方法所在的类是不是泛型类没有关系。

只使用类的泛型的方法不是泛型方法。

**泛型方法可以是静态的**，因为静态方法在调用的时候会指定泛型方法的类型参数。

泛型方法的格式：

`权限符 <泛型> 返回类型 方法名（泛型 参数1，...） `

比如：

```java
public class Test {
    //返回值前面的<E>泛型必须要有。参数传入的泛型类型就是E的类型
    public static <E> List<E> copyFromArrayToList(E[] arr){ //泛型方法
        //传入的String类型，此时方法中用到泛型的地方都是String类型
        ArrayList<E> list = new ArrayList<>(Arrays.asList(arr));
        System.out.println(list);
        return list;
    }
    @Test
    public void test(){
        String[] a = new String[]{"AA","BB","CC"};
        copyFromArrayToList(a); //传入的String类型
    }
}
```



# 四、泛型在继承上的体现

如果类`A`是类`B`的父类，那么`class <A>` 和`class <B>`二者不具备子父类关系，二者是并列关系。

比如`ArrayList<Object> list1`和`ArrayList<String> list2`，那么`list1 = list2`是错误的。

如果`A`是类`B`的父类，`A<E> `是 `B<E> `的父类。

举例：

```java
public class GenericTest {
    @Test
    public void test1(){
        Object obj = null;
        String str = null;
        obj = str;  //正确，String是Object的子类
        
        List<Object> list1 = null;
        List<String> list2 = new ArrayList<String>();
        //此时的list1和list2的类型不具有子父类关系
        list1 = list2;  //错误，编译不通过
        /*反证法：
        假设list1 = list2正确
        那么list1.add(123)会导致混入非String的数据，出错。
         */
    }

    @Test
    public void test2(){
        List<String> list1 = null;
        ArrayList<String> list2 = null;
        //ArrayList是List的实现类，可以赋值，但是必须保证泛型参数相同
        list1 = list2;  
    }
}
```



# 五、通配符

通配符`?`代表具体的类型参数，例如`List<?>`是`List<String>`、`List<Object>`等各种泛型`List`的父类。

假设现在有`List<?>`的一个对象`list`，那么可以安全**读取**`list`中的元素，因为一定是`Object`类型的。不可以向`list`中**写入**元素，因为不知道`list`中元素的类型，但是可以写入`null`，其他都都不可以。

代码举例：

```java
public class Test{
	@Test
    public void test3(){
        List<Object> list1 = null;
        List<String> list2 = null;
        List<?> list = new ArrayList<>(); //list是list1和list2的父类
        
        list = list1;//正确
        list = list2;//正确

        //添加(写入)：对于List<?>不能向其内部添加数据。
        //除了添加null之外。
        list.add("DD"); //错误
        list.add(null); //正确

        //获取(读取)：允许读取数据，读取的数据类型为Object。
        Object o = list.get(0); //可以读取数据，类型是Object
        System.out.println(o);
    }
}
```

注意点：

* `public static <?> void test(ArrayList<?> list){}`编译错误，不能用在泛型方法声明上，返回值类型前面`<>`不能使用`?`通配符。返回值必须是确定的类型。
* `class GenericTypeClass<?>{}`编译错误，`?`不能用在泛型类的声明上。
* `ArrayList<?> list2 = new ArrayList<?>();`编译错误，`?`不能用在创建对象上，右边属于创建集合对象，必须是确定的类型。

**有限制的通配符**

* `? extends A`：理解为<=A类，类型限定了只能是A类或者是A类的子类。
  * `  class<? extends A> `可以作为`class<A>`和`class<B>`的父类，其中B是A的子类
* `? super A`：理解为>=A类，类型限定了只能是A类或者是A类的父类。
  * `class<? super A> `可以作为`class<A>`和`class<B>`的父类，其中B是A的父类
* 对于接口，`<? extends Comparable>`表示只允许泛型为实现`Comparable`接口的实现类的引用调用。

使用举例：

```java
public class GenericTest {
    @Test
    public void test4(){
        List<? extends Person> list1 = null; //只能是Person或Person的子类
        List<? super Person> list2 = null; //只能是Person或Person的父类
        
        List<Student> list3 = new ArrayList<Student>();
        List<Person> list4 = new ArrayList<Person>(); //Person是Student的父类
        List<Object> list5 = new ArrayList<Object>();

        list1 = list3;	//正确
        list1 = list4;  //正确
        list1 = list5;  //错误

        list2 = list3;  //错误
        list2 = list4;	//正确
        list2 = list5;	//正确

        //读取数据：
        list1 = list3;
        Person p = list1.get(0); //要用最大类型接收
        //编译不通过，因为如果是Person类型，不能赋给Student引用
        //Student s = list1.get(0);

        list2 = list4;
        Object obj = list2.get(0); //用Object类型接收
        //编译不通过，因为如果是Person的父类，不能赋给Person引用
		//Person obj = list2.get(0);

        //写入数据：
        //编译不通过，extends不能写
		//list1.add(new Student()); 

        //编译通过
        //super可以写
        list2.add(new Person());
        list2.add(new Student());
    }
}
```

总结：

* 使用`extends`通配符只能读，不能写(只能写`null`)，类似于`?`通配符。因为有上限，返回的任意类型都能够向上转型到确定的类型，但是不允许使用`set`方法传入引用(`null`除外)。
* 使用`super`通配符可以写，也可以读。写的时候可以安全地向上转型，读的时候，只能使用`Object`类型接收，因为`Object`是所有类的根父类，可以向上转型为`Object`。