### Hibernate

***

1. ##### 为什么要使用Hibernate？

   - Hibernate对JDBC访问数据库做了封装，简化了数据访问层繁琐的代码。
   - Hibernate是一个基于JDBC的主流持久层框架，是一个优秀的orm实现，简化了dao层编码工作。
   - Hibernate使用java的反射机制，而不是字节码增强程序类增强透明性。
   - Hibernate是一个轻量级框架，性能很好。支持很多关系型数据库，从一对一到多对多的各种复杂关系。

2. ##### 什么是ORM框架？

   Object Relational Mapping：对象关系映射。框架采用元数据（描述数据的数据，对数据及信息资源的描述性信息）来描述对象与关系映射的细节，元数据一般采用XML格式，并且存放专门的对象到映射文件中。

   只要提供了持久化与表的映射关系，ORM框架就能在运行时**参照映射文件的信息，把对象持久化到数据库中。**

   ORM是通过使用描述对象和数据库之间映射的元数据,在我们想到描述的时候自然就想到了xml和特性(Attribute).目前的ORM框架中,Hibernate就是典型的使用xml文件作为描述实体对象的映射框架,而大名鼎鼎的Linq则是使用特性(Attribute)来描述的 

3. ##### Hibernate中如何在控制台查看打印的SQL语句?

   在 Config 里面把 hibernate. show_SQL 设置为 true 就可以。但不建议开启，开启之后会降低程序的运行效率。

   **示例：**

   找到hibernate的配置文件-- hibernate.cfg.xml
   加入：

   ~~~java
   <property name="hibernate.show_sql">true</property>
   ~~~

   如果你用spring那么就要：

   修改spring里面的配置文件：

   ~~~java
   <property name="hibernateProperties">
   <props>
   	<prop key = "hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
   	<prop key = "hibernate.show_sql">false</prop>
   ~~~

4. ##### Hibernate有几种查询语句？

   Hibernate有4种查询方法。

   - HQL 通过Hibernate提供的查询语言进行查询。Hibernate Query lanague。

   - EJBQL(JPQL 1.0) 是EJB提供的查询语言。

   - QBC(query by cretira)通过Cretira接口进行查询。

   - QBE(query by Example) 通过Example编程接口进行查询。

5. ##### Hibernate实体类可以被定义为final么？

   是的，你可以将Hibernate的实体类定义为final类，但这种做法并不好。因为Hibernate会使用代理模式在延迟关联的情况下提高性能，如果你把实体类定义成final类之后，因为 Java不允许对final类进行扩展，所以Hibernate就无法再使用代理了，如此一来就限制了使用可以提升性能的手段。不过，如果你的持久化类实现了一个接口而且在该接口中声明了所有定义于实体类中的所有public的方法的话，就能够避免出现前面所说的不利后果。 

6. ##### Hibernate里实体类用int和Integer的区别？

   返回数据库字段的值为null的话，int类型会报错。int是基本数据类型，其声明的是变量，而null是对象。所以实体类建议使用Integer。

   通过jdbc将实体存储到数据库的操作通过sql语句，基本数据类型可以直接存储，对象需要序列化存储。

   在很多应用中，需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便长期保存。

7. ##### Hibernate是如何工作的？

   1. 通过Configuration config = new Configuration().configure();[//读取并解析hibernate.cfg.xml配置文件](https://link.zhihu.com/?target=//%E8%AF%BB%E5%8F%96%E5%B9%B6%E8%A7%A3%E6%9E%90hibernate.cfg.xml%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

   2. 由hibernate.cfg.xml中的<mapping resource="com/xx/User.hbm.xml"/>读取并解析映射信息

   3. 通过SessionFactory sf = config.buildSessionFactory();//创建SessionFactory

   4. Session session = sf.openSession();//打开Sesssion

   5. Transaction tx = session.beginTransaction();//创建并启动事务Transation

   6. persistent operate操作数据，持久化操作

   7. tx.commit();//提交事务

   8. 关闭Session

   9. 关闭SesstionFactory

8. ##### get()和load()的区别?

   - load()支持延迟加载，get()不支持延迟加载。
   - 数据查询时，没有OID指定的对象，get()返回null，load()返回一个代理对象。

9. ##### 说一下Hibernate的缓存机制？

   - 一级缓存：也叫Session缓存，只在Session作用范围内有效，不需要用户干涉，由Hibernate自身维护。可以通过evict(object)清除object的缓存，clear()清除一级缓存中的所有缓存。flush()刷出缓存。
   - 二级缓存：应用级别的缓存，在所有Session对象中都有效，支持第三方的缓存。如EhCache。

10. ##### Hibernate对象有哪些状态？

    - 临时/瞬时状态：直接new出来的对象，该对象还没被持久化（没保存在数据库中），不受Session管理。
    - 持久化状态：当调用Session的save/saveOrupdate/get/load/list等方法的时候，对象就是持久化状态。
    - 游离状态：Session关闭后对象就是游离状态。

11. ##### 在Hibernate中getCurrentSession和openSession的区别是什么？

    - getCurrentSession：会绑定当前线程，而openSession不会。
    - getCurrentSession事务是Spring控制的，并且不需要手动关闭，而openSession需要我们自己手动开启和提交事务。

12. ##### Hibernate实体类必须要有无参构造函数么？为什么？

    Hibernate中每个实体类必须提供一个无参构造函数，因为Hibernate框架要使用reflection api，通过调用Class.newInstance()来创建实体类的实例，如果没有无参构造函数就会抛出异常。