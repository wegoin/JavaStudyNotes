## 在Spring中整合MyBatis

有了Spring JDBC的理论基础，Spring整合MyBatis进行数据访问就比较简单了。

### 添加所需依赖

首先为项目做类库准备，由于内容比较多，我们划分成**4部分**：

1. #### Spring相关的库

   ```xml
   <!-- spring-context：Spring基础依赖 -->
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>5.3.2</version>
   </dependency>
   <!-- spring-jdbc：只要Spring中用到了JDBC，就必须加入此依赖 -->
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-jdbc</artifactId>
       <version>5.3.2</version>
   </dependency>
   <!-- spring-aspects：只要Spring中用到了AOP，就必须加入此依赖 -->
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aspects</artifactId>
       <version>5.3.2</version>
   </dependency>
   ```

   

2. #### MyBatis相关依赖

   ```xml
   <!-- mybatis：MyBatis基础依赖 -->
   <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis</artifactId>
       <version>3.5.6</version>
   </dependency>
   <!-- mybatis-spring： 只要Spring中用到了MyBatis，就必须加入此依赖-->
   <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis-spring</artifactId>
       <version>2.0.6</version>
   </dependency>
   <!-- mybatis-generator-core：MyBatis逆向工程 -->
   <dependency>
       <groupId>org.mybatis.generator</groupId>
       <artifactId>mybatis-generator-core</artifactId>
       <version>1.4.0</version>
   </dependency>
   ```

   <font color=red>注意：spring整合mybatis的包“mybatis-spring”是由mybatis公司制作提供的。</font>

3. #### 数据访问相关依赖

   ```xml
   <!-- mysql-connector-java：MySQL的驱动 -->
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>5.1.47</version>
   </dependency>
   <!-- commons-dbcp2：DBCP2数据源 -->
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-dbcp2</artifactId>
       <version>2.8.0</version>
   </dependency>
   ```

   

4. #### 其他

   ```xml
   <!-- 单元测试 -->
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>3.8.1</version>
       <scope>test</scope>
   </dependency>
   <!-- log4j-core：Log4j2日志框架 -->
   <dependency>
       <groupId>org.apache.logging.log4j</groupId>
       <artifactId>log4j-core</artifactId>
       <version>2.14.0</version>
   </dependency>
   ```

   

配置日志 log4j2.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="STDOUT"  target="SYSTEM_OUT">
            <PatternLayout pattern="%-5p  [%C{2}] (%F:%L) - %m%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="STDOUT" />
        </Root>
    </Loggers>
</Configuration>
```

使用 <font color=green>**MyBatis Generator** </font>工具生成好相应的实体类、映射文件和数据访问接口。

### 搭建项目架构

1. #### 新建resources资源目录，并创建mbg.xml

   mgb.xml配置文件用来配置Generator代码生成器工具

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE generatorConfiguration PUBLIC  "-//mybatis.org//DTD MyBatis Generator  Configuration 1.0//EN"  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
   <generatorConfiguration>
       <context id="DB2Tables"  targetRuntime="MyBatis3">
           <!-- 配置内置的或者自定义的Plugin -->
           <plugin  type="org.mybatis.generator.plugins.SerializablePlugin"/>
           <commentGenerator>
               <property name="suppressAllComments"  value="true" />
           </commentGenerator>
           <!-- 数据库链接URL、用户名、密码 -->
           <jdbcConnection  driverClass="com.mysql.jdbc.Driver"  connectionURL="jdbc:mysql://127.0.0.1:3306/bank"  userId="root" password="root">
           </jdbcConnection>
           <javaTypeResolver>
               <property name="forceBigDecimals"  value="false" />
           </javaTypeResolver>
           <!-- 生成模型的包名和位置 -->
           <javaModelGenerator  targetPackage="com.turing.sm.entity"  targetProject=".\src\main\java">
               <property name="enableSubPackages"  value="true" />
               <property name="trimStrings"  value="true" />
           </javaModelGenerator>
           <!-- 生成的Mapper接口和位置 -->
           <sqlMapGenerator  targetPackage="com.turing.sm.mapper"  targetProject=".\src\main\java">
               <property name="enableSubPackages"  value="true" />
           </sqlMapGenerator>
           <!-- 生成Mapper配置文件和位置 -->
           <javaClientGenerator type="XMLMAPPER"  targetPackage="com.turing.sm.mapper"  targetProject=".\src\main\java">
               <property name="enableSubPackages"  value="true" />
           </javaClientGenerator>
           <!-- 要生成哪些表(更改tableName和domainObjectName就可以) -->
           <table tableName="account" />
       </context>
   </generatorConfiguration>
   ```

   

2. #### 执行Generator代码生成器工具

   在src/test/java/com.turing.sm包下新建类MybatisTest

   ```java
   @Test
   public void create() throws Exception{
       List<String> warnings = new  ArrayList<String>();
       boolean overwrite = true;
       File configFile = new File(MybatisTest.class.getClassLoader().getResource("mbg.xml").getPath());
       ConfigurationParser cp = new  ConfigurationParser(warnings);
       Configuration config =  cp.parseConfiguration(configFile);
       DefaultShellCallback callback = new  DefaultShellCallback(overwrite);
       MyBatisGenerator myBatisGenerator = new  MyBatisGenerator(config, callback, warnings);
       myBatisGenerator.generate(null);
   }
   ```

   使用单元测试执行。

   把资源包下，新建一个和mapper一样的包，把Generator代码生成器工具生成的xml放到里面（为了使开发环境下xml文件和Java文件分开，实际上在编译后他们还是在同一个路径下）。

### 配置

1. #### 数据源配置

   新建包com.turing.ms.config。新建类SpringConfig

   ```java
   @Configuration
   @ComponentScan(basePackages="com.tuling")
   public class SpringConfig {
   
       //配置数据源
       @Bean
       public DataSource dataSource(){
           BasicDataSource ds = new BasicDataSource();
           ds.setDriverClassName("com.mysql.jdbc.Driver");
           ds.setUrl("jdbc:mysql:///bank");
           ds.setUsername("root");
           ds.setPassword("root");
   
           ds.setInitialSize(5);
           ds.setMaxTotal(15);
           ds.setMaxIdle(8);
           ds.setMinIdle(3);
           ds.setMaxWaitMillis(10000);
           ds.setValidationQuery("select 1");
   
           return ds;
       }
   }
   ```

   

2. #### SqlSessionFactory配置

   由于MyBatis对JDBC组件进行了轻量级封装，最终通过<font color=green>SqlSession</font>进行数据访问，所以我们接下来配置<font color=green>SqlSessionFactory</font>，好让后续的bean能得到<font color=green>SqlSession</font>。

   在SpringConfig类中加入：

   ```java
   /**
   	 * 配置sqlSessionFactory，其实就等同于之前MyBatis配置的mybatis-config.xml
   	 * @param dataSource
   	 * @return
   	 * @throws Exception
   	 */
   @Bean
   public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception{
       //创建SqlSessionFactoryBean对象
       SqlSessionFactoryBean bean=new SqlSessionFactoryBean();
       //设置数据源
       bean.setDataSource(dataSource);
       //返回SqlSessionFactory对象
       return bean.getObject();
   }
   ```

   在上面这段配置中，MyBatis将会得到DBCP数据源，对JDBC中的基础组件进行封装，最后产生<font color=green>`SqlSessionFactory`</font>实例，交给Spring容器进行管理。对比之前单独使用MyBatis的配置，从基本内容上来看是一致的，只不过表现方式不一样。

3. #### SqlSessionTemplate配置

   配置好<font color=green>`SqlSessionFactory`</font>以后，我们就可以使用spring为我们准备的<font color=green>`SqlSessionTemplate`</font>进行数据问开发了，在配置<font color=green>`SqlSessionTemplate`</font>的时候我们需要传入<font color=green>`SqlSessionFactory`</font>进行构造

   在SpringConfig类中加入：

   ```java
   //使用MyBatis的模板---SqlSessionTemplate
   @Bean
   public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory){
       return new SqlSessionTemplate(sqlSessionFactory);
   }
   ```

   

### 代码设计

+ #### 业务层：

  + 接口

    ```java
    /**
     * Account业务层接口
     * @author miwei
     *
     */
    public interface AccountService {
    
        /**
    	 * 添加
    	 * @param acc
    	 * @return
    	 */
        int addAccount(Account acc);
    
    }
    ```

    

  + 实现类

    ```java
    @Service
    public class AccountServiceImpl implements AccountService{
    
        @Autowired
        private SqlSessionTemplate template;
    
        @Override
        public int addAccount(Account acc) {
            return template.insert("insert", acc);
        }
    
    }
    ```

    

+ #### 扫描mapper

  在SpringConfig类上加上注解

  `@MapperScan(basePackages="com.turing.sm.mapper") //扫描Mapper`

+ 使用MyBatis模板

  在SpringConfig类中加入：

  ```java
  //使用MyBatis的模板---SqlSessionTemplate
  @Bean
  public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory){
      return new SqlSessionTemplate(sqlSessionFactory);
  }
  ```

  

+ #### 测试

  ```java
  //测试添加
  public void testAdd(){
      //创建上下文对象
      ApplicationContext ctx=new AnnotationConfigApplicationContext(SpringConfig.class);
      //获取bean
      AccountService accountService = ctx.getBean(AccountService.class);
      //调用添加方法
      Account acc=new Account();
      acc.setAname("刘德华");
      acc.setAmoney(100);
      accountService.addAccount(acc);
      System.out.println("添加成功！");
  }
  ```

  

## Spring中的事务处理

在业务层接口中添加转账的方法，“转账”需要依赖实体类对象，所以添加“根据id查询”的方法。

```java
/**
	 * 转账
	 * @return
	 */
int transAccount();

/**
	 * 根据id查询
	 * @param id
	 * @return
	 */
Account findById(int id);
```

对应的实现类：

```java
@Override
public int transAccount() {
    //张三给李四转100元

    //李四的钱+100
    Account lisi=findById(2);
    lisi.setAmoney(lisi.getAmoney()+100);
    int result1=template.update("updateByPrimaryKeySelective", lisi);

    int a=1/0;

    //张三的钱-100
    Account zhangsan=findById(1);
    zhangsan.setAmoney(zhangsan.getAmoney()-100);
    int result2=template.update("updateByPrimaryKeySelective", zhangsan);
    return result1+result2;
}

@Override
public Account findById(int id) {
    return template.selectOne("selectByPrimaryKey", id);
}
```

### 添加事务管理器

Spring在管理事务的时候，提供了一个跨平台管理事务的组件`PlatformTransactionManager`。

这个接口下面，对应每一个数据持久化框架，都有对应的实现类，MyBatis对应的为`DataSourceTransactionManager`，这个类将会通过基本的JDBC接口来对事务进行管理，我们只需要在配置类中声明这个类就可以了。

在SpringConfig类加入：

```java
//配置事务管理器
@Bean
public DataSourceTransactionManager transactionManager(DataSource dataSource){
    return new DataSourceTransactionManager(dataSource);
}
```

#### 使用xml配置方式：

声明`DataSourceTransactionManager`后，如果没有额外的配置，那么默认的事务边界就是数据访问接口的每一个方法，也即是，每一个数据访问接口的方法都是自成一个事务单元。

但在实际开发中，我们的事务边界一般都是定义在服务层的，接下来我们将使用Xml的声明式事务配置来定义事务边界。

以Xml的方式定义事务，我们除开要使用Spring提供的beans这个基础服务外，还需要tx及aop服务，为此新建`spring-transaction.xml`，并在头文件中声明如下命名空间：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
</beans>
```



在resources资源包下新建spring-transaction.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 配置事务通知 -->
    <!-- transaction-manager就是IOC容器中配置的事务管理器，建议名称保持一致 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!-- 配置事务的传播行为 -->
        <tx:attributes>
            <!-- 参数，
               	name：需要配置的方法名；
               	propagation：传播行为； 
                REQUIRED：需要事务，通常用在增删改
                SUPPORTS：不需要事务，通常用在查询         
               	read-only：是否只读
                false：需要事务
                true：不需要事务
             -->

            <!-- 增删改需要事务 -->
            <tx:method name="transAccount" propagation="REQUIRED" read-only="false"/>
            <tx:method name="add*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="upd*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="del*" propagation="REQUIRED" read-only="false"/>

            <!-- 查询方法不要事务 -->
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>

    <!-- AOP配置 -->
    <aop:config>
        <!-- 配置切入点 -->
        <aop:pointcut expression="execution(* com.turing.sm.service..*.*(..))" id="pc"/>
        <!-- 配置切面 -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pc"/>
    </aop:config>

</beans>
```

为了加载这份事务配置，我们需要在BeanConfig.java上增加一个导入Xml配置的注解：`@ImportResource("classpath:spring-transaction.xml")//文件的加载方式：classpath:类路径（默认）；file：文件路径`

<font color=red>注意：最新版的`spring-aspects`依赖存在不兼容问题。我们需要修改依赖版本</font>

```xml
<!-- spring-aspects：只要Spring中用到了AOP，就必须加入此依赖 -->
<!-- <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>5.3.2</version>
  </dependency> -->
<!-- 在Spring最新版中使用aspectjweaver1.9报错，降级到1.8 -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.14</version>
    <scope>runtime</scope>
</dependency>
```

#### 使用注解配置方式

单独为某个方法设置事务，可以在需要事务的方法上，添加一个注解：

`@Transactional(propagation=Propagation.REQUIRED,readOnly=true)`

例：

```java
@Transactional(propagation=Propagation.REQUIRED,readOnly=false)
public int transAccount() {
    //张三给李四转100元

    //李四的钱+100
    Account lisi=findById(2);
    lisi.setAmoney(lisi.getAmoney()+100);
    int result1=template.update("updateByPrimaryKeySelective", lisi);

    int a=1/0;

    //张三的钱-100
    Account zhangsan=findById(1);
    zhangsan.setAmoney(zhangsan.getAmoney()-100);
    int result2=template.update("updateByPrimaryKeySelective", zhangsan);
    return result1+result2;
}
```

在Spring-Config配置类上，添加开启事务注解：`@EnableTransactionManagement`

