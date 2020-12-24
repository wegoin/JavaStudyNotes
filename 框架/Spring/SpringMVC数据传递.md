# **Spring_Day07_SpringMVC数据传递**

## 一、ViewResolver视图解析器

ViewResolver是视图解析器，可以根据不同的视图，相应不同的结果，比如jsp或json。

SpringMVC默认：

`org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver`

InternalResourceViewResolver：支持默认视图，采用forward，redirect。

视图名规则：

+ 不写前缀默认为"转发"。

+ 视图名字符串前缀：
  + forward:/xxx.jsp 采用转发。 例如：`new ModelAndView("forward:/carList"); `   
  + redirect:/xxx.jsp 采用重定向。例如：`new ModelAndView("redirect:/carList");`

注册视图解析器

设置视图路径的前后缀，该配置可以让我们写视图路径的时候更简单。

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
 <property name="prefix" value="/WEB-INF/web">
 <property name="suffix" value=".jsp">
</bean>
```



## 二、数据接收

以下列表单为例

```html
<body>
    <form action="/login" method="get">
        用户名：<input type="text" name="user">
        地 址：<input type="text" name="city">
        <input type="submit" value="提交">
    </form>
</body>
```

1. 通过控制器的执行方法参数来接收表单参数。

   ```java
   //接收表单参数
   @RequestMapping("/login")
   public ModelAndView login(@RequestParam("user")String username,String city){
       System.out.println(username+","+city);
       return null;
   }
   ```

2. 通过HttpServletRequest接收

   ```java
   //接收表单参数
   @RequestMapping("/login2")
   public ModelAndView login2(HttpServletRequest request,HttpServletResponse
                              response){
       String username = request.getParameter("user");
       String city = request.getParameter("city");
       System.out.println(username+","+city);
       return null;
   }
   ```

3. 通过模型对象来接收

   只需要在形参中提供用于接收数据的模型对象。

   ```java
   //方式三：接收表单参数
   @RequestMapping("/login3")
   public ModelAndView login3(User u){
       System.out.println(u.getUser()+","+u.getCity());
       return null;
   }
   ```

   <font color=red>注意：如何某个参数在模型对象中没有，则单独列出。如： public ModelAndView login3(User u,String rePwd)</font>

4. 路径变量参数 接收 （Restful风格）

   ```java
   //方式四：路径变量接受
   //http://localhost/login4/5 把5赋给id
   @RequestMapping("/login4/{id}")
   public ModelAndView login4(@PathVariable("id")Long id){
       System.out.println(id);
       return null;
   }
   ```

## 三、数据传递

1. 通过request对象进行数据传递

   ```java
   //使用Servlet的请求对象，传递数据(不推荐)
   @RequestMapping("/out1")
   public ModelAndView out1(HttpServletRequest request){
       request.setAttribute("msg", "Hello");
       ModelAndView mav = new ModelAndView("forward:/show.jsp");
       return mav;
   }
   ```

2. 使用ModelAndView进行数据传递

   ```java
   //使用ModelAndView
   @RequestMapping("/out2")
   public ModelAndView out2(){
       ModelAndView mav = new ModelAndView("forward:/show.jsp");
       // mav.addObject("msg", "今天放假啦");
       // mav.addObject("今天放假啦");//使用类型的首字母小写
       // mav.addObject("哈哈");//使用类型的首字母小写 有多个相同类型，后面的会覆盖前面的
       User user = new User();
       user.setName("张三");
       user.setCity("北京");
       mav.addObject(user);
       return mav;
   }
   ```

   

3. 通过返回值传递数据

   ```java
   //使用返回值传递数据
   //默认视图名为请求的路径名
   @RequestMapping("/out3")
   public User out3(){
       User user = new User();
       user.setName("张三");
       user.setCity("北京");
       return user;
   }
   ```

   

4. 通过Model对象进行数据传递 

   springmvc会自动创建模型对象传入到方法中，我们只需要往这个模型对象上面添加数据即可。返回值是一个字符串，如果返回值是一个字符串就代表"视图名";

   ```java
   //使用Model对象传递数据
   @RequestMapping("/out4")
   public String out4(Model model){
       model.addAttribute("msg", "哈哈");
       return "forward:/show.jsp";
   }
   ```

   

## 四、返回JSON

1. 加入jackson工具依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
   <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-databind</artifactId>
       <version>2.9.7</version>
   </dependency>
   ```

2. 如果要返回json，声明返回类型是对象，在方法上添加注解`@ResponseBody`

   ```java
   @RequestMapping("/method3")
   @ResponseBody
   public User execute(){
       User user = new User();
       user.setUser("张三");
       user.setCity("长沙");
       System.out.println("method3");
       return user;
   }
   ```

3. 返回集合

   ```java
   //返回json
   //response.getWriter().print(s);
   @RequestMapping("/out6")
   @ResponseBody
   public List<User> out6(){
       List<User> list = new ArrayList<User>();
       for(int i=0;i<10;i++){
           User user = new User();
           user.setName("张三"+i);
           user.setCity("北京"+i);
           list.add(user);
       }
       return list;
   }
   ```

   

## 五、配置模板整理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/mvc
                           http://www.springframework.org/schema/mvc/spring-mvc.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
    <bean name="/test1" class="com.tuling.controller.MyController"></bean>
    <!-- 扫描组件 -->
    <context:component-scan base-package="com.tuling"></context:component-scan>
    <!-- 开启mvc注解 -->
    <mvc:annotation-driven/>
    <!-- 开启默认静态资源访问 -->
    <mvc:default-servlet-handler/>
    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

