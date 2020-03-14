### Spring MVC注解

***

1. ##### @Controller

   与Spring的controller注解作用一致，注册一个bean到spring上下文中。

2. ##### @RequestMapping

   控制器可以指定的URL请求。

3. ##### @RequestBody

   用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上，在把HttpMessageConverter返回的对象数据绑定到controller中方法的参数上。

4. ##### @ModelAttribute

   - 在方法定义上使用 @ModelAttribute 注解：Spring MVC 在调用目标处理方法前，会先逐个调用在方法级上标注了@ModelAttribute 的方法

   - 在方法的入参前使用 @ModelAttribute 注解：可以从隐含对象中获取隐含的模型数据中获取对象，再将请求参数绑定到对象中，再传入入参将方法入参对象添加到模型中

5. ##### @RequestParam

   在处理方法入参处使用 @RequestParam 可以把请求参数传递给请求方法 。

   - value：请求参数名（必须配置）

   - required：是否必需，默认为 true，即 请求中必须包含该参数，如果没有包含，将会抛出异常（可选配置）

   - defaultValue：默认值，如果设置了该值，required 将自动设为 false，无论你是否配置了required，配置了什么值，都是 false（可选配置）

6. ##### @PathVariable

   绑定URL占位符到入参。

7. ##### @ExceptionHandler

   注解到方法上，出现异常时执行该方法。

8. ##### @ControllerAdvice

   使一个Contoller成为全局的异常处理类，类中用@ExceptionHandler方法注解的方法可以处理所有Controller发生的异常 