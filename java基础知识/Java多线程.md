### Java多线程

****

1. ##### 并行和并发有什么区别？

   - 并行：两个或多个事件在同一时刻发生。是不同实体上的多个事件。
   - 并发：两个或多个事件在同一时间间隔发生。是同一实体上的多个事件。

2. ##### 线程和进程的区别

   - 性质不同：进程是计算机中的程序关于某数据集合的一次运行活动，是系统进行资源(指的就是计算机里的中央处理器，内存，文件等等)分配和调度的基本单位，是操作系统结构的基础。线程是操作系统能够进行运算调度的最小单位；他被包含在进程之中，是进程中的实际运作单位。
   - 同类的线程可以共享进程的堆和方法区，但每个线程都有自己的程序计数器、虚拟机栈和本地方法栈，所以系统在产生一个线程，或是在各个线程线程之间切换工作时，负担要比进程小得多。

   [阮一峰解释]( http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html )

3. ##### 守护线程是什么？

   守护线程是运行在后台的一种特殊进程，它**独立于控制终端并且周期性的执行某种任务或等待处理某些发生的事件**。例如java中的垃圾回收线程就是守护线程。

4. ##### 创建线程的几种方式。

   - 继承Thread重写run方法。

   - 实现Runnable接口（里面只有一个抽象方法run），Thread就是实现了Runnable接口。

     ~~~java
     public interface Runnable {
         public abstract void run();
     }
     ~~~

   - 实现Callable接口（里面只有一个call方法）。

     ~~~java
     public interface Callable<V> {
         V call() throws Exception;
     }
     ~~~

5. ##### Runnable和Callable的区别

   - 实现Callable接口的任务线程能返回执行结果；而实现Runnable接口的任务线程不能返回结果。
   - Callable接口的call()方法允许抛出异常；而Runnable接口的run()方法的异常只能在内部消化，不能继续上抛； 

6. ##### 线程有那些状态？

   - NEW：尚未启动
   - RUNNABLE：正在执行中
   - BLOCKED：阻塞状态
   - WAITING：永久等待状态
   - TIMED_WAITING：等待指定的时间重新被唤醒的状态
   - TERMINATED：执行完成

7. ##### sleep()和wait()的区别？

   - sleep来自于Thread，wait来自于Object类。
   - **sleep不释放锁，wait释放锁。**
   - sleep到时间会自动恢复，wait可以使用`notify()`或者`notifyAll()`唤醒。
   - sleep()方法通常用于暂停执行，wait()方法通常用于线程间交互/通信。

8. ##### 线程的run和start两个方法有什么区别？

   start() 方法用于启动线程，run() 方法用于执行线程的运行时代码。run() 可以重复调用，而 start() 只能调用一次。 

9. ##### 为什么我们调用 start() 方法时会执行 run() 方法，为什么不能直接执行 run() 方法？

   new一个 Thread ，线程就会进入新建状态；调用start()，会启动一个线程并使线程进入了就绪状态，当分配到时间片就可以开始运行了。start()会执行线程的相应准备工作，然后自动执行run()方法的内容，这是真正的多线程工作。而直接进行run()的调用，会把run()方法当作一个main()线程下的普通方法去执行，也就是在运行在主线程中，代码在程序中是顺序执行的，所以不会有解决耗时操作的问题。

   只有子线程开始了，才会有异步的效果。

10. ##### 创建线程池的方式

   线程池创建有七种方式，最核心的是最后一种：

   - `newSingleThreadExecutor()`：它的特点在于工作线程数目被限制为 1，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处于活动状态，并且不允许使用者改动线程池实例，因此可以避免其改变线程数目；
   - `newCachedThreadPool()`：它是一种用来处理大量短时间工作任务的线程池，具有几个鲜明特点：它会试图缓存线程并重用，当无缓存线程可用时，就会创建新的工作线程；如果线程闲置的时间超过 60 秒，则被终止并移出缓存；长时间闲置时，这种线程池，不会消耗什么资源。其内部使用` SynchronousQueue `作为工作队列；
   - `newFixedThreadPool(int nThreads)`：重用指定数目（`nThreads`）的线程，其背后使用的是无界的工作队列，任何时候最多有 `nThreads `个工作线程是活动的。这意味着，如果任务数量超过了活动队列数目，将在工作队列中等待空闲线程出现；如果有工作线程退出，将会有新的工作线程被创建，以补足指定的数目 `nThreads`；
   - `newSingleThreadScheduledExecutor()`：创建单线程池，返回 `ScheduledExecutorService`，可以进行定时或周期性的工作调度；
   - `newScheduledThreadPool(int corePoolSize)`：和`newSingleThreadScheduledExecutor()`类似，创建的是个 `ScheduledExecutorService`，可以进行定时或周期性的工作调度，区别在于单一工作线程还是多个工作线程；
   - `newWorkStealingPool(int parallelism)`：这是一个经常被人忽略的线程池，Java 8 才加入这个创建方法，其内部会构建`ForkJoinPool`，利用Work-Stealing算法，并行地处理任务，不保证处理顺序；
   - `ThreadPoolExecutor()`：是最原始的线程池创建，上面1-3创建方式都是对`ThreadPoolExecutor`的封装。

11. ##### 线程池有哪些状态？

    - RUNNING：正常状态，接受新任务，处理等待队列中的任务。

    - SHUTDOWN：不接受新的任务提交，但是会继续处理等待队列中的任务。

    - STOP：不接受新的任务提交，不在处理等待队列中的任务，中断正在执行的线程。

    - TIDYING：所有的任务都被销毁了，`workCount`为零，线程池的状态在转换为TIDYING状态时，回执行钩子方法terminated()。

      [钩子方法]( https://www.cnblogs.com/yanlong300/p/8446261.html )

12. ##### 线程池中的submit()和execute()有什么区别？

    - execute()：只能执行 Runnable 类型的任务。
    - submit()：可以执行 Runnable 和 Callable 类型的任务。

13. ##### 在Java程序中如何保证多线程的运行安全？

    - 使用安全类，`java.util.concurrent`下的类。
    - 使用自动锁synchronized。
    - 使用手动锁Lock。

       ~~~java
    Lock lock = new ReentrantLock();
    lock. lock();
    try {
        System. out. println("获得锁");
    } catch (Exception e) {
        // TODO: handle exception
    } finally {
        System. out. println("释放锁");
        lock. unlock();
    }
       ~~~

14. ##### synchronized解决了什么问题?

    解决的是多个线程之间访问资源的同步性问题，synchronized关键字可以保证它修饰的方法或者代码块在任意时刻只能有一个线程执行。

15. ##### 说一下synchronized底层实现原理。

    synchronized是由一对`monitorenter/monitorexit`指令实现的，monitor(监视器锁)对象是同步的基本实现单元。而监视器锁依赖于底层操作系统的互斥锁实现，Java线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙实现。而操作系统实现线程之间的切换需要从用户态转到内核态，这就需要相对较长的时间，所以早期的synchronized效率低下。JDK1.6对synchronized做了优化，如自旋锁，偏向锁，轻量级锁。

16. ##### synchronized和volatile的区别是什么？

    - volatile是变量修饰符，synchronized是修饰类，方法，代码段。
    - volatile仅能实现变量的修改可见性（`cpu`高速缓存向主存的同步），不能保证原子性；而synchronized可以保证变量的可见性和原子性。
    - volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。

    [volatile关键字]( https://www.cnblogs.com/dolphin0520/p/3920373.html )

17. ##### synchronized和Lock有什么区别？

    - Lock只能给代码块加锁，而synchronized可以给类，方法，代码段加锁。
    - synchronized是自动锁，不需要手动获取和释放锁，使用简单，发生异常时自动释放锁，不会造成死锁。Lock锁是手动锁，需要自己加锁和释放锁，如果使用不当没有unlock()去释放锁就会造成死锁。
    - 通过Lock可以知道有没有成功获取锁，synchronized不可以。

18. ##### synchronized和`ReentrantLock`有什么区别？

    - `ReentrantLock`使用灵活(实现了Lock接口)，但是必须有释放锁的配合动作。
    - `ReentrantLock`也是手动锁，synchronized是自动锁。
    - `ReentrantLock`只能给代码块加锁，而synchronized可以给类，方法，代码段加锁。

19. ##### 多线程中synchronized锁升级的原理是什么？

    **synchronized 锁升级原理**：在锁对象的对象头里面有一个 `threadid` 字段，在第一次访问的时候` threadid` 为空，`jvm `让其持有偏向锁，并将 `threadid` 设置为其线程 id，再次进入的时候会先判断 `threadid` 是否与其线程 id 一致，如果一致则可以直接使用此对象，如果不一致，则升级偏向锁为轻量级锁，通过自旋循环一定次数来获取锁，执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会把锁从轻量级升级为重量级锁，此过程就构成了 synchronized 锁的升级。

    **锁的升级的目的**：锁升级是为了减低了锁带来的性能消耗。在 Java 6 之后优化 synchronized 的实现方式，使用了偏向锁升级为轻量级锁再升级到重量级锁的方式，从而减低了锁带来的性能消耗。

    [简单阐述]( https://www.cnblogs.com/ConstXiong/p/11687975.html )

20. ##### 说一下atomic的原理。

    atomic主要利用CAS（Compare And Swap）乐观锁（不会进行内核态的操作）和volatile和native方法来保证原子性，从而避免synchronized的高开销（用户态和内核态的切换），执行效率大为提升。

21. ##### 什么是死锁？

    线程A持有独占锁a，并尝试获取独占锁b的同时，线程B持有独占锁b，并尝试获取独占锁a，会发生AB两个线程都在等待对方释放锁，而发生阻塞的现象，称之为死锁。

22. ##### 产生死锁必须具备的条件？

    - 互斥条件：该资源任意时刻只由一个线程占用。
    - 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
    - 不剥夺条件：线程以获得的资源在没有使用完之前不能被其他线程强行剥夺，只能自己使用完释放。
    - 循环等待条件：若干进程循环等待资源。

23. ##### 怎么防止死锁？

    对必须的条件只要破坏一个就行（互斥破不了，单个CPU一次只能处理一个线程，分配时间片）

    - 一次性申请所有资源（破坏请求与保持条件）。
    - 占用部分资源的线程申请其他资源时，获取不到，可以主动释放自己占有的资源（破坏不剥夺条件）。
    - 按某一顺序申请资源，释放资源则反序释放（破坏循环等待条件）。

    **具体做法：**

    - 对锁设置超时时间，超时就退出竞争防止死锁。
    - 尽量使用`java.util.concurrent`并发类代替手写锁。
    - 尽量降低锁的使用粒度，尽量不要几个功能使用同一把锁。
    - 尽量减少同步的代码块。

24. ##### `ThreadLocal`是什么？有哪些使用场景？

    `ThreadLocal` 为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。 