---
title: Java学习笔记14-网络编程
excerpt: 网络编程概述，IP和端口号，网络协议，TCP/UDP网络编程，URL编程
mathjax: true
date: 2021-04-17 23:00:21
tags: Java
categories: Java
keywords: Java,网络编程概述,IP和端口号,网络协议,TCP/UDP网络编程,URL编程
---

# 一、网络编程概述

Java提供了网络类库，可以轻松实现网络连接，底层细节隐藏在Java的本机安装系统里，由JVM控制。Java实现了一个跨平台的网络库，程序员面对的是一个统一的网络编程环境。

网络编程的目的：直接或间接地通过网络协议与其它计算机实现数据交换，进行通讯。

实现网络编程面对的两个问题：

* 如何准确定位某台主机，如何定位主机上的特定应用。
  * 解决方法：使用IP地址定位主机，端口号定位特定应用
* 找到主机后如何可靠高效地进行数据传输。
  * 解决方法：提供网络通信协议。

# 二、网络通信两个要素

想要实现网络中的主机互相通信，需要满足两个条件：

* 已知通信双方的地址，包括**IP和端口号**。
* 一定的规则，即网络通信协议。有OSI和TCP/IP两个参考模型 ，OSI参考模型过于理想化，主要是使用**TCP/IP参考模型（TCP/IP协议）**。

<table>
	<tr>
		<td><font color="red"><b>OSI参考模型</b></font></td>
		<td><font color="red"><b>TCP/IP参考模型</b></font></td>
        <td><font color="red"><b>TCP/IP参考模型各层对应协议</b></font></td>
	</tr>
	<tr>
        <td>应用层</td>
        <td rowspan="3">应用层</td>
		<td rowspan="3">HTTP、FTP、Telnet、DNS...</td>
	</tr>
	<tr>
		<td>表示层</td>
	</tr>
    <tr>
		<td>会话层</td>
	</tr>
    <tr>
		<td>传输层</td>
        <td>传输层</td>
        <td>TCP、UDP...</td>
	</tr>
    <tr>
		<td>网络层</td>
        <td>网络层</td>
        <td>IP、ICMP、ARP...</td>
	</tr>
    <tr>
		<td>数据链路层</td>
        <td rowspan="2">物理+数据链路层</td>
        <td rowspan="2">以太网、无线LAN...</td>
	</tr>
    <tr>
        <td>物理层</td>
    </tr>
</table>





## 1、IP和端口号

**IP地址**

IP：唯一的标识Internet上的计算机（通信实体），Java中使用`InetAddress`类代表IP。

Internet上的主机有两种方式表示地址：

* 域名(hostName，主机名)：比如 www.baidu.com。其中本地主机名用localhost表示
* IP地址(hostAddress)：比如 182.61.200.7。本地地址用127.0.0.1表示

`InetAddress`没有公共的构造器，提供了两个静态方法用于实例化：

* `public static InetAddress getLocalHost()`：返回localhost地址的InetAddress对象
* `public static InetAddress getByName(String host)`：返回指定主机地址的InetAddress对象，host参数可以是域名或者ip地址。

`InetAddress`常用的方法：

* `public String getHostName()`：返回当前主机的域名(主机名)
* `public String getHostAddress()`：返回当前主机的ip地址
* `public boolean isReachable(int timeout)`：测试指定时间内是否可以到达当前地址

应用实例：

```java
public class InetAddressTest {
    @Test
    public void test() throws IOException {
        InetAddress inet1 = InetAddress.getByName("www.baidu.com");
        System.out.println(inet1.getHostAddress()); // 182.61.200.7
        System.out.println(inet1.getHostName()); // www.baidu.com

        InetAddress inet2 = InetAddress.getByName("182.61.200.7");
        System.out.println(inet2.getHostAddress()); // 182.61.200.7
        System.out.println(inet2.getHostName()); // 182.61.200.7
        
        System.out.println(inet1.isReachable(500)); //true
        System.out.println(inet2.isReachable(500)); //true
    }
}
```



**端口号**

端口号用来标识正在计算机上运行的程序（进程）。**不同的进程有不同的端口号**，端口号用一个16位的整数表示，范围是`0-65535`

端口分类：

* 公认端口：`0-1023`。被预先定义的服务通信占用，如HTTP占用端口80，FTP占用端口21，Telnet占用端口23等。
* 注册端口：`1024-49151`，分配给用户进程或应用程序。比如Tomcat占用端口8080，MySQL占用端口3306，Oracle占用端口1521等。
* 动态/私有端口：`49152-65535`，即`1100 0000 0000 0000-1111 1111 1111 1111`

端口号和IP地址的组合，得到**网络套接字：Socket**。

## 2、网络通信协议

网络通信协议对速率、传输代码、代码结构、传输控制步骤、出错控制等制定标准。

以**TCP/IP协议**为例。TCP/IP协议将网络协议分成了**物理链路层**、**网络层**、**传输层**和**应用层**四层，其中传输层有两个非常重要的协议：

* **TCP**(Transmission Control Protocol)：传输控制协议。
  * 使用TCP协议前，须先建立TCP连接，形成传输数据通道。
  * 传输前，采用“三次握手”方式，点对点通信，**是可靠的**。
  * TCP协议进行通信的两个应用进程：客户端、服务端。
  * 在连接中可**进行大数据量的传输**。
  * 传输完毕，**需释放已建立的连接，效率低**。
* **UDP**(User Datagram Protocol)：用户数据报协议。
  * 将数据、源、目的封装成数据包，**不需要建立连接**。
  * 每个数据报的大小限制在64K内。
  * 发送不管对方是否准备好，接收方收到也不确认，**是不可靠的**。
  * 可以广播发送。
  * 发送数据结束时**无需释放资源，开销小，速度快**。

TCP的三次握手和四次挥手：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1401_1.png'/>
</div>

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1401_2.png'/>
</div>






**Socket**

网络上具有唯一标识的**IP地址**和**端口号**组合在一起，构成唯一能识别的标识符**套接字(Socket)**。

通信的两端都要有Socket，是两台机器间通信的端点。

网络通信其实就是Socket间的通信。

Socket允许程序把网络连接当成一个流，数据在两个Socket间通过IO传输。

一般主动发起通信的应用程序属**客户端**，等待通信请求的为**服务端**。

**Socket分类**：

* 流套接字（stream socket）：使用**TCP**提供**可依赖**的字节流服务。
* 数据报套接字（datagram socket）：使用**UDP**提供**“尽力而为”**的数据报服务。



**`Socket`类的常用构造器**：

* `public Socket(InetAddress address, int port)`：创建一个流套接字并将其连接到指定IP地址的指定端口号。
* `public Socket(String host, int port)`：创建一个流套接字并将其连接到指定主机上的指定端口号。

**`Socket`类的常用方法**：

* `public InputStream getInputStream()`：返回此套接字的输入流。可以用于接收网络消息
* `public OutputStream getOutputStream()`：返回此套接字的输出流。可以用于发送网络消息
* `public InetAddress getInetAddress()`：此套接字连接到的远程IP地址；如果套接字是未连接的，则返回null。
* `public InetAddress getLocalAddress()`：获取套接字绑定的本地地址。即本端的IP地址
* `public int getPort()`：此套接字连接到的远程端口号；如果尚未连接套接字，则返回0。
* `public int getLocalPort()`：返回此套接字绑定到的本地端口。如果尚未绑定套接字，则返回-1。即本端的端口号。
* `public void close()`：关闭此套接字。套接字被关闭后，便不可在以后的网络连接中使用（即无法重新连接或重新绑定）。需要创建新的套接字对象。关闭此套接字也将会关闭该套接字的InputStream和OutputStream。
* `public void shutdownInput()`：如果在套接字上调用shutdownInput()后从套接字输入流读取内容，则流将返回EOF（文件结束符）。即不能在从此套接字的输入流中接收任何数据。
* `public void shutdownOutput()`：禁用此套接字的输出流。对于TCP套接字，任何以前写入的数据都将被发送，并且后跟TCP的正常连接终止序列。如果在套接字上调用shutdownOutput()后写入套接字输出流，则该流将抛出IOException。即不能通过此套接字的输出流发送任何数据。



# 三、TCP网络编程

## 1、TCP通信流程

**TCP连接的套接字函数**：

<img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1401_3.png' style = "transform:scale(0.8)"/>

基于Socket的TCP编程，基本步骤：

**客户端**：

* **创建Socket**：根据指定服务端的IP地址或端口号构造Socket类对象（使用Socket类的两个构造器方法）。若服务器端响应，则建立客户端到服务器的通信线路。若连接失败，会出现异常。
* **打开连接到Socket的输入/输出流**：用getInputStream()方法获得输入流，使用getOutputStream()方法获得输出流，进行数据传输。
* **按照一定的协议对Socket进行读/写操作**：通过输入流读取服务器放入线路的信息（但不能读取自己放入线路的信息），通过输出流将信息写入线程。
* **关闭Socket**：断开客户端到服务器的连接，释放线路。

> 客户端建立socketAtClient对象的过程就是向服务器发出套接字连接请求。
>
> 客户端可以自定义，可以是浏览器

**服务器**：

* **调用ServerSocket（int port）**：创建一个服务端套接字，并绑定到指定端口上，用于监听客户端的请求。
* **调用accept（）**：监听连接请求，如果客户端请求连接，则接受连接。此方法返回通信套接字对象。
* **调用返回的Socket类对象的getOutputStream()和getInputStream()**：获取输出流和输入流，开始网络数据的发送和接收。
* **关闭ServerSocket和Socket对象**：客户端访问结束，关闭通信套接字。

> ServerSocket对象负责等待客户端请求建立套接字连接。服务器必须实现建立一个等待客户请求建立socket连接的ServerSocket对象。
>
> accept接收客户的socket请求，并返回一个socket对象。
>
> 服务端可以自定义，可以是Tomcat服务器

## 2、TCP网络编程案例

应用举例1，客户端发送文件给服务端， 服务端保存到本地，并返回“接收成功”给客户端：

```java
public class TCPTest3 {
    @Test
    public void client() throws IOException {
        //1.创建Socket，指定要连接的主机地址和端口号
        Socket socket = new Socket(InetAddress.getByName("127.0.0.1"),9090);
        //2.打开连接到socket的输出流
        OutputStream os = socket.getOutputStream();
        //3.建立文件流
        FileInputStream fis = new FileInputStream(new File("beauty.jpg"));
        //4.写数据
        byte[] buffer = new byte[1024];
        int len;
        while((len = fis.read(buffer)) != -1){
            os.write(buffer,0,len);
        }
        //写完数据以后要关闭数据输出流，不然会一直处于阻塞状态
        socket.shutdownOutput(); 
        //5.接收来自于服务器端的数据，并显示到控制台上
        InputStream is = socket.getInputStream();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] bufferr = new byte[20];
        int len1;
        while((len1 = is.read(buffer)) != -1){
            baos.write(buffer,0,len1);
        }

        System.out.println(baos.toString());
        //6.关闭流
        fis.close();
        os.close();
        socket.close();
        is.close();
        baos.close();
    }
    @Test
    public void server() throws IOException {
        //1.创建ServerSocket对象，设置端口为9090
        ServerSocket ss = new ServerSocket(9090);
        //2.调用accept监听连接请求，连接成功后才会执行下面的代码
        Socket socket = ss.accept();
        //3.获取输入流
        InputStream is = socket.getInputStream();
        //4.构建节点流
        FileOutputStream fos = new FileOutputStream(new File("beauty2.jpg"));
        //5.将输入流内容写到文件中
        byte[] buffer = new byte[1024];
        int len;
        while((len = is.read(buffer)) != -1){
            fos.write(buffer,0,len);
        }

        System.out.println("文件传输完成");
        //6.服务器端给予客户端反馈
        OutputStream os = socket.getOutputStream();
        os.write("我是服务器，文件已收到".getBytes());

        //7.关闭连接。这里需要使用try-catch捕获异常。
        fos.close();
        is.close();
        socket.close();
        ss.close();
        os.close();
    }
}
```

应用举例2，客户端向服务器发送信息，服务器返回接受成功，直到输入“exit"断开连接：

```java
/*
Client.java文件
*/
package com.atguigu.Exer;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetAddress;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

public class Client {
    public static void main(String[] args) {
        new Client().start();
    }
    public void start() {
        Socket socket = null;
        OutputStream os = null;
        Scanner sc = null;
        ByteArrayOutputStream baos = null;
        while (true) {
            try {
                //1.
                socket = new Socket(InetAddress.getByName("127.0.0.1"), 9999);
                //2.
                os = socket.getOutputStream();
                //3.
                System.out.println("Input message：");
                sc = new Scanner(System.in);
                String message = sc.next();
                os.write(message.getBytes(StandardCharsets.UTF_8));
                socket.shutdownOutput();//写完数据以后要关闭数据输出流
                //接收来自服务器的消息
                InputStream is = socket.getInputStream();

                baos = new ByteArrayOutputStream();
                int len = 0;
                while ((len = is.read()) != -1) {
                    baos.write(len);
                }
                System.out.println(baos.toString());

                if ("exit".equals(message)) break;
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (os != null) {
                    try {
                        os.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (socket != null) {
                    try {
                        socket.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}

/*
Server.java文件
*/
package com.atguigu.Exer;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static void main(String[] args) {
        new Server().start();
    }
    public void start() {
        ServerSocket ss = null;
        Socket socket = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;
        try {
            ss = new ServerSocket(9999);
            while (true) {
                socket = ss.accept();
                is = socket.getInputStream();
                int len = 0;
                baos = new ByteArrayOutputStream();
                while ((len = is.read()) != -1) {
                    baos.write(len);
                }
                String message = baos.toString();
                System.out.print("From " + socket.getInetAddress().getHostName() 
                                 + ": ");
                System.out.println(message);
                socket.getOutputStream().write(
                    "-----Message received-----".getBytes());
                socket.shutdownOutput();//写完数据以后要关闭数据输出流
                if ("exit".equals(message)) break;
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (ss != null) {
                try { ss.close(); }
                catch (IOException e) { e.printStackTrace(); }
            }
            if (socket != null) {
                try { socket.close(); }
                catch (IOException e) { e.printStackTrace(); }
            }
            if (is != null) {
                try { is.close(); }
                catch (IOException e) { e.printStackTrace(); }
            }
            if (baos != null) {
                try { baos.close(); }
                catch (IOException e) { e.printStackTrace(); }
            }
        }
    }
}
```



# 四、UDP网络编程

## 1、UDP通信流程

**UDP连接的套接字函数**：

<img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/java-note-1401_4.png' style = "transform:scale(0.8)"/>

类`DatagramSocket`和`DatagramPacket`实现了基于UDP协议网络程序。

`DatagramSocket`，数据报套接字，负责发送和接收UDP数据报(数据包)，系统不保证UDP数据报一定能够安全送到目的地，也不能确定什么时候可以抵达。

`DatagramPacket`对象封装了UDP数据报(数据包)，在数据报中包含了发送端的IP地址和端口号以及接收端的IP地址和端口号。

UDP协议中每个数据报都给出了完整的地址信息，因此无须建立发送方和接收方的连接。如同发快递包裹一样。

**DatagramSocket类的常用方法**：

* `public DatagramSocket(int port)`：创建数据报套接字并将其绑定到本地主机上的指定端口。套接字将被绑定到通配符地址，IP地址由内核来选择。
* `public DatagramSocket(int port,InetAddress laddr)`：创建数据报套接字，将其绑定到指定的本地地址。本地端口必须在0到65535之间（包括两者）。如果IP地址为0.0.0.0，套接字将被绑定到通配符地址，IP地址由内核选择。
* `public void close()`：关闭此数据报套接字。
* `public void send(DatagramPacket p)`：从此套接字发送数据报包。DatagramPacket包含的信息指示：将要发送的数据、其长度、远程主机的IP地址和远程主机的端口号。
* `public void receive(DatagramPacket p)`：从此套接字接收数据报包。当此方法返回时，DatagramPacket的缓冲区填充了接收的数据。数据报包也包含发送方的IP地址和发送方机器上的端口号。此方法在接收到数据报前一直阻塞。数据报包对象的length字段包含所接收信息的长度。如果信息比包的长度长，该信息将被截短。
* `public InetAddress getLocalAddress()`：获取套接字绑定的本地地址。
* `public int getLocalPort()`：返回此套接字绑定的本地主机上的端口号。
* `public InetAddress getInetAddress()`：返回此套接字连接的地址。如果套接字未连接，则返回null。
* `public int getPort()`：返回此套接字的端口。如果套接字未连接，则返回-1。

**DatagramPacket类的常用方法**：

* `public DatagramPacket(byte[] buf,int length)`：构造DatagramPacket，用来接收长度为length的数据包。length参数必须小于等于buf.length。
* `public DatagramPacket(byte[] buf,int length,InetAddress address,int port)`：构造数据报包，用来将长度为length的包发送到指定主机上的指定端口号。length参数必须小于等于buf.length。
* `public InetAddress getAddress()`：返回某台机器的IP地址，此数据报将要发往该机器或者是从该机器接收到的。
* `public int getPort()`：返回某台远程主机的端口号，此数据报将要发往该主机或者是从该主机接收到的。
* `public byte[] getData()`：返回数据缓冲区。接收到的或将要发送的数据从缓冲区中的偏移量offset处开始，持续length长度。
* `public int getLength()`：返回将要发送或接收到的数据的长度。

**UDP网络通信步骤**：

由于UDP通信不需要建立连接，因此客户端和服务端步骤相同。

* 使用DatagramSocket创建socket。
* 使用DatagramPacket建立数据包，将要发送/接收的数据、IP地址、端口号封装到数据包中。
* 调用socket的发送、接收方法。
* 关闭socket。



## 2、UDP网络编程案例

应用实例，发送端向接收端发送信息：

```java
public class UDPTest {

    //发送端
    @Test  //为了看着简洁，使用throws处理异常。正确方式是try-catch
    public void sender() throws IOException {
        //1.创建数据包套接字对象
        DatagramSocket socket = new DatagramSocket();
        String str = "我是UDP发送端";
        byte[] data = str.getBytes();
        InetAddress inet = InetAddress.getLocalHost();
        //2.将要发送的数据以及IP和端口号封装到数据包中
        DatagramPacket packet = new DatagramPacket(data,0,data.length,inet,9090);
        //3.发送数据
        socket.send(packet);
        //4.关闭socket，这里同样应该使用try-catch处理异常。
        socket.close();
    }
    //接收端
    @Test
    public void receiver() throws IOException {
        DatagramSocket socket = new DatagramSocket(9090);
        byte[] buffer = new byte[100];
        DatagramPacket packet = new DatagramPacket(buffer,0,buffer.length);
        //将接收的数据存入数据包
        socket.receive(packet);
        System.out.println(new String(packet.getData(),0,packet.getLength()));
        socket.close();
    }
}
```



# 五、URL编程

## 1、URL介绍

**URL（Uniform Resource Locator）**：统一资源定位符，表示Internet上某一资源的地址。

通过url可以访问Internet上的各种网络资源，浏览器通过解析给定的url，在网络上查找相应的文件或其他资源。

URL基本结构有5部分：`<传输协议>://<主机名>:<端口号>/<文件名>#片段名?参数列表`。

例如：`http://127.0.0.1:8080/helloworld/index.jsp#a?username=admin&password=123`

> **#片段名**：即锚点，例如小说定位到章节。
>
> **参数列表格式**：参数名=参数值&参数名=参数值...

## 2、URL类

`java.net.URL`类用于表示URL，其有四种构造器：

* `public URL (String spec)`：通过一个表示URL地址的字符串可以构造一个URL对象。
  * 例如：`URL url= new URL ("http://www.baidu.com/");`
* `public URL(URL context, String spec)`：通过基URL和相对URL构造一个URL对象。
  * 例如：`URL downloadUrl= new URL(url, “download.html");`
* `public URL(String protocol, String host, String file)`
  * 例如：`newURL("http","www.baidu.com",“download.html");`
* `public URL(String protocol, String host,intport, String file);`
  * 例如:`URL gamelan = newURL("http","www.baidu.com", 80,“download.html");`

> URL类的构造器都声明抛出非运行时异常，必须对这种异常进行处理，通常使用try-catch语句捕获。



**URL类的常用方法**：

* `public String getProtocol()`：获取该URL的协议名
* `public String getHost()`：获取该URL的主机名
* `public String getPort()`：获取该URL的端口号
* `public String getPath()`：获取该URL的文件路径
* `public String getFile()`：获取该URL的文件名
* `publicString getQuery()`：获取该URL的查询名

## 3、URL编程

URL的方法`openStream()`能从网络上读取数据。

若希望输出数据，例如向服务器端的CGI（公共网关接口-Common GatewayInterface-的简称，是用户浏览器和服务器端的应用程序进行连接的接口）程序发送一些数据，则必须先与URL建立连接，然后才能对其进行读写，此时需要使用`URLConnection`类。

`URLConnection`是抽象类，`HttpURLConnection`抽象类继承了`URLConnection`，`HttpsURLConnectionImpl`继承了`HttpURLConnection`抽象类。

`URLConnection`表示到URL所引用的远程对象的连接。当与一个URL建立连接时，首先要在一个URL对象上通过方法`openConnection()`生成对应的`URLConnection`对象。如果连接过程失败，将产生IOException.

* `URL url = new URL ("http://www.baidu.com/index.html");`
* `URLConnectonn u = url.openConnection( );`

通过`URLConnection`对象获取的输入流和输出流，可以与现有的CGI程序进行交互。

* `public Object getContent() throws IOException`
* `public int getContentLength( )`
* `public String getContentType( )`
* `public long getDate( )`
* `public long getLastModified( )`
* `public InputStream getInputStream( ) throws IOException`
* `public OutputSteram getOutputStream( ) throws IOException`

应用实例，使用URL编程下载网络资源文件：

```java
public class URLTest1 {

    public static void main(String[] args) {

        HttpURLConnection urlConnection = null;
        InputStream is = null;
        FileOutputStream fos = null;
        try {
            //1.构造URL对象
            URL url = new URL("https://www.baidu.com/img/
                              PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png");
            //2.生成URLConnection对象，其实是HttpsURLConnectionImpl对象
            urlConnection = (HttpURLConnection) url.openConnection();
            //3.建立连接
            urlConnection.connect();
            //4.获取并下载资源
            is = urlConnection.getInputStream();
            fos = new FileOutputStream("D:\\BaiduLogo.jpg");
                            
            byte[] buffer = new byte[1024];
            int len;
            while((len = is.read(buffer)) != -1){
                fos.write(buffer,0,len);
            }
            System.out.println("下载完成");
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源
            if(is != null){
                try {is.close();} 
                catch (IOException e) {e.printStackTrace();}
            }
            if(fos != null){
                try {fos.close();} 
                catch (IOException e) {e.printStackTrace();}
            }
            if(urlConnection != null){urlConnection.disconnect();}
        }
    }
}
```

## 4、URI、URL和URN的区别

**URI(uniform resource identifier)**，统一资源标识符，用来唯一的标识一个资源。

**URL(uniform resource locator)**，统一资源定位符，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。

**URN(uniform resource name)**，统一资源命名，通过名字来标识资源，比如`mailto:java-net@java.sun.com`。

也就是说，URI是以一种抽象的，高层次概念定义统一资源标识，而**URL和URN则是具体的资源标识的方式，URL和URN都是一种URI**。

在Java的URI中，一个URI实例可以代表绝对的，也可以是相对的，只要符合URI语法规则即可。而URL类不仅符合语义，还包含了定位该资源的信息，因此它不能是相对的。