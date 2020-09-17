#### JDBC相关

***

1. ##### 请你谈谈JDBC的反射，以及它的作用？

   通过反射com.mysql.jdbc.Driver类，实例化该类的时候会执行该类内部的静态代码块，该代码块会在Java实现的DriverManager类中注册自己,DriverManager管理所有已经注册的驱动类，当调用DriverManager.geConnection方法时会遍历这些驱动类，并尝试去连接数据库，只要有一个能连接成功，就返回Connection对象，否则则报异常。 

2. ##### 请你谈谈Statement和PrepareStatement的区别。

   - PreparedStatement接口代表预编译的语句，它主要的优势在于可以减少SQL的编译错误并增加SQL的安全性（减少SQL注射攻击的可能性）。
   
   - PreparedStatement中的SQL语句是可以带参数的，避免了用字符串连接拼接SQL语句的麻烦和不安全  。
   - 当批量处理SQL或频繁执行相同的查询时，PreparedStatement有明显的性能上的优势，由于数据库可以将编译优化后的SQL语句缓存起来，下次执行相同结构的语句时就会很快（不用再次编译和生成执行计划）。 
   
3. ##### 传统JDBC开发存在的问题？

   - 频繁的创建数据库连接对象，释放，容易造成系统资源的浪费，影响系统性能。
   - sql语句定义、参数设置、结果及处理存在硬编码，实际项目中的sql语句变化的可能性较大，一旦发生变化，需要重新修改java代码，系统需要重新编译，重新发布，不好维护。
   - 使用preparedStatement向占位符传参数存在硬编码，因为sql语句的where条件不一定，可能多可能少，修改sql还要修改代码，系统不易维护。
   - 结果集处理存在重复代码，处理麻烦。

