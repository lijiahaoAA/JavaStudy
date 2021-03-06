### Java设计模式

***

1. ##### JDK中常见的几个设计模式。

   - 单例模式：保证被创建一次，节省系统开销。

     > java.lang.Runtime#getRuntime() 
     >
     >  Runtime类封装了运行时的环境。每个 Java 应用程序都有一个 Runtime 类实例，使应用程序能够与其运行的环境相连接。
     > 一般不能实例化一个Runtime对象，应用程序也不能创建自己的 Runtime 类实例，但可以通过 getRuntime 方法获取当前Runtime运行时对象的引用。
     > 一旦得到了一个当前的Runtime对象的引用，就可以调用Runtime对象的方法去控制Java虚拟机的状态和行为。 

   - 工厂模式：解耦代码, 创建对象而不将实例化逻辑暴露给客户端，并通过公共接口引用新创建的对象。 

     > java.lang.Object#toString() (在所有子类中重写) 
     >
     > java.lang.Class#newInstance() 
     >
     > java.lang.Integer#valueOf(String) 
     >
     > java.lang.Class#forName() 

   - 抽象工厂：提供用于创建一系列相关对象的接口，而无需显式指定其类。  

     > java.sql.DriverManager#getConnection() 
     >
     >  java.util.Arrays#asList() 
     >
     >  java.sql.Statement#executeQuery() 
     >
     >  java.sql.Connection#createStatement() 

   - 原型模式：复制对象，深复制浅复制。

     > Object.clone
     >
     > Cloneable 

   - 适配器模式：使不兼容的接口相容

     >java.io.InputStreamReader(InputStream)
     >
     >java.io.OutputStreamWriter(OutputStream) 

   - 代理模式：透明的调用被代理对象，增加被代理类的功能，无需知道实现细节。

     > 动态代理

   - 观察者模式：定义了对象之间的一对多的依赖，这样一来，当一个对象改变时，他的所有的依赖者都会收到通知并自动更新。

     >java.util.Observer,Observable
     >Swing中的Listener 

   - 外观模式：提供一个统一的接口，用来访问子系统中的一群接口，外观定义了一个高层的接口，让子系统更容易使用。

     > java.util.logging包 

   - 模板方法模式：定义了一个算法骨架，而将一些步骤延迟到子类中，模板方法可以使得子类可以在不改变算法结构的情况下，重新定义算法的步骤。

     > ThreadPoolExecutor.Worker 

   - 装饰器模式：动态的扩展一个类的功能。被用于多个Java类中。

     >java.io包
     >
     >java.util.Collections#synchronizedList(List) 

2. ##### 什么是设计模式，你是否在你的代码里使用过任何设计模式？

   设计模式是用来解决特定设计问题的尝试和测试的方法。是代码可重用性的延申。

3. ##### Java中什么叫单例设计模式？请用Java写出线程安全的单例模式。

   单例模式重点在于整个系统上共享一些创建时比较耗费资源的对象。整个应用中只维护一个特定类示例，他被所有的组件共享。```Java.lang.Runtime```是单例模式的经典例子。

4. ##### 什么叫做观察者模式？

   观察者模式是基于对象的状态变化和观察者的通讯，以便他们做出相应的操作。简单的例子就是一个天气系统，当天气变化时必须在展示给公众的视图中进行反映。这个视图对象是一个主体，而不同的视图是观察者。 

5. ##### 使用工厂模式的好处在哪里？在哪里使用？

   工厂模式的最大好处是增加了创建对象时的封装层次。如果你使用工厂来创建对象，之后你可以使用更高级和更高性能的是实现来代替原始的产品实现类，这不需要再调用层做任何修改。

6. ##### 举一个用Java实现的装饰者模式？他是作用于对象层次还是类层次？

   装饰模式增强了单个对象的能力。Java IO使用了装饰模式。例如Buffered系列，`BufferedReader`和`BufferedWriter`，他们增强了`Reader`和`Writer`对象，以实现提升性能的Buffer层次的读取和写入。

7. ##### java编程为什么不允许从静态方法中访问非静态变量？

   Java中不能从静态上下文访问非静态数据只是因为非静态变量时跟具体的对象实例相关的，而静态的却没有和任何实例关联。

8. ##### 如果需要设计一个ATM机，你的设计思路是什么？

   保持正确的状态（事务），加锁，错误条件，边界条件等等。

9. ##### 在Java语言中，什么时候重载，什么时候重写？

   - 重载：发生在方法上，不同的输入做同一件事。
   - 重写： 体现在继承关系上，子类把父类本身有的方法重新写了一遍。

10. ##### 请说明什么情况下更倾向于使用抽象类而不是接口？

   - 在 Java 中，你只能继承一个类，但可以实现多个接口。所以一旦你继承了一个类，你就失去了继承其他类的机会了。
   - 接口通常被用来表示附属描述或行为如：`Runnable、Clonable、Serializable`等等，因此当你使用抽象类来表示行为时，你的类就不能同时是`Runnable`和`Clonable`(注：这里的意思是指如果把`Runnable`等实现为抽象类的情况)，因为在 Java 中你不能继承两个类，但当你使用接口时，你的类就可以同时拥有多个不同的行为。
   - 在一些对时间要求比较高的应用中，倾向于使用抽象类，它会比接口稍快一点。
   - 如果希望把一系列行为都规范在类继承层次内，并且可以更好地在同一个地方进行编码，那么抽象类是一个更好的选择。有时，接口和抽象类可以一起使用，接口中定义函数，而在抽象类中定义默认的实现。

11. ##### 简单工厂和抽象工厂有什么区别？

    - 简单工厂：用来生产同一等级结构中的任意产品，对于增加新的产品，无能为力。
    - 工厂方法：用来生产同一等级结构中的固定产品，支持增加任意产品。
    - 抽象工厂：用来生产不同产品族的全部产品，对于增加新的产品，无能为力；支持增加产品族。