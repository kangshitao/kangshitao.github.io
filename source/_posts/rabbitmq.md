---
title: RabbitMQ学习笔记
excerpt: RabbitMQ工作模式，Exchange、Queue基本概念，发布确认机制，死信队列、延迟队列，RabbitMQ集群简介
mathjax: false
date: 2021-10-26 22:16:25
tags: ['消息队列','RabbitMQ']
categories: 消息队列
keywords: RabbitMQ,MQ,Exchange,Queue,DLX
---



# 一、消息队列

## 1.1 MQ相关概念

### 1.1.1 什么是MQ

消息队列（MQ，Message Queue）本质是一个队列，队列中存放的内容是message，是一种跨进程的通信机制，用于上下游传递消息。

使用MQ以后，消息发送上游只需要依赖MQ，不用依赖其他服务。

### 1.1.2 MQ的作用

* 流量削峰

* 应用解耦

* 异步处理

### 1.1.3 MQ分类

常见的MQ有以下几种：

* ActiveMQ：高吞吐量场景较少使用。
* Kafka：为大数据而生，百万级TPS，吞吐量高，在日志领域比较成熟。适合有日志采集需求的大型企业。
* RocketMQ：出自阿里巴巴，单机吞吐量十万级，消息0丢失，支持10亿级别的消息堆积。适合金融互联网。
* RabbitMQ：由Erlang语言开发，在AMQP（高级消息队列协议）基础上完成，当前最流行的MQ。吞吐量万级，支持多种语言。适合数据量不是特别大的中小型公司。



## 1.2 RabbitMQ

### 1.2.1 核心概念

**RabbitMQ**

RabbitMQ是一个消息中间件，它接受、存储、转发消息数据。

**生产者（Producer）**

产生数据、发送消息的程序是生产者。

**交换机（Exchange）**

交换机是RabbitMQ的一个重要组件。它一方面接收来自生产者的消息，另一方面将消息推送到队列中。交换机决定了将消息推送到特定队列还是推送到多个队列。

**队列（Queue）**

队列是RabbitMQ内部使用的一种数据结构。消息只能存储在队列中，队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者也可以尝试从一个队列接收数据。

**消费者（Consumer）**

消费者指的是等待接受消息的程序。同一个应用程序既可以是生产者也可以是消费者。



### 1.2.2 安装

RabbitMQ官方文档：[Docs](https://www.rabbitmq.com/documentation.html)

RabbitMQ基于Erlang环境，因此需要先安装Erlang。

安装之前需要确保RabbitMQ和Erlang的版本要对应：[RabbitMQ Erlang Version Requirements](https://www.rabbitmq.com/which-erlang.html)

本机环境：

* CentOS-7.9
* Erlang-23.2.3
* RabbitMQ-3.8.15

使用`rpm`方式，在packagecloud网站下载安装包。

**1、安装Erlang**

下载对应版本的安装包，[packagecloud](https://packagecloud.io/rabbitmq/erlang?page=1)：

```bash
wget --content-disposition https://packagecloud.io/rabbitmq/erlang/packages/el/7/erlang-23.2.3-1.el7.x86_64.rpm/download.rpm
```

安装：

```bash
rpm -ivh erlang-23.2.3-1.el7.x86_64.rpm
```

安装完后，输入`erl`进入Erlang的命令行界面，安装成功。



**2、安装socat**

除了Erlang环境，还需要安装socat：

```bash
yum install socat logrotate -y
```



**3、安装RabbitMQ**

下载对应版本的安装包，[packagecloud](https://packagecloud.io/rabbitmq/rabbitmq-server?page=1)：

```bash
wget --content-disposition https://packagecloud.io/rabbitmq/rabbitmq-server/packages/el/8/rabbitmq-server-3.8.15-1.el8.noarch.rpm/download.rpm
```

安装：

```bash
rpm -ivh rabbitmq-server-3.8.15-1.el8.noarch.rpm
```

安装完成后，启动rabbit服务：

```bash
systemctl start rabbitmq-server.service
```

**4、安装插件**

可以通过以下命令开启web管理插件，需要先停止rabbitmq服务：

```bash
rabbitmq-plugins enable rabbitmq_management
```

插件开启成功后就可以在浏览器中访问，默认端口号为`15672`（记得关闭防火墙或者开放端口）。

管理界面需要账号密码登陆，默认的账号和密码都是`guest`。

查看当前所有用户：

```bash
rabbitmqctl list_users
```



添加新用户`admin`，密码也为`admin`：

```bash
rabbitmqctl add_user admin admin
```



设置用户角色（标签），将`admin`设置为`administrator`

```bash
rabbitmqctl set_user_tags admin administrator
```



设置用户权限：

```bash
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
```





### 1.2.3 原理和工作模式

**工作原理**

RabbitMQ的工作原理如图所示：

![RabbitMQ工作原理](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_1.png)



生产者将消息通过Channel发送到Exchange，Exchange决定将消息分发到哪个队列，然后由消费者从队列中接收消息。

其中的核心概念：

* **Broker**：接收和分发消息的应用，RabbitMQ Server就是Message Broker
* **Virtual host**：虚拟的分组，当多个不同的用户使用同一个RabbitMQ server提供的服务时，可以划分出多个vhost，每个用户在自己的vhost创建exchange/queue等。
* **Connection**：publisher/consumer和broker之间的TCP连接。
* **Channel**：Channel是在connection内部建立的逻辑连接，多线程情况下通常每个线程创建单独的channel进行通讯。AMQP method包含了channel id帮助客户端和broker识别channel，**channel是完全隔离的**。**Channel作为轻量级的Connection极大减少了操作系统建立TCP connection的开销。**
* **Exchange**：交换机。消息到达broker中会首先到达Exchange，Exchange根据分发规则，匹配查询表中的routing key，分发消息到queue中去。常用的类型有：direct（point-to-point）、topic（publish-subscribe）、fanout（multicast）。
* **Queue**：消息被送到Queue中，然后被消费者取走。
* **Binding**：**exchange**和**queue**之间的虚拟连接。binding中可以包含Routing key，binding信息被保存到exchange中的查询表中，用于message的分发依据。声明binding关系的时候，可以声明RoutingKey参数

**工作模式**

RabbitMQ一共有7种工作模式，参考[Get Started](https://www.rabbitmq.com/getstarted.html)：

![RabbitMQ工作模式](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_2.png)





# 二、Hello World

## 2.1 介绍

[Hello World](https://www.rabbitmq.com/tutorials/tutorial-one-java.html)模式是RabbitMQ最简单的一个模式。下图中的P表示生产者，C是消费者，中间框是一个队列，是RabbitMQ代表消费者保存的消息缓冲区。

![Hello World模式](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_3.png)





## 2.2 实现

**1、在IDEA中创建Maven项目，然后引入依赖：**

```xml
<dependencies>
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.8.0</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>
</dependencies>
```



**2、编写生产者的代码**

```java
public class Producer {
    // 队列名称
    public static final String QUEUE_NAME = "hello";

    //发消息
    public static void main(String[] args) throws IOException, TimeoutException {
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置工厂的ip，连接队列
        factory.setHost("192.168.198.198"); //RabbitMQ服务主机的ip
        //设置用户名和密码
        factory.setUsername("admin");
        factory.setPassword("admin");

        //创建连接，每个连接有多个channel，channel是用来发消息的。
        Connection connection = factory.newConnection();

        //获取channel
        Channel channel = connection.createChannel();

        //生成一个队列用于通信，简单起见，使用默认的交换机
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //发消息
        String message = "hello, world";
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println("消息发送完毕！");
    }
}
```

其中关键方法的说明：

* `queueDeclare()`，用于声明一个队列，其中的各个参数依次解释如下：
  * 队列名。
  * 队列的消息是否持久化，默认情况下消息存储在内存中(不持久化）。
  * 该队列是否进行消费共享，true表示允许多个消费者消费。
  * 是否自动删除 最后一个消费者端开连接以后，该队列是否自动删除。
  * 其他参数。
* `basicPublish()`，用于发布消息：
  * 交换机名称，用于指定发送到哪个交换机。
  * routingKey，路由的key值。这里使用的是channel名字作为routingKey
  * 其他参数信息。
  * 发送消息的消息体。



**3、编写消费者的代码**

```java
public class Consumer {
    // 队列名称
    public static final String QUEUE_NAME = "hello";
    
    //接收消息
    public static void main(String[] args) throws IOException, TimeoutException {
        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置ip
        factory.setHost("192.168.198.198");
        //设置用户名和密码
        factory.setUsername("admin");
        factory.setPassword("admin");
        //创建连接
        Connection connection = factory.newConnection();
        //创建信道
        Channel channel = connection.createChannel();

        //消费者接收消息（消费消息）
        //接收消息的回调函数
        DeliverCallback deliverCallback = (consumerTag,message)->{
            System.out.println(new String(message.getBody()));
        };
        CancelCallback cancelCallback = (consumerTag)->{
            System.out.println("消息消费被中断");
        };
        channel.basicConsume(QUEUE_NAME,true, deliverCallback,cancelCallback);
    }
}
```



`basicConsume()`的参数说明：

* 指定消费的队列名，即从哪个队列中取消息。
* 消费成功之后是否自动应答。
* 消费者接收的回调函数。
* 消费者取消消费的回调函数。



> 如果需要修改现有的exchange和queue，需要删除现有的队列，重新创建。



**4、运行**

启动生产者程序，会创建Channel并发送消息，然后启动消费者程序，会收到来自生产者的消息。



# 三、Work Queues

[Work Queues](https://www.rabbitmq.com/tutorials/tutorial-two-java.html)模式的主要思想是避免因立即执行资源密集型任务而不得不等待它完成。在这个模式中，我们将任务封装为消息，并将其发送到队列。当有多个工作线程时，这些工作线程将一起处理这些任务。

![Work Queues](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_4.png)



## 3.1 轮询分发消息

Work Queues模式下使用的轮询分发的机制，对于多个消费者线程，会轮流分发任务。

下面我们以一个生产者，两个消费者线程来模拟。

**抽取工具类**

创建channel之前的代码是相同的，因此可以单独抽取出来，作为工具类：

`RabbitMqUtils.java`

```java
public class RabbitMqUtils {
    public static Channel getChannel(){
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.198.198");
        factory.setUsername("admin");
        factory.setPassword("admin");
        Connection connection = null;
        try {
            connection = factory.newConnection();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        Channel channel = null;
        try {
            channel = connection.createChannel();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return channel;
    }
}
```

**生产者**

启动发送线程：

```java
public class Task01 {
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        //从控制台接收信息发送到消费者
        Scanner scanner = new Scanner(System.in);
        while(scanner.hasNext()){
            String message = scanner.next();
            channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
            System.out.println(message+"发送完成！");
        }
    }
}
```

**启动两个工作线程**

```java
public class Worker01 {
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();
        //消息的接收
        DeliverCallback deliverCallback = (consumerTag,message)->{
            System.out.println("接收到的消息："+new String(message.getBody()));
        };
        CancelCallback cancelCallback = (consumerTag)->{
            System.out.println(consumerTag+"消息被取消接受");
        };
        System.out.println("Consumer 1------");
        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
    }
}
```

使用IDEA设置并行运行，`Edit Configurations`->`Allow multiple instances`，这样可以同时启动两个消费者线程。

**运行**

生产者发送消息，消费者1和消费者2会轮流处理消息。

比如生产者发送1、2、3、4、5、6，消费者1接收到1、3、5，消费者2接收到2、4、6。



## 3.2 消息应答

**概念**

消费者完成一个任务需要耗费一定的时间，RabbitMQ一旦向消费者发送消息后，会将此消息标记为删除，这种情况下，如果消费者处理任务的过程中出现故障，会导致任务丢失。**为了保证消息不会丢失，RabbitMQ引入了消息应答机制**。

**消息应答（Message acknowledgment）**：消费者在接收到消息并且处理完该消息之后，告诉RabbitMQ此消息已经被处理，RabbitMQ可以将该消息删除。这样就保证当某一个消费者线程故障后，消息会被重新发送给其他消费者，确保消息不会丢失（前提是RabbitMQ无故障）。

消息应答包括**自动应答**和**手动应答**两种方式：

* **自动应答**：消息发送后立即被认为已经传送成功。这种方式没有对传递的消息数量做限制，会导致消费者端消息积压，线程被系统杀死。这种模式仅适用于消费者可以高效并以某种速率能够处理这些消息的情况下使用。
* **手动应答**：默认是手动应答模式。如果一个消费者线程宕机，其消息可以被其他消费者线程消费，而不会出现消息丢失的情况。



手动应答示例：

```java
/**
 * 消息在手动应答时不丢失、放回队列中重新消费
 */
public class Worker03 {
    public static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel(); //获取channel
        System.out.println("消费者1等待接收消息，处理时间较短------");

        //手动应答
        boolean autoAck = false;
        DeliverCallback deliverCallback = (consumerTag, message)->{
            //沉睡1s，模拟处理信息场景。
            SleepUtils.sleep(1); //工具类SleepUtils用来睡眠线程。
            System.out.println("接收到的消息："+new String(message.getBody(),"UTF-8"));
            //手动应答。参数1为消息的标记tag，参数2表示是否批量应答
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
        };
        CancelCallback cancelCallback = (consumerTag)->{
            System.out.println(consumerTag+"消息被取消接受");
        };
        //设置为手动应答
        channel.basicConsume(TASK_QUEUE_NAME,autoAck,deliverCallback,cancelCallback);
    }
}
```

手动应答主要是在接收消息的回调方法中调用`basicAck()`方法，已经在`basicConsume()`方法中设置自动应答方式为false。

其中`basicAck()`方法的第二个参数表示是否批量化应答。如果是**批量化应答(Multiple)**，则每次会应答一个批次的消息。

**手动应答的好处就是可以批量应答并且减少网络拥堵**。



**消息重新入队**：如果某个消费者由于某些原因失去连接（或发生故障），导致消息未发送ACK确认，RabbitMQ将了解到消息未完全处理，并将其重新排队发送给其他消费者。这样，即使某个消费者偶尔死亡，也可以确保不丢失消息。



## 3.3 RabbitMQ持久化

**消息应答**能够确保消费者线程故障后，消息不会丢失，如何保障当RabbitMQ服务停掉以后消息也不丢失？

这就**需要将队列和消息都标记为持久化。**

**队列持久化**

在声明队列时将是否持久化的参数置为`true`即可：

```java
channel.queueDeclare(TASK_QUEUE_NAME,true,false,false,null);
```

第二个参数就表示是否持久化队列。

如果已经存在的队列需要持久化，需要将队列删除，重新创建。

**消息持久化**

仅将队列持久化不能保证消息不丢失，因为如果消费者线程宕机断开连接，仍然有可能出现消息丢失的情况。

设置消息持久化，需要在`channel`发布消息时设置：

```java
channel.basicPublish("",TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes("UTF-8"));
```

这种方式只是尽量保证持久化，如果绝对保证持久化，需要使用发布确认机制。



**预取值**

`basicQos()`方法可以设置消费者线程的预取值。

预取值表示一个消费者线程对应的信道最大可以堆积的消息个数，即**通道上允许的未确认消息的最大数量**。

> 类似于缓冲池，预取值最大值就是缓存池的最大值。最多只能存放预取值个数的未确认消息。

如果不设置预取值，可能会有大量已传递但尚未处理的消息的数量堆积，导致消费者RAM消耗。



**不公平分发**

轮询方式是不管每个消费者的处理速度，给每个消费者线程轮流分发任务。

不公平分发是指**根据每个消费者线程的处理能力，为每个消费者线程分配不同个数的消息**。——能者多劳。

在**消费者**的信道上，设置Qos的值为1，就可以表示按处理能力分发：

```java
//将Qos设置为1，就是不公平分发；默认为0，表示轮询分发
channel.basicQos(1); 
```

Qos的值为1时，表示根据消费者线程的处理能力分发，最多堆积一个任务。



# 四、发布确认

如果想要确保消息一定不会丢失，除了上面提到的**队列持久化**和**消息持久化**，还需要使用**发布确认（Publisher confirm）**。这三种设置保证了消息不会丢失。

官方文档：[Publisher Confirms](https://www.rabbitmq.com/tutorials/tutorial-seven-java.html)



## 4.1 发布确认原理

生产者将channel设置为confirm模式，一旦channel进入**confirm模式**，**所有在该channel上面发布的消息都会被指派一个唯一的ID（从1开始）**。一旦消息被投递到所有匹配的队列之后，broker会发送一个确认给生产者。

生产者得知消息已经正确到达目的队列后，如果消息和队列是可持久化的，确认消息会在消息写入磁盘后发出。

> broker回传给生产者的确认消息中delivery-tag域中包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。
>
> **发布确认（Publisher confirm）是broker给生产者发送的确认消息。**
>
> **消息应答（Message acknowledgment）是消费者处理完消息发送给broker的ack确认。**



**confirm模式**的好处在于它是异步的，发布一条消息后，生产者可以边等确认消息边发送下一条消息。消息得到确认或者丢失，生产者都会通过相应的回调方法进行处理。



## 4.2 发布确认策略

### 4.2.1 开启发布确认

生产者创建信道后，调用`confirmSelect()`方法即可开启发布确认模式。

```java
channel.confirmSelect();
```

发布确认可以有单个确认、批量确认、异步确认三种方式。



### 4.2.2 单个确认发布

单个确认发布，即对每一条消息进行同步确认，生产者发布一条消息后只有它被确认发布后，后续的消息才能继续发布。

```java
public static void publishMessageIndividually() throws Exception{
    Channel channel = RabbitMqUtils.getChannel();
    //信道名字使用随机的UUID
    String queueName = UUID.randomUUID().toString();
    channel.queueDeclare(queueName,true,false,false,null);
    //开启发布确认
    channel.confirmSelect();
    long start = System.currentTimeMillis();
    //批量发消息
    for(int i=0;i<1000;i++){
        String message = i+"";
        channel.basicPublish("",queueName,null,message.getBytes());
        //单个消息马上确认
        boolean flag = channel.waitForConfirms();
        if(flag){
            System.out.println("第"+i+"条消息发送成功");
        }
    }
    long end = System.currentTimeMillis();
    System.out.println("发布1000个单独确认消息，耗时"+(end-start)+"ms");
}
```

单个确认发布的速度是最慢的，因为要每条消息都确认一次。



### 4.2.3 批量确认发布

批量确认发布是指根据批次大小确认发布，这种方式的缺点是当发生故障导致发布出现问题时，不知道哪个消息出问题。

```java
//批量发布确认
public static void publishMessageBatch() throws Exception{
    Channel channel = RabbitMqUtils.getChannel();
    //信道名字使用随机的UUID
    String queueName = UUID.randomUUID().toString();
    channel.queueDeclare(queueName,true,false,false,null);
    //开启发布确认
    channel.confirmSelect();
    long start = System.currentTimeMillis();
    //批量发消息，并批量确认消息
    int batchSize = 100; //每100条确认一次
    for(int i=0;i<1000;i++){
        String message = i+"";
        channel.basicPublish("",queueName,null,message.getBytes());
        if((i+1)%batchSize==0){ //每100条消息确认一次
            boolean flag = channel.waitForConfirms();
            if(flag){
                System.out.println("确认前"+i+"条数据");
            }
        }
    }
    long end = System.currentTimeMillis();
    System.out.println("发布1000个批量确认消息，耗时"+(end-start)+"ms");
}
```



### 4.2.4 异步确认发布

异步确认发布是效率和可靠性最高的。对于已确认消息和未确认消息，异步确认方式都能够处理。

异步确认发布主要是通过`addConfirmListener`方法监听确认和未确认的消息，使用哈希表记录所有发布的消息，对于成功确认的消息从哈希表中删除，剩下的是未确认的消息。

```java
//异步批量确认
public static void publishMessageAsync() throws Exception{
    Channel channel = RabbitMqUtils.getChannel();
    //信道名字使用随机的UUID
    String queueName = UUID.randomUUID().toString();
    channel.queueDeclare(queueName,true,false,false,null);
    //开启发布确认
    channel.confirmSelect();

    //使用哈希表记录所有消息，便于多个线程进行消息的添加和删除
    /*线程安全的哈希表，适用于高并发的情况。
    1.哈希表能够轻松的将序号和消息进行关联。
    2.可以轻松地批量删除条目，只需要知道序号
    3.支持高并发（多线程）
    */
    ConcurrentSkipListMap<Long,String> map = new ConcurrentSkipListMap<>();
    
    long start = System.currentTimeMillis();
    //创建消息的监听器，监听哪些消息成功了，哪些消息失败了。
    //消息确认成功 回调函数
    /*参数：1. 消息的标记（序号）2.是否为批量确认*/
    ConfirmCallback ackCallback = (deliveryTag, multiple) ->{
        //2.删除掉已经确认的消息，剩下的就是未确认的消息
        if(multiple){ //默认multiple是true，即批量确认的
            //如果是批量确认的，则找到所有小于当前序号的值，并清除。这样剩下的就是未确认的消息
            //headMap方法就是返回所有小于指定序号的值的map，第二个参数表示是否找出等于序号的。
            ConcurrentNavigableMap<Long, String> navigableMap = map.headMap(deliveryTag, true);
            navigableMap.clear();
        }else{//如果不是批量确认，则只删除当前已经确认的消息即可。
            map.remove(deliveryTag);
        }
        System.out.println("确认的消息："+deliveryTag);
    };
    //监听确认失败的消息
    ConfirmCallback nackCallback = (deliveryTag, multiple) ->{
        //3.打印未确认的消息
        System.out.println("未确认的消息："+deliveryTag);
    };
    /*参数：1.监听确认成功的消息 2. 监听确认失败的消息*/
    channel.addConfirmListener(ackCallback,nackCallback);
    //批量发消息，异步确认消息
    int batchSize = 100; //每100条确认一次
    for (int i = 0; i < 1000; i++) {
        String message = "消息" + i;
        //1.记录下所有要发送的消息
        map.put(channel.getNextPublishSeqNo(),message);
        //发布消息
        channel.basicPublish("",queueName,null,message.getBytes());
    }
    long end = System.currentTimeMillis();
    System.out.println("发布"+MESSAGE_COUNT+"个异步确认消息，耗时"+(end-start)+"ms");
}
```



# 五、交换机



## 5.1 相关概念

生产者生产的消息不会直接发送到队列，只能将消息发送到**交换机（exchange）**，然后由交换机发送到队列。

**交换机**的功能：①接收来自生产者的消息。②将消息推入队列。



交换机主要的三个类型：

* **fanout**：这种类型的交换机不分析Routing Key，将消息转发到所有和该交换机绑定的队列中。用于[Publish/Subscribe模式](https://www.rabbitmq.com/tutorials/tutorial-three-java.html)。
* **direct**：这类交换机需要精准匹配Routing Key，只将消息转发到指定Routing Key的队列中。用于[Routing模式](https://www.rabbitmq.com/tutorials/tutorial-four-java.html)。
* **topic**：这类交换机按照一定规则匹配Routing Key，将消息转发到匹配到的队列中，通常是一组相同主题的队列。用于[Topics模式](https://www.rabbitmq.com/tutorials/tutorial-five-java.html)



**临时队列**

创建一个随机名称的临时队列：

```java
String queueName = channel.queueDeclare().getQueue();
```

**绑定（bindings）**

binding是指exchange和queue之间的关系，将exchange和queue进行绑定。

其中一个交换机和一个队列之间可以有多个binding key



## 5.2 Publish/Subscribe

[发布订阅模式](https://www.rabbitmq.com/tutorials/tutorial-three-java.html)是使用的扇出（fanout）类型的交换机。

交换机会**将消息推送至所有和他绑定的队列，不会匹配Routing Key**。无论绑定的Routing Key是什么值，都会发送到所有和其绑定的队列中。

![Publish/Subscribe](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_5.png)





**实例**

如上图所示，一个交换机，两个队列。

生产者：

```java
public class EmitLog {
    //交换机名字为logs
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();
        //声明为fanout类型，可以用枚举类型或者字符串。
        //channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);
        Scanner scanner = new Scanner(System.in);
        while(scanner.hasNext()){
            String message = scanner.next();
            //绑定信息为空
            channel.basicPublish(EXCHANGE_NAME,"",null,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息："+message);
        }
    }
}
```

消费者1

```java
public class ReceiveLogs01 {
    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();
        /*声明临时队列
        临时队列的队列名称是随机的
        当消费者断开与队列的连接后，队列会被自动删除。
         */
        String queueName = channel.queueDeclare().getQueue();

        /*
        将队列和交换机绑定 binding，routingKey为空
         */
        channel.queueBind(queueName,EXCHANGE_NAME,"");

        DeliverCallback deliverCallback = (consumerTag, message)->{
            System.out.println("消费者1控制台打印接收到的消息："+new String(message.getBody(),"UTF-8"));
        };
        CancelCallback cancelCallback = consumerTag->{};
        channel.basicConsume(queueName,deliverCallback,cancelCallback);
    }
}
```

消费者2的代码和消费者1相同，会生成另一个随机名称的临时队列。

这样，当生产者每次发送一条消息，消费者1和消费者2都能接收到。



## 5.3 Routing

[Routing模式](https://www.rabbitmq.com/tutorials/tutorial-four-java.html)使用的是direct类型交换机，这种模式下，交换机需要精准匹配Routing Key，只将消息转发到指定Routing Key的队列中。

![Routing](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_6.png)



如图，交换机X绑定了Q1和Q2两个队列，其中和Q1之间的Binding Key为`orange`，和Q2的Binding Key包括`black`和`green`两个。

Routing Key为`orange`的消息会被推送到Q1队列。Routing Key为`black`和`green`的消息会被推送到Q2队列。

> 多重绑定：允许不同队列和交换机之间的Binding Key是相同的，这种情况下效果和Publish/Subscribe模式相同。

**实现**

生产者：

```java
public class DirectProducer {
    public static final String EXCHANGE_NAME = "X";
    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();
        //声明交换机,类型为direct
        channel.exchangeDeclare(EXCHANGE_NAME,"direct");
        
        Scanner scanner = new Scanner(System.in);
        while(scanner.hasNext()){
            String message = scanner.next();
            //当Bingding Key取不同值时，会根据情况发送的相应的队列。
            channel.basicPublish(EXCHANGE_NAME,"orange",null,message.getBytes("UTF-8"));
//            channel.basicPublish(EXCHANGE_NAME,"black",null,message.getBytes("UTF-8"));
//            channel.basicPublish(EXCHANGE_NAME,"green",null,message.getBytes("UTF-8"));
            System.out.println("Direct类型，生产者发出消息："+message);
        }
    }
}
```



消费者1：

```java
public class DirectConsumer01 {
    public static final String EXCHANGE_NAME = "X";
    public static final String QUEUE_NAME = "Q1";
    public static final String BINGDING_NAME = "orange";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();
        //声明交换机,类型为direct
        channel.exchangeDeclare(EXCHANGE_NAME,"direct");
        //声明名为Q1的队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        //将队列和交换机绑定
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,BINGDING_NAME);

        DeliverCallback deliverCallback = (consumerTag, message)->{
            System.out.println("C1接收到消息："+new String(message.getBody(),"UTF-8"));
        };
        CancelCallback cancelCallback = consumerTag->{};
        channel.basicConsume(QUEUE_NAME,deliverCallback,cancelCallback);
    }
}
```

消费者2和消费者1类似，不同的是有两个Bingding：

```java
//多重绑定，一个channel和交换机有两个绑定,两个不同的routing key
channel.queueBind("Q2","X","black");
channel.queueBind("Q2","X","green");
```

这样，当发送消息的Routing Key为`orange`时，消息会被推送到Q1，Routing Key为`black`或`green`时，消息被推送到Q2。



## 5.4 Topics

[Topics](https://www.rabbitmq.com/tutorials/tutorial-five-java.html)模式使用的是Topic类型的交换机，队列可以匹配一定规则的多个Routing Key。

Topic模式的Routing Key必须符合一定的要求：**必须是一个单词列表，以`.`号分开**，单词可以是任意的，比如`stock.usd.nyse`, `nyse.vmw`, `quick.orange.rabbit`等。单词列表最多为255个字节。

`*`号可以代替一个单词。

`#`号可以代替0个或多个单词。

![Topics](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_7.png)



如上图所示，交换机X和Q1的Binding Key为`*.orange.*`，X和Q2的Binding Key为`*.*.rabbit`和`lazy.#`。

一些案例：

①`quick.orange.rabbit`：Q1、Q2接收到消息。

②`quick.orange.rabbit` ： Q1、Q2接收到消息。

③`lazy.orange.elephant`：Q1、Q2 接收到消息。

④`quick.orange.fox`：Q1 接收到消息。

⑤`lazy.brown.fox`：Q2 接收到消息。

⑥`lazy.pink.rabbit` ：Q2 接收一次消息，虽然两种绑定都匹配，但只接收一次。

⑦`quick.brown.fox` ：不匹配任何绑定，被丢弃。

⑧`quick.orange.male.rabbit` ：是四个单词，不匹配任何绑定，被丢弃。

⑨`lazy.orange.male.rabbit` ：是四个单词，Q2接收到消息。



**实现**

声明交换机：

```java
channel.exchangeDeclare("X","topic");
```

队列Q1绑定：

```java
channel.queueBind("Q1","X","*.orange.*");
```

队列Q2绑定：

```java
channel.queueBind("Q1","X","*.*.rabbit");
channel.queueBind("Q1","X","lazy.#");
```





# 六、死信队列

关于死信队列的官方文档：[死信队列](https://www.rabbitmq.com/dlx.html)

队列中的消息如果发生以下情况就会变成**死信（Dead Letter）**：

* 消息被拒绝（`basic.reject`或`basic.nack`），并且`requeue`参数为`false`
* 消息TTL超时，即消息过期。
* 队列长度超过最大限制。

> 队列过期不会导致消息变为死信。

死信交换机（Dead Letter eXchanges，DLXs）是正常的交换机，它可以将Dead Letter转发给死信队列，进一步处理。

如图，正常情况下，消息通过`normal_exchange`推送到`normal_queue`，然后被`C1`消费；如果消息变为死信，`normal_queue`会将死信转发给死信交换机`DLX`，`DLX`将死信推送给死信队列`dead_letter_queue`，然后被`C2`消费。

![死信队列](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_8.png)





**实现**

生产者：

```java
public class Producer  {
    public static final String NORMAL_EXCHANGE="normal_exchange";
    public static final String NORMAL_ROUTING_KEY="normal_routing_key";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();
        channel.exchangeDeclare(NORMAL_EXCHANGE,"direct");
        //设置过期时间为10s
        //AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().expiration("10000").build();

        //模拟发10条消息
        for (int i = 0; i < 10; i++) {
            String message = String.valueOf(i);
            //超过ttl变为死信
            //channel.basicPublish(NORMAL_EXCHANGE,NORMAL_BINDING,properties,message.getBytes("UTF-8"));
            //超过最大长度变为死信
            //channel.basicPublish(NORMAL_EXCHANGE,NORMAL_BINDING,null,message.getBytes("UTF-8")); 
            channel.basicPublish(NORMAL_EXCHANGE,NORMAL_ROUTING_KEY,null,message.getBytes("UTF-8")); //消息被拒绝变为死信
            System.out.println("生产者发出消息："+message);
        }
    }
}
```



消费者C1：

```java
public class Consumer01 {
    public static final String NORMAL_EXCHANGE="normal_exchange";
    public static final String DEAD_EXCHANGE="dead_exchange";
    public static final String NORMAL_QUEUE="normal_queue";
    public static final String DEAD_QUEUE="dead_queue";
    public static final String NORMAL_ROUTING_KEY="normal_routing_key";
    public static final String DEAD_ROUTING_KEY="dead_routing_key";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();
        //声明普通交换机和死信交换机，类型均为direct
        channel.exchangeDeclare(NORMAL_EXCHANGE,"direct");
        channel.exchangeDeclare(DEAD_EXCHANGE,"direct");

        //声明正常队列。正常队列需要将死信消息转发到死信交换机，需要用到map参数
        Map<String, Object> arguments = new HashMap<>();
        //过期时间,比如为10s；也可以在生产者发送消息的时候设置过期时间
        //arguments.put("x-message-ttl",10000);
        //指定队列的最大长度，一旦消息个数超出这个长度，就会成为死信
        //arguments.put("x-max-length",6);
        //设置死信交换机，即死信消息将要转发到的交换机
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        //设置死信routingkey,死信消息通过此路由键发送到死信队列。
        arguments.put("x-dead-letter-routing-key",DEAD_ROUTING_KEY);

        channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);
        channel.queueBind(NORMAL_QUEUE,NORMAL_EXCHANGE,NORMAL_ROUTING_KEY); //绑定

        //声明死信队列
        channel.queueDeclare(DEAD_QUEUE,false,false,false,null);
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,DEAD_ROUTING_KEY); //绑定

        DeliverCallback deliverCallback = (consumerTag, message)->{
            //模拟拒绝消息
            String msg = new String(message.getBody(),"UTF-8");
            if(Integer.parseInt(msg)%2==0){
                System.out.println(msg+"被拒绝");
                //拒绝消息
                channel.basicReject(message.getEnvelope().getDeliveryTag(),false);
            }else{
                System.out.println("C1打印接收到的消息："+msg);
                //手动应答消息
                channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
            }
        };
        CancelCallback cancelCallback = consumerTag->{};
        //模拟拒绝消息时，要关闭自动应答。
        channel.basicConsume(NORMAL_QUEUE,false,deliverCallback,cancelCallback);
    }
}
```

> 疑问：为什么要在正常队列中设置`x-dead-letter-routing-key`? 不设置会导致死信队列收不到消息，但是下文中也设置了死信队列和DLX和Routing Key，二者如果不一致也会导致死信队列收不到死信消息。



消费者C2：

```java
//C2只需要从死信队列中接收消息即可
public class Consumer02 {
    public static final String DEAD_QUEUE="dead_queue";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMqUtils.getChannel();

        DeliverCallback deliverCallback = (consumerTag, message)->{
            System.out.println("C2打印接收到的消息："+new String(message.getBody(),"UTF-8"));
        };
        CancelCallback cancelCallback = consumerTag->{};
        channel.basicConsume(DEAD_QUEUE,true,deliverCallback,cancelCallback);
    }

}

```



# 七、延迟队列



## 7.1 概念理解

**延迟队列**是用来存放需要在指定时间被处理的元素的队列。如果我们希望一条消息在指定时间到了以后或之前处理，可以使用延迟队列。

延迟队列常见的使用场景：订单一段时间内未支付则自动取消、预定会议开始前十分钟通知与会人员参加会议。

**TTL**是RabbitMQ中一个消息或者队列的属性，表明**一条消息或者一个队列中所有消息的最大存活时间**。单位是`ms`

如果一条消息设置了TTL属性，或者一条消息进入了设置TTL属性的队列，则这条消息如果在TTL设置的时间内没有被消费，就会成为“死信”。

**如果同时配置了消息的TTL和队列的TTL，较小的值会被使用**。



## 7.2 设置TTL

**消息设置TTL**

```java
//在发送消息时，针对当前这条消息单独设置ttl时间为ttlTime
rabbitTemplate.convertAndSend("exchange","routingKey",message,msg->{
            msg.getMessageProperties().setExpiration(ttlTime);
            return msg;
        });
```



**队列设置TTL**

```java
arguments.put("x-message-ttl",10000);
Queue q = QueueBuilder.durable(QUEUE_A).withArguments(arguments).build();
```

其中`argumengts`是`Map`，用于给队列设置参数。`x-message-ttl`表示队列的延迟时间。

`durable()`意为设置队列为持久化队列。



**注意**

* 如果设置了**队列**的TTL，那么消息一旦超过了这个时间就会被丢弃（如果有死信队列会被丢弃到死信队列）

* 如果设置的是**消息**的TTL，消息过期了不一定马上被抛弃。因为**消息是否过期是在即将投递到消费者之前判定的**，当队列出现消息积压的情况时，已过期的消息仍然能在队列中存活。

  > 不设置TTL，表示消息永远不会过期。
  >
  > TTL设置为0，表示除非此时可以直接投递该消息到消费者，否则改消息将会被丢弃。





## 7.3 DLX+TTL实现延迟队列

我们可以使用消息和队列的TTL的属性和死信队列，实现延迟队列。思路是将过期队列转发到死信队列，消费者只要消费死信队列中的消息即可，就实现了延迟队列的功能。

**1、创建SpringBoot项目**

创建SpringBoot项目，然后引入以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

	<!-- RabbitMQ依赖 -->
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--fastjson-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.62</version>
    </dependency>
    <!-- RabbitMQ -->
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```



**2、修改配置文件**

在`application.properties`配置文件中，添加RabbitMQ的相关配置：

```properties
# 配置RabbitMQ的相关配置
spring.rabbitmq.host=192.168.198.198
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=admin
```



**3、创建配置类**

根据下面的架构图，`X`为正常的交换机，`Y`是死信交换机，`QA`和`QB`是具有TTL属性的延迟队列，`QC`是没有TTL属性的正常队列，`QD`是死信队列。各个队列和交换机之间的`binding`关系如图所示。

![架构图](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_9.png)



首先创建一个配置类，将各个组件构建出来。其中包括两个交换机，四个队列，四个绑定关系：

```java
import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TtlQueueConfig {
    //普通交换机的名称
    public static final String X_EXCHANGE = "X";
    //死信交换机的名称
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    //普通队列的名称
    public static final String QUEUE_A = "QA";
    public static final String QUEUE_B = "QB";
    //死信队列的名称
    public static final String DEAD_LETTER_QUEUE = "QD";
    //普通队列，没有过期时间。可以适用于任意过期时间的消息。不需要每个过期时间都创建一个新队列
    public static final String QUEUE_C = "QC";

    //声明QC
    //普通队列C，不设置过期时间
    @Bean("queueC")
    public Queue queueC(){
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
        arguments.put("x-dead-letter-routing-key","YD");
        return QueueBuilder.durable(QUEUE_C).withArguments(arguments).build();
    }
    //绑定.队列QC和交换机X绑定，Routing Key为XC
    @Bean
    public Binding queueCBindingX(@Qualifier("queueC") Queue queueC,
                                  @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueC).to(xExchange).with("XC");
    }

    //声明普通交换机，别名为xExchange
    @Bean("xExchange")
    public DirectExchange xExchange(){
        return new DirectExchange(X_EXCHANGE);
    }

    //声明死信交换机，别名为yExchange
    @Bean("yExchange")
    public DirectExchange yExchange(){
        return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
    }

    //声明队列
    //普通队列A，过期时间为10s
    @Bean("queueA")
    public Queue queueA(){
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
        arguments.put("x-dead-letter-routing-key","YD");
        //设置过期时间,单位是ms
        arguments.put("x-message-ttl",10000);
        return QueueBuilder.durable(QUEUE_A).withArguments(arguments).build();
    }
    //普通队列B，过期时间为40s
    @Bean("queueB")
    public Queue queueB(){
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
        arguments.put("x-dead-letter-routing-key","YD");
        //设置过期时间,单位是ms
        arguments.put("x-message-ttl",40000);
        return QueueBuilder.durable(QUEUE_B).withArguments(arguments).build();
    }
    //死信队列QD
    @Bean("queueD")
    public Queue queueD(){
        return QueueBuilder.durable(DEAD_LETTER_QUEUE).build();
    }
    //绑定.队列QA和交换机X绑定，Routing Key为XA
    @Bean
    public Binding queueABindingX(@Qualifier("queueA") Queue queueA,
                                  @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueA).to(xExchange).with("XA");
    }
    //绑定.队列QB和交换机X绑定，Routing Key为XB
    @Bean
    public Binding queueBBindingX(@Qualifier("queueB") Queue queueB,
                                  @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueB).to(xExchange).with("XB");
    }
    //绑定.队列QD和交换机Y绑定，Routing Key为YD
    @Bean
    public Binding queueDBindingY(@Qualifier("queueD") Queue queueD,
                                  @Qualifier("yExchange") DirectExchange yExchange){
        return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }
}

```

创建队列可以使用`new Queue()`方式，也可以使用`QueueBuilder`构建器的方式。

同样地，声明交换机和Binding都有`new`和构建器两种方法。



**4、创建生产者和消费者**

生产者：

```java
@RestController
@RequestMapping("/ttl")
public class SendMsgController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable String message){
        System.out.println("发送消息："+message);
        rabbitTemplate.convertAndSend("X","XA","消息来自ttl为10s的队列："+message);
        rabbitTemplate.convertAndSend("X","XB","消息来自ttl为40s的队列："+message);
    }

    //发送带有ttl的消息，消息的过期时间由消息本身确定，不需要单独创建队列
    @GetMapping("/sendExpirationMsg/{message}/{ttlTime}")
    public void sendMsg(@PathVariable String message,@PathVariable String ttlTime){
        System.out.println(new Date()+"发送一条时长为"+ttlTime+"毫秒的消息:"+message);
        rabbitTemplate.convertAndSend("X","XC",message,msg->{
            //设置消息的ttl大小
            msg.getMessageProperties().setExpiration(ttlTime);
            return msg;
        });
    }
}
```



消费者：

```java
@Component
public class DeadLetterQueueConsumer {
    //从QD死信队列接收消息
    @RabbitListener(queues = "QD")
    public void receiveD(Message message) throws Exception{
        String msg = new String(message.getBody());
        System.out.println(new Date()+"收到死信队列的消息："+msg);
    }
}
```



**5、运行结果**

* **测试`QA`和`QB`这两个设置TTL属性的队列**。
  * 在浏览器中访问`http://localhost:8080/ttl/sendMsg/hello world`，消费者在10s和40s后会收到内容为`hello world`的两条消息。这两条消息的延迟时间是由队列本身决定的。
* **测试指定消息TTL**。
  * 在浏览器中访问`http://localhost:8080/ttl/sendExpirationMsg/hello world/10000`，消费者会在10s后收到消息。这种方式可以任意指定TTL的大小，可以适用于任意的延迟时间。



**比较队列TTL和消息TTL两种方式**

* 队列设置TTL属性实现延迟队列，这种方式中，当队列到了过期时间，一定会被放到死信队列。但是这种方式不够灵活，如果想要改变延迟时间，就需要新建队列，面对大量不同时间需求，无法实现。

* 消息设置TTL属性实现延迟队列，这种方式足够灵活，可以满足任意的延迟时间的需求。但是这种方式的严重缺陷是如果队列中消息积压，会导致过期的消息无法被丢弃（放到死信队列），导致死信队列的消费者无法按时收到消息。

  > 比如，如果第一个消息的过期时间较长，第二个消息的过期时间较短，则两个消息如果先后发送，会同时被消费者收到。因为RabbitMQ只检查当前第一个消息，直到等到第一个消息到了TTL时间，才会被死信队列收到，然后第二个消息被处理。
  >
  > 正确的结果应该是根据消息的TTL，第二个消息先到达。



为了解决TTL消息的这种缺陷，我们可以使用插件实现延迟队列，满足不同延迟时间的需求。



## 7.4 RabbitMQ插件实现延迟队列



### 7.4.1 安装插件

`rabbitmq_delayed_message_exchange`插件正是为了解决TTL消息无法及时死亡的问题。

在https://www.rabbitmq.com/community-plugins.html 地址下载插件并安装。

下载完以后，将`.ez`后缀的文件复制到`usr/lib/rabbitmq/lib/rabbitmq_server-3.x.x/plugins`目录，这个目录用于存放RabbitMQ的插件。

进入到这个目录，然后执行语句：

```bash
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

即可安装插件。

安装成功以后，在web管理界面，添加交换机可以看到新增的`x-delayed-message`类型。

### 7.4.2 代码实现

如图，使用插件实现延迟队列的原理是创建一个**类型为`x-delayed-message`的交换机（延迟交换机）**，这个类型的交换机支持**延迟投递机制**，即消息传递到以后先暂存到mnesia表中，到达指定的延迟时间以后才将消息投递出去。

> 这种方式在发送消息的时候设置每条消息的延迟时间，延迟交换机根据7延迟时间发送消息。
>
> ![结构图](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_10.png)





实现代码框架和思路和7.3相同：

1、编写配置类，创建各个组件：

```java
@Configuration
public class DelayedQueueConfig {
    //延迟交换机
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    //队列
    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    //Routingkey
    public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

    //声明交换机,使用自定义类型的交换机
    //交换机类型为"x-delayed-message"，传播模式为direct
    @Bean
    public CustomExchange delayedExchange(){
        Map<String,Object> arguments = new HashMap<>();
        arguments.put("x-delayed-type","direct");  //声明匹配模式
        return new CustomExchange(DELAYED_EXCHANGE_NAME,"x-delayed-message",true,false,arguments);
    }
    //声明队列
    @Bean
    public Queue delayedQueue(){
        return new Queue(DELAYED_QUEUE_NAME);
    }

    //binding
    @Bean
    public Binding delayQueueBindingDelayedExchange(
        @Qualifier("delayedQueue") Queue delayedQueue,
        @Qualifier("delayedExchange") CustomExchange delayedExchange){
        //自定义类型交换机需要调用noargs()或args()方法指定是否有参数
        return BindingBuilder.bind(delayedQueue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
    }
}
```



* 创建延迟交换机，需要创建自定义交换机，即`CustomExchange`，类型为`x-delayed-message`，然后设置分配模式。

* `CustomExchange`交换机的绑定，需要调用`noargs()`或`args()`指定有无参数。



2、生产者代码：

```java
//发送消息。基于插件，使用延迟交换机。
@GetMapping("/sendDelayedMsg/{message}/{delayTime}")
public void sendDelayedMsg(@PathVariable String message,@PathVariable Integer delayTime){
    System.out.println(new Date()+"给延迟交换机发送一条延迟时间为"+delayTime+"毫秒的消息:"+message);
    rabbitTemplate.convertAndSend(DelayedQueueConfig.DELAYED_EXCHANGE_NAME,
                                  DelayedQueueConfig.DELAYED_ROUTING_KEY,message,
                                  msg->{
                                      //设置延迟时间
                                      msg.getMessageProperties().setDelay(delayTime);
                                      return msg;
                                  });
}
```

在发送消息时，调用`setDelay()`方法设置延迟时间。

3、消费者代码：

```java
@Component
public class DelayQueueConsumer {
    @RabbitListener(queues = DelayedQueueConfig.DELAYED_QUEUE_NAME)
    public void receiveDelayQueue(Message message) {
        String msg = new String(message.getBody());
        System.out.println(new Date() + "收到延迟队列的消息：" + msg);
    }
}
```



这样，在浏览器中访问`http://localhost:8080/ttl/sendDelayedMsg/hello world/10000`，消费者会在10s后收到消息。并且，对于先后发送的多条延迟时间不同的消息，消费者也能在正确的延迟时间后收到消息。



# 八、发布确认SpringBoot版



发布确认用于确保消息的可靠投递，如果消息投递成功，返回确认信息，如果投递失败，返回失败信息并确保消息不会丢失。

第四章提到过发布确认，这里要讲的是在SpringBoot项目中使用发布确认机制。



## 8.1 发布确认

发布确认的回调方法可以判断出交换机是否收到消息，当交换机宕机或其他原因导致交换机没有收到生产者发出的消息时，回调方法能够做出反馈。

**实现代码**

**1、配置文件**

需要在配置文件中配置开启发布确认：

```properties
# 配置交换机发布确认，correlated表示发布消息成功到交换机后会触发回调方法
spring.rabbitmq.publisher-confirm-type=correlated
```

确认类型有三种：

* `none`表示禁用发布确认模式。
* `correlated`表示发布消息成功到交换器后会触发回调方法。
* `simple`不仅有和`correlated`相同的功能，此外，`simple`模式在发布消息成功后使用`rabbitTemplate`调用`watiForConfirms`或`waitForConfirmsOrDie`方法等待`broker`节点返回发送结果，根据返回结果来判定下一步的逻辑。



**2、添加配置类，构建各个组件**

`ConfirmConfig.java`:

```java
@Configuration
public class ConfirmConfig {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm_exchange"; //交换机
    public static final String CONFIRM_QUEUE_NAME = "confirm_queue"; //队列
    public static final String CONFIRM_ROUTING_KEY = "key1"; //绑定

    //声明确认交换机
    @Bean
    public DirectExchange confirmExchange(){
        return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }

    //声明确认队列
    @Bean
    public Queue confirmQueue(){
        return new Queue(CONFIRM_QUEUE_NAME,true);
    }

    //确认交换机和确认队列的绑定
    @Bean
    public Binding queueBingdingExchange(@Qualifier("confirmExchange") DirectExchange directExchange,
                                         @Qualifier("confirmQueue") Queue queue){
        return BindingBuilder.bind(queue).to(directExchange).with(CONFIRM_ROUTING_KEY);
    }
}
```

**3、生产者代码**

生产者代码可以指定消息id，即回调接口中消息的id。

此外，可以通过修改交换机和队列的名字，模拟交换机/队列宕机的故障情况。

`ProducerController.java`：

```java
@RestController
@RequestMapping("/confirm")
public class ProducerController {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/send/{message}")
    public void sendMessage(@PathVariable String message){
        //发送消息
        CorrelationData correlationData = new CorrelationData("1"); //设置回调接口中的消息,id为1
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME,
                                      ConfirmConfig.CONFIRM_ROUTING_KEY,message,correlationData);

        //模拟队列宕机的情况
        CorrelationData correlationData2 = new CorrelationData("2"); //设置回退接口中的消息,id为2
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME,
                                      ConfirmConfig.CONFIRM_ROUTING_KEY+"aaaa",
                message,correlationData2);
        System.out.println("已发送消息");
    }
}
```



**4、回调接口**

我们需要实现`RabbitTemplate`类的一个内部接口，并重写其中的`confirm`方法，用来判断交换机是否收到消息。

`MyCallBack.java`

```java
@Component
public class MyCallBack implements RabbitTemplate.ConfirmCallback{
    //由于实现的是内部类，RabbitTemplate对象不包括当前对象。
    //需要将MyCallBack对象注入到RabbitTemplate中。
    //第二种方式是不使用这种实现接口的方式，在配置类中实现接口
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    //将当前实现类注入到RabbitTemplate中,postConstruct会在autowired之后执行
    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(this);
    }
  
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String reason) {
        String id = correlationData != null ? correlationData.getId() : "";
        if(ack){
            System.out.println("交换机收到了id为"+id+"的消息");
        }else{
            System.out.println("交换机未收到消息，原因为"+reason);
        }
    } 
}
```



关于`@PostConstruct`注解，参考https://www.cnblogs.com/lay2017/p/11735802.html

`confirm()`方法的参数解析：

* `correlationData`保存回调消息的ID及相关信息
* `ack`表示交换机是否收到了消息。`true`表示交换机接收到了消息，否则表示接收消息失败。
* `reason`表示消息接收失败的原因，如果接收成功则为`null`



**5、消费者代码**

消费者代码就是正常的接收消息。

`ConfirmConsumer.java`：

```java
@Component
public class ConfirmConsumer {
    @RabbitListener(queues = ConfirmConfig.CONFIRM_QUEUE_NAME)
    public void receiveConfirmMessage(Message message){
        System.out.println("消费者接收到消息："+new String(message.getBody()));
    }
}
```



**结果分析**

* 状态正常时，消费者接收到消息，回调方法`ack`为`true`。
* 交换机宕机，消费者收不到消息，回调方法`ack`为`false`。
* 队列宕机，或者消息无法路由，消费者收不到消息，但是回调方法`ack`为`true`，表示交换机收到了消息（虽然没有被路由到队列）。



## 8.2 回退消息

由上一节的发布确认机制可知，生产者通过回调方法只能得知消息有没有被发送到交换机，**如果消息不可路由**，那么消息会被直接丢弃，生产者并不知道消息被丢弃。为了解决这一问题，需要**回退消息**机制。

**回退消息用在消息无法被路由（队列宕机）的情况**。

回退消息的实现和发布确认类似，不同之处为：

**1、设置参数：**

```yml
# 开启消息回退机制，如果消息无法被路由，则消息会被回退
spring.rabbitmq.publisher-returns=true
```



**2、配置类：**

配置类中需要实现`RabbitTemplate.ReturnsCallback`内部接口，并实现其中的方法。

`MyCallBack.java`： 

```java
@Component
public class MyCallBack implements RabbitTemplate.ReturnsCallback {
    @Autowired
    private RabbitTemplate rabbitTemplate;
  
    @PostConstruct
    public void init(){
        //true表示交换机发现消息无法被路由时，会将消息返回给生产者；false表示直接丢弃。
        rabbitTemplate.setMandatory(true); 
        rabbitTemplate.setReturnsCallback(this);
    }
    //回退消息，当消息无法被路由，会被回退。
    @Override
    public void returnedMessage(ReturnedMessage returnedMessage) {
        String replyText = returnedMessage.getReplyText();
        System.out.println("消息被回退，回退原因为："+replyText);
    }
}
```



生产者和消费者的实现并无特殊。当发送的消息无法被路由（比如队列宕机、RoutingKey错误等）时，会调用回退方法。

通过发布确认机制，生产者可以得知消息是否被交换机接收；通过回退消息机制，生产者可以得知消息消息是否被分发到队列中。



## 8.3 备份交换机



通过回退消息，我们可以感知到无法被路由的消息，但是只能把消息回退，无法继续处理。想要处理无法被路由的消息，需要使用**备份交换机**。

**备份交换机**可以理解为RabbitMQ中交换机的备胎，当交换机收到一条不可路由消息时，将会把这条消息转发到备份交换机中。通常备份交换机的类型为`Fanout`，这样就能将不可路由消息广播到所有和它绑定的队列中。

备份交换机除了添加备份队列以外，还可以添加一个报警队列，这样如果有无法被路由的消息，报警队列的消费者可以发出警报信息进行提示。



> 和死信队列不同的是，死信队列是消息已经转发到了队列，而备份交换机处理的是不能被路由转发到队列的消息。

![备份交换机](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_11.png)





如图，正常交换机无法路由的消息会交给备份交换机处理，备份交换机接收到消息后，将消息广播，其中一个队列是报警队列，用于提示消息无法路由。

实现备份交换机，需要在声明普通交换机的时候添加备份交换机：

```java
//声明一个交换机，其备份交换机为"backup_exchange"
Exchange exchange = ExchangeBuilder.directExchange("confirm_exchange").
    durable(true).
    withArgument("alternate-exchange","backup_exchange").
    build();
```



备份交换机需要声明为`fanout`类型：

```java
FanoutExchange exchange = new FanoutExchange("backup_exchange");
```



然后声明报警队列和消费者队列，并且声明各自和交换机之间的绑定。

其中，报警队列的消费者可以用来发出警告信息，表示生产者发出的消息无法被路由：

```java
@Component
public class WarningConsumer {
    //报警队列
    @RabbitListener(queues = "warning_queue")
    public void receiveWarningMessage(Message message){
        System.out.println("报警！发现不可路由消息："+new String(message.getBody()));
    }
}
```



如果**消息回退**机制和**备份交换机**同时配置，则备份交换机机制优先。因为备份队列收到了消息，理解为消息成功路由。



# 九、RabbitMQ其他知识点

 

## 9.1 幂等性

**幂等性(idempotence)**指的是用户对同一操作发起的一次请求或多次请求的结果是一致的，不会因为多次点击而产生了副作用。消息重复消费时需要考虑消息的幂等性。

**消息重复消费**：如果MQ已经把消息发送给了消费者，消费者在返回ack时网络中断，MQ未收到确认消息，会将消息重新发送给其他的消费者，或者再次发送给该消费者，就会造成消息重复消费。

**解决方式**：

* 对于消费端，可以使用全局ID或者唯一标识比如时间戳等唯一的id，每次消费时用该id判断该消息是否已消费过。
* 对于发送端，有两种方式：①唯一ID+指纹码机制（基于业务规则拼接的唯一字符串），查询数据库判断是否重复；②利用redis的原子性，`setnx`指令天然具有幂等性。



## 9.2 优先队列

优先队列可以根据消息的优先级进行消息的分发，优先队列会**将队列中的消息按照优先级进行排序**，按照优先级从高到低的顺序进行消息的发送，而不考虑消息进入队列的顺序。因此，使用优先级队列需要确保所有消息都发送到队列中以后，消费者才开始消费，**确保这些消息能够在队列中排序**。

使用优先队列需要满足两个条件：①队列是优先队列；②消息需要设置优先级。

声明优先队列：

```java
Map<String, Object> params = new HashMap();
params.put("x-max-priority",10); //设置队列为优先级队列，且消息的最大优先级为10
channel.queueDeclare("Q1",true,false,false,params);
```

设置消息的优先级：

```java
// 在生产者代码中，设置消息的优先级为5
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
channel.basicPublish("exchange","routingKey",properties,message.getBytes("UTF-8"));
```



当不同优先级的消息都进入优先队列以后，优先队列将消息进行排序，然后消费者进行消费，接收到的消息是按照优先级从高到低排列。



## 9.3 惰性队列

RabbitMQ 3.6.0版本引入了**惰性队列**的概念。惰性队列会尽可能的将消息存入磁盘中，消费者消费到对应的消息时，才会被加载到内存中。

惰性队列的目的是为了能够支持更长的队列，当消费者下线或者宕机等情况导致长时间不能消费消息造成堆积时，惰性队列就发挥作用了。

开启惰性队列：

```java
Map<String, Object> params = new HashMap();
params.put("x-queue-mode",lazy); //设置队列为惰性队列
channel.queueDeclare("Q1",false,false,false,params);
```

`x-queue-mode`表示队列的模式，默认为`default`模式，如果想要声明惰性队列，则将其设置为`lazy`模式即可。

在面对大量消息积压的情况下，**惰性队列能够占用更少的内存，从而存储更多的消息。**



# 十、RabbitMQ集群



## 10.1 集群搭建

RabbitMQ集群是指搭建多台RabbitMQ服务器，这样当其中的某台服务器出现故障时，其他服务器可以保障服务可用。并且集群能够提升服务的性能。

**搭建步骤**

需要准备三台机器。

建立三个节点，node1-node2-node3

**1、修改3台机器的主机名称**

将3台机器的主机名分别修改为`node1`、`node2`、`node3`，方便识别。

```bash
# 在/etc/hostname文件中修改
vim /etc/hostname
```

**2、配置各个节点的`hosts`文件**

配置`hosts`文件，确保各个节点能够互相认识对方。

```bash
vim /etc/hosts
```

在每台主机的`/etc/hosts`文件中添加三个节点的ip地址和主机名，比如：

```
123.123.123.1 node1
123.123.123.2 ndoe2
123.123.123.3 node3
```



**3、确保各个节点的cookie文件相同**

在其中一个节点上，比如node1节点上，执行远程复制命令，将本机的`cookie`文件远程复制到另外两个节点中：

```bash
scp /var/lib/rabbitmq/.erlang.coolie root@node2:/var/lib/rabbitmq/.erlang.cookie
scp /var/lib/rabbitmq/.erlang.coolie root@node3:/var/lib/rabbitmq/.erlang.cookie
```



**4、启动RabbitMQ服务**

在每台机器上启动RabbitMQ服务：

```bash
# detached代表以后台守护进程方式启动
rabbitmq-server start -detached
```



**5、将节点加入集群**

在node2中执行下列指令，将node2加入到node1的集群：

```bash
# stop_app表示之关闭RabbitMQ服务，stop则会将Erlang虚拟机也关闭
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node1
rabbitmqctl start_app
```

同理，将node3加入到node2的集群。



**6、查看集群状态**

```bash
rabbitmqctl cluster_status
```



**7、需要重新设置用户**

```bash
# 添加用户，用户名admin，密码123
rabbitmqctl add_user admin 123
# 设置用户角色
rabbitmqctl set_user_tags admin administrator
# 设置用户权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
```

至此，包括三个节点的集群搭建完成。

如果想要解除集群节点，需要在节点中执行解除指令，比如 在节点1中解除节点2，需要在节点1中执行下面的指令：

```bash
# node1解除node2节点
rabbitmqctl forget_cluster_node rabbit@node2
```





## 10.2 镜像队列



[**镜像队列**](https://www.rabbitmq.com/ha.html)是指将队列镜像到集群中的其他Broker节点之上，这样当集群中某一节点失效后，队列能够自动切换到镜像中的另一个节点上保证服务的可用性。

> 和集群保证的可用性不同的是，集群中某一节点宕机后，其中的队列都不可用了，队列中的消息也无法被消费。而镜像队列能够保证集群中的节点宕机后，队列中的消息也不会丢失。



**搭建步骤**

以3个集群节点为例，通过web管理页面创建镜像队列。

**1、启动三个集群节点**

**2、添加policy**

选一个节点，比如node1，然后`Add/update a policy`，进行如下设置：

![镜像队列配置](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_12.png)





> 如果在代码中设置镜像队列，参数名称前需要加`x-`



这样，在node1上创建一个队列，则其他两个节点都有这个队列的镜像队列。当node1宕机以后，可以继续使用node2节点中的队列，保证了高可用性。



## 10.3 高可用负载均衡

**1、使用Haproxy实现负载均衡。**

Nginx、lvs、Haproxy之间的区别：http://www.ha97.com/5646.html

假设当前有一个Haproxy服务器，要负责一个3个节点的RabbitMQ集群的负载均衡，需要在Haproxy服务器配置如下：

安装Haproxy：

```bash
yum -y install haproxy
```

修改`haproxy.cfg`：

```bash
vim /etc/haproxy/haproxy.cfg
```

修改以下内容，将IP改为集群中节点的IP地址：

```
# 检测心跳频率
server rabbitmq_node1 123.123.123.1:5672 check inter 5000 rise 2 fall 3 weight1
server rabbitmq_node1 123.123.123.2:5672 check inter 5000 rise 2 fall 3 weight1
server rabbitmq_node1 123.123.123.3:5672 check inter 5000 rise 2 fall 3 weight1
```



启动Haproxy：

```bash
haproxy -f /etc/haproxy/haproxy.cfg
# 查看服务是否启动
ps -ef | grep haproxy
```

访问地址： `http://haproxy主机地址:8888/stats`



**2、Keeplived实现双机热备份**

上面只使用了一个Haproxy主机进行负载均衡，如果Haproxy主机宕机，虽然RabbitMQ集群没有故障，但是对于外界客户端来说所有的连接都会断开。为了确保负载均衡服务的可靠性，使用Keepalied做高可用，实现故障转移。

Keepalived能够通过自身健康检查，资源接管功能做双机热备份。

Keepalived实现双机热备的步骤可参考：https://blog.51cto.com/hellocjq/2089450



## 10.4 Federation Plugin

[Federation插件](https://www.rabbitmq.com/federation.html)的目的是在不同集群的Broker之间同步消息。

Federation plugin可以实现[Federation Exchange](https://www.rabbitmq.com/federated-exchanges.html)和[Federation Queue](https://www.rabbitmq.com/federated-queues.html)两种功能。

**Federation Exchange**

比如，位于两个地区（不同集群）的两个交换机，如果需要两个地区的用户访问两个节点速度一致，不会出现A地区用户访问B地区服务器延迟比较高的情况，需要使用Federation exchange。

将其中一个设置为上游交换机，另一个设置为下游交换机，这样发送到上游交换机的信息会通过federation机制同步到下游交换机。这样下游交换机的消费者访问两个交换机的速度是一致的。

![Federation Plugin](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_13.png)



搭建步骤参考：https://www.cnblogs.com/yeyongjian/p/13964161.html



**Federation Queue**

联邦队列可以在多个Broker节点或集群之间，为单个队列提供负载均衡的功能。一个联邦队列可以连接一个或多个上游队列，并从这些上游队列中获取消息以满足本地消费者消费消息的需求。

![Federation Queue](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/rabbitmq_14.png)



## 10.5 Shovel Plugin

[Shovel plugin](https://www.rabbitmq.com/shovel.html)能够可靠、持续地从一个Broker中的队列（源，source）拉去数据并转发到另一个Broker中的交换器中（目的端，destination）。

源端的队列和目的端的交换机可以同时位于同一个Broker，也可以位于不同的Broker上。

部署方法可以参考官方文档：https://www.rabbitmq.com/shovel.html

