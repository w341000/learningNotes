如果一个Bean配置了多个生命周期机制，并且含有不同的方法名，执行的顺序如下：  
-	包含@PostConstruct注解的方法  
-	在InitializingBean接口中的afterPropertiesSet()方法  
-	自定义的init()方法   

销毁方法的执行顺序和初始化的执行顺序相同：  
-	包含@PreDestroy注解的方法  
-	在DisposableBean接口中的destroy()方法  
-	自定义的destroy()方法  
