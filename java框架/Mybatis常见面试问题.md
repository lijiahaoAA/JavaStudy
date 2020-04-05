### Mybatis

***

1. ##### Mybatis中#{}和${}的区别是什么？

   将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号。
   如：where username=#{username}，如果传入的值是111,那么解析成sql时的值为where username="111", 如果传入的值是id，则解析成的sql为where username="id".　
   $将传入的数据直接显示生成在sql中。
   如：where username=${username}，如果传入的值是111,那么解析成sql时的值为where username=111；

   所以若传入的参数为;drop table user，那么使用${}接收参数后就会发生sql注入。

   **区别：**

   - \#方式能够很大程度上防止sql注入，$方式无法阻止sql注入。
   - $方式一般用于传入数据库对象，例如传入表名和列名，还有排序时使用order by动态参数时需要使用$，ORDER BY ${columnName}。
   - 一般能用#就别用$，如不得不使用$传参，就要手工做好过滤工作，来防止sql注入。
   - 在MyBatis中，“${xxx}”这样格式的参数会直接参与SQL编译，从而不能避免注入攻击。但涉及到动态表名和列名时，只能使用“${xxx}”这样的参数格式。所以，这样的参数需要我们在代码中手工进行处理来防止注入。

   > 在编写MyBatis的映射语句时，尽量采用“#{xxx}”这样的格式。若不得不使用“${xxx}”这样的参数，要手工地做好过滤工作，来防止SQL注入攻击。 

2. ##### Mybatis有几种分页方式？

   分为物理分页和逻辑分页

   - 物理分页：自己手写SQL分页或使用分页插件PageHelper，去数据库查询指定条数的分页数据的形式。
   - 逻辑分页：使用Mybatis自带的RowBounds进行分页，他是一次性查询很多数据，然后在数据中在进行检索。

3. ##### RowBounds是一次性查询全部结果么？为什么？

   RowBounds 表面是在“所有”数据中检索数据，其实并非是一次性查询出所有数据，因为 MyBatis 是对 jdbc 的封装，在 jdbc 驱动中有一个 Fetch Size 的配置，它规定了每次最多从数据库查询多少条数据，假如你要查询更多数据，它会在你执行 next()的时候，去查询更多的数据。就好比你去自动取款机取 10000 元，但取款机每次最多能取 2500 元，所以你要取 4 次才能把钱取完。只是对于 jdbc 来说，当你调用 next()的时候会自动帮你完成查询工作。这样做的好处可以有效的防止内存溢出。 

4. ##### Mybatis逻辑分页和物理分页的区别是什么？

   - 逻辑分页是一次性查询很多数据，然后再在结果中检索分页的数据。这样做弊端是需要消耗大量的内存、有内存溢出的风险、对数据库压力较大。
   - 物理分页是从数据库查询指定条数的数据，弥补了一次性全部查出的所有数据的种种缺点，比如需要大量的内存，对数据库查询压力较大等问题。

5. ##### Mybatis是否支持延迟加载？延迟加载的原理是什么？

   Mybatis支持延迟加载，是指lazyLoadingEnable=true即可。

   原理：调用的时候触发加载，而不是在初始化的时候就进行加载。延迟加载的原理的是调用的时候触发加载，而不是在初始化的时候就加载信息。比如调用 a. getB(). getName()，这个时候发现 a. getB() 的值为 null，此时会单独触发事先保存好的关联 B 对象的 SQL，先查询出来 B，然后再调用 a. setB(b)，而这时候再调用 a. getB(). getName() 就有值了，这就是延迟加载的基本原理。 

6. ##### 说一下Mybatis的一级缓存和二级缓存？

   - **一级缓存**：指的是mybatis的SqlSession对象的缓存。当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供的一块区域中，该区域的结构是一个HashMap，key为hashcode+statementId+sql语句。Value为查询出来的结果集映射成的java对象。 当我们再次拿到同样的数据时，mybatis会先去SqlSession中查询是否有，有就拿出来用。

     ​		当SqlSession对象消失时，mybatis的一级缓存也就消失了。

     ​		SqlSession执行insert、update、delete等操作commit后会清空该SQLSession缓存。

   - **二级缓存**：指的是Myabtis中SqlSessionFactory对象的缓存，由同一个SqlSessionFactory对象创建的SqlSession共享其缓存。

7. ##### Mybatis和Hibernate的区别有哪些？

   - 灵活性：Mybatis更灵活，支持高度的自定义SQL语句。
   - 可移植性：MyBatis 有很多自己写的 SQL，因为每个数据库的 SQL 可以不相同，所以可移植性比较差。 
   - 二级缓存：Hibernate有更好的二级缓存，他的二级缓存可以自行更换为第三方的二级缓存。

8. ##### Mybatis有哪些执行器？

   有三种基本的Executor执行器：

   - SimpleExecutor：每执行一次 update 或 select 就开启一个 Statement 对象，用完立刻关闭 Statement 对象； 
   - ReuseExecutor：执行 update 或 select，以 SQL 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后不关闭 Statement 对象，而是放置于 Map 内供下一次使用。简言之，就是**重复使用 Statement 对象；**
   - BatchExecutor：执行 update（没有 select，jdbc 批处理不支持 select），将所有 SQL 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理，与 jdbc 批处理相同。

9. ##### Mybatis分页查询的实现原理是什么？

   使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的SQL，根据dialect方言，添加对应的物理分页参数。

10. ##### Mybatis如何编写一个自定义插件？

    **自定义插件实现原理**

    MyBatis 自定义插件针对 MyBatis 四大对象（Executor、StatementHandler、ParameterHandler、ResultSetHandler）进行拦截：

    - Executor：拦截内部执行器，它负责调用 StatementHandler 操作数据库，并把结果集通过 ResultSetHandler 进行自动映射，另外它还处理了二级缓存的操作；
    - StatementHandler：拦截 SQL 语法构建的处理，它是 MyBatis 直接和数据库执行 SQL 脚本的对象，另外它也实现了 MyBatis 的一级缓存；
    - ParameterHandler：拦截参数的处理；
    - ResultSetHandler：拦截结果集的处理。

    **自定义插件实现关键**

    MyBatis 插件要实现 Interceptor 接口，接口包含的方法，如下：

    ```text
    public interface Interceptor {   
       Object intercept(Invocation invocation) throws Throwable;       
       Object plugin(Object target);    
       void setProperties(Properties properties);
    }
    ```

    - setProperties 方法是在 MyBatis 进行配置插件的时候可以配置自定义相关属性，即：接口实现对象的参数配置；
    - plugin 方法是插件用于封装目标对象的，通过该方法我们可以返回目标对象本身，也可以返回一个它的代理，可以决定是否要进行拦截进而决定要返回一个什么样的目标对象，官方提供了示例：return Plugin. wrap(target, this)；
    - intercept 方法就是要进行拦截的时候要执行的方法。

    **自定义插件实现示例**

    官方插件实现：

    ```java
    @Intercepts({@Signature(type = Executor. class, method = "query",
            args = {MappedStatement. class, Object. class, RowBounds. class, ResultHandler. class})})
    public class TestInterceptor implements Interceptor {
       public Object intercept(Invocation invocation) throws Throwable {
         Object target = invocation. getTarget(); //被代理对象
         Method method = invocation. getMethod(); //代理方法
         Object[] args = invocation. getArgs(); //方法参数
         // do something . . . . . .  方法拦截前执行代码块
         Object result = invocation. proceed();
         // do something . . . . . . . 方法拦截后执行代码块
         return result;
       }
       public Object plugin(Object target) {
         return Plugin. wrap(target, this);
       }
    }
    ```