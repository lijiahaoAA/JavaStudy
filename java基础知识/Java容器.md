### Java容器	

***

1. ##### Java容器有哪些？

   Java容器分为两类Collection和Map

   - Collection：List，ArrayList，LinkedList，Set，HashSet，LinkedHashSet，TreeSet，Stack，Vector
   - Map：HashMap，LinkedHashMap，TreeMap，ConcurrentHashMap，Hashtable

2. ##### Collection和Collections有什么区别？

   - Collection是一个集合接口，提供了对集合对象进行基本操作的通用接口方法，所有集合都是它的子类，List，Set...
   - Collections是一个包装类，包含很多静态方法，不能被实例化，类似一个工具类，例如Collections.sort(list)。

3. ##### List，Set，Map之间的区别是什么？

   - List：可重复，有序，Vector线程安全
   - Set：不可重复（equals判断），无序（HashCode决定）
   - Map：不可重复，key-value

   ![区别](https://pic3.zhimg.com/80/v2-c5c9fb0e2e52cd762f1bf1a73216c6c2_720w.jpg)

4. ##### HashMap和HashTable有什么区别？

   - 效率：HashMap是非线程安全的，HashTable是线程安全的，所以HashMap效率高于HashTable。
   - null：HashMap中，null可以作为键，这样的键只有一个，可以有一个或多个键对应的值为null。HashTable只要put进的值只要有一个null，直接抛出NullPointerException。
   - 线程安全：HashTable是同步（经过synchronized修饰）的，HashMap不是，所以HashMap更适合单线程，HashTable适用于多线程，但现在不怎么使用HashTable，应为他是遗留类，有很多内部实现没优化和冗余，且如今可以使用ConcurrentHashMap（引入分段锁）代替。
   - 关于扩容：创建时不指定容量初始值，HashTable默认为11，每次扩容，容量变为原来的2n+1；HashMap默认为16，每次扩容为原来的两倍。创建时给定了容量初始值，HashTable使用给出的，HashMap将其扩充为二的幂次方作为哈希表的大小。
   - 底层数据结构：JDK1.8之后HashMap在解决哈希冲突时有了较大的变化，链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。HashTable没有这样的机制。

5. ##### ConcurrentHashMap和HashTable的区别？

   > 参考: [博客主dreamcatcher-cx]( https://www.cnblogs.com/chengxiao/p/6842045.html )
   >
   > ​		  [GitHub Guide哥]( https://github.com/Snailclimb/JavaGuide )

   - 底层数据结构：JDK1.7底层采用 数组+链表 实现。JDK1.8采用 数组+链表/红黑二叉树 实现。HashTable一直都采用的是 数组+链表 的形式，数组是HashMap的主体，链表主要是解决冲突而存在的。

   - 实现线程安全的方式：

     - ConcurrentHashMap：JDK1.7，ConcurrentHashMap（分段锁）对整个桶数组进行了分割分段（Segment），每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据端的数据就没有锁竞争，提高了并发量。JDK1.8采用 Node节点数组+链表+红黑树 实现，并发控制synchronized和CAS来操作。

     **JDK1.7分段锁：**

      ![img](https://images2015.cnblogs.com/blog/1024555/201705/1024555-20170514174100832-1891630860.png) 

     **JDK1.8实现：TreeBin：红黑二叉树节点 Node：链表节点**

     ![](D:\博客图片\博客园\1584512054149.png)

     - HashTable：使用 `synchronized` 来保证数据安全。效率低下，当一个线程访问同步方法时，其他线程也访问同方法，可能会进入阻塞或轮询的状态。例如put添加元素，另一个线程就不能使用put，也不能使用get，效率低下。

     HashTable实现: ![img](https://images2015.cnblogs.com/blog/1024555/201705/1024555-20170514173954488-1353945142.png) 

6. ##### JDK1.8之前的HashMap底层实现？

   数组+链表结合在一起使用的，也就是散列链表。HashMap通过key的hashCode经过扰动函数处理后得到的hash值，通过`（n-1）&hash` 判断当前元素存放的位置，如果当前位置存在元素，就判断该元素与要存入的元素的hash值以及key是否相同，相同就用新的value覆盖，否则就使用拉链法，加入链表（头插法）。

7. ##### JDK1.8的HashMap底层实现？

   > 参考：[CSND安琪拉]( https://blog.csdn.net/zhengwangzw/article/details/104889549/ )

   数组+链表+红黑树。 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031712385760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZW5nd2FuZ3p3,size_16,color_FFFFFF,t_70) 

   - 判断数组是否为空，为空进行初始化;
   - 不为空，计算 key 的 hash 值，通过(n - 1) & hash (n是数组长度) 计算应当存放在数组中的下标 index;
   - 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；
   - 存在数据，说明发生了hash冲突(存在二个节点key的hash值一样), 继续判断key是否相等，相等，用新的value替换原数据(onlyIfAbsent为false)；
   - 如果不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；
   - 如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于 8， 大于的话链表转换为红黑树，红黑树节点小于6，转化为链表；
   - 插入完成之后判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍。

8. ##### JDK1.8对HashMap做了哪些优化?以及优化的原因？

   > 参考：[CSND安琪拉]( https://blog.csdn.net/zhengwangzw/article/details/104889549/ )

   - 数组+链表改成了数组+链表或红黑树。 
   - 链表的插入方式从头插法改成了尾插法，简单说就是插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后。
   - 扩容的时候1.7需要对原数组中的元素进行重新hash定位在新数组的位置，1.8采用更简单的判断逻辑，位置不变或索引+旧容量大小。
   - 在插入时，1.7先判断是否需要扩容，再插入，1.8先进行插入，插入完成再判断是否需要扩容。

   **优化原因：**

   - 防止发生hash冲突，链表长度过长，将时间复杂度由`O(n)`降为`O(logn)`。
   - 因为1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环。

9. ##### HashMap的长度为什么是2的幂次方？或者扩充为什么2倍？

   为了让HashMap存取高效，应尽量减少碰撞，把数据分配均匀。传入对象到 hashCode() 方法，返回一个int类型的数据。所以hash值的范围=int的范围，为-2^31 到  2^31-1这个范围，前后大概40亿的映射空间，只要哈希函数映射的比较均匀松散，一般很难出现碰撞。但40亿的数据，内存是存不下的，这个散列值不能直接拿来使用。用之前先对数组长度进行取模运算，得到的余数才是用来存放的位置，也就是数组的下标 。这个下标的计算方法为`(n-1)&hash`。这也就解释了为什么HashMap的长度为2的幂次级别（HashMap默认长度为16）。

   ~~~java
   算法设计：取余 （hash值） % (数组长度n)
       而这个取余得到的数值与 （n-1）& hash 值相同
       并且计算机按位“&”运算效率高于“%”
       所以最终选择了下面的方式。
      	运算方式见下：
   ~~~

    ![引用自CSDN安琪拉的博客](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE3LmNuYmxvZ3MuY29tL2Jsb2cvNTA4MzY0LzIwMTcxMi81MDgzNjQtMjAxNzEyMjgxNTU4MDkwNjktMjExOTIzODgwNy5qcGc?x-oss-process=image/format,png)

10. ##### 如何决定使用HashMap还是TreeMap？

   对于在Map中进行插入，删除，定位一个元素这种操作，HashMap是最好的方式，因为它基于散列表实现，通过key查找value。但如果是要对一个key集合进行有序的遍历，选择TreeMap更好，基于[红黑树]( [https://baike.baidu.com/item/%E7%BA%A2%E9%BB%91%E6%A0%91/2413209?fr=aladdin](https://baike.baidu.com/item/红黑树/2413209?fr=aladdin) )。

11. ##### HashMap和HashSet的区别？

    |       HashMap       |                           HashSet                            |
    | :-----------------: | :----------------------------------------------------------: |
    |     实现Map接口     |                         实现Set接口                          |
    |     存储键值对      |                          仅存储对象                          |
    |    put()添加元素    |                        add()添加元素                         |
    | 使用Key计算hashCode | 使用成员对象计算hashCode，对于两个对象来说hashCode可能相同，所以需要使用equals()方法来判断对象是否相同。 |

12. ##### HashMap的实现原理。

    HadhMap是基于Hash实现的，我们通过put(key，value)存储，get(key)来获取，key值不允许有重复。当传入key时，HashMap会根据key.hashCode()计算出hash值，根据哈希值将value保存在bucket中。若计算出的hash值相同时，我们称之为哈希冲突，HashMap的做法是用链表和红黑树存储相同hash值的value，冲突个数比较少时，使用链表，否则使用红黑树。

13. ##### 说一下Hash的表数据结构。

    HashMap通常会用一个指针数组（假设为table[]）来做分散所有的key，当一个key被加入时，会通过Hash算法通过key算出这个数组的下标i，然后就把这个<key, value>插到table[i]中，如果有两个不同的key被算在了同一个i，那么就叫冲突，又叫碰撞，这样会在table[i]上形成一个链表。

    我们知道，如果table[]的尺寸很小，比如只有2个，如果要放进10个keys的话，那么碰撞非常频繁，于是一个O(1)的查找算法，就变成了链表遍历，性能变成了O(n)，这是Hash表的缺陷（可参看《[Hash Collision DoS 问题](https://coolshell.cn/articles/6424.html)》）。

    所以，Hash表的尺寸和容量非常的重要。一般来说，Hash表这个容器当有数据要插入时，都会检查容量有没有超过设定的thredhold，如果超过，需要增大Hash表的尺寸，但是这样一来，整个Hash表里的无素都需要被重算一遍。这叫rehash，这个成本相当的大。

14. ##### HashSet的实现原理。

    HashSet是根据HashMap实现的，HashSet底层使用HashMap来保存所有元素。操作基本上是直接调用HashMap的相关方法来完成，（除了clone()、writeObject()、readObject()必须实现的方法外）且不允许有重复值。

15. ##### HashSet如何实现检查重复？

    HashSet不能添加重复的元素，当调用add（Object）方法时候，
    首先会调用Object的hashCode方法判hashCode是否已经存在，如不存在则直接插入元素；
    如果已存在则调用Object对象的equals方法判断是否返回true， 如果为true则说明元素已经存在，如为false则插入元素。

16. ##### ArrayList和LinkedList的区别。

    - 二者都是不同步的，不保证线程安全。

    - ArrayList是基于动态数组（object数组）实现，LinkedList是基于双向链表的数据结构实现。
    - 随机访问：ArrayList支持根据索引随机访问，LinkedList是线性的数据存储方式，寻找时需要移动指针从前往后查找。
    - 增加删除：LinkedList效率更高，只需要修改指针，而ArrayList需要修改其他数据的下标。
    - 空间占用：ArrayList的空间浪费主要体现在在list结尾会预留一定的容量空间，而LinkedList花费空间体现在他的每一个元素的节点都比ArrayList空间更大，用来存储前驱后继节点信息以及元素数据。

17. ##### 如何实现数组和List之间的转换？

    - 数组转List：使用Arrays.asList(array)。
    - List转数组：List自带的toArray()方法。

18. ##### ArrayList和Vector的区别

    - 线程安全：Vector使用了Synchronized来实现线程同步，所以线程安全。ArrayList非线程安全。
    - 扩容：都是动态调整的，Vector扩容每次增加一倍，ArrayList只会增加50%。

19. ##### Array和ArrayList的区别

    - Array可以存储基本数据类型和对象，ArrayList只能存储对象。
    - Array是指定固定大小的，而ArrayList是自动扩容的。
    - Array的内置方法没有ArrayList多，addAll，removeAll，iteration等方法只有ArrayList有。

20. ##### 在 Queue 中 poll()和 remove()有什么区别？

    - 相同点：都返回第一个元素，并在列队中删除返回的对象。
    - 不同点：如果没有元素poll()会返回null，而remove()会直接抛出NoSuchElementExcrption异常。

21. ##### 那些集合类是线程安全的？

    Vector、Stack、HashTable是线程安全的。HashMap的线程安全类是ConcurrentHashMap。

22. ##### 迭代器Iterator怎么使用？

    ~~~java
    List<String> list = new ArrayList<>();
    Iterator<String> it = list. iterator();
    while(it. hasNext()){
      String obj = it. next();
      System. out. println(obj);
    }
    ~~~

     Iterator 的特点是更加安全，因为它可以确保，在当前遍历的集合元素被更改的时候，就会抛出 ConcurrentModificationException 异常。 

23. ##### 迭代器Iterator是什么？

    Iterator接口提供遍历任何Collection的接口。我们可以从一个Collection中使用迭代器方法来获取迭代器实例。迭代器取代了java集合框架中的Enumeration，迭代器允许调用者在迭代过程中移除元素。

24. ##### Iterator和ListIterator有什么区别？

    - Iterator可以遍历Set和List集合，而ListIterator只能遍历List。
    - Iterator 只能单向遍历，而 ListIterator 可以双向遍历（向前/后遍历） 。
    - ListIterator 从 Iterator 接口继承，然后添加了一些额外的功能，比如添加一个元素、替换一个元素、获取前面或后面元素的索引位置。 

    ~~~java
    public interface ListIterator<E> extends Iterator<E> 
    ~~~

25. ##### 怎么确保一个集合不能被修改？

    可以使用 Collections。

    unmodifiableCollection(Collection c) 方法来创建一个只读集合，这样改变集合的任何操作都会抛出 Java. lang. UnsupportedOperationException 异常。 

    ~~~java
    List<String> list = new ArrayList<>();
    list. add("x");
    Collection<String> clist = Collections. unmodifiableCollection(list);
    clist. add("y"); // 运行时此行报错
    System. out. println(list. size());
    ~~~

18. ##### Arrays工具类的常见操作

    >1. 排序 : `sort()`
    >2. 查找 : `binarySearch()`
    >3. 比较: `equals()`
    >4. 填充 : `fill()`
    >5. 转列表: `asList()`
    >6. 转字符串 : `toString()`
    >7. 复制: `copyOf()`

19. ##### Collections工具类的常见操作

    ```
    void reverse(List list)//反转
    void shuffle(List list)//随机排序
    void sort(List list)//按自然排序的升序排序
    void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
    void swap(List list, int i , int j)//交换两个索引位置的元素
    void rotate(List list, int distance)//旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面。
    ```

20. ##### 如何选用集合?

    主要根据集合的特点来选。

    需要根据键值获取元素值就选用Map接口下面的的集合，需要排序时选择TreeMap，不需要排序时选择HashMap，需要线程安全就选择ConcurrentHashMap。

    只需要存放元素值时，就选择Collection接口的集合，需要元素唯一时选择实现Set接口的集合如HashSet，TreeSet，不需要实现就选择List接口的比如ArrayList或LinkedList。

