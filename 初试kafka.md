### 初试kafka

#### 目标"kafka的默认配置运行以及实现topic的创建,消息的生产及订阅

### 一kafka的介绍

#### 1.概述

Kafka是最初由Linkedin公司开发，是一个分布式、分区的、多副本的、多订阅者，基于zookeeper协调的分布式日志系统（也可以当做MQ系统），常见可以用于web/nginx日志、访问日志，消息服务等等，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。

主要应用场景是：日志收集系统和消息系统。

Kafka主要设计目标如下：

- 以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间的访问性能。
- 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒100K条消息的传输。
- 支持Kafka Server间的消息分区，及分布式消费，同时保证每个partition内的消息顺序传输。
- 同时支持离线数据处理和实时数据处理。
- Scale out:支持在线水平扩展

#### 2.kafka的优点

- 解耦:生产者和消费者可以独立的修改扩展,只要满足接口约定即可.
- 顺序保证:消息队列是具有有序性的,而在大多数环境下数据处理的顺序都很重要
- 可恢复性:系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。
- 缓冲:例如，加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行———写入队列的处理会尽可能的快速。该缓冲有助于控制和优化数据流经过系统的速度。
- 异步通信:很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

### 二kafka的安装使用

#### 1.下载安装包

由于最新版kafka内置了zookeeper,所以不需要额外去安装zookeeper

http://mirror.bit.edu.cn/apache/kafka/2.3.1/kafka_2.12-2.3.1.tgz

tgz文件只需解压到目标文件下就行:

此处我解压在D:\kafka\kafka_2.12-2.3.1\kafka_2.12-2.3.1

#### 2.使用默认配置启动zookeeper

默认端口2181

进入到上面的解压路径下:

cmd下

```
bin\windows\zookeeper-server-start.bat config\zookeeper.properties
```

![image-20191210152028678](C:\Users\w1660\AppData\Roaming\Typora\typora-user-images\image-20191210152028678.png)

#### 3.启动kafka

新建cmd

```
bin\windows\kafka-server-start.bat config\server.properties
```

![image-20191210151916437](C:\Users\w1660\AppData\Roaming\Typora\typora-user-images\image-20191210151916437.png)

#### 4.创建topic主题

端口号为9092 topic 名称为demo

```
bin\windows\kafka-topics.bat --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic demo
```

查看创建的topic

```
bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```

![image-20191210152311070](C:\Users\w1660\AppData\Roaming\Typora\typora-user-images\image-20191210152311070.png)

#### 5.启动生产者producer

声明主题为demo

```
bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic demo
```

进入页面后发送消息:   测试 你好

#### 6.启动消费者 customer

订阅主题为demo

```
bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic demo --from-beginning
```

![image-20191210152632222](C:\Users\w1660\AppData\Roaming\Typora\typora-user-images\image-20191210152632222.png)

**此处由于使用默认配置config所以输入中文乱码,后期有待改进改**

### 三java代码实现本地的kafka通信

#### 1.导入jar包依赖

```xml
<dependency>    
    <groupId>org.apache.kafka</groupId>    
    <artifactId>kafka-clients</artifactId>   
    <version>2.3.0</version>
</dependency>
```

#### 2.创建topic类

**在执行下面操作前必须先启动zookeeper 和KafkaServer**

```
public class Topic {
    /**
     * AdminClient可以控制对kafka服务器配置,我们这里使用NewTopics("topic_test",1,(short)1)
     * 创建一个名为topic_test ,分区数为1,复制因子为1的topic
     */
    public static void createTopic(){
        Properties props=new Properties();
        props.put("bootstrap.servers","localhost:9092");
        AdminClient adminClient=AdminClient.create(props);
        ArrayList<NewTopic> topics=new ArrayList<NewTopic>();
        NewTopic newTopic=new NewTopic("topic_test",1,(short)1);
        topics.add(newTopic);
        CreateTopicsResult result=adminClient.createTopics(topics);
        try {
            result.all().get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    /**
     * 删除topic
     * todo 未删除成功
     */
    public static void deleteTopic(){
        Properties props=new Properties();
        props.put("bootstrap.servers","localhost:9092");
        AdminClient adminClient=AdminClient.create(props);
        Collection<String> collection=new HashSet<String>();
        collection.add("test");
        adminClient.deleteTopics(collection);
    }

    /**
     * 展示当前端口所有的topic
     */
    public static void showTopics(){
        Properties props=new Properties();
        props.put("bootstrap.servers","localhost:9092");
        AdminClient adminClient=AdminClient.create(props);
        ListTopicsResult listTopicsResult = adminClient.listTopics();
        KafkaFuture<Set<String>> names = listTopicsResult.names();
        Set<String> strings;
        try {
            strings = names.get();
            for(String s:strings){
                System.out.println(s);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println(listTopicsResult);
    }
    public static void main(String[] args) {
        createTopic();
    }
}
```

#### 3.创建生产者类

```java
public class ProducerPojo {
    public static void sendMessage(String msg){
        Properties props=new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"localhost:9092");
        props.put("acks","all");
        props.put("retries",0);
        props.put("batch.size",16384);
        props.put("linger.ms",1);
        props.put("buffer.memory",33554432);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());
        //生产者对象
        Producer<String,String> producer=new KafkaProducer<String, String>(props);
        producer.send(new ProducerRecord<String, String>("demo",msg));
        System.out.println("已发送");
        producer.close();
    }

    public static void main(String[] args) {
        sendMessage("hello veryOne");
    }
}
```

#### 4.创建消费者类

```java
public class ProducerPojo {
    public static void sendMessage(String msg){
        Properties props=new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"localhost:9092");
        props.put("acks","all");
        props.put("retries",0);
        props.put("batch.size",16384);
        props.put("linger.ms",1);
        props.put("buffer.memory",33554432);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());
        //生产者对象
        Producer<String,String> producer=new KafkaProducer<String, String>(props);
        producer.send(new ProducerRecord<String, String>("demo",msg));
        System.out.println("已发送");
        producer.close();
    }

    public static void main(String[] args) {
        sendMessage("hello veryOne");
    }
}
```

### 四springboot整合kafka

