### Spring MVC

***

1. ##### 说一下Spring MVC的运行流程？

   - Spring MVC先将请求发送给DispatchServlet。
   - DispatchServlet查询一个或多个HandlerMapping，找到处理请求的Controller。
   - DispatchServlet再把请求提交给对应的Controller。
   - Controller进行业务逻辑处理后，会返回一个ModelAndView。
   - DispatchServlet查询一个或多个ViewResolver视图解析器，找到ModelAndView对象指定的视图对象。
   - 视图对象负责渲染然后返回给客户端。

2. ##### Spring MVC有哪些资源？

   - 前端控制器：DispatchServlet
   - 处理器映射器：HandlerMapping
   - 处理器：Controller
   - 模型和视图：ModelAndView
   - 视图解析器：ViewResolver

3. ##### @RequestMapping的作用是什么？

   @RequestMapping是一个用来处理请求地址映射的注解，可用于类或者方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径 。

4. ##### @AutoWired的作用是什么？

   @Autowired 它可以对类成员变量、方法及构造函数进行标注，完成自动装配（注入）的工作，通过@Autowired 的使用来消除 set/get 方法。 