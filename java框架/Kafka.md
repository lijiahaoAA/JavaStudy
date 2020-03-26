### Kafka

***

1. ##### kafka可以脱离zookeeper单独使用么？为什么？

   kafka不能脱离zookeeper单独使用，因为kafka使用zookeeper管理和协调kafka的节点服务器。

2. ##### zookeeper在kafka中的作用？

   > [简书_博弈史密斯]( https://www.jianshu.com/p/a036405f989c )

   - Borker注册：**Broker是分布式部署并且相互之间独立，但是需要有一个注册系统能够将整个集群中的Broker管理起来**，此时就使用到了Zookeeper。在Zookeeper上会有一个专门**用来进行Broker服务器列表记录**的节点：

     /brokers/ids

     每个Broker在启动时，都会到Zookeeper上进行注册，即到/brokers/ids下创建属于自己的节点，如/brokers/ids/[0...N]。

     Kafka使用了全局唯一的数字来指代每个Broker服务器，不同的Broker必须使用不同的Broker ID进行注册，创建完节点后，**每个Broker就会将自己的IP地址和端口信息记录**到该节点中去。其中，Broker创建的节点类型是临时节点，一旦Broker宕机，则对应的临时节点也会被自动删除。

   - Topic注册：在Kafka中，同一个**Topic的消息会被分成多个分区**并将其分布在多个Broker上，**这些分区信息及与Broker的对应关系**也都是由Zookeeper在维护，由专门的节点来记录，如：

     /borkers/topics

     Kafka中每个Topic都会以/brokers/topics/[topic]的形式被记录，如/brokers/topics/login和/brokers/topics/search等。Broker服务器启动后，会到对应Topic节点（/brokers/topics）上注册自己的Broker ID并写入针对该Topic的分区总数，如/brokers/topics/login/3->2，这个节点表示Broker ID为3的一个Broker服务器，对于"login"这个Topic的消息，提供了2个分区进行消息存储，同样，这个分区节点也是临时节点。

   - 负载均衡：zookeeper可以根据当前的消费者数量和分区数量首先动态负载均衡。

3. ##### 什么是kafka？

    Kafka是一个分布式流式处理平台。

    是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者在网站中的所有动作流数据。 

    流平台具有的三个关键功能：

    - 消息队列：发布和订阅消息，kafka的主要用途。
    - 容错的持久方式存储记录消息流：kafka会把消息持久化到磁盘。
    - 流式平台处理：在消息发布的时候进行处理，kafka提供了一个完整的流式处理类库。

4. ##### 说一下发布-订阅模型

   首先，发布-订阅模型是为了解决队列模型存在的问题（我们需要将生产者产生的消息分发给多个消费者并且每个消费者都能接收到完成的消息内容）。

   发布订阅模型使用主题作为消息通信载体，类似于广播模式：发布者发布一条消息，将该消息通过主题传递给所有的订阅者，这之后才订阅主题的消费者是收不到这条消息的。

   若只有一个订阅主题的消费者 == 队列模型

   kafka没有队列的概念，与之对应的是partition分区。

5. ##### 说一下kafka的消息模型。

   kafka将生产者发布的消息发送到Topic中，需要这些消息的消费者可以订阅这些Topic。如下图：

   ![img](https://img2018.cnblogs.com/blog/1325651/201907/1325651-20190718235525920-1173211187.png)

   - Producer：生产者生产消息。
   - Topic：主题，生产者将消息发送到特定的主题，消费者通过订阅主题消费信息。一个Topic可以有多个Partition，并且同一Topic下的Partition可以分布在不同的Broker上。一个Topic可以跨越多个Broker。
   - Consumer：消费者消费信息。
   - Partition：Topic的一部分。
   - Broker：代理，可以看做一个独立的Kafka实例，包含多个Topic。

6. ##### kafka多副本机制了解么？

   kafka为partition引入了多副本机制，分区Partition中的多个副本之间会有一个叫做leader的家伙，其他副本成为follower。我们发送的消息会被发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步。

7. ##### kafka有几种数据保留的策略？

   探究的是kafka的数据生产出来之后究竟落到了那个分区（partition）里面去了。

   - 第一种分区策略：给定了分区号，直接将数据发送到指定的分区里面去了。
   - 第二种分区策略：没有给定分区号，给定数据的key，通过key取上hashCode进行分区。
   - 第三种分区策略：既没有给定分区策略，也没有给定数据的key，直接轮询进行分区。
   - 第四种分区策略：自定义分区。

   前两种方式可以保证消息的消费顺序（Partition是真正保存消息的地方）。

8. ##### kafka同时设置了7天和10G清除数据，到第五天的时候消息到了10G，这个时候kafka会如何处理？

   这个时候kafka会执行数据清除工作，时间和空间无论哪个满足条件，都会清空数据。

9. ##### 怎么设置kafka topic数据存储时间？

   Kafka创建topic命令很简单，一条命令足矣：bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic test 。 

   这条命令会创建一个名为test的topic，有3个分区，每个分区需分配3个副本。 

   topic创建主要分为两个部分：命令行部分+后台(controller)逻辑部分。 

   后台逻辑会监听zookeeper下对应的目录节点，一旦发起topic创建命令，该命令会创建新的数据节点从而触发后台的创建逻辑。 

   确定分区副本的分配方案(就是每个分区的副本都分配到哪些broker上)；创建zookeeper节点，把这个方案写入/brokers/topics/<topic>节点下。 

10. ##### 什么情况会导致kafka运行变慢？

   - 磁盘读写瓶颈。
   - 网络瓶颈。
   - CPU性能瓶颈。

11. ##### 使用kafka集群需要注意什么?

    - 集群的数量最好是单数，因为超过一半故障集群就不能用了，设置为单数容错率高。
    - 集群的数量不是越多越好，最好不要超过7个，因为节点越多，消息复制的时间就越长，整个集群的吞吐量就越低。