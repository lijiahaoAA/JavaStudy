### Redis

***

1. ##### Redis是什么? 有哪些使用场景?

   Redis是一个使用C语言开发的高速缓存数据库。

   应用场景：

   - 记录帖子点赞数、点击数、评论数。
   - 缓存近期热帖。
   - 缓存文章详细信息。
   - 记录用户会话信息。

2. ##### Redis有哪些功能？

   - 数据缓存功能。
   - 分布式锁的功能。
   - 支持数据持久化。
   - 支持事务。
   - 支持消息队列。

3. ##### redis事务是怎么实现的？

   redis通过MULTI，EXEC，WATCH等命令来实现事务功能。事务提供了一种将多个命令请求打包，然后一次性、按顺序的执行多个命令的机制，并且在事务执行期间，服务器不会中断事务而去执行其他客户端的命令，它会将事务中所有命令都执行完毕，然后才去处理其他客户端的请求。

4. ##### redis的线程模型。

   redis内部使用文件事件处理器`file event handler`，这个文件事件处理器是单线程的，所以redis才叫做单线程的模型。它采用IO多路复用机制监听多个socket，根据socket上的事件来选择对应的事件处理器进行处理。

   **文件事件处理器的结构：**

   - 多个socket
   - IO多路复用程序
   - 文件事件分派器
   - 事件处理器（连接应答处理器，命令请求处理器，命令回复处理器）

   多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。 

5. ##### Redis和memcache有什么区别？

   - 存储方式不同：memcache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小；Redis有部分存在磁盘上，这样能保证数据的持久性。
   - 数据支持类型：memcache对数据类型支持相对简单（k-v）；Redis有复杂的数据类型支持（k-v，list，set，zset，hash）。
   - 使用底层模型不同：他们之间底层实现方式，以及客户端之间通信的应用协议不同，Redis自己构建了vm机制，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
   - value值不同：Redis最大可达到512MB，memcache只有1MB。
   - memcached是多线程，非阻塞IO复用的模型，redis使用单线程的多路IO复用机制。
   - redis支持持久化RDB，AOF。

6. ##### Redis为什么是单线程的？

   因为 cpu 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存或者网络带宽。既然单线程容易实现，而且 cpu 又不会成为瓶颈，那就顺理成章地采用单线程的方案了。**采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗 。**

   而且单线程并不代表就慢 ，nginx 和 nodejs 也都是高性能单线程的代表。

7. ##### redis持久化机制（怎么保证redis挂掉之后重启数据可以进行恢复）

   - 快照持久化（RDB）：redis可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。创建快照之后，可以对快照进行备份、可以将快照复制到其他服务器。还可以将快照留在原地以便重启服务器时候使用。
   - AOF持久化：主流的方式，通过appendonly参数开启，开启之AOF持久化后每执行一条更改redis中的数据命令，redis就会将该命令写入硬盘中的AOF文件。

8. ##### 什么是缓存穿透？怎么解决？

   缓存穿透：指查询一个一定不存在的数据，由于缓存是不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。

   **解决方案：**最简单粗暴的方法如果查询一个返回数据为空（不管是数据不存在，还是系统故障），我们就把这个空结果进行缓存，但他的过期时间会很短，最长不超过5分钟。

   **布隆过滤器：**

   > [可以参考]( https://github.com/Snailclimb/JavaGuide/blob/master/docs/dataStructures-algorithms/data-structure/bloom-filter.md )

   可以非常方便的判断一个给定数据是否存在于海量数据中。只需做判断key是否合法。

9. ##### 什么是缓存雪崩?怎么解决?

   缓存同一时间大面积失效，大部分的请求落在数据库上，造成数据库短时间内承受大量请求而崩掉。

   **解决办法：**

   - 事前：尽量保证整个redis集群的高可用性，发现机器宕机尽快补上。选择适合的内存淘汰策略。
   - 事中：本地ehcache缓存+hystrix限流&降级，避免MySQL崩掉。
   - 事后：利用redist持久化机制保存的数据尽快恢复缓存。

10. ##### Redis支持的数据类型有哪些？

   - string字符串（k-v）
   - list列表
   - hash字典
   - set集合
   - zset有序集合
   
11. ##### Redis支持的Java客户端有哪些？

    Redisson，jedis，lettuce。。。

12. ##### jedis和Redisson有哪些区别？

    - jedis：提供了比较全面的Redis命令的支持。
    - Redisson：实现了分布式和可扩展的Java数据结构，功能相对简单，不支持排序、事务、管道、分区等Redis特性。

13. ##### 怎么保证缓存和数据库数据的一致性？

    - 合理设置缓存的过期时间。
    - 新增，更改，删除数据库操作同时同步跟新Redis，可以使用事务机制来保证了数据的一致性。

14. ##### Redis持久化有几种方式？

    - RDB：Redis Database，指定的时间间隔能对你的数据进行快照存储。
    - AOF：Append Only File，每一个收到的写命令都通过write函数追加到文件中。

15. ##### Redis怎么实现分布式锁？

    Redis 分布式锁其实就是在系统里面占一个“坑”，其他程序也要占“坑”的时候，占用成功了就可以继续执行，失败了就只能放弃或稍后重试。

    占坑一般使用 setnx(set if not exists)指令，只允许被一个程序占有，使用完调用 del 释放锁。

16. ##### Redis分布式锁有什么缺陷？

    Redis分布式锁不能解决超时的问题，分布式锁有一个超时的时间，程序的执行如果超出了锁的超时时间就会出现问题。

17. ##### Redis如何做内存优化？

    尽量使用 Redis 的散列表，把相关的信息放到散列表里面存储，而不是把每个字段单独存储，这样可以有效的减少内存使用。比如将 Web 系统的用户对象，应该放到散列表里面再整体存储到 Redis，而不是把用户的姓名、年龄、密码、邮箱等字段分别设置 key 进行存储。 

18. ##### Redis淘汰策略有哪些？

    volatile-lru：从已设置过期时间的数据集（server. db[i]. expires）中挑选**最近最少使用**的数据淘汰。

    volatile-ttl：从已设置过期时间的数据集（server. db[i]. expires）中挑选**将要过期**的数据淘汰。

    volatile-random：从已设置过期时间的数据集（server. db[i]. expires）中任意**选择**数据淘汰。

    allkeys-lru：内存不足以容纳新写入数据时，从数据集（server. db[i]. dict）中挑选**最近最少使用**的数据淘汰，移除最近最少使用的key。

    allkeys-random：从数据集（server. db[i]. dict）中**任意**选择数据淘汰。

    no-enviction（驱逐）：**禁止**驱逐数据。当内存不足以容纳新写入数据时，写入操作会报错。

19. ##### 单线程的Redis为什么这么快？

    - 纯内存操作。
    - 单线程操作，避免了频繁的山下文切换。
    - 采用了非阻塞I/O多路复用机制。

20. ##### Redis常见的性能问题有哪些？该如何解决？

    - 主服务器写内存快照，会阻塞主线程的工作，当快照比较大时对性能的影响是非常大的。
    - Redis主从复制的性能问题，为了主从复制的速度和连接的稳定性，主从库最好在同一个局域网内。