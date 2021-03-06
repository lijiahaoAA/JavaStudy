### Spring

***

1. ##### 为什么要使用Spring？

   - 方便解耦，便于开发（spring就类似于一个大工厂，可以将对象的创建和依赖关系维护都交给spring管理）。
   - spring支持AOP（面向切面编程，可以方便的实现对程序进行权限拦截和运行监控等功能）。
   - 声明式事务的支持（通过配置就完成对事务的支持，不需要手动编程）。
   - spring支持junit，可以通过注解方便的测试spring程序。
   - 方便继承各种优秀的框架（mybatis等）。
   - 封装了JDBC，javamail，远程调用等复杂的API。

2. ##### 解释一下什么是AOP？

   AOP：面向切面编程。在运行时，动态的将代码切入到类的指定方法、指定位置上的编程思想就是面向切面编程。

   使用AOP技术，可以将一些系统性相关的编程工作，独立提取出来，独立实现，然后通过切面切入进系统。

   从而避免了在业务逻辑的代码中混入很多的系统相关的逻辑——比如权限管理，事物管理，日志记录等等。

   这些系统性的编程工作都可以独立编码实现，然后通过AOP技术切入进系统即可。从而达到了 将不同的关注点分离出来的效果。

3. ##### Spring AOP和AspectJ AOP有什么区别？

   - Spring AOP属于运行时增强，AspectJ AOP是编译时增强。
   - Spring AOP基于动态代理，AspectJ基于字节码操作。
   - Spring AOP集成了AspectJ，切面太多时，选择AspectJ，速度快。

4. ##### 解释一下什么是IOC？

   IOC：控制反转。是spring的核心，对于spring框架来说，就是由spring来负责去控制对象的生命周期和对象间的关系。

   通俗地讲：就是把原本你自己制造，使用的对象，现在交由别人制造，而通过构造函数，setter方法或方法（这里指使用这个对象的方法）参数的方式传给你，由你使用。 

5. ##### Spring有哪些主要模块？

   Spring Core，ORM，DAO，MVC，WEB，Context，AOP

   ![模块图](https://img2018.cnblogs.com/blog/1704520/201906/1704520-20190624204106525-880966760.gif)

   - Spring Core：Core是Spring的核心类库，Spring的所有功能都依赖于该库，Core主要实现IOC功能。Spring的所有功能都是借助IOC实现的。
   - AOP：AOP模块是Spring的AOP库，提供了AOP（拦截器）机制，并提供常用的拦截器，供用户自定义和配置。基于动态代理，代理对象实现了接口，Spring AOP使用JDK Proxy去创建代理对象，对于没有实现接口的对象，无法使用JDK Proxy代理，Spring AOP会使用Cglib生成一个被代理对象的子类。
   - ORM：提供对常用的ORM框架的管理和辅助支持。例如Hibernate，ibtas等框架，Spring本身不对ORM进行实现，仅对常见的ORM框架进行封装。
   - DAO模块：Spring提供对JDBC的支持，对JDBC进行封装，允许JDBC使用Spring资源，并能统一管理JDBC事务，并不对JDBC实现。
   - WEB模块：WEB模块提供对常见框架Struts，JSF的支持。Spring能够管理这些框架，将Spring的资源注入给框架，也能在这些框架的前后插入拦截器。
   - Context模块：Context模块提供框架式的Bean访问方式，其他程序可以通过Context访问Spring的Bean资源，相当于资源注入。
   - MVC模块：WEB MVC模块为Spring提供了一套轻量级的MVC实现，实际开发中，我们既可以使用Struts，也可以使用Spring自己的MVC框架。

6. ##### Filter过滤器，Interceptor拦截器，Spring AOP拦截器的区别？

   - Filter过滤器：拦截Web访问url地址。
   - Interceptor：拦截以.action结尾的url，拦截Action的访问。
   - Spring AOP：只能拦截Spring管理Bean的访问（业务层Service）。

7. ##### Spring常用的注入方式？

   - setter属性注入
   - 构造方法注入
   - 注解注入

8. ##### Spring中bean是线程安全的么？

   Spring容器中的Bean是否线程安全，容器本身并没有提供Bean的线程安全策略，因此可以说**Spring容器中的Bean本身不具备线程安全的特性**，但是具体还是要结合具体scope的Bean去研究。

   对于原型(多例)Bean,每次创建一个新对象，也就是线程之间并不存在Bean共享，自然是不会有线程安全的问题。

   对于单例Bean,所有线程都共享一个单例实例Bean,因此是存在资源的竞争。

   如果单例Bean,是一个无状态Bean，也就是线程中的操作不会对Bean的成员执行查询以外的操作，那么这个单例Bean是线程安全的。比如Spring mvc 的 Controller、Service、Dao等，这些Bean大多是无状态的，只关注于方法本身。

   对于有状态的bean，Spring官方提供的bean，一般提供了通过ThreadLocal去解决线程安全的方法，比如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等。

   - 有状态就是有数据存储功能。
   - 无状态就是不会保存数据。

9. ##### Spring支持几种bean的作用域？

   - singleton：单例模式，Spring IOC容器中只存在一个Bean实例，Bean以单例模式存在。系统默认值。
   - prototype：每次请求（容器调用bean）时都会创建一个新的实例，即每次getBean()相当于执行new Bean()操作。
   - Web下的bean作用域：
     - request：每次http请求都会创建一个bean。
     - session：同一个http session共享一个bean实例。
     - global-session：提供一个全局的http session。

10. ##### 请谈一下@Autowired 和@Resource区别是什么？

   - @Autowired：默认按照byType注入。默认情况下他要求依赖对象必须存在，如果允许null值，可以设置他的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。 
   - @Resource： 默认按照byName自动注入 。@Resource有两个属性，name和type，Spring将name解析为bean的名字，type解析为bean的类型。使用那个属性就用那种方式注入。

11. ##### Spring自动装配bean有哪些方式？

   - no：默认值，表示自动装配，应使用显式bean引用进行装配。
   - byName：根据bean的名称注入对象依赖项。
   - byType：根据类型注入对象依赖项。
   - 构造函数：通过构造函数注入依赖项。
   - autodetect：容器首先通过构造函数使用@Autowired装配，如果不能，则通过byType自动装配。

12. ##### Spring事务实现的方式有哪些？

    - 声明式事务：基于xml配置文件或者基于注解。
    - 硬编码实现。

13. ##### 说一下什么是事务传播行为？

    用来描述某一个事务传播行为修饰的方法被嵌套进另一个方法时事务如何传播。

14. ##### Srring的七种事务传播行为？

    | 事务传播行为              | 说明                                                         |
    | ------------------------- | ------------------------------------------------------------ |
    | PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
    | PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
    | PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。               |
    | PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
    | PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
    | PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
    | PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

15. ##### 说一下Spring的事务隔离。

    spring有五大隔离界别，默认值为ISOLATION_DEFAULT.

    - ISOLATION_DEFAULT：用底层数据库的设置隔离界别，数据库设置什么我就用什么。
    - ISOLATION READ UNCOMMITTED：未提交读，最底隔离级别，事务未提交前，就可被其他事务读取。（会出现幻读，脏读，不可重复读）
    - ISOLATION READ COMMITTED：提交读，一个事务提交之后才能被其他事务读取到。（会造成幻读，不可重复读）
    - ISOLATION REPEATABLE READ：可重复读，保证多次读取同一个数据时，其值都和事务刚开始时的内容是一致的，禁止读取到别的事务未提交的数据（会造成幻读）。
    - ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该级别能防止脏读、不可重复读、幻读。

16. ##### 脏读，不可重复读，幻读是什么？

    - 脏读：表示一个事务能够读取另一个事务中还未提交的数据。比如，某个事务尝试插入记录 A，此时该事务还未提交，然后另一个事务尝试读取到了记录 A。 
    - 不可重复读：不可重复读是指在事务1内，读取了一个数据，事务1还没有结束时，事务2也访问了这个数据，修改了这个数据，并提交。紧接着，事务1又读这个数据。由于事务2的修改，那么事务1两次读到的的数据可能是不一样的，因此称为是不可重复读。 
    - 幻读：指同一个事务内多次查询返回的结果集不一样。比如同一个事务 A 第一次查询时候有 n 条记录，但是第二次同等条件下查询却有 n+1 条记录，这就好像产生了幻觉。发生幻读的原因也是另外一个事务新增或者删除或者修改了第一个事务结果集里面的数据，同一个记录的数据内容被修改了，所有数据行的记录就变多或者变少了。 

    >注意：不可重复读和幻读的区别是：前者是指读到了已经提交的事务的更改数据（修改或删除），后者是指读到了其他已经提交事务的新增数据。
    >幻读和不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。 
    >原文链接：https://blog.csdn.net/Vincent2014Linux/article/details/89669762