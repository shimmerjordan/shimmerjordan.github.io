---
title: Spring IOC底层原理
date: 2022-03-28 12:41:34
tags:
  - Spring
  - Interview
  - IOC
categories:
  - Spring
id: spring-ioc-intro
---

# IOC概念和原理

## 什么是IOC

1. 控制反转：把对象创建和对象之间的调用过程，交给Spring进行管理

2. 使用IOC的目的：为了耦合度降低

<!--more-->

## IOC底层原理

1. xml解析、工厂模式、反射

2. 工厂模式利用工厂一定程度降低耦合

3. IOC过程：
   
   1. xml配置文件，配置创建的对象：
      
      ```xml
      <bean id="dao" class="com.shimmerjordan.UserDao"></bean>
      ```
   
   2. 有Service类和Dao类，创建工厂类（进一步降低耦合）
      
      ```java
      class UserFactory{
          public static UserDao getDao(){
              String classValue = class属性值;    //1. xml解析
              // 2. 通过反射创建对象（反射：得到类的字节码文件）
              Class clazz = Class.forName(classValue);
              return (UserDao) clazz.newInstance();
          }
      }
      ```

## IOC接口

1. IOC思想基于IOC容器完成，IOC容器底层就是对象工厂

2. Spring提供IOC容器实现的两种方式（两个接口）：
   
   1. `BeanFactory`：IOC容器基本实现，是Spring内部使用接口，不提供开发人员使用。**加载配置文件的时候不会创建对象，在获取对象（使用）的时候才会创建**
   
   2. `ApplicationContext`：BeanFactory接口的子接口，提供更多更强大的功能（继承），一般由开发人员使用。**在加载配置文件的时候就会把在配置文件中的对象进行创建**

3. `ApplicationContext`接口有很多实现类，包括`FileSystemXmlApplicationContext`和`ClassPathXmlApplicationContext`

# IOC操作Bean管理

## 什么是Bean管理（指两个操作）

1. Spring创建对象（实例化）

2. Spring注入属性（初始化）

## Bean管理操作的两种方式

1. 基于xml配置文件方式实现

2. 基于注解方式实现

## 基于xml配置文件方式进行Bean管理

1. 基于xml配置文件进行对象创建
   
   - 在Spring配置文件中，使用`bean`标签，标签里面添加对应属性即可完成对象创建：
   
   ```xml
   <bean id="dao" class="com.shimmerjordan.UserDao"></bean>
   ```
   
   - `bean`标签包含很多属性，常用属性：
     
     - `id`：唯一标识
     
     - `class`：类全路径（包含路径）
     
     - `name`：类似于`id`，但`name`可以包含特殊符号
   
   - 创建对象的时候，默认执行无参构造方法完成对象创建

2. 基于xml方式注入属性
   
   1. DI：依赖注入：就是注入属性，需要在创建对象的基础上完成。**是IOC中的一种具体实现**。
      
      - `set`方法注入
        
        1. 创建类，定义属性和对应的set方法
           
           ```java
           // Book.java
           public class Book{
               // 创建属性
               private String bname;
               private String bauthor;
               // 创建属性对应的set方法
               public void setBname(String bname){
                   this.bname = bname;
               }
               public void setBauthor(String bauthor){
                   this.bauthor = nauthor
               }
           }
           ```
        
        2. 在Spring配置文件配置对象的创建，配置属性的注入
           
           ```xml
           <!-- xml文件 -->
           <!-- set方法注入属性 -->
           <bean id="book" class="com.shimmerjordan.Book">
               <property name="bname" value="balabala"></property>
               <property name="bauthor" value="muximuxi"></property>
           </bean>    
           ```
      
      - 有参构造注入
        
        1. 创建类，定义属性，创建属性对应有参数构造方法
           
           ```java
           // Book.java
           public class Book{
               // 创建属性
               private String bname;
               private String bauthor;
               // 有参构造
               public Book(String bname, String bauthor){
                   this.bname = bname;
                   this.bauthor = bauthor;
               }
           }
           ```
        
        2. 在Spring配置文件配置对象的创建，配置属性的注入
           
           ```xml
           <!-- xml文件 -->
           <!-- 有参构造注入属性 -->
           <bean id="book" class="com.shimmerjordan.Book">
               <constructor-arg name="bname" value="balabala"></constructor-arg>
               <constructor-arg name="bauthor" value="muximuxi"></constructor-arg>
               <!-- 若使用index="k"属性，则是有参构造中第k个属性 -->
           </bean> 
           ```
      
      - p名称空间注入
        
        使用p名称空间注入，可以简化基于xml配置方式（底层依旧是set方法）
        
        - 添加p名称空间在配置文件中
        
        ```xml
        <!-- 在<beans ...>中添加p空间使其成为 -->
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:p="http://www.springframework.org/schema/p"
            xsi:schemaLocation="
                http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        ```
        
        - 进行属性注入，在bean标签里进行操作
        
        ```xml
        <bean id="book" class="com.shimmerjordan.Book" p:bname="balabala" p:bauthor="muximuxi">
        </bean>
        ```

## Bean管理（xml注入其他类型属性）

1. 字面量：属性的静态预设值
   
   1. Null值
      
      ```xml
      <property name="bname">
          <null/>
      </property>
      ```
   
   2. 包含特殊符号
      
      - 转义
      
      - 使用`CDATA`数据结构
      
      ```xml
      <property name="bname">
          <value> <![CDATA[<<南京>>]]> </value>
      </property>
      ```

2. 注入属性-外部Bean
   
   （1）创建两个类`service`类和`dao`类
   
   （2）在`service`调用`dao`里面的方法
   
   （3）在Spring配置文件中进行配置
   
   ```java
   // UserService.java
   public class UserService{
       // 创建UserDao类型属性，生成set方法
       private UserDao userDao;
       public void setUserDao(UserDao userDao){
           this.userDao = userDao;
       }
       public void add(){
           System.out.println("service add");
           userDao.update();
           // 原始方法
           //    UserDao userDao = new UserDaoImpl();
           //    userDao.update();
       }
   }
   ```
   
   ```xml
   <!-- service和dao对象创建 -->
   <bean id="userService" class="com.shimmerjordan.service.UserService">
       <!-- 注入userDao对象
           name属性：类里面属性名称
           ref属性：创建userDao对象bean标签的id值
       -->
       <property name="userDao" ref="userDaoImpl"></property >
   </bean> 
   <bean id="userDaoImpl" class="com.shimmerjordan.dao.UserDaoImpl"></bean>
   ```

3. 注入属性-内部Bean
   
   - 一对多关系：部门和员工。
   
   - 在实体类之间表示一对多的关系
   
   ```java
   // Dept.java
   public class Dept{
       private String dname;
       public void setDname(String dname){
           this.dname = dname;
       }
   }
   ```
   
   ```java
   // Emp.java
   public class Dept{
       private String ename;
       private String gender;
       // 员工属于某一个部门，使用对象形式表示、
       private Dept dept;
       public void setDept(Dept dept){
           this.dept = dept;
       }
   
       public void setDname(String ename){
           this.ename= ename;
       }
       public void setGender(String gender){
           this.gender= gender;
       }
   }
   ```
   
   ```xml
   <bean id="emp" class="com.shimemrjordan.bean.Emp">
       <property name="ename" value="lucy"></property>
       <property name="gender" value="female"></property>
       <!-- 设置对象类型属性 -->
       <property name="dept">
           <bean id="dept" class="com.shimmerjordan.bean.Dept">
               <property name="dname" value="HR"></property>
           </bean>
       </property>
   </bean>
   ```

4. 级联赋值
   
   - 第一种写法
   
   ```xml
   <bean id="emp" class="com.shimemrjordan.bean.Emp">
       <property name="ename" value="lucy"></property>
       <property name="gender" value="female"></property>
       <!-- 设置对象类型属性（级联赋值） -->
       <property name="dept" ref="dept"></property>
   </bean>
   <bean id="dept" class="com.shimmerjordan.bean.Dept">
       <property name="dname" value="HR"></property>
   </bean>
   ```
   
   - 第二种写法
   
   ```java
   // 需要在Emp.java中添加生成dept的get方法
   public Dept getDept(){
       return dept;
   }
   ```
   
   ```xml
   <bean id="emp" class="com.shimemrjordan.bean.Emp">
       <property name="ename" value="lucy"></property>
       <property name="gender" value="female"></property>
       <!-- 设置对象类型属性（级联赋值） -->
       <property name="dept.dname" value="HR"></property>
   </bean>
   ```

## Bean管理（xml注入集合属性）

1. 注入数组类型属性 

2. 注入List集合类型属性

3. 注入Map集合类型属性
   
   ```java
   // Stu.java
   package com.shimmerjrodan.collectiontype
   public class Stu{
       private String[] courses;
       private List<String> list;
       private Map<String, String> maps;
       private Set<String> sets;
       // 各自set方法省略
   }
   ```
   
   ```xml
   <bean id="stu" class="com.shimemrjordan.collectiontype.Stu">
       <!-- 数组类型属性注入 -->
       <property name="courses">
           <array>
               <value>Java</value>
               <value>Python</value>
           </array>
       </property>
        <!-- List类型属性注入 -->
       <property name="list">
           <list>
               <value>Java</value>
               <value>Python</value>
           </list>
       </property>
       <!-- Map类型属性注入 -->
       <property name="maps">
           <map>
               <entry key="JAVA" value="java"></entry>
               <entry key="PYTHON" value="python"></entry>
           </map>
       </property> 
       <!-- Set类型属性注入 -->
       <property name="list">
           <set>
               <value>Java</value>
               <value>Python</value>
           </set>
       </property>
   </bean>
   ```

4. 注入集合属性的值为对象
   
   ```xml
   <bean id="stu" class="com.shimemrjordan.collectiontype.Stu">
       <property name="list">
           <list>
               <ref bean="course1"></ref>
               <ref bean="course2"></ref>
           </list>
       </property>
   </bean>
   <bean id="course1" class="com.shimemrjordan.collectiontype.Course">
       <property name="cname" value="balabala"></property>
   </bean>
   <bean id="course2" class="com.shimemrjordan.collectiontype.Course">
       <property name="cname" value="muximuxi"></property>
   </bean>
   ```

5. 提取集合类型属性注入（`util`）
   
   ```xml
   <!-- 在<beans ...>中添加    util空间使其成为 -->
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
       <!-- 提取list集合类型属性 -->
       <util:list id="bookList">
           <value>Python</value>
           <value>Java</value>
       </util:list>
   
       <!-- 注入使用 -->
       <bean id="book" class="com.shimemrjordan.collectiontype.Book">
           <property name="list" ref="bookList"></property>
       </bean>
   </beans>
   ```

## Bean管理（FactoryBean）

1. Spring有两种类型的bean，普通bean以及FactoryBean
   
   - 普通bean：在配置文件中定义bean类型就是返回类型
   
   - FactoryBean：在配置文件中定义bean类型可以和返回类型不一样
     
     . 具体流程
     
       第一步：创建类，让这个类作为工厂bean，实现接口FactoryBean
     
     ```java
     public class MyBean implements FactoryBean<Course>{
         // 定义返回bean
         @Override
         public Object getObject() throws Exception{
             Course course = new Course();
             course.setCname("Python");
             return course;
         }
         @Override
         public Class<?> getObjectType(){
             return null;
         }
     }
     ```
     
     ```xml
     <bean id="myBean" class="com.shimmerjordan.factorybean.MyBean">
     </bean>
     ```
     
       第二步：实现接口里面的方法，在实现的方法中定义返回的bean类型
     
     ```java
     // TestDemo.java
     @Test
     public void test(){
         ApplicationContext context = 
             new ClassPathXmlApplicationContext(configLocation:"bean.xml");
         Course course = context.getBean(s:"myBean", Course.class);
         System.out.println(course);
     }
     ```

## Bean管理（作用域）

1. 在Spring里面，设置创建bean实例是单实例还是多实例。

2. Spring默认创建单实例
   
   ```java
   Book book1 = context.getBean(s:"book", Book.class);
   Book book2 = context.getBean(s:"book", Book.class);
   System.out.println(book1);
   System.out.println(book2);
   ```
   
   将输出同一地址（默认创建单实例）

3. 如何设置多实例或者单实例
   
   - 通过配置文件bean标签中`scope`属性进行设置
   
   - 常用`scope`值：
     
     - `singleton`（默认单实例），**加载**Spring配置文件时候就会创建单实例对象
     
     - `prototype`（多实例），不在加载时创建，而是在**调用**`getBean`方法的时候创建多实例对象
     
     - `request`, `session`

## Bean管理（生命周期）

1. 生命周期：从对象创建到对象销毁的过程

2. bean生命周期
   
   - 1）通过构造器创建bean实例（执行无参构造）
   
   - 2）为bean的属性设置值和对其它bean的引用（调用set方法）
   
   - 3）调用bean的初始化方法（需要进行配置）
   
   - 4）bean可以使用（获取得对象）
   
   - 5）当容器关闭时，调用bean的销毁方法（需要进行配置销毁方法）
   
   **bean生命周期演示**
   
   ```java
   // Order.java
   public class Orders{
       // 无参数构造
       public Orders(){
           System.out.println("第一步 执行无参构造创建bean实例")；
       }
       private String oname;
       public void setOname(String onam){
           this.oname = oname;
           System.out.println("第二步 调用set方法设置属性值");
       }
   
       // 创建执行的初始化方法
       public viud initMethod(){
           System.out.println("第三步 执行初始化方法");
       }
   
       // 执行bean的销毁方法
       public void destroyMethod(){
           System.out.println("第五步 执行销毁的方法");
       }
   }
   ```
   
   ```xml
   <bean id="Orders" class="com.shimmerjordan.bean.Orders"
       init-method="initMethod" destory-method="destroyMethod">
       <property name="oname" value="phone"></property>
   </bean>
   ```
   
   ```java
   // Test.java
   @Test
   public void test(){
       ApplicationContext context =
           new ClassPathXmlApplicationContext(configLocation:"bean.xml");
       Orders orders = context.getBean(s:"orders", Orders.class);
       System.out.println("第四步 获取创建bean实例对象");
       System.out.println(course);
       // 手动销毁bean
       ((ClassPathXmlApplicationContext) context).close();
   }
   ```

3. Bean的后置处理器（BeanPostProcessor）
   
   若使用后置处理器，则为**七步**生命周期：
   
   - 1）通过构造器创建bean实例（执行无参构造）
   
   - 2）为bean的属性设置值和对其它bean的引用（调用set方法）
   
   - **2.1）把bean实例传递bean后置处理器的方法`postProcessBeforeInitialization`**
   
   - 3）调用bean的初始化方法（需要进行配置）
   
   - **3.1）把bean实例传递bean后置处理器的方法`postProcessAfterInitialization`**
   
   - 4）bean可以使用（获取得对象）
   
   - 5）当容器关闭时，调用bean的销毁方法（需要进行配置销毁方法）
   
   **生命周期演示：**
   
   创建类，实现接口`BeanPostProcessor`，创建后置处理器
   
   ```java
   // MyBeanPost.java
   public class MyBeanPost implements BeanPostProcessor{
       @Override
       public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException{
           System.out.println("2.1 在初始化之前执行的方法");
           return bean;
       }
       @Override
       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException{
           System.out.println("3.1 在初始化之后执行的方法");
           return bean;
       }
   }
   ```
   
   xml文件中需要加入后置处理器的配置，为**所有**实例添加此后置处理器。
   
   ```xml
   <bean id="myBeanPost" class="com.shimmerjordan.bean.MyBeanPost">
   </bean>
   ```

## Bean管理（xml自动装配）

1. 什么是自动装配：根据指定装配规则（属性名称或者属性类型），Spring自动将匹配的属性值进行注入。

2. 演示自动装配过程：
   
   ```xml
   <!-- 实现自动装配
        bean标签属性autowire配置自动装配
        autowire属性常用两个值：
           byName：根据属性名称注入，注入值bean的id值和类属性名称一致
           byType：根据属性类型注入（多个相同类型可能导致错误）
   -->
   <bean id="emp" class="com.shimmerjordan.bean.Emp" autowire="byName">
   </bean>
   <bean id="dept" class="com.shimmerjordan.bean.Dept"></bean>
   ```

## Bean管理（外部属性文件）

1. 直接配置数据库信息：
   
   - 配置`druid`连接池
   
   - 引入`druid.jar`并创建连接池对象
     
     ```xml
     <!-- DruidDataSource dataSource = new DruidDataSource(); -->
     <bean id="dataSource" class="class.alibaba.druid.pool.DruidDataSource">
         <!-- dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
             set方法注入
         -->
         <!-- 获取propertied文件内容，根据key获取，使用Spring表达式获取 -->
         <property name="driverClassName" value="${jdbc.driverclass}"></property>
         <property name="url" value="${jdbc.url}"></property>
         <property name="username" value="${jdbc.username}"></property>
         <property name="password" value="${jdbc.password}"></property>
     </bean>
     ```

2. 引入外部属性文件配置数据库连接池
   
   - 创建
     
     ```properties
     # jdbc.properties
     jdbc.driverclass=com.mysql.cj.jdbc.Driver
     jdbc.url=jdbc:mysql://localhost:3306/userDb
     jdbc.username=root
     jdbc.password=password
     ```
   
   - 在xml文件中引入外部属性文件
     
     - 加入`context`名称空间：
       
       ```xml
       <!-- 在<beans ...>中添加context空间使其成为 -->
       <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="
               http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
               http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
       ```
     
     - 引入配置属性
       
       ```xml
       <context:property-placeholder location="classpath:jdbc.properties"/>
       ```

## Bean管理（基于注解方式）

1. 什么是注解
   
   - 注解是代码的特殊标记
   
   - 格式：`@注解名称(属性名称=属性值, 属性名称=属性值...)`
   
   - 注解可以作用在类、方法、属性上
   
   - 注解的目的：简化xml配置

2. Spring针对Bean管理中创建对象提供注解
   
   - `@Component`
   
   - `@Service`
   
   - `@Controller`
   
   - `@Repository`
   
   *以上四个注解的功能一样，都可以用来创建bean实例*

3. 基于注解方式实现对象创建
   
   1）引入依赖：`spring-aop-x.x.x.RELEASE.jar`
   
   2）开启组件扫描
   
   - 引入`context`名称空间
   
   - 开启组件扫描
     
     - 多个属性使用`,`分割，例如`base-package="com.shimmerjordan.dao,com.shimmerjordan.service"`
     
     - 多个属性配置共同上层目录，例如`base-package="com.shimmerjordan">`
     
     ```xml
     <context:component-scan base-package="com.shimmerjordan.dao"></context:component-scan>
     ```
   
   3）创建类，在类上面添加创建对象注解
   
   ```java
   // UserService.java
   // 注解里value属性值可以省略，默认是类名称的首字母小写
   @Component(value = "userService")    // <bean id="userService" class=".."/>
   public class UserService{
       public void add(){
           System.out.println("service add");
       }
   }
   ```

4. 开启组件扫描的细节配置
   
   示例一：
   
   - `use-default-filters="false"`表示不适用默认`filter`，需要自己配置`filter`规则
   
   - 使用`context:include-filter`设置扫描内容
   
   ```xml
   <context:component-scan base-package="com.shimmerjordan" use-default-filters="false">
       <!-- 只扫描带Controller注解的类 -->
       <context:include-filter type="annotation"
               expression="org.springframework.stereotype.Controller"/>
   </context:component-scan>
   ```
   
   示例二：
   
   - 扫描所有内容，但是使用`context:exclude-filter`**排除**特定扫描内容
   
   ```xml
   <context:component-scan base-package="com.shimmerjordan">
       <!-- 不扫描带Controller注解的类 -->
       <context:exclude-filter type="annotation"
               expression="org.springframework.stereotype.Controller"/>
   </context:component-scan>
   ```

5. 实现属性的注入
   
   - `@AutoWired`：根据属性类型进行自动装配
     
     1） 创建service和dao对象，在service和dao类添加创建对象注解
     
     ```java
     // UserDaoImpl.java
     @Repository
     public class UserDaoImpl implements UserDao{
         @Override
         public void add(){
             System.out.println("dao add");
         }
     }
     ```
     
     ```java
     // UserService.java
     @Repository
     public class UserService{
         public void add(){
             System.out.println("service add");
         }
     }
     ```
     
     2）在serive注入dao对象，在service类添加dao类型属性，在属性上面使用注解
     
     ```java
     // UserService.java
     @Repository
     public class UserService{
         // 定义dao类型属性
         // 不需要添加set方法
         // 添加注入属性注解
         @AutoWired
         private UserDao userDao;
         public void add(){
             System.out.println("service add");
             userDao.add();
         }
     }
     ```
   
   - `@Qualifier`：根据属性名称进行注入
     
     需要和`@AutoWired`一起使用，类似bean标签中autowire配置的`byName`
     
     若`UserDao`有多个实现类，需要在实现类上加注解`@Repository(value = "userDaoImpl1")`进行标识（默认`value`为类名首字母小写）。并在`UserService`类的属性上添加`@Qualifier(value="userDaoImpl1")`（搭配`@Autowired`同时存在
   
   - `@Resource`：可以根据属性类型进行注入，也可以根据属性名称进行注入。默认按照属性名注入，按照类型注入要使用`@Resource(type=UserDao)`。（此注解属于javax包，jdk11移除了此注解）
   
   - `@Value`：注入普通类型属性
     
     ```java
     @Value(value = "AD")
     private String uname;
     ```

6. 完全注解开发
   
   1. 创建配置类，替代xml文件
      
      ```java
      // SpringConfig.java
      @Configuration    // 作为配置类替代xml文件
      @ComponentScan(basePackages = {"com.shimmerjordan"})
      public class SpringConfig{
      
      }
      ```
   
   2. 编写测试类
      
      - `ClassPathXmlApplicationContext`需要改写为`AnnotationConfigApplicationContext`方法
      
      ```java
      // TestDemo.java
      @Test
      public void test(){
          // 加载配置类
          ApplicationContext context = 
              new AnnotationConfigApplicationContext(SpringConfig.class);
          Course userService= context.getBean(s:"userService", UserService.class);
          System.out.println(userService);
      }
      ```
