### Java异常

***

1. ##### throw和throws的区别？

   - throw用于方法内部，throws用于方法声明上。

   - throw后跟异常对象，throws后跟异常类型。

   - throw后只能跟一个异常对象，throws后可以一次声明多种异常类型。

     > throw new RuntimeErrorException;
     >
     > public void test() throws IOException,SQLException{}

2. ##### final、finally、finalize有什么区别？

   - final：是修饰符，如果修饰类，此类不能被继承；如果修饰方法和变量，则表示此方法和此变量不能在被改变，只能使用。
   - finally：是 try{} catch{} finally{} 最后一部分，表示不论发生任何情况都会执行，finally 部分可以省略，但如果 finally 部分存在，则一定会执行 finally 里面的代码。
   - finalize： 是 Object 类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法。

3. ##### try-catch-finally中哪个部分可以省略？

   try-catch-finally 其中 catch 和 finally 都可以被省略，但是不能同时省略，也就是说有 try 的时候，必须后面跟一个 catch 或者 finally。 

4. ##### try-catch-finally中，如果catch中return了，finally还会执行么？

   finally 一定会执行，即使是 catch 中 return 了，catch 中的 return 会等 finally 中的代码执行完之后，才会执行。 

   ###### 代码示例：

   执行结果：30

   ~~~java
   /*
    * 如果catch里面有return语句，finally里面的代码还会执行吗？
    */
   public class FinallyDemo2 {
       public static void main(String[] args) {
           System.out.println(getInt());
       }
    
       public static int getInt() {
           int a = 10;
           try {
               System.out.println(a / 0);
               a = 20;
           } catch (ArithmeticException e) {
               a = 30;
               return a;
               /*
                * return a 在程序执行到这一步的时候，这里不是return a 而是 return 30；这个返回路径就形成了
                * 但是呢，它发现后面还有finally，所以继续执行finally的内容，a=40
                * 再次回到以前的路径,继续走return 30，形成返回路径之后，这里的a就不是a变量了，而是常量30
                */
           } finally {
               a = 40;
           }
    
   //      return a;
       }
   }
   ~~~

   执行结果：40

   ~~~java
   /*
    * 如果catch里面有return语句，finally里面的代码还会执行吗？
    */
   public class FinallyDemo2 {
       public static void main(String[] args) {
           System.out.println(getInt());
       }
    
       public static int getInt() {
           int a = 10;
           try {
               System.out.println(a / 0);
               a = 20;
           } catch (ArithmeticException e) {
               a = 30;
               return a;
               /*
                * return a 在程序执行到这一步的时候，这里不是return a 而是 return 30；这个返回路径就形成了
                * 但是呢，它发现后面还有finally，所以继续执行finally的内容，a=40
                * 再次回到以前的路径,继续走return 30，形成返回路径之后，这里的a就不是a变量了，而是常量30
                */
           } finally {
               a = 40;
               return a; //如果这样，就又重新形成了一条返回路径，由于只能通过1个return返回，所以这里直接返回40
           }
    
   //      return a;
       }
   }
   ~~~

   

5. ##### 常见的异常类。

   - NullPointerException 当应用程序试图访问空对象时，则抛出该异常。

   - SQLException 提供关于数据库访问错误或其他错误信息的异常。

   - IndexOutOfBoundsException指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出。

   - NumberFormatException当应用程序试图将字符串转换成一种数值类型，但该字符串不能转换为适当格式时，抛出该异常。

   - FileNotFoundException当试图打开指定路径名表示的文件失败时，抛出此异常。

   - IOException当发生某种I/O异常时，抛出此异常。此类是失败或中断的I/O操作生成的异常的通用类。

   - ClassCastException当试图将对象强制转换为不是实例的子类时，抛出该异常。

   - ArrayStoreException试图将错误类型的对象存储到一个对象数组时抛出的异常。

   - IllegalArgumentException 抛出的异常表明向方法传递了一个不合法或不正确的参数。

   - ArithmeticException当出现异常的运算条件时，抛出此异常。例如，一个整数“除以零”时，抛出此类的一个实例。

   - NegativeArraySizeException如果应用程序试图创建大小为负的数组，则抛出该异常。

   - NoSuchMethodException无法找到某一特定方法时，抛出该异常。

   - SecurityException由安全管理器抛出的异常，指示存在安全侵犯。

   - UnsupportedOperationException当不支持请求的操作时，抛出该异常。

   - RuntimeExceptionRuntimeException 是那些可能在Java虚拟机正常运行期间抛出的异常的超类。