### Zookeeper

***

1. ##### zookeeper是什么？

   - `ZooKeeper`是一个**分布式**的，开放源码的分布式**应用程序协调服务**，是Google的Chubby一个开源的实现，它是**集群的管理者**，**监视着集群中各个节点的状态根据节点提交的反馈进行下一步合理操作**。最终，将简单易用的接口和性能高效、功能稳定的系统提供给用户。
   - 客户端的**读请求**可以被集群中的**任意一台机器处理**，如果读请求在节点上注册了监听器，这个监听器也是由所连接的zookeeper机器来处理。对于**写请求**，这些请求会同**时发给其他zookeeper机器并且达成一致后，请求才会返回成功**。因此，随着**zookeeper的集群机器增多，读请求的吞吐会提高但是写请求的吞吐会下降**。
   - 有序性是zookeeper中非常重要的一个特性，所有的**更新都是全局有序的**，每个更新都有一个**唯一的时间戳**，这个时间戳称为`zxid`（Zookeeper Transaction Id）**。而**读请求只会相对于更新有序**，也就是读请求的返回结果中会带有这个**zookeeper最新的`zxid`。

2. ##### zookeeper 都有哪些功能？

   - 集群管理：监控节点存活状态，运行请求等。
   - 主节点选举：主节点挂掉了之后可以从备用的节点开始新一轮选主，主节点选举说的就是这个选举的过程，使用 zookeeper 可以协助完成这个过程。 
   - 分布式锁：zookeeper提供两种锁：独占锁、共享锁。独占锁即一次只能占有一个线程使用资源，共享锁是读写共享，读写互斥，即可以有多线程同时读取一个资源，如果要使用写锁也只能有一个线程使用。zookeeper可以对分布式锁进行控制。
   - 统一命名服务：在分布式系统中，通过使用命名服务，客户端应用能够根据指定名字来获取资源或者服务的地址，提供者信息等。
   - 配置管理：将配置信息保存在Zookeeper的某个目录节点中，一旦配置信息发生变化,每台应用机器就会收到`ZooKeeper`的通知，然后从Zookeeper获取新的配置信息应用到系统中。 
   - 队列管理：当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达，这种是同步队列。队列按照 FIFO 方式进行入队和出队操作，例如实现生产者和消费者模型。 

3. ##### zookeeper 有几种部署模式？

   zookeeper 有三种部署模式：

   - 单机部署：一台集群上运行；
   - 集群部署：多台集群运行；
   - 伪集群部署：一台集群启动多个 zookeeper 实例运行。

4. ##### zookeeper 怎么保证主从节点的状态同步？

   zookeeper 的核心是原子广播，这个机制保证了各个 server 之间的同步。实现这个机制的协议叫做 `zab` 协议。 `zab` 协议有两种模式，分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导者崩溃后，`zab` 就进入了恢复模式，当领导者被选举出来，且大多数 server 完成了和 leader 的状态同步以后，恢复模式就结束了。状态同步保证了 leader 和 server 具有相同的系统状态。 

5. ##### ZAB协议工作机制？

   ZAB主要是用来实现保持各集群中主备副本之间的**数据一致性**。

   **当 Leader 服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB 协议就会进人恢复模式并选举产生新的Leader服务器。这个过程大致是这样的：**

   1. Leader election（选举阶段）：节点在一开始都处于选举阶段，只要有一个节点得到超半数节点的票数，它就可以当选准 leader。
   2. Discovery（发现阶段）：在这个阶段，followers 跟准 leader 进行通信，同步 followers 最近接收的事务提议。
   3. Synchronization（同步阶段）：同步阶段主要是利用 leader 前一阶段获得的最新提议历史，同步集群中所有的副本。同步完成之后 准 leader 才会成为真正的 leader。
   4. Broadcast（广播阶段）：到了这个阶段，Zookeeper 集群才能正式对外提供事务服务，并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。

6. ##### 集群中为什么要有主节点？

   在分布式环境中，有些业务逻辑只需要集群中的某一台机器进行执行，其他的机器可以共享这个结果，这样可以大大减少重复计算，提高性能，所以就需要主节点。 

7. ##### 集群中有三台服务器，其中一个节点宕机，这个时候zookeeper 还可以继续使用么？

   可以继续使用，单数服务器只要没超过一半的服务器宕机就可以继续使用。 

8. ##### 说一下`zookeeper`的通知机制。

   client端会对某个`znode（数据节点）`建立一个**watcher事件**，当该`znode`发生变化时，这些client会收到`zookeeper`的通知，然后client可以根据`znode`变化来做出业务上的改变等。 

9. ##### ZAB和PAXOS算法的相同和区别？

   - 相同点：

     - 两者都存在一个类似于Leader进程的角色，由其负责协调多个Follower进程的运行。
     
     - Leader进程都会等待超过半数的Follower做出正确的反馈后，才会将一个提案进行提交。
   
   - 不同点：
   
     - ZAB协议中，每个Proposal中都包含一个 epoch 值来代表当前的Leader周期，`Paxos`中名字为Ballot。
   
     - ZAB用来构建高可用的分布式数据主备系统（Zookeeper），`Paxos`是用来构建分布式一致性状态机系统。
   
     - 在`Paxos`算法中，一个新选举产生的主进程会进行两个阶段的工作。第一阶段被称为读阶段，在这个阶段中，这个新的主进程会通过和所有其他进程进行通信的方式来收集上一个主进程的提案，并将他们提交。第二阶段被称为写阶段，在这个阶段，当前主进程开始提出他自己的提案。 