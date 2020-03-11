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

   - HashMap是非线程安全的，HashTable是线程安全的，所以HashMap效率高于HashTable。
   - HashMap允许键值为null，HashTable不可以。
   - HashTable是同步的，HashMap不是，所以HashMap更适合单线程，HashTable适用于多线程，但现在不怎么使用HashTable，应为他是遗留类，有很多内部实现没优化和冗余，且如今可以使用ConcurrentHashMap代替。

5. ##### 如何决定使用HashMap还是TreeMap？

   对于在Map中进行插入，删除，定位一个元素这种操作，HashMap是最好的方式，因为它基于散列表实现，通过key查找value。但如果是要对一个key集合进行有序的遍历，选择TreeMap更好，基于[红黑树]( [https://baike.baidu.com/item/%E7%BA%A2%E9%BB%91%E6%A0%91/2413209?fr=aladdin](https://baike.baidu.com/item/红黑树/2413209?fr=aladdin) )。

6. ##### HashMap的实现原理。

   HadhMap是基于Hash实现的，我们通过put(key，value)存储，get(key)来获取，key值不允许有重复。当传入key时，HashMap会根据key.hashCode()计算出hash值，根据哈希值将value保存在bucket中。若计算出的hash值相同时，我们称之为哈希冲突，HashMap的做法是用链表和红黑树存储相同hash值的value，冲突个数比较少时，使用链表，否则使用红黑树。

7. ##### HashSet的实现原理。

   HashSet是根据HashMap实现的，HashSet底层使用HashMap来保存所有元素。操作基本上是直接调用HashMap的相关方法来完成，且不允许有重复值。

8. ##### ArrayList和LinkedList的区别。

   - ArrayList是基于动态数组实现，LinkedList是基于双向链表的数据结构实现。
   - 随机访问：ArrayList支持根据索引随机访问，LinkedList是线性的数据存储方式，寻找时需要移动指针从前往后查找。
   - 增加删除：LinkedList效率更高，只需要修改指针，而ArrayList需要修改其他数据的下标。

9. ##### 如何实现数组和List之间的转换？

   - 数组转List：使用Arrays.asList(array)。
   - List转数组：List自带的toArray()方法。

10. ##### ArrayList和Vector的区别

    - 线程安全：Vector使用了Synchronized来实现线程同步，所以线程安全。ArrayList非线程安全。
    - 扩容：都是动态调整的，Vector扩容每次增加一倍，ArrayList只会增加50%。

11. ##### Array和ArrayList的区别

    - Array可以存储基本数据类型和对象，ArrayList只能存储对象。
    - Array是指定固定大小的，而ArrayList是自动扩容的。
    - Array的内置方法没有ArrayList多，addAll，removeAll，iteration等方法只有ArrayList有。

12. ##### 在 Queue 中 poll()和 remove()有什么区别？

    - 相同点：都返回第一个元素，并在列队中删除返回的对象。
    - 不同点：如果没有元素poll()会返回null，而remove()会直接抛出NoSuchElementExcrption异常。

13. ##### 那些集合类是线程安全的？

    Vector、Stack、HashTable是线程安全的。HashMap的线程安全类是ConcurrentHashMap。

14. ##### 迭代器Iterator怎么使用？

    ~~~java
    List<String> list = new ArrayList<>();
    Iterator<String> it = list. iterator();
    while(it. hasNext()){
      String obj = it. next();
      System. out. println(obj);
    }
    ~~~

     Iterator 的特点是更加安全，因为它可以确保，在当前遍历的集合元素被更改的时候，就会抛出 ConcurrentModificationException 异常。 

15. ##### 迭代器Iterator是什么？

    Iterator接口提供遍历任何Collection的接口。我们可以从一个Collection中使用迭代器方法来获取迭代器实例。迭代器取代了java集合框架中的Enumeration，迭代器允许调用者在迭代过程中移除元素。

16. ##### Iterator和ListIterator有什么区别？

    - Iterator可以遍历Set和List集合，而ListIterator只能遍历List。
    - Iterator 只能单向遍历，而 ListIterator 可以双向遍历（向前/后遍历） 。
    - ListIterator 从 Iterator 接口继承，然后添加了一些额外的功能，比如添加一个元素、替换一个元素、获取前面或后面元素的索引位置。 

    ~~~java
    public interface ListIterator<E> extends Iterator<E> 
    ~~~

17. ##### 怎么确保一个集合不能被修改？

    可以使用 Collections。

    unmodifiableCollection(Collection c) 方法来创建一个只读集合，这样改变集合的任何操作都会抛出 Java. lang. UnsupportedOperationException 异常。 

    ~~~java
    List<String> list = new ArrayList<>();
    list. add("x");
    Collection<String> clist = Collections. unmodifiableCollection(list);
    clist. add("y"); // 运行时此行报错
    System. out. println(list. size());
    ~~~

