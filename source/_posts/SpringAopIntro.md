---
title: Spring AOP 初探
date: 2022-04-01 11:41:34
tags:
  - Spring
  - AOP
categories:
  - Spring
id: spring-aop-intro
---

# AOP概述

1. AOP为面向切面编程，利用AOP可以**隔离业务逻辑**的各个部分，进一步**解耦**并提高程序**可重用性**，以此来提高**开发效率**。

2. 通俗描述：不通过修改源代码的方式，添加新功能（类似python装饰器）

<!--more-->

## 底层原理

1. AOP底层使用动态代理实现
   
   - 有两种情况的动态代理：
     
     - 第一种 有接口情况，使用JDK动态代理
       
       创建**接口实现类**的代理对象，增强类的方法
     
     - 第二种 没有接口情况，使用CGLIB动态代理
       
       创建**子类**的代理对象，增强类的方法

## JDK动态代理

1）调用`newProxyInstance`方法：

```java
static Object newProxyInstance(ClassLoader loader, 类<?>[] interfaces, InvocationHandler h)
```

2）方法有三个参数：

- `arg0`：类加载器

- `arg1`：增强方法所在的类，这个类实现的接口（支持多个接口）

- `arg2`：实现这个接口`InvocationHandler`，创建代理对象，添加增强方法。

### JDK动态代理代码

1. 创建接口，定义方法
   
   ```java
   // UserDao.java
   public interface UserDao{
       public int add(int a, int b);
       public String update(String id);
   }
   ```

2. 创建接口实现类，实现方法
   
   ```java
   // UserDaoImpl.java
   public class UserDaoImpl implements UserDao{
       @Override
       public int add(int a, int b){
           return a + b;
       }
       @Override
       public String update(String id){
           return id
       }
   }
   ```

3. 使用`Proxy`类创建接口代理对象
   
   - 匿名内部类（`InvocationHandler`）
   
   ```java
   // JDKProxy.java
   import java.lang.reflect.Proxy
   
   public class JDKProxy{
       public static void main(String[] args){
           // 创建接口实现类代理对象
           Class[] interfaces = (UserDao.class);
           Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), new InvocationHandler(){
               @Override
               public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
                   return null;
               }
           });
       }
   }
   ```
   
   - 创建代理对象类
   
   ```java
   // JDKProxy.java
   import java.lang.reflect.Proxy
   
   public class JDKProxy{
       public static void main(String[] args){
           UserDaoImpl userDao = new UserDaoImpl();
           UserDao da = (UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), new UserDaoProxy(userDao ));
           int result = dao.add(a:1, b:2);
           System.out.println("result: " + result);
       }
   }
   
   // 创建代理对象
   class UserDaoProxy implements InvocationHandler{
       // 把创建的是谁的代理对象，把这个”谁“传递过来
       // 方法1（有参构造传递）
       private Object obj;
       public UserDaoProxy(Object obj){
           this.obj = obj;
       }
       // 增强的逻辑
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
           System.out.prinln("执行方法之前执行" + method.getName() + "；传递的参数：" + Arrarys.toString(args));
           Object res = method.invoke(obj, args);
           System.out.prinln("执行方法之后执行" + obj);
           return res;
       }
   }
   ```

# AOP术语

1. 连接点
   
   类里面的哪些方法**可以被增强**，这些方法被称为连接点

2. 切入点
   
   实际被**真正增强**的方法称为切入点

3. 通知（增强）
   
   - 实际被增强的逻辑部分成为通知（增强）
   
   - 通知有多种类型：
     
     - 前置通知：在需要方法之前进行增强
     
     - 后置通知
     
     - 环绕通知
     
     - 异常通知：执行方法出现异常的增强方法
     
     - 最终通知：类似`finally`，一定会执行

4. 切面（是一个动作）
   
   把通知应用到切入点的过程

# AOP操作（准备）

1. Spring框架一般基于`AspectJ`实现AOP操作
   
   - 什么是`AspectJ`：不是Spring组成部分，是一个独立的AOP框架。一般把`AspectJ`和SPring框架一起使用，进行AOP操作。

2. 基于`AspectJ`实现AOP操作
   
   1）基于xml配置文件
   
   2）基于注解方式实现（一般使用此方式）

3. 在项目工程引入相关依赖
   
   `spring-aop-x.x.x.RELEASE.jar`, `spring-aspects-x.x.x.RELEASE.jar`, `com.springsource.net.sf.cglib-x.x.x.jar`, `com.springsource.org.aopalliance-x.x.x.jar`, `com.springsource.org.aspectj.weaver-x.x.x.RELEASE.jar`

4. 切入点表达式
   
   1）切入点表达式作用：知道对哪个类里面的哪个方法进行增强
   
   2）语法结构：
   
   - `execution([权限修饰符] [返回类型] [类全路径] [方法名称]([参数列表]))`
   
   3）案例：
   
   1. 对`com.shimmerjordan.dao.BookDao`类里面的`add`方法进行增强
      
      `execution(* com.shimmerjordan.dao.BookDao.add(..))`
   
   2. 对`com.shimmerjordan.dao.BookDao`类里面所有方法进行增强
      
      `execution(* com.shimmerjordan.dao.BookDao.*(..))`
   
   3. 对`com.shimmerjordan.dao`包里面的所有类里面所有方法进行增强
      
      `execution(* com.shimmerjordan.dao.*.*(..))`

# 注解实现AOP操作

## 操作流程

1. 创建类（被增强类），在类里面定义方法
   
   ```java
   // User.java
   @Component
   public class User{
       public void add(){
           System.out.println("add...");
       }
   }
   ```

2. 创建增强类（编写增强逻辑）
   
   - 在增强类里面，创建方法，让不同方法代表不同通知类型。
     
     ```java
     // UserProxy.java
     @Component
     @Aspect
     public class UserProxy {
         // 前置通知
         public void before(){
             System.out.println("before...");
         }
     }
     ```

3. 进行通知的配置
   
   1）在Spring配置文件中，开启注解扫描
   
   ```xml
   <!-- 在<beans ...>中添加context和aop空间使其成为 -->
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!-- 开启注解扫描 -->
       <context:component-scan base-package="com.shimmerjordan.aopanno"></context:component-scan>
   </beans>
   ```
   
   2）使用注解创建`User`和`UserProxy`对象
   
   如1，2中代码，使用`@Component`分别注解被增强类和增强类
   
   3）在增强类上面添加注解`@Aspect`
   
   如2中代码，使用`@Aspect`注解表示生成代理对象
   
   4）在Spring配置文件中开启生成代理对象
   
   ```xml
   <!-- 开启Aspect生成代理对象 -->
   <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
   ```
   
   若是使用配置类的注解方法开启，则是使用`@EnableAspectJAutoProxy`

4. 配置不同类型通知
   
   - 在增强类里面，在作为通知的方法上面添加通知类型注解，使用切入点表达式配置
     
     ```java
     // UserProxy.java
     @Component
     @Aspect
     @Order(2)
     public class UserProxy {
         // 前置通知
         @Before(value = "execution(* com.shimmerjodan.aopanno.User.add(..))")
         public void before(){
             System.out.println("before...");
         }
     
         // 环绕通知
         @Around(value = "execution(* com.shimmerjodan.aopanno.User.add(..))")
         public void around(ProceedingJoinPoint p) throws Throwable {
             System.out.println("before around...");
             p.proceed();
             System.out.println("after around...");
         }
     }
     ```
   
   - `@After`为最终通知的注解，`@AfterReturning`为后置通知的注解

## 细节问题

1. 公共切入点抽取（重用切入点）
   
   ```java
   // 相同切入点抽取
   @Pointcut(value = "execution(* com.shimmerjordan.aopanno.User.add(..))")
   public void pointdemo(){
   
   }
   
   @Before(value = "pointdemo()")
   public void before(){
   
   }
   ```

2. 有多个增强类对同一个方法进行增强，设置增强类优先级
   
   - 在增强类上面添加注解`@Order(数字类型值)`，数字类型值越小优先级别越高
   
   ```java
   @Component
   @Aspect
   @Order(1)
   public class PrersonProxy {
       @Before(value = "execution(* com.shimmerjordan.aopanno.User.add(..))")
       public void enhencement() {
           System.out.println("Person Before...");
       }
   }
   ```

# xml配置文件实现AOP操作

## 操作流程

1. 创建两个类和被增强类并创建方法
   
   ```java
   // Book.java
   public class Book{
       public void buy() {
           System.out.println("buy...");
       }
   }
   ```
   
   ```java
   // BookProxy.java
   public class BookProxy{
       public void before() {
           System.out.println("before...");
       }
   }
   ```

2. 在Spring配置文件中创建两个类对象
   
   ```xml
   <!-- 在<beans ...>中添加context和aop空间（此处省略） -->
   <!-- 创建对象 -->
   <bean id="book" class="com.shimmerjordan.aopxml.Book"></bean>
   <bean id="bookProxy" class="com.shimmerjordan.aopxml.BookProxy"></bean>
   ```

3. 在Spring配置文件中配置切入点
   
   ```xml
   <!-- 配置aop增强 -->
   <aop:config>
       <!-- 切入点 -->
       <aop:pointcut id="p" expression="execution(* com.shimmerjordan.aopxml.Book.buy(..)"/>
   
       <!-- 配置切面 -->
       <aop:aspect ref="bookProxy">
           <!--  增强作用在具体的方法上 -->
           <aop:before method="before" pointcut-ref="p"/>
       </aop:aspect>
   </aop:config>
   ```
