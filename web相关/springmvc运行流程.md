SpringMvc工作流程描述
  1. 用户向服务器发送请求，请求被Spring 前端控制Servelt DispatcherServlet捕获；

  2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI）。然后根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；(换句话说HandlerMapping用来定位url对应的Controller)  

  3. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法）
  4.  提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。
  执行handler方法之前会先执行拦截器的方法.
  (换句话说HandlerAdapter是用来具体执行hanler方法和拦截器)


  5.  Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象；
  6.  根据返回的ModelAndView，选择一个适合的ViewResolver（必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet ；
  7. ViewResolver 结合Model和View，来渲染视图  
  
  8. 将渲染结果返回给客户端。
