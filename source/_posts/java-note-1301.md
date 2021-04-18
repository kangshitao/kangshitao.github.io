---
title: Java学习笔记13-IO流
excerpt: IO流,节点流,缓冲流,转换流,字符集,字符编码,打印流,数据流,序列化,反序列化
mathjax: true
date: 2021-04-16 20:51:33
tags: Java
categories: Java
keywords: Java,IO流,节点流,缓冲流,转换流,字符集,字符编码,序列化,反序列化
---

# 一、File类

## 1、File类的使用

`java.io.File`类的一个对象，表示一个**文件**或一个**文件目录**，与平台无关。

`File`能新建、删除、重命名文件和目录，但不能访问文件内容本身，访问文件内容需要使用输入输出流。

Java中表示一个真实存在的文件或目录，必须有一个`File`对象，但Java程序的一个`File`对象可能没有一个真实存在的文件或目录。

`File`对象可以作为参数传递给流的构造器。

## 2、构造File类对象

`File`类的构造器有四种：

* `public File(String pathname)`：以pathname为路径创建File对象，可以是绝对路径和相对路径。
* `public File(String parent,String child)`：以parent为父路径，child为子路径创建File对象。
* `public File(File parentFile,String childPath)`：根据父File对象和子文件路径创建File对象。
* `public File(URI uri)`：根据给定的url创建File对象。

> 1. IDEA中，main方法中默认的相对路径是当前project下，Test方法则是默认当前module。
> 2. 路径中每级目录之间用路径分隔符隔开，Windows和Dos系统默认采用`\`表示，UNIX和URL使用`/`表示分隔符。
> 3. 为了支持跨平台，File类提供了常量`public static final String separator`，它能根据不同系统动态提供分隔符。

代码示例：

```java
public class FileTest {
    @Test
    public void test1(){
        //构造器1
        File file1 = new File("hello.txt");  
        File file2 =  new File("D:Java\\JavaSenior\\day08\\hello.txt");

        System.out.println(file1);  // hello.txt
        System.out.println(file2);  // D:Java\JavaSenior\day08\hello.txt
        //构造器2：
        File file3 = new File("D:Java","JavaSenior");
        System.out.println(file3); // D:Java\JavaSenior

        //构造器3：
        File file4 = new File(file3,"hi.txt");
        System.out.println(file4); // D:Java\JavaSenior\hi.txt
    }
}
```

## 3、File类中的方法

`File`类中的常用方法：

File类的**获取**功能：

* `public String getAbsolutePath()`：获取绝对路径
* `public String getPath()`：获取路径
* `public String getName()`：获取名称
* `public String getParent()`：获取上层文件目录路径。若无，返回null
* `public long length()`：获取文件长度（即：字节数）。不能获取目录的长度。
* `public long lastModified()`：获取最后一次的修改时间，毫秒值
* `public String[] list()`：获取**指定目录**下的所有文件或者文件目录的名称数组
* `public File[] listFiles()`：获取**指定目录**下的所有文件或者文件目录的File数组

FIle类的**重命名**功能：

* `public boolean renameTo(File dest)`：把当前文件重命名为指定的文件路径，要求当前文件必须存在的且目标dest不存在。

File类的**判断**功能：

* `public boolean isDirectory()`：判断是否是文件目录
* `public boolean isFile()`：判断是否是文件
* `public boolean exists()`：判断是否存在
* `public boolean canRead()`：判断是否可读
* `public boolean canWrite()`：判断是否可写
* `public boolean isHidden()`：判断是否隐藏

File类的**创建**功能：

* `public boolean createNewFile()`：创建文件。若文件存在，则不创建，返回false
* `public boolean mkdir()`：创建文件目录。如果此文件目录存在，就不创建。如果此文件目录的上层目录不存在，也不创建。
* `public boolean mkdirs()`：创建文件目录。如果此文件目录存在，就不创建。如果上层文件目录不存在，一并创建。

File类的**删除**功能：

* `public boolean delete()`：删除文件或者文件目录

  > Java中的删除不经过回收站。
  >
  > 如果要删除的文件目录不为空，则删除失败。

# 二、IO流的原理及分类

## 1、IO流概述

Java中，对于数据的输入/输出操作以`流(stream)`的方式进行。

**流的分类**：

* 根据操作的**数据单位**不同，分为：`字节流（8 bit）`，`字符流（16 bit）`

* 根据流的**流向**不同，分为：`输入流`、`输出流`
* 根据流的**角色**不同，分为：`节点流`、`处理流`

IO流的四个基类，其他类都是继承自这四个抽象类：

| （抽象基类） | 字节流         | 字符流   |
| ------------ | -------------- | -------- |
| 输入流       | `InputStream`  | `Reader` |
| 输出流       | `OutputStream` | `Writer` |

IO流的类层次图：

<div align='center'>
    <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1301_1.svg"/>
</div>

**节点流**：直接从数据源或目的地读写数据。

上图中的`FileInputStream`/`FileOutputStream`/`FileReader`/`FileWrider`都是节点流。

**处理流**：不直接连接到数据源或目的地，而是“连接”在已存在的流（节点流或处理流）之上，通过对数据的处理为程序提供更强大的读写功能。比如缓冲流、打印流等。

## 2、IO流基类

`InputStream`和`Reader`是所有输入流的基类，分别用于字节流和字符流。二者的典型实现类分别为`FileInputStream`和`FileReader`，使用`FileInputStream`和`FileReader`从文件系统中的某个文件中获得输入字节/字符。

**InputStream**中的方法：

* `int read()`：从输入流中读取数据的下一个字节。返回0-255内的int字节值，如果到达流末尾，返回-1
* `int read(byte[] b)`：从输入流中最多将b.length个字节的数据读入byte数组中。如果到达流末尾，返回-1，否则以整数形式返回实际读取的字节数。
* `int read(byte[] b, int off, int len)`：读取最多len个字节数据，存放到数组b中off以后的位置。如果到达流末尾，返回-1
* `public void close() throws IOException`：关闭此输入流，并释放与该流关联的所有系统资源。

**Reader**中的方法：

* `int read()`：读取单个字符，作为整数读取的字符，范围在0-65535之间（0x00-0xff，2个字节的Unicode码），如果到达流末尾，返回-1
* `int read(char[] c)`
* `int read(char[] c, int off, int len)`
* `public void close() throws IOException`

`OutputStream`和`Writer`是所有输出流的基类，其典型实现类分别是`FileOutputStream`和`Writer`，用于写出数据。

**OutputStream**中的方法：

* `void write(int b)`：将单个字节b写入指定的输出文件
* `void write(byte[] b)`：将字节数组b中的数据写出到文件
* `void write(byte[] b,int off, int len)`：将数组中从off开始的长度为len的所有字节写出到文件
* `void flush()`：刷新
* `void close()`：关闭

**Writer**中的方法：

* `void write(int c)`：将单个字符c写入指定的输出文件
* `void write(char[] c)`：将字符数组c中的数据写出到文件
* `void write(char[] c,int off, int len)`：将数组中从off开始的长度为len的所有字符写出到文件
* `void flush()`：刷新
* `void close()`：关闭

> IO流部署于内存里的资源，GC无法回收该资源，操作结束后，必须要**显式关闭文件IO资源**。

# 三、节点流（文件流）

`FileInputStream`/`FileOutputStream`/`FileReader`/`FileWrider`都是节点流，用于读入/读出数据。

节点流的构造器，以`FileInputStream`为例，其余都相似：

* `public FileInputStream(File file)`：根据File对象file创建输入节点流
* `public FileInputStream(String name)`：根据name创建输入节点流，其实底层还是根据字符串创建了File对象，再使用第一个构造器。



读入/读出数据的大体步骤：

* 创建File类对象，指明要操作的文件
* 提供具体的节点流
* 调用`read`/`write`方法`读入`/`写出`数据
* 关闭流

应用实例，将文件1的内容读取到控制台，然后复制到文件2：

```java 
public class FileInputOutputStreamTest{
    FileInputStream fis = null;
    FileOutputStream fos = null;
    try{
        //1&2. 创建File对象，创建文件流
        fis = new FileInputStream(new File("hello1.txt"));
        fos = new FileOutputStream(new File("hello2.txt"));
        //3.进行读写操作
        byte[] buffer = new byte[1024];
        int len;
        while((len = fis.read(buffer)) != -1){
            fos.write(buffer,0,len); //这里必须是长度为len，读多少写多少
        }
    }catch(IOException e){
        e.printStackTrace();
    }finally{ //4.关闭操作。为保证关闭操作一定会执行，写在finally中
        if(fos != null){
            try{
                fos.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
        if(fis != null){
            try{
                fis.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
}
```

> 1. 字符流只能处理字符，操作普通文本文件(`.txt`,`.java`,`.c`,`.cpp`等)，`.doc`不是文本文件。
>
> 2. 字节流可以操作普通文本文件，也可以操作`.mp3`,`.avi`,`.rmvb`,`.mp4`,`.jpg`,`.doc`,`.ppt`等。
> 3. 使用`read()`读取文件时，必须保证文件存在。
> 4. `FileWriter`/`FileOutputStream`有两个构造器，一个参数的构造器，默认对原有文件进行覆盖。两个参数的构造器，第二个参数是`append`，用于判断是进行追加还是覆盖。如果文件不存在，则自动创建。
> 5. 字符流底层也是用字节流实现的。

# 四、缓冲流

缓冲流是为了提高读写速度，在使用时创建内部缓冲区（默认缓冲区大小为8192字节，8KB）的一种处理流。缓冲流要“套接”在节点流之上才能使用。

根据数据操作单位不同，有两种输入输出缓冲流：

* `BufferedInputStream`/`BufferedOutputStream`
* `BufferedReader`/`BufferedWriter`

下面以 `BufferedInputStream`/`BufferedOutputStream`为例，说明缓冲流的原理及使用方法。

`BufferedInputStream`读取数据时，先一次性读取8KB数据存入缓冲区，然后读操作从缓冲区中获取数据。

`BufferedOutputStream`写入数据时，先写到缓冲区中，将缓冲区写满，然后write操作从缓冲区中写到文件中。使用`flush()`可以强制将缓冲区的内容全部写入输出流。

关闭流时，只需要关闭最外层即可，最外层关闭时会自动将节点流关闭。

缓冲流关闭时会自动执行`flush`操作。

使用缓冲流读入/读出数据的大体步骤：

* 创建File类对象，指明要操作的文件
* 提供具体的节点流
* 创建缓冲流
* 调用`read`/`write`方法`读入`/`写出`数据
* 关闭缓冲流

代码实例，使用缓冲流复制视频：

```java
public class FileInputOutputStreamTest{
    BufferedInputStream bis = null;
    BufferedOutputStream bos = null;
    try{
        //1&2. 创建File对象，创建文件流
        FileInputStream fis = new FileInputStream(new File("视频1.mp4"));
        FileOutputStream fos = new FileOutputStream(new File("视频2.mp4"));
        //3.创建缓冲流
        //默认缓冲区大小是8192Byte，可以指定大小
        bis = new BufferedInputStream(fis); 
        bos = new BufferedOutputStream(fos);
        //4.进行读写操作
        byte[] buffer = new byte[1024];
        int len;
        /*
        读取的时候，先一次性将缓冲区存满，然后每次read操作从缓冲区读取1024字节到数组buffer中。
        写入的时候，先将缓冲区写满，然后再根据字节数组，每次将1024字节写入文件。
        */
        while((len = bis.read(buffer)) != -1){
            bos.write(buffer,0,len);
        }
    }catch(IOException e){
        e.printStackTrace();
    }finally{ //5.关闭操作。只需要关闭最外层即可，且会自动调用flush方法。
        if(bos != null){
            try{
                bos.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
        if(bis != null){
            try{
                bis.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
}
```



# 五、转换流

## 1、转换流概述

**转换流**用于字节流和字符流之间的转换，Java中有两个转换流：

* `InputStreamReader`：将InputStream转换为Reader，字节、字节数组→字符数组、字符串，相当于解码。有以下常用构造器：
  * `public InputStreamReader(InputStream in)`：使用默认字符集对`in`进行解码
  * `public InputStreamReader(InputStream in, String charsetName)`：使用指定字符集解码
* `OutputStreamWriter`：将Writer转换为OutputStream，字符数组、字符串→字节、字节数组，相当于编码。有构造器：
  * `public OutputStreamWriter(OutputStream out)`：使用默认字符集（字符编码）对`out`进行编码
  * `public OutputStreamWriter(OutputStream out, String charsetName)`：使用指定字符集（字符编码）编码

> 1. 字符流底层操作是基于字节流，使用转换流进行转换。过程为：文本1-->字节流-->InputStreamReader-->字符流-->程序-->字符流-->OutputStreamWriter-->字节流-->文本2
> 2. 对于字符数据，使用字符流操作更高效。
> 3. 转换流通常用于处理文件乱码问题，实现编码和解码操作。
> 4. 未解决疑问：JDK源码中将UTF-8描述为字符集(charset)，好像没有区分字符集和字符编码的概念？Java中字符编码器是什么东西？

具体用法：

```java
//输入
FileInputStream fis = new FileInputStream("dbcp1.txt");
InputStreamReader isr = new InputStreamReader(fis,"UTF-8");
//输出
FileOutputStream fos = new FileOutputStream("dbcp2.txt");
OutputStreamWriter osw = new OutputStreamWriter(fos,"gbk");
```

## 2、字符编码

* **字符集（Charset）**：字符集指的是一个系统支持的所有抽象字符的集合。字符集中的每个字符的位置叫做**码位（Code Point）**，码位上的值为**码位值（Code point value）**。**字符集就是把抽象字符映射为码位值**。常见的字符集有GB2312、GBK、GB18030、Big5、Unicode、ASCII等字符集。

* **字符编码（Character Encoding）**：字符编码是一种映射规则，根据这个映射规则可以将某个字符映射成其他形式的数据（比如比特模式、自然数序列、8位组、电脉冲等），以便在计算机中存储和传输。简而言之就是字符编码是字符集和实际存储数值之间的转换关系。

> 1. 总结：字符集建立起数字和字符之间的索引关系。某个字符在计算机中怎么表示，具体占用几个字符等，这就需要编码规则来解决，即字符编码，它规定了字符如何编码成二进制，存储在计算机中。
> 2. 直接使用字符集中的码位值存储，会造成空间浪费，比如本来ASCII字符只需要一个字节，如果直接使用字符集编码，需要三个字节，因此出现了UTF-8等变长编码，其对于ASCII字符仍然使用一个字节存储。
> 3. Unicode出现之前，几乎所有的字符集和其编码方式都是绑定的，可以说字符集就是编码方式。
> 4. Unicode字符集有多种编码方式，即UTF-8、UTF-16、UTF-32等。



**问题一，关于ASCII码、ANSI和ISO8859-XXX：**

**ASCII**码是最早出现的字符集，包括128个字符，字符集表如下：

<div align='center'>
    <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1301_2.png"/>
</div>



**ANSI**是一种字符代码，表示**扩展的ASCII编码**。对于`0x00~0x7F`的128个字符，与ASCII码一致，超过此范围的使用`0x80~0xFFFF`来编码。这种**使用多个字节来代表一个字符的各种汉字延伸编码的方式，称为ANSI编码**，比如GB2312、GBK、GB18030、Big5、Shift_JIS等。

* 在英文的Windows操作系统中，ANSI编码表示ISO-8859-1编码(Latin-1，西欧常用字符，包括德法两国的字母)。
* 在简体中文的Windows操作系统中，ANSI编码表示GBK编码。
* 在繁体中文的Windows操作系统中，ANSI编码表示Big5编码。
* 在日文的Windows操作系统中，ANSI编码表示Shift_JIS编码

欧洲国家对ASCII码进行扩充，只扩充128-255这一段，0-127与ASCII码相同，形成了在欧洲国家的一些子标准，比如**ISO8859-1字符集**、**ISO8859-2字符集**等，具体可以参考[链接](https://baike.baidu.com/item/ISO8859)。



**问题二，关于GBxxxx编码：**

**GB2312**（或GB2312-80）是最早出现的汉字编码，收录6763个汉字，由于其不能处理一些罕见字，出现了**GBK**编码，兼容GB2312，并且包括了**BIG5**的所有汉字。

2000年发布了**GB18030**字符集，采用单字节、双字节、四字节三种编码方式，完全兼容ASCII码和GBK码。2005年又进行了修订与扩展，推出了**GB18030-2005**，共收录70244个汉字。



**问题三，Unicode和UTF-8的关系：**

结论：**UTF-8是Unicode的实现方式之一**。

**Unicode是字符集**，为每个字符规定了唯一的编号。实现它的方式有多种，比如UTF-8、UTF-16、UTF-32等字符编码都是Unicode的实现方式，Unicode字符集具体存储成什么样的字节流，取决于字符编码方案。

**UTF-8是一种变长的编码方式**，每次8个位传输数据，使用1-4个字节表示一个符号，根据不同的符号而变化字节长度 。UTF-16是每次16个位传输数据，具体可参考[链接1](https://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html)和[链接2](https://zhuanlan.zhihu.com/p/260192496)。

UTF-8的编码方式，参考[链接](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)：

<div align='center'>
    <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1301_3.png"/>
</div>



# 六、标准输入输出流

标准输入输入流，分别代表系统标准的输入和输出设备，默认输入设备是键盘，默认输出设备是显示器：

* `System.in`：标准的输入流，默认从键盘输入，类型是InputStream，其源码为：`public static final InputStream in`.

* `System.out`：标准的输出流，默认从控制台输出，类型是PrintStream，其源码为：`public static final PrintStream out`.

`System`类的`set`方法可以重新指定输入和输出的流：

* `public static void setIn(InputStream in)`
* `public static void setOut(PrintStream out)`

例题，使用标准输入输出流，从键盘输入字符，将读取到的整行字符串转出大写输出，并继续输出，直到输入“exit”时退出程序：

```java
public class SystemStreamTest {
    /*
    方法一：使用Scanner实现，调用next()返回一个字符串
    方法二：使用System.in实现。
    		System.in  --->  转换流 ---> BufferedReader的readLine()
     */
    public static void main(String[] args) {
        BufferedReader br = null;
        try {
            //System.in是InputStream类型
            InputStreamReader isr = new InputStreamReader(System.in);
            br = new BufferedReader(isr);
            while (true) {
                System.out.println("请输入字符串：");
                String data = br.readLine();
                if ("exit".equalsIgnoreCase(data)) {
                    System.out.println("程序结束");
                    break;
                }
                String upperCase = data.toUpperCase();
                System.out.println(upperCase);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                try { br.close();} 
                catch (IOException e) {e.printStackTrace();}
            }
        }
    }
}
```



# 七、打印流

打印流`PrintStream`和`PrintWriter`提供了一系列重载的`print()`和`println()`方法，用于多种数据类型的输出。

`PrintStream`和`PrintWriter`的输出不会抛出异常。

`PrintStream`和`PrintWriter`有自动flush功能。

`PrintStream`打印的所有字符使用平台默认的字符编码转换为字节，在需要写入字符的情况下，应该使用`PrintWriter`类。

`System.out`返回的是`PrintStream`的实例。

代码实例，输出ASCII字符：

```java
public class PrintStreamTest{
    public void test2() {
        PrintStream ps = null;
        try {
            FileOutputStream fos =
                new FileOutputStream(new File("D:\\IO\\text.txt"));
            // 创建打印输出流,设置为自动刷新模式(写入换行符或字节 '\n' 时都会刷新输出缓冲区)
            ps = new PrintStream(fos, true);
            if (ps != null) {// 把标准输出流(控制台输出)改成文件
                System.setOut(ps);
            }
            for (int i = 0; i <= 255; i++) { // 输出ASCII字符
                System.out.print((char) i);
                // 每50个数据一行
                if (i % 50 == 0)  System.out.println();
            }
        }
        catch (FileNotFoundException e) {e.printStackTrace();} 
        finally {if (ps != null) {ps.close();}}
    }
}
```



# 八、数据流

数据流用于方便地操作Java语言的基本数据类型和String类型。

数据流有两个类：

* `DataInputStream`：套接在InputStream子类的流上。其包含的方法如下
  * `boolean readBoolean()`
  * `char readChar()`
  * `double readDouble()`
  * `long readLong()`
  * `String readUTF()`
  * `byte readByte()`
  * `float readFloat()`
  * `short readShort()`
  * `int readInt()`
  * `void readFully(byte[] b)`
* `DataOutputStream`：套接在OutputStream子类的流上。其中的方法与DataInputStream中一致，只需要将read改为write即可。

应用实例：

```java
public class OtherStreamTest {
    /*
    将内存中的字符串、基本数据类型的变量写出到文件中
    这里处理异常应该使用try-catch-finally.
     */
    @Test
    public void test3() throws IOException {
        //1.
        DataOutputStream dos = 
            new DataOutputStream(new FileOutputStream("data.txt"));
        //2.
        dos.writeUTF("数据结构");
        dos.flush(); //刷新操作，将内存中的数据写入文件，每次写完都要刷新
        dos.writeInt(23);
        dos.flush();
        dos.writeBoolean(true);
        dos.flush();
        //3.
        dos.close();

    }
    /*
    将文件中存储的基本数据类型变量和字符串读取到内存中，保存在变量中。
    注意点：读取不同类型的数据的顺序要与当初写入文件时，保存的数据的顺序一致！
     */
    @Test
    public void test4() throws IOException {
        //1.
        DataInputStream dis = 
            new DataInputStream(new FileInputStream("data.txt"));
        //2.
        String name = dis.readUTF();
        int age = dis.readInt();
        boolean isMale = dis.readBoolean();
        System.out.println("name = " + name);
        System.out.println("age = " + age);
        System.out.println("isMale = " + isMale);
        //3.
        dis.close();
    }
}
```



# 九、对象流

## 1、对象流概述

对象流`ObjectInputStream`和`ObjectOutputStream`，用于存储和读取基本类型数据或对象的处理流。可以把Java中的对象写入到数据中（序列化），或者把对象从数据源中还原回来（反序列化）。

* **序列化**：用`ObjectOutputStream`类保存基本数据类型或对象的机制。
* **反序列化**：用`ObjectInputStream`类读取基本数据类型或对象的机制。



## 2、序列化和反序列化

**transient关键字**

transient意为短暂的、临时的。用transient关键字修饰的变量无法被序列化。

同样，用static关键字修饰的变量也无法被序列化。

**对象序列化机制**

对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。当其它程序获取了这种二进制流，就可以恢复成原来的Java对象。

**序列化**就是将内存中的java对象保存到磁盘中或通过网络传输出去。而**反序列化**就是将磁盘中的对象还原成内存中的一个java对象。

序列化的好处在于**可将任何实现了Serializable接口的对象转化为字节数据，使其在保存和传输时可被还原**。

序列化是RMI（Remote Method Invoke–远程方法调用）过程的参数和返回值都必须实现的机制，RMI是JavaEE的基础，因此序列化机制是JavaEE平台的基础。

如果需要让某个对象支持序列化机制，则必须让**对象所属的类及其属性**是可序列化的，为了让某个类是可序列化的，该类必须实现如下两个接口之一。否则，会抛出`NotSerializableException`异常

* Serializable
* Externalizable

> 实现Serializable接口的类，都有一个表示序列化版本标识符的静态变量：
>
> `private static final long serialVersionUID`
>
> 其用来表明类的不同版本之间的兼容性。目的是以序列化对象进行版本控制，有关各版本反序列化时是否兼容。
>
> 如果实现类没有显式定义此变量，它的值是Java运行时环境根据类的内部细节自动生成的。若类的实例变量做了修改，`serialVersionUID`可能发生变化，导致无法反序列化。建议显式声明此常量。
>
> 反序列化时，JVM会把传来的字节流的`serialVersionUID`与本地相应实体类的`serialVersionUID`进行比较，如果相同就认为是一致的，可以进行反序列化，否则会出现异常`InvalidCastException`



## 3、使用对象流序列化对象

序列化步骤：

* 创建`ObjectOutputStream`对象，记为oos
* 调用oos的`writeObject(对象)`方法，输出可序列化对象
* 每写出一次，就需要`flush`一次。

反序列化步骤：

* 创建`ObjectInputStream`对象，记为ois
* 调用ois的`readObject()`方法，读取流中的对象

> 可序列化的类中的属性，如果是引用类型，其必须也是可序列化的。String类型是可序列化的。

实现代码：

```java
//序列化，将对象变为二进制流
ObjectOutputStream oos = new ObjectOutputStream(
    new FileOutputStream("data.txt"));
Personp = newPerson("韩梅梅", 18, "中华大街", newPet());
oos.writeObject(p);
oos.flush();
oos.close();

//反序列化，将磁盘中数据读出并还原为对象
ObjectInputStream ois = new ObjectInputStream(
    new FileInputStream("data.txt"));
Personp1 = (Person)ois.readObject();
System.out.println(p1.toString());
ois.close();                                                              
```



**关于java.io.Serializable接口的理解**

* `Serializable`接口是空方法接口，这样的接口称为标志接口。实现了Serializable接口的对象，可将它们转换成一系列字节，并可在以后完全恢复回原来的样子。**这一过程亦可通过网络进行。这意味着序列化机制能自动补偿操作系统间的差异**。换句话说，可以先在Windows机器上创建一个对象，对其序列化，然后通过网络发给一台Unix机器，然后在那里准确无误地重新“装配”。不必关心数据在不同机器上如何表示，也不必关心字节的顺序或者其他任何细节。
* 由于大部分作为参数的类如`String`、`Integer`等都实现了`java.io.Serializable`的接口，也可以利用多态的性质，作为参数使接口更灵活。

# 十、随机存取文件流

`RandomAccessFile`实现了`DataInput`和`DataOutput`这两个接口，意味着这个类既可以读也可以写。

`RandomAccessFile`包含一个记录指针，用来标识当前读写所处的位置，也就是说，类支持“随机访问”的方式，可以从文件的任意地方读、写文件。这个指针可以自由移动：

* `long getFilePointer()`：获取文件记录指针的当前位置
* `void seek(long pos)`：将文件记录指针定位到pos位置

`RandomAccessFile`的构造器：

* `public RandomAccessFile(File file,String mode)`：mode参数用于指定访问模式
* `public RandomAccessFile(String name, String mode)`

mode有四种值：

* `r`：以只读方式打开。如果文件不存在，则报异常。
* `rw`：以读写方式打开，可读可写。如果文件不存在，会自动创建文件。
* `rwd`：可读可写，并且同步文件内容的更新
* `rws`：可读可写，同步文件内容和元数据的更新

`RandomAccessFile`对象写数据，是以对目标文件覆盖的形式写入，默认指针从头开始，将内容逐个覆盖。可以通过指针实现文件内容插入的操作。

实现内容插入效果：

```java
public class RandomAccessFileTest {
    /*
    使用RandomAccessFile实现数据的插入效果,从第3个位置插入数据
     */
    @Test
    public void test3() throws IOException {
        RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");
        raf1.seek(3);//将指针调到角标为3的位置
        //保存指针3后面的所有数据到StringBuilder中
        //也可以将StringBuilder替换为ByteArrayOutputStream
        StringBuilder builder = new StringBuilder(
            (int) new File("hello.txt").length());
        byte[] buffer = new byte[20];
        int len;
        while((len = raf1.read(buffer)) != -1){
            builder.append(new String(buffer,0,len)) ;
        }
        //调回指针，写入“xyz”
        raf1.seek(3);
        raf1.write("xyz".getBytes());
        //将StringBuilder中的数据写入到文件中
        raf1.write(builder.toString().getBytes());
        raf1.close();
    }
}
```



# 十一、NIO.2中Path/Paths/Files类的使用

JDK1.4版本提出NIO（New IO，Non-Blocking IO），以更高效的方式进行文件的读写操作。

JDK1.7对NIO进行了扩展，称之为NIO.2。早期的File类功能有限，大部分方法出错时仅返回失败，不会提供异常信息。NIO.2引入了Path接口，并提供了Files、Paths工具类，都声明在`java.nio.file`包下。

Path接口可以看成File类的升级版本，表示一个文件或目录，文件或目录可以不存在。

使用Path接口创建对象：`Path path = Paths.get("hello.txt");`

Paths工具类，提供静态的get()方法用来获取Path对象：

* `static Pathget(String first, String … more) `：用于将多个字符串串连成路径
* `static Path get(URI uri)`：返回指定uri对应的Path路径



**Path接口常用方法**

* `String toString()`：返回调用Path对象的字符串表示形式
* `boolean startsWith(String path) `:判断是否以path路径开始
* `boolean endsWith(String path) `:判断是否以path路径结束
* `boolean isAbsolute() `:判断是否是绝对路径
* `Path getParent()`：返回Path对象包含整个路径，不包含Path对象指定的文件路径
* `Path getRoot()`：返回调用Path对象的根路径
* `Path getFileName() `:返回与调用Path对象关联的文件名
* `int getNameCount()` :返回Path根目录后面元素的数量
* `Path getName(int idx) `:返回指定索引位置idx的路径名称
* `Path toAbsolutePath() `:作为绝对路径返回调用Path对象
* `Path resolve(Path p) `:合并两个路径，返回合并后的路径对应的Path对象
* `File toFile()`:将Path转化为File类的对象

**Files工具类常用方法**

* `Path copy(Path src, Path dest, CopyOption … how) `:文件的复制
* `Path createDirectory(Path path, FileAttribute<?> … attr)` :创建一个目录
* `Path createFile(Path path, FileAttribute<?> … arr) `:创建一个文件
* `void delete(Path path)` :删除一个文件/目录，如果不存在，执行报错
* `void deleteIfExists(Path path) `: Path对应的文件/目录如果存在，执行删除
* `Path move(Path src, Path dest, CopyOption…how) `:将src移动到dest位置
* `long size(Path path)` :返回path指定文件的大小

用于判断：

* `boolean exists(Path path, LinkOption … opts)` :判断文件是否存在
* `boolean isDirectory(Path path, LinkOption … opts) `:判断是否是目录
* `boolean isRegularFile(Path path, LinkOption … opts)` :判断是否是文件
* `boolean isHidden(Path path) `:判断是否是隐藏文件
* `boolean isReadable(Path path) `:判断文件是否可读
* `boolean isWritable(Path path)` :判断文件是否可写
* `boolean notExists(Path path, LinkOption … opts) `:判断文件是否不存在

用于操作内容：

* `SeekableByteChannel newByteChannel(Path path, OpenOption…how) `:获取与指定文件的连接，how指定打开方式。
* `DirectoryStream<Path>  newDirectoryStream(Path path)` :打开path指定的目录
* `InputStream newInputStream(Path path, OpenOption…how)`:获取InputStream对象
* `OutputStream newOutputStream(Path path, OpenOption…how) `:获取OutputStream对象