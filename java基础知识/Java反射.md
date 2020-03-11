### Java反射

***

1. ##### 什么是反射？

   指程序可以访问，检测或修改他本身状态或行为的一种能力。

   java中反射：指在运行状态中，对于任何一个类都能知道这个类有哪些属性和方法，对于任何一个对象都能对他的方法和属性进行调用。

2. ##### 什么是Java序列化？

   Java序列化是为了保存各种对象在内存中的状态，并且可以把保存的对象状态在读出来。

3. ##### 什么情况下需要序列化？

   - 想把内存中的对象保存到文件或数据库中时。
   - 想用套接字在网络上传输对象的时候。
   - 想通过远程方法调用传输对象的时候。

   ##### 注意：

   - 某个类可以被序列化，它的子类也可以序列化。
   - 声明为static和transient的成员变量，不能被序列化。static成员变量是描述类级别的属性，transient表示临时数据。
   - 反序列化读取序列化对象的顺序需要保持一致。

   ##### 测试：

   ~~~java
   import java.io.FileInputStream;
   import java.io.FileOutputStream;
   import java.io.IOException;
   import java.io.ObjectInputStream;
   import java.io.ObjectOutputStream;
   import java.io.Serializable;
   
   /**
    * @Author Lijiahao
    * @Description 测试序列化
    * @Date 18:15 2020/3/3
    * @Param
    * @return
    **/
   public class TestSerializable implements Serializable {
   
       private static final long serialVersionUID = 5887391604554532906L;
   
       private int id;
   
       private String name;
   
       public TestSerializable(int id, String name) {
           this.id = id;
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "TestSerializable [id=" + id + ", name=" + name + "]";
       }
   
       @SuppressWarnings("resource")
       public static void main(String[] args) throws IOException, ClassNotFoundException {
           //序列化
           ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("TestSerializable.obj"));
           oos.writeObject("测试序列化");
           oos.writeObject(618);
           TestSerializable test = new TestSerializable(1, "lijiahao");
           oos.writeObject(test);
   
           //反序列化
           ObjectInputStream ois = new ObjectInputStream(new FileInputStream("TestSerializable.obj"));
           System.out.println((String)ois.readObject());
           System.out.println(ois.readObject());
           System.out.println(ois.readObject());
       }
   
   }
   ~~~

4. ##### 动态代理是什么？

   在运行时，创建目标类，可以调用或扩展（增强）目标类的方法。

5. ##### 动态代理的使用场景。

   Java中实现动态的方式：JDK中的动态代理和Java类库`cglib`

   - Spring 中AOP就是采用动态代理的机制来实现的面向切面。
   - 统一的日志输出。
   - 统计每个API的请求耗时。

6. ##### 怎么实现动态代理？

   - 使用Java中JDK的动态代理方式

     ~~~java
     package test.jdk;
     
     /**
      * @Author: lijiahao
      * @Description:
      * @Data: Create in 18:30 2020/3/3
      * @Modified By:
      */
     public interface Rent {
         public void rent();
     }
     
     ~~~

     ~~~java
     package test.jdk;
     
     /**
      * @Author: lijiahao
      * @Description:
      * @Data: Create in 18:30 2020/3/3
      * @Modified By:
      */
     public class Landlord implements Rent{
     
         @Override
         public void rent() {
             System.out.println("房东要出租房子了！");
         }
     }
     
     ~~~

     ~~~java
     package test.jdk;
     
     /**
      * @Author: lijiahao
      * @Description:
      * @Data: Create in 18:31 2020/3/3
      * @Modified By:
      */
     import java.lang.reflect.InvocationHandler;
     import java.lang.reflect.Method;
     
     public class Intermediary implements InvocationHandler{
     
         private Object post;
     
         Intermediary(Object post){
             this.post = post;
         }
         @Override
         public Object invoke(Object proxy, Method method, Object[] args)
                 throws Throwable {
             Object invoke = method.invoke(post, args);
             System.out.println("中介：该房源已发布！");
             return invoke;
         }
     }
     
     ~~~

     ~~~java
     package test.jdk;
     
     /**
      * @Author: lijiahao
      * @Description:
      * @Data: Create in 18:31 2020/3/3
      * @Modified By:
      */
     import java.lang.reflect.Proxy;
     
     public class Test {
         public static void main(String[] args) {
             Rent rent = new Landlord();
             Intermediary intermediary = new Intermediary(rent);
             //loader: 用哪个类加载器去加载代理对象
             //interfaces:动态代理类需要实现的接口
             //第三个参数:动态代理方法在执行时，会调用该类里面的invoke方法去执行
             Rent rentProxy = (Rent) Proxy.newProxyInstance(rent.getClass().getClassLoader(), rent.getClass().getInterfaces(), intermediary);
             rentProxy.rent();
         }
     }
     
     ~~~

   - 使用类库`cglib`

     ~~~java
     public class Landlord {
         public void rent(){
             System.out.println("房东要出租房子了！");
         }
     }
     ~~~

     ~~~java
     import java.lang.reflect.Method;
     
     import net.sf.cglib.proxy.MethodInterceptor;
     import net.sf.cglib.proxy.MethodProxy;
     
     public class Intermediary implements MethodInterceptor {
     
         @Override
         public Object intercept(Object object, Method method, Object[] args,MethodProxy methodProxy) throws Throwable {
             Object intercept = methodProxy.invokeSuper(object, args);
             System.out.println("中介：该房源已发布！");
             return intercept;
         }
     }
     ~~~

     ~~~java
     import net.sf.cglib.proxy.Enhancer;
     
     public class Test {
         public static void main(String[] args) {
             Intermediary intermediary = new Intermediary();
             
             Enhancer enhancer = new Enhancer();  
             enhancer.setSuperclass(Landlord.class);
             enhancer.setCallback(intermediary);
             
             Landlord rentProxy = (Landlord) enhancer.create();
             rentProxy.rent();
         }
     }
     ~~~

     

