### SpringBoot

***

1. ##### 什么是SpringBoot？

   SpringBoot使为Spring服务的，是用来简化spring应用的初始搭建以及开发过程的。SpringBoot是Spring开源组织下的子项目，是Spring组件一站式结局方案，主要是简化了使用Spring的难度，省略了繁重的配置，提供了各种启动容器，可快速上手。

2. ##### 为什么要使用SpringBoot？

    为了解决java开发中的，繁多的配置、低下的开发效率，复杂的部署流程，和第三方技术集成难度大的问题，产生了spring boot。 

3. ##### SpringBoot的核心配置文件是什么？

   两个核心配置文件

   - bootstrap（.yml或者.properties）：bootstrap由父类ApplicationContext加载的，比application优先加载，且bootstrap里面的属性不能被覆盖。
   - application（.yml或者.properties）：用于SpringBoot项目的自动化配置。

4. ##### SpringBoot配置文件有哪几种类型？有什么区别？

   配置文件有 . properties 格式和 . yml 格式，它们主要的区别是书法风格不同。

   . projjperties 配置如下：

   ```pijava
   spring.RabbitMQ.port=5672
   ```

   . yml 配置如下：

   ```java
   spring:
       RabbitMQ:
           port: 5672
   ```

   . yml 格式不支持 @PropertySource 注解导入。

5. ##### SpringBoot与那些方式可以实现热部署？

   - 使用 devtools 启动热部署，添加 devtools 库，在配置文件中把 spring. devtools. restart. enabled 设置为 true；
   - 使用 Intellij Idea 编辑器，勾上自动编译或手动重新编译。

6. ##### jpa和hibernate有什么区别？

   JPA（Java Persistence API），Hibernate是一个JPA实现，而Spring Data JPA是一个JPA数据访问对象。

   使用Spring Data，您可以使用Hibernate，Eclipse Link或任何其他JPA提供程序。一个非常有趣的好处是[您可以使用@Transactional注释以声明方式控制事务边界](https://link.zhihu.com/?target=https%3A//vladmihalcea.com/a-beginners-guide-to-transaction-isolation-levels-in-enterprise-java/)。
   Spring Data JPA不是一个实现或JPA提供程序，它只是一个抽象，用于显着减少为各种持久性存储实现数据访问层所需的样板代码量。
   Hibernate提供了Java Persistence API的参考实现，使其成为具有松散耦合优势的ORM工具的绝佳选择。
   请记住，Spring Data JPA始终需要JPA提供程序，如[Hibernate](https://link.zhihu.com/?target=http%3A//www.javaguides.net/p/hibernate-tutorial.html)或Eclipse Link。 

