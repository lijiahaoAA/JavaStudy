### Kafka

***

1. ##### kafka可以脱离zookeeper单独使用么？为什么？

   kafka不能脱离zookeeper单独使用，因为kafka使用zookeeper管理和协调kafka的节点服务器。

2. ##### 什么是kafka？

    Kafka是一种高吞吐量的[分布式](https://baike.baidu.com/item/分布式/19276232)发布订阅消息系统，它可以处理消费者在网站中的所有动作流数据。 

3. ##### kafka有几种数据保留的策略？

   探究的是kafka的数据生产出来之后究竟落到了那个分区里面去了。

   - 第一种分区策略：给定了分区号，直接将数据发送到指定的分区里面去了。
   - 第二种分区策略：没有给定分区号，给定数据的key，通过key取上hashCode进行分区。
   - 第三种分区策略：既没有给定分区策略，也没有给定数据的key，直接轮询进行分区。
   - 第四种分区策略：自定义分区。

4. ##### kafka同时设置了7天和10G清除数据，到第五天的时候消息到了10G，这个时候kafka会如何处理？

   这个时候kafka会执行数据清除工作，时间和空间无论哪个满足条件，都会清空数据。

5. ##### 怎么设置kafka topic数据存储时间？

   Kafka创建topic命令很简单，一条命令足矣：bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic test 。 

   这条命令会创建一个名为test的topic，有3个分区，每个分区需分配3个副本。 

   topic创建主要分为两个部分：命令行部分+后台(controller)逻辑部分。 

   后台逻辑会监听zookeeper下对应的目录节点，一旦发起topic创建命令，该命令会创建新的数据节点从而触发后台的创建逻辑。 

   确定分区副本的分配方案(就是每个分区的副本都分配到哪些broker上)；创建zookeeper节点，把这个方案写入/brokers/topics/<topic>节点下。 

   确定分区副本的分配方案(就是每个分区的副本都分配到哪些broker上)；创建zookeeper节点，把这个方案写入/brokers/topics/<topic>节点下。 

6. ##### 什么情况会导致kafka运行变慢？

   - 磁盘读写瓶颈。
   - 网络瓶颈。
   - CPU性能瓶颈。

7. ##### 使用kafka集群需要注意什么?

   - 集群的数量最好是单数，因为超过一半故障集群就不能用了，设置为单数容错率高。
   - 集群的数量不是越多越好，最好不要超过7个，因为节点越多，消息复制的时间就越长，整个集群的吞吐量就越低。