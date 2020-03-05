### JavaWeb

***

1. ##### JSP和servlet有什么区别？各自特点是什么？

   jsp经编译后就变成了Servlet，jsp的本直接就是servlet

   ###### 联系：

   JSP是Servlet技术的扩展，本质上就是Servlet的简易方式。JSP编译后是“类servlet”。Servlet和JSP最主要的不同点在于，Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML里分离开来。而JSP的情况是Java和HTML可以组合成一个扩展名为.jsp的文件。JSP侧重于视图，Servlet主要用于控制逻辑。

   ###### 区别：

   - Servlet在Java代码中通过HttpServletResponse对象动态输出HTML内容。
   - JSP在静态HTML内容中嵌入Java代码，Java代码被动态执行后生成HTML。

   ###### 特点：

   - Servlet能够很好地组织业务逻辑代码，但是在Java源文件中通过字符串拼接的方式生成动态HTML内容会导致代码维护困难、可读性差。
   - JSP虽然规避了Servlet在生成HTML内容方面的劣势，但是在HTML中混入大量、复杂的业务逻辑同样也是不可取的。

2. ##### JSP有哪些内置对象，作用是什么？

   jsp有九大内置对象

   - request（请求）：封装客户端的请求，其中包含来自get或post请求的参数。
   - response（响应）：封装服务器对客户端的响应。
   - pageContext：通过该对象可以获取其他对象。
   - session：封装用户会话的对象。
   - application：封装服务器运行环境的对象。
   - out：输出服务器响应的输出流对象。
   - config：Web应用的配置对象。
   - page：JSP页面本身（相当于java程序中的this）。
   - exception：封装页面抛出异常的对象。

3. ##### 说一下JSP的四种作用域？

   - **page：代表与一个页面相关的对象和属性。**
   - **request：代表于客户端发出的一个请求相关的对象和属性。**一个请求可能跨越多个页面，涉及多个Web组件；需要在页面显示的临时数据可以置于此作用域。
   - **session：代表与某个用户与服务器建立的一次与会话相关的对象和属性。**跟某个用户相关的数据应该放在用户自己的session中。
   - **application：代表与整个Web应用程序相关的对象和属性。**它实质上是跨越整个Web应用程序，包括多个页面、请求和会话的一个全局作用域。

4. ##### session和cookie有什么区别？

   - 存储位置不同：session存储在服务器端，cookie存储在浏览器端。
   - 安全性不同：cookie在浏览器端可以被伪造和篡改。
   - 存储的多样性：session可以存储在Redis，数据库，应用程序中。而cookie只能存储在浏览器中。

5. ##### 说一下session的工作原理？

   session工作原理是客户端登陆完成之后，服务端会创建对应的session，session创建完之后，会把session的id发送给客户端。客户端在存储在浏览器中（cookie）。这样客户端每次访问服务器时，都会在cookie中带着Session ID，服务器拿到sessionid之后，在内存找到sessionid对应的session这样就可以正常交互了。

6. ##### 假定在客户端禁止cookie下session如何使用？

   - 用文件，数据库形式保存SessionID，在跨页过程中手动调用。
   - 手动通过URL传值，隐藏表单传递SessionID。

7. ##### springmvc和struts2的区别是什么？

   - 拦截级别：springmvc是方法级别的拦截，struts2是类级别的拦截。
   - 拦截机制：struts2有自己的interceptor机制，springmvc用的是独立的aop方式，这样导致struts2的配置文件的量比springmvc大。
   - 对ajax的支持：springmvc集成了ajax，所以使用ajax方便，只需一个注解@RequestBody就可以实现了；而struts2一般需要安装插件或自己写代码才行。
   - 数据独立性：spring mvc 的方法之间基本上独立的，独享 request 和 response 数据，请求数据通过参数获取，处理结果通过 ModelMap 交回给框架，方法之间不共享变量；而 struts2 虽然方法之间也是独立的，但其所有 action 变量是共享的，这不会影响程序运行，却给我们编码和读程序时带来了一定的麻烦。 

8. ##### 如何避免SQL注入？

   - 使用正则表达式过滤掉字符中的特殊字符（例如sql关键字）。
   - 使用预处理PreparedStatement。

9. ##### 什么是XSS攻击，如何避免？

   跨站脚本攻击，原理是攻击者往Web页面里插入恶意的脚本代码（css,js等）,用户浏览时，嵌入其中的脚本代码会被执行，从而达到恶意攻击用户的目的，如盗取用户的cookie，破坏页面结构，重定向到其它网站等。

10. ##### 什么是XSRF攻击，如何避免？

    CSRF：Cross-Site Request Forgery，即跨站请求伪造，可以理解为攻击者盗用了你的身份，以你的名义发送恶意请求。比如以你的名义发送邮件，发消息，购买商品，虚拟货币转账等。

    如何避免：

    - 使用post请求
    - 敏感操作添加验证码
    - 添加自定义http请求头
    - 添加并验证token
    - CSRF检测工具检测漏洞