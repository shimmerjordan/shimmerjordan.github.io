---
title: IOC介绍及其简单实现
date: 2020-06-14 00:07:38
tags:
	- Web开发
	- Spring
	- IOC
categories: Web框架
id: IOCIntroAndCompl
---

​		控制反转（Inversion of Control，英文缩写为IoC）是一个重要的面向对象编程的法则来削减计算机程序的耦合问题，也是轻量级的Spring框架的核心。 控制反转一般分为两种类型，依赖注入（Dependency Injection，简称DI）和依赖查找。依赖注入应用比较广泛，这里只介绍依赖注入。

<!--more-->

# 一、IOC简介

​		控制反转IOC，它最主要反映的是与传统面向对象（OO）编程的不同。通常我们编程实现某种功能都需要几个对象相互作用，从编程的角度出发，也就是一个主对象要保存其他类型对象的引用，通过调用这些引用的方法来完成任务。如何获得其他类型的对象引用呢？一种方式是主对象内部主动获得所需引用（也就是通常我们使用的new一个对象）；另一种方式是在主对象中设置setter 方法，通过调用setter方法或构造方法传入所需引用。后一种方式就叫IOC，也是我们常常所说的依赖注入DI。以下我们用一个简单的例子来说明传统OO编程与IOC编程的差别。

​		这个例子的目的是根据时间不同返回不同的问候字符串， 比如Good Morning，world或Good afternoon，World。

**服务端口**

```java
package cn.test.ioc;
public interface HelloIF {
    String sayHello();
}
```

​		**传统实现**

```java
package cn.test.ioc;

import java.util.Calendar;

//传统实现(非IOC方式)
public class HelloIFImpl implements HelloIF {
    private Calendar cal; // 我们需要的引用

    public HelloIFImpl() {
        cal = Calendar.getInstance(); // 主动获取
    }

    public String sayHello(){
        if(cal.get(Calendar.AM_PM) == Calendar.AM){
            return "Good morning, World";
        }else{
            return "Good afternoon, World";
        } 
    }
    
    public static void main(String args[]){
        HelloIFImpl hf = new HelloIFImpl();
        System.out.println(hf.sayHello());
    }
}
```

​		**采用IOC方式：**

```java
package cn.test.ioc;

import java.util.Calendar;

//IOC方式实现
public class HelloIFImpl2 implements HelloIF {
    private Calendar cal; // 我们需要的引用

    public void setCal(Calendar cal) {
        this.cal = cal;
    } // 依赖注入

    public String sayHello(){
        if(cal.get(Calendar.AM_PM) == Calendar.AM){
            return "Good morning, World";
        }else{
            return "Good afternoon, World";
        } 
    }
    
    public static void main(String args[]){
        HelloIFImpl2 hf = new HelloIFImpl2();
        hf.setCal(Calendar.getInstance());
        System.out.println(hf.sayHello());
    }
}
```

​		在这里你或许会有疑问，这页看不出有太大的差别，并且依赖注入还需要我先创建外部的`Calendar`对象，然后再传到`HelloIFImpl`对象中。但是，假如我们事先已经在类用`new orderOracle()`，但是后来由于需求变更，我们需要使用`new orderSqlServer()`，这样我们还需要修改所有使用`orderOracle()`的类，这样好麻烦。如果我们使用IOC方法编程并且使用了spring框架，这样我们只需要修改xml配置文件即可。

​		IoC则是一种 软件设计模式，它告诉你应该如何做，来解除相互依赖模块的耦合。控制反转（IoC），它为相互依赖的组件提供抽象，将依赖（低层模块）对象的获得交给第三方（系统）来控制，即依赖对象不在被依赖模块的类中直接通过new来获取。

# 二、IOC的一个应用举例

​		我们以一个struts2和Spring整合的例子来说明。

##		1）整合struts2和Spring

　　首先要整合Spring和Struts2，需要先要拷入Spring需要的jar包，既包括Spring本身的jar包，也包括Struts2的Spring插件。将以下几个jar包拷入到我们的web工程的WEB-INF\lib中：`org.springframework.asm-3.0.5.RELEASE.jar`；`spring-*.jar`(struts2中lib包里所有的jar，共6个); `struts2-spring-plugin-*.jar `。

## 2）编写逻辑层接口

```java
package cn.test.springDemo;

public interface SampleService {
    public String getNameById(String userId);  
}
```

## 3）编写逻辑层实现类

```java
package cn.test.springDemo;

public class SampleServiceImpl implements SampleService{  
    public String getNameById (String userId) {  
        //根据userId到数据层进行查询，获取相应的name  
        String name = "hello,"+ userId;  
        return name;  
    }  
}  
```

## 4）编写ACTION

```java
package cn.test.springDemo;

import com.opensymphony.xwork2.ActionSupport;

public class SampleAction extends ActionSupport {
    // 通过setter方式,由Spring来注入SampleService实例
    private SampleService service;

    public void setService(SampleService service) {
        this.service = service;
    }

    private String name;
    private String userId;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String execute() throws Exception {
        name = this.service.getNameById(userId);
        return SUCCESS;
    }
}
```

　　在execute方法中不再直接new一个`SampleServiceImpl`的实例了，而是声明了一个`SampleSerivce`类型的属性，并提供对应的setter方法，这个setter方法是留给Spring注入对象实例的时候调用的，可以不用提供getter方法。也就是说，现在的`SampleAction`已经不用知道逻辑层的具体实现了。

## 5）编写Spring的配置文件applicationContext.xml

　　要让Spring来管理`SampleAction`和`SampleServiceImpl`的实例，还需要新建一个Spring的配置文件。在src下新建一个`applicationContext.xml`文件，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
        http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

    <bean name="service" class="cn.test.springDemo.SampleServiceImpl" />
    <bean name="sampleAction" class="cn.test.springDemo.SampleAction" scope="prototype" > 
        <property name="service" ref="sampleService"/>  
    </bean>  
</beans>  
```

　　这个xml的根元素是<beans>，在<beans>中声明了它的schema引用，除此之外，还有两个<bean>元素，定义了由Spring管理的SampleServiceImpl和SampleAction。

- 对于第一个<bean>元素来说
  - name属性为它设置了一个名字
  - class元素指定了它的实现类的全类名

- 对于第二个<bean>元素来说
  -  name属性和class属性的含义与第一个<bean>元素完全一样。
  - scope属性，赋值为prototype（原型）。scope属性非常重要，它管理了注册在它里面的Bean的作用域。Spring容器默认的作用域是单例，即每次外界向Spring容器请求这个Bean，都是返回同一个实例；但是，Struts2的Action是需要在每次请求的时候，都要新建一个Action实例，所以，在配置对应Action的<bean>元素时，必须把它的scope属性赋值为prototype，以保证每次请求都会新建一个Action实例。
  -  <property>子元素。<property>元素的name属性为service，代表`SampleAction`这个类有一个setter方法叫setSampleService；<property>元素的ref属性为`sampleService`，代表Spring容器会将一个名为`sampleService`的已经存在的Bean，注入给`sampleAction`的service属性。

## 6）在web.xml中引用Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

    <filter>
        <filter-name>Struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>Struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

　　listener实现了当这个Web工程启动的时候，就去读取Spring的配置文件，这个类是由Spring提供的，这里只需要配置上去就可以了。上下文参数的配置里面，`contextConfigLocation`的值`classpath*:applicationContext.xml`，表明了所有出现在classpath路径下的`applicationContext.xml`文件，都是上面的这个Listener要读取的Spring配置文件。

##　7）编写struts.xml　

　　就快要大功告成了，最后一步，来修改`struts.xml`，需要做两件事：

　　首先，添加常量`struts.objectFactory`，其值为`spring`，这就指定了Struts2使用Action的时候并不是自己去新建，而是去向Spring请求获取Action的实例。示例如下：

```xml
<constant name="struts.objectFactory" value="spring"/>  
```

　　然后，<action>元素的class属性，现在并不是要填Action类的全类名了，而是要填一个在Spring配置文件中配置的Action的Bean的名字，也就是<bean>元素的name属性，很显然，需要的是sampleAction这个Bean。示例如下：

```xml
<package name="springPackage" extends="struts-default">
        <action name="sampleActionName" class="sampleAction">
            <result>/spring/success.jsp</result>
        </action></package>
```

　　只有<action>元素的class属性变了，其他部分不变。来测试一下，运行：http://localhost:8080/struts2Deepen2/sampleActionName.action?userId=test。【其中，struts2Deepen2为web工程的名称】。运行一切正常，说明Struts2与Spring整合并不是为了实现新功能，而是为了让表现层组件和逻辑层组件解耦，`SampleAction`类不用再知道`SampleServiceImpl`这个具体实现了，只需要知道`SampleService`这个接口就可以了。

**参考资料：**

　　http://blog.csdn.net/Kettas2008/article/details/2447809

　　[http://www.iteye.com/topic/1124526](http://blog.csdn.net/Kettas2008/article/details/2447809)

　　[http://www.cnblogs.com/liuhaorain/p/3747470.htm#title_4](http://blog.csdn.net/Kettas2008/article/details/2447809) 【这篇文章详细介绍了DIP、IoC、DI以及IoC容器，推荐看看】

​		https://www.cnblogs.com/ningvsban/p/3757890.html