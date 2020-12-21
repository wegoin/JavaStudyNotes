## 什么是AOP？

<font color=red>Aspect Oritention Programming</font> 面向切面编程。换句话说AOP是一种更高级的动态代理的使用。

## AOP术语

在上一章编写动态代理案例的时候，我们把很多辅助逻辑都封装在了各种Handler中。这些售前服务、售后服务也就是所谓的横切关注点，它是可以被模块化为特殊的类的，这些类被称为切面（aspect）。这样做有两个好处：

+  首先，现在每个关注点都集中于一个地方，而不是分散到多处代码中；

+ 其次，模块服务更简洁，因为它们只包含主要关注点（或核心功能）的代码，而次要关注点的代码被转移到切面中去了。

为了能更准确的使用Spring提供的AOP技术，我们首先有必要了解和掌握一部分常见的AOP术语：

1. #### 通知 Advice

   通知定义了切面是什么（**what**）以及何时使用（**when**）。除了描述切面要完成的工作，通知还解决了**何时**执行这个工作的问题。它应该应用在某个方法被调用**之前**？**之后**？还是只在方法**抛出异常时**调用？

   Spring切面可以应用5种类型的通知：

   <table>
       <tr>
           <td>通知类型</td>
           <td>说明</td>
       </tr>
       <tr>
           <td>前置通知<font color=green>Before</font></td>
           <td>在目标方法被调用之前调用。</td>
       </tr>
       <tr>
       	<td>后置通知<font color=green>After</font></td>
           <td> 在目标方法完成之后调用通知，此时不会关心方法的输出是什么。</td>
       </tr>
       <tr>
       	<td> 返回通知<font color=green>After-returning</font></td>
           <td> 在目标方法成功执行之后调用。</td>
       </tr>
       <tr>
       	<td> 异常通知<font color=green>After-throwing</font></td>
           <td> 在目标方法抛出异常后调用。</td>
       </tr>
       <tr>
       	<td> 环绕通知<font color=green>Around</font></td>
           <td> 包裹了被通知的方法，在被通知的方法调用之前和之后执行自定义的行为。</td>
       </tr>
   </table>

   

2. #### 切入点 Pointcut

   如果说通知定义了切面的“什么”（what）和“何时”（when）的话，那么切点就定义了“何处”（where）。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。

   关于切点表达式：

   + execution：在方法执行时触发

   + *：返回任意类型- com.turing.pojo.Customer：方法所属的类，也可以使用通配符

   +  buy：被通知的方法

   +  (..)：使用任意参数

3. #### 切面

   切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和何处完成其功能。

## 使用SpringAOP

首先，需要给项目增加spring-aspects模块：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.19.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.19.RELEASE</version>
</dependency>
```

1. #### 使用XML方式配置

   业务层实现类

   ```java
   public class UserService {
       public void save(){
           System.out.println("用户保存成功...");
       }
   }
   ```

   自定义事务

   ```java
   public class ServiceHandler {
       //在方法执行之前执行
       public void before(){
           System.out.println("before...");
       }
   
       //在方法正常执行之后
       public void afterReturning(){
           System.out.println("afterReturning...");
       }
   
       //在方法执行之后
       //相当于在finally中
       public void after(){
           System.out.println("after...");
       }
   
       //在抛出异常的时候
       public void throwing(Throwable ex){
           System.out.println("throwing..."+ex.getMessage());
       }
   
       //方法环绕
       public Object around(ProceedingJoinPoint pjp){
           try {
               System.out.println("before...");
               Object proceed = pjp.proceed();
               System.out.println("after returning");
               return proceed;
           } catch (Throwable e) {
               System.out.println("after throwing");
               e.printStackTrace();
           } finally{
               System.out.println("after");
           }
           return null;
       }
   }
   ```

   测试

   ```java
   public class MainTest {
       public static void main(String[] args) {
           //初始化容器
           ApplicationContext ctx = new
               ClassPathXmlApplicationContext("applicationContext.xml");
           //获取bean
           UserService userService = ctx.getBean("userService",
                                                 UserService.class);
           //调用bean的方法
           userService.save();
       }
   }
   ```

   

   ##### 主要配置

   1. 创建`applicationContext.xml`配置文件

   2. 手动加入命名空间：

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:aop="http://www.springframework.org/schema/aop"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans
                                 http://www.springframework.org/schema/beans/spring-beans.xsd
                                 http://www.springframework.org/schema/aop
                                 http://www.springframework.org/schema/aop/spring-aop.xsd">
      ```

      

   3. AOP配置：

      ​	---配置切面 

      ​		---切入点 pointcut 

      ​		---通知 aop:before...

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:aop="http://www.springframework.org/schema/aop"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans
                                 http://www.springframework.org/schema/beans/spring-beans.xsd
                                 http://www.springframework.org/schema/aop
                                 http://www.springframework.org/schema/aop/spring-aop.xsd">
          <bean id="userService" class="com.tuling.pojo.UserService">
          </bean>
          <bean
                id="serviceHandler" class="com.tuling.pojo.ServiceHandler"></bean>
      
          <!-- 配置AOP -->
          <aop:config>
              <!-- 配置切面 -->
              <aop:aspect ref="serviceHandler">
                  <!-- 配置切入点 -->
                  <aop:pointcut expression="execution(* com.tuling.pojo.UserService.*
                                            (..))" id="pc"/>
                  <!-- 配置时机 -->
                  <!-- 
       <aop:before method="before" pointcut-ref="pc"/>
       <aop:after-returning method="afterReturning" pointcutref="pc"/>
       <aop:after method="after" pointcut-ref="pc"/>
       <aop:after-throwing method="throwing" pointcutref="pc" throwing="ex"/>
       -->
                  <aop:around method="around" pointcut-ref="pc"/>
              </aop:aspect>
          </aop:config>
      </beans>
      ```

      

   ##### 常用expression示例：

   + The execution of any public method:

     `execution(**public** * *(..))`

   + The execution of any method with a name that begins with set:

     `execution(* set*(..))`

   + The execution of any method defined by the AccountService interface:

     `execution(* com.xyz.service.AccountService.*(..))`

   + The execution of any method defined in the service package:

     `execution(* com.xyz.service.*.*(..))`

   + The execution of any method defined in the service package or one of its sub-packages:

     `execution(* com.xyz.service..*.*(..))`

2. #### 使用注解方式配置

   + 没有配置文件
   + 配置一个切面的类，打上注解@Aspact
     + 类中的某个方法，就是我们的通知。
     + 切入点使用一个方法来标识。
   + 在测试类中，要记得开启Aspact这个开关，`@EnableAspectJAutoProxy`

   业务层实现类：

   ```java
   @Component 
   public class UserService { 
       public void save(){ 
           System.out.println("用户保存成功..."); 
       } 
   }
   ```

   自定义事务：

   ```java
   @Component
   @Aspect
   public class ServiceHandler {
   
       @Pointcut("execution(* com.tuling.pojo.aspactj.UserService.*
                 (..))")
           public void pc(){
   
       }
       @Before("pc()")
       public void beforeService(){
           System.out.println("开启事务...");
       }
   
       @AfterReturning("pc()")
       public void afterService(){
           System.out.println("提交事务...");
       }
   }
   ```

   测试：

   ```java
   @Configuration
   @ComponentScan
   @EnableAspectJAutoProxy
   public class MainTest {
       public static void main(String[] args) {
           //初始化容器
           ApplicationContext ctx = new
               AnnotationConfigApplicationContext(MainTest.class);
           //获取bean
           UserService userService = ctx.getBean("userService",
                                                 UserService.class);
           //调用bean的方法
           userService.save();
       }
   }
   ```

   