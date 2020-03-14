### Spring常用注解

***

1. ##### @Component

   主要用于将一个Java类注入到Spring框架中，它相当于xml配置文件中的：

   ~~~java
   <bean id=”xxx” class=”xxx”/>
   ~~~

   使用了Spring注解后，我们需要在配置文件中添加`<context:component-scan base-package=""/>`来扫描添加要扫描注解的类，这样子声明注解的类才能起作用。

   Bean实例的名称默认是Bean类的首字母小写，其他部分不变。

2. ##### @Autowired

   他可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作（DI依赖注入工作），目的是为了消除java代码中的get和set方法以及Bean中的property属性。

   Spring通过@Autowired注解实现Bean之间的依赖关系。

   按照默认的byType方式进行bean匹配。

3. ##### @Qualifier

   如果容器中有一个以上匹配的Bean时，则可以通过该注解限定注入的Bean的名称。

4. ##### @Scope

   用来显示的指定Bea作用范围。

5. ##### @Controller

   用于对Controller实现类注解。

6. ##### @Service

   用于对Service实现类注解。

7. ##### @Repository

   用于对Dao实现类注解。

8. ##### @Resource

   @Resource注解与@Autowired注解作用非常类似。

   - `@Resource`后面没有任何内容，默认通过name属性去匹配bean，找不到再按type去匹配

   - 指定了name或者type则根据指定的类型去匹配bean

   - 指定了name和type则根据指定的name和type去匹配bean，任何一个不匹配都将报错