---
title: IDEA整合Maven+SSM框架
date: 2020-06-14 18:36:38
tags:
	- Spring
	- SpringMVC
	- Mybatis
categories: SSM
id: IDEAMavenSSM
---

前面对于Spring、SpringMVC以及Mybatis的学习后，这里基于IDEA开发工具将SSM整合为Maven工程。

<!--more-->

# 1. 搭建整合环境

## 1.1 整合说明

​		整合说明：SSM整合可以使用多种方式，这里选择XML + 注解的方式，这并没有什么不妥，反正这样最简洁

## 1.2 整合的思路：

1. 先搭建整合的环境

2. 搭建Spring的配置环境

3. 再使用Spring整合SpringMVC框架

4. 随后使用Spring整合MyBatis框架

5. 最后Spring整合MyBaits框架配置事务（Spring声明式事务管理）

## 1.3 创建数据库和表格结构语句

建表sql代码：

```sql
create database ssm;
use ssm;
create table account (
id int primary key auto_increment,
name varchar(50),
money double
);
```

## 1.4 创建Maven工程

1. 创建Twossm_parent父工程（打包方式选择pom，必须的）
2. 创建Twossm_web子模块（打包方式是war包）
3. 创建Twossm_service子模块（打包方式是jar包）
4. 创建Twossm_dao子模块（打包方式是jar包）
5. 创建Twossm_domain子模块（打包方式是jar包）
6. web依赖于service，service依赖于dao，dao依赖于domain
7. 在Twossm_parent的pom.xml文件中引入坐标依赖，找到对应的< properties >标签，以及< dependencies >标签，复制粘贴即可
   版本控制是在< properties >标签中控制，从坐标依赖中可以看出版本号：spring5X、MySQL8.0.15、mybatis3.4.5

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <spring.version>5.0.2.RELEASE</spring.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <mysql.version>8.0.15</mysql.version>
    <mybatis.version>3.4.5</mybatis.version>
  </properties>


<dependencies>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.6.8</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>${mysql.version}</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency> <!-- log start -->
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>${log4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>${slf4j.version}</version>
    </dependency> <!-- log end -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>${mybatis.version}</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.0</version>
    </dependency>
    <dependency>
      <groupId>c3p0</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.1.2</version>
      <type>jar</type>
      <scope>compile</scope>
    </dependency>
  </dependencies>
```

8. 部署Twossm_web的项目，只要把Twossm_web项目加入到tomcat服务器中即可

## 1.5 编写实体类，在Twossm_domain项目中编写

在这里，我记录一下IDEA的快捷键，详情见{% post_link IDEA常用快捷键整理 IDEA常用快捷键整理 %}

```java
package com.gx.domain;

import java.io.Serializable;

public class Account implements Serializable {
    private Integer id;
    private String name;
    private Double money;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```

## 1.6 编写Dao接口

在dao包中编写dao接口IAccountdao

```java
package com.gx.dao;

import com.gx.domain.Account;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;

import java.util.List;

public interface IAccountdao {
    public List<Account> findAll();
    public void saveAccount(Account account);
}
```

## 1.7 编写service接口和实现类

service接口：

```java
package com.gx.service;

import com.gx.domain.Account;

import java.util.List;

public interface AccountService {
    // 查询所有账户
    public List<Account> findAll();
    // 保存帐户信息
    public void saveAccount(Account account);
}
```

service接口实现类：

```java
package com.gx.service.Impl;

import com.gx.domain.Account;
import com.gx.service.AccountService;
import org.springframework.stereotype.Service;

import java.util.List;
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Override
    public List<Account> findAll() {
        System.out.println("Service业务层：查询所有账户...");
        return null;
    }

    @Override
    public void saveAccount(Account account) {
        System.out.println("Service业务层：保存帐户...");
    }
}
```

到这里，整合环境就搭建好了效果如下，接下来搭建Spring的配置！

![NSazgx.jpg](https://s1.ax1x.com/2020/06/14/NSazgx.jpg)

# 2. Spring框架代码的编写

搭建和测试Spring的开发环境

## 2.1 创建resources的资源文件目录管理xml配置文件

创建一个叫resources的资源文件目录，用来管理放置XML配置文件

![NSDNY4.png](https://s1.ax1x.com/2020/06/14/NSDNY4.png)

![NSDvXq.png](https://s1.ax1x.com/2020/06/14/NSDvXq.png)

## 2.2 编写applicationContext.xml配置文件

在resources资源文件中创建applicationContext.xml的配置文件，编写具体的配置信息

applicationContext.xml中的配置信息：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/tx
	http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--开启注解的扫描，希望处理service和dao，controller不需要Spring框架去处理-->
    <context:component-scan base-package="com.gx" >
        <!--配置哪些注解不扫描-->
    	<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" />
    </context:component-scan>
    
</beans>
```

## 2.3 在项目中编写测试方法，进行测试

创建Test包后在test包中创建一个叫TestSpring的class类，具体的内容如下：

```java
package com.gx.test;

import com.gx.domain.Account;
import com.gx.service.AccountService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestSpring {
    @Test
    public void run1(){
        ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        AccountService as = (AccountService) ac.getBean("accountService");
        as.findAll();
    }
}
```

运行如下效果，说明搭建Spring的开发环境成功！

![NSrGjI.png](https://s1.ax1x.com/2020/06/14/NSrGjI.png)

到这里，Spring的开发环境成功！接下来搭建SpringMVC框架环境。

# 3. SpringMVC框架代码的编写

## 3.1 web.xml配置

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://java.sun.com/xml/ns/javaee"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
  <display-name>Archetype Created Web Application</display-name>
   
    <!--配置前端控制器-->
  <servlet>
     <servlet-name>dispatcherServlet</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <!--加载springmvc.xml配置文件-->
      <init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>classpath:springmvc.xml</param-value>
      </init-param>
      <!--启动服务器，创建该servlet-->
      <load-on-startup>1</load-on-startup>
  </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--解决中文乱码的过滤器-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

## 3.2 配置SpringMVC.xml文件

同样是在resources资源文件夹中创建springmvc.xml配置文件并写入以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启注解扫描，只扫描Controller注解-->
    <context:component-scan base-package="com.gx">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--配置的视图解析器对象-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!--过滤静态资源-->
    <mvc:resources location="/css" mapping="/css/**"/>
    <mvc:resources location="/images/" mapping="/images/**"/>
    <mvc:resources location="/js/" mapping="/js/**"/>
    <!--开启SpringMVC注解的支持-->
    <mvc:annotation-driven/>
</beans>
```

## 3.3 创建jsp页面，编写Controller代码

编写index.jsp页面

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<body>
<a href="account/findAll">测试SpringMVC查询</a>
</body>
</html>
```

在controller层中的AccountController的class类中编写代码

```java
package com.gx.controller;

import com.gx.domain.Account;
import com.gx.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
public class AccountController {

    @RequestMapping("/account/findAll")
    public String findAll(){
        System.out.println("Controller表现层：查询所有账户...");
        return "list";  //在视图解析器中配置了前缀后缀
    }
}
```

这时候就要创建controller跳转的list.jsp页面了：

![NSgZT0.png](https://s1.ax1x.com/2020/06/14/NSgZT0.png)

list.jsp页面创建好了，编写一下内容，只是看是否跳转成功，输出一句话即可：

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%--
  Created by IntelliJ IDEA.
  User: Bule
  Date: 2019/9/2
  Time: 7:32
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<html>
<head>
    <title>Title</title>
</head>
<body>
            <h2>查询所有的账户</h2>
          
</body>
</html>
```

## 3.4 部署Tomcat进行测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902162117301.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902162457840.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902163745680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NTQzNTA4,size_16,color_FFFFFF,t_70)
		到这里，spring、springmvc的开发环境就都搭建好了。接下来是整合Spring和SpringMVC了！

#  4. Spring整合SpringMVC框架

## 4.1 Spring整合SpringMVC的框架原理分析

​		整合成功的表现：在controller（SpringMVC）中能成功的调用service（Spring）对象中的方法。要想在controller中调用service方法，就要注入service到controller中来，有service对象才可以调用service方法，方法是这样没有错，但是有一个问题，就是启动Tomcat之后试想一下，在web.xml中配置有前端控制器，web容器会帮我们加载springmvc.xml配置文件，在springmvc.xml配置文件中我们配置情况是只扫描controller，别的不扫，而spring.xml文件就从头到尾没有执行过，spring中的配置扫描自然也不会去扫描，就相当于没有将spring交到IOC容器当中去，所以，现在的解决方案就是，在启动服务器时就加载spring配置文件,怎么实现呢？这时候监听器listener就派上用场了，具体实现如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902171005382.png)

## 4.2 在web.xml中配置ContextLoaderListener监听器
在项目启动的时候，就去加载applicationContext.xml的配置文件，在web.xml中配置ContextLoaderListener监听器（该监听器只能加载WEB-INF目录下的applicationContext.xml的配置文件）。要想加载applicationContext.xml的配置文件有两种方法，第一种（不建议）：

第二种（强烈建议）：在web.xml中配置加载路径

```xml
 <!--配置Spring的监听器，默认只加载WEB-INF目录下的applicationContext.xml配置文件-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
 <!--设置配置文件的路径-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
```

因为我们在整合过程中会有许多配置文件，我们自定义一个类似pages资源文件夹专门管理这些配置文件，方便管理，方便维护！！！

## 4.3 controller中注入service对象，调用service对象方法并测试

这时候，启动服务器时也会加载spring配置文件了，那么，我们可以在controller中注入service了，于是开始编写controller代码：

```java
package com.gx.controller;

import com.gx.domain.Account;
import com.gx.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
public class AccountController {

    @Autowired   //按类型注入
    private AccountService accountService;

    @RequestMapping("/account/findAll")
    public String findAll(Model model){
        System.out.println("Controller表现层：查询所有账户...");

        List<Account> list = accountService.findAll();
        return "list";
    }
}
```

编写完成，开始测试，启动Tomcat，效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902174053298.png)

# 5. MyBaits框架代码的编写

MyBatis环境搭建首先是dao，搭建mybatis，之前要编写mapper映射的配置文件，其实挺麻烦的，所以我选择使用注解！

## 5.1 在IAccountdao接口方法上添加注解，编写SQL语句

```java
package com.gx.dao;

import com.gx.domain.Account;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository  //此注解代表这是一个持久层，用法类似@controller、@service
public interface IAccountdao {

    @Select("select * from account")
    public List<Account> findAll();
    @Insert("insert into account (name,money) value(#{name},#{money})")
    public void saveAccount(Account account);
}
```

## 5.2 SqlMapConfig.xml的配置文件

写入如下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///ssm"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments> 
    <!-- 使用的是注解 -->
    <mappers> 
        <!-- <mapper class="com.gx.dao.IAccountdao"/> --> <!-- 该包下所有的dao接口都可以使用 -->
        <package name="com.gx.dao"/>
    </mappers>
</configuration>
```

因为我使用的是注解，我觉得还是有必要提一下以下三种方法：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902181214119.png)

## 5.3 MyBatis测试方法

```java
package com.gx.test;

import com.gx.dao.IAccountdao;
import com.gx.domain.Account;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class TestMyBatis {

    @Test
    public void run1() throws IOException {
        Account account =new Account();
        account.setName("杜永蓝");
        account.setMoney(200d);
        // 加载配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 创建SqlSessionFactory对象
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        // 创建SqlSession对象
        SqlSession session = factory.openSession();
        // 获取到代理对象
        IAccountdao dao = session.getMapper(IAccountdao.class);

        // 保存
        dao.saveAccount(account);

        // 提交事务
        session.commit();

        // 关闭资源
        session.close();
        in.close();
    }
    
    @Test
    public void run2() throws Exception {
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");

        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);

        SqlSession session = factory.openSession();

        IAccountdao dao = session.getMapper(IAccountdao.class);

        List<Account> list = dao.findAll();
        for (Account account: list ) {
            System.out.println(account);
        }
        session.close();
        in.close();
    }
}
```

运行测试：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902182219742.png)

​		运行效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902182609492.png)

# 6. Spring整合MyBaits框架

​		Spring整合MyBatis框架之前，先想一想，怎样才算整合成功呢？其实，这和之前的spring整合springMVC的套路差不多，其实就是，Service能成功调用dao对象，能够做查询操作或者新增数据能存进数据库。现在spring已经是在IOC容器中了，dao是一个接口，可以通过程序帮这个接口生成代理对象，我们要是可以把这个代理对象也放进IOC容器，那么service就可以拿到这个对象，之后在service中做一个注入，service从而调用dao代理对象的方法，那么我们怎么去实现dao接口生成的代理对象放入IOC容器呢？
​		整合目的：把SqlMapConfig.xml配置文件中的内容配置到applicationContext.xml配置文件中

## 6.1 在applicationContext.xml中配置数据库连接池

```xml
<!--Spring整合MyBatis框架-->
    <!--配置连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql:///ssm"/>
        <property name="user" value="root"/>
        <property name="password" value="password"/>
    </bean>
```

## 6.2 在applicationContext.xml中配置SqlSessionFactory工厂
​		没配置工厂之前，我们用Test测试的时候，每次都要先创建工厂，因为工厂能够给我们创建SqlSession,有了SqlSession就可以通过SqlSession拿到代理对象。现在我们直接在applicationContext.xml中配置SqlSessionFactory工厂，这就相当于IOC容器中有了工厂，就可以去创建SqlSession，进而通过SqlSession拿到代理对象，没必要每次测试都去创建工厂。

```xml
 <!--配置SqlSessionFactory工厂-->
<bean id="sqlSessonFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
 </bean>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902205722751.png)

## 6.3 在applicationContext.xml中配置IAccount

​		因为工厂有了，SqlSession也有了，那代理谁呢，所以我们要配置IAccountdao接口所在包，告诉SqlSession去代理接口所在包中的代理，从而存到IOC容器中

```xml
 <!--配置IAccountdao接口所在包-->
<bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.gx.dao"/>
</bean>
```

## 6.4 小结上面的三个配置

​		其实，上面的操作就是**把mybatis中的配置（SqlMapConfig.xml）转移到spring中去，让它产生代理并存到IOC容器中**！

## 6.5 完善Service层代码

​		在AccountServiceImpl实现类中编写代码：

```java
package com.gx.service.Impl;

import com.gx.dao.IAccountdao;
import com.gx.domain.Account;
import com.gx.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private IAccountdao iaccountdao;

    @Override
    public List<Account> findAll() {
        System.out.println("Service业务层：查询所有账户...");
        return iaccountdao.findAll();
    }

    @Override
    public void saveAccount(Account account) {
        System.out.println("Service业务层：保存帐户...");
    }
}
```

## 6.6 完善Controller层代码

```java
package com.gx.controller;

import com.gx.domain.Account;
import com.gx.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
public class AccountController {

    @Autowired
    private AccountService accountService;

    @RequestMapping("/account/findAll")
    public String findAll(Model model){  //存数据， Model对象
        System.out.println("Controller表现层：查询所有账户...");
        // 调用service的方法
        List<Account> list = accountService.findAll();
        model.addAttribute("list",list);
        return "list";
    }
}
```

## 6.7 完善list.jsp页面

```jsp
<%--
  Created by IntelliJ IDEA.
  User: Bule
  Date: 2019/9/2
  Time: 7:32
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<html>
<head>
    <title>Title</title>
</head>
<body>
    <h2>查询所有的账户</h2>
    <c:forEach items="${list}" var="account">
        ${account.name}
    </c:forEach>
</body>
</html>
```

## 6.8 运行测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902211018192.png)
到这里，SSM整合就基本完成！

# 7. Spring整合MyBatis框架配置事务（Spring的声明式事务管理）

​		细心的小伙伴可能发现了，我在整合spring、mybatis测试的时候（TestMybatis中），新增数据保存的时候手动的提交过事务 session.commit()，如果不写这一句，就会出现数据没提交的情况，因此为了完美的整合ssm，我们必须配置Spring的声明式事务管理！

## 7.1 在applicationContext.xml中配置Spring框架声明式事务管理

```xml
 <!--配置Spring框架声明式事务管理-->
    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!--配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" isolation="DEFAULT"/>
        </tx:attributes>
    </tx:advice>

    <!--配置AOP增强-->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.gx.service.Impl.*ServiceImpl.*(..))"/>
    </aop:config>
```

# 7.2 完善index.jsp页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<a href="account/findAll">测试查询</a>

<h3>测试包</h3>

<form action="account/save" method="post">
    姓名：<input type="text" name="name" /><br/>
    金额：<input type="text" name="money" /><br/>
    <input type="submit" value="保存"/><br/>
</form>

</body>
</html>
```

## 7.3 完善Service层、Controller层代码

Service层：在AccountServiceImpl实现类中调用service中的`saveAccount(account)`方法

```java
 @Override
    public void saveAccount(Account account) {
        System.out.println("Service业务层：保存帐户...");
        iaccountdao.saveAccount(account);  //调用service中的saveAccount(account)方法
    }
```

Controller层代码：在AccountController类中添加一个保存save的方法

```java
    @RequestMapping("/account/save")
    public void save(Account account, HttpServletRequest request, HttpServletResponse response) throws IOException {
        accountService.saveAccount(account);
        response.sendRedirect(request.getContextPath()+"/account/findAll");
        return;
    }
```

## 7.4 运行测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902220904301.png)

OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOKKKKKKKKKKKKKKKKKKK！

这么曲折终于整合完毕，但是到底还是SpringBoot香啊，烦神！

 