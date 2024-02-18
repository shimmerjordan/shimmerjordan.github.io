---
title: Spring工厂模式
date: 2020-6-13 21:36:44
tags: 
	- Spring
	- 设计模式
categories: Spring
id: SpringFactoryModes
---

# 前言

工厂模式是一种在工程中广泛应用的设计模式，对代码的解耦合起到了很大的作用。实际上，我们可以将Spring理解成封装了我们工程中大量重复代码的一种工具。众所周知，Spring中最重要的组件就是IOC，而IOC中非常重要的部分就是应用了工厂模式的代码。而工厂模式依赖于Java的反射机制，所以，我们从反射机制讲起，一步步了解Spring的Bean工厂。
<!--more-->

# 一、Java中的反射机制

## 1、简介

简单的说，反射机制就是在程序的运行过程中被允许对程序本身进行操作，比如自我检查，进行装载，还可以获取类本身，类的所有成员变量和方法，类的对象，还可以在运行过程中动态的创建类的实例，通过实例来调用类的方法，这就是反射机制一个比较重要的功能了。那么要通过程序来理解反射机制，首先要理解类的加载过程。

![txArSf.jpg](https://s1.ax1x.com/2020/06/13/txArSf.jpg)

在Java程序执行的时候，要经历三个步骤：加载、连接和初始化。首先程序要加载到JVM的方法区中，然后进行连接，最后初始化。这里就主要介绍一下类的加载。如上图，首先，JVM会从硬盘中读取Java源文件并将其加载到方法区中同时生成类名.class文件，也就是类对象，这个类对象中包含了我们创建类的实例时所需要的模板信息，也就是源代码中的成员变量和方法等。Class本身也是一个类，它的主要功能之一就是生成类加载时的class文件，为类的初始化及实例化做准备。而我们在程序中通过关键字new创建的对象创建的是类的对象，而不是类对象，二者的区别如图中所示。

对类的加载有了一个大致的理解之后，我们来看一下实现反射机制的具体操作：

反射机制在我们所学习的框架中有很大的应用，而在我们实际开发中用的并不多，所以理解反射机制对我们学习框架来说很有帮助。

首先我们创建一个实体类User.java

```java
package cn.itcast_01;
 
public class User extends Person{
	
	public String name;
	private int age;
	
	public User(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}
	
	private User(int age) {
		super();
		this.age = age;
		
	}
	
	public User(String name) {
		super();
		this.name = name;
	}
	
	public User() {
		super();
		// TODO Auto-generated constructor stub
	}
	@Override
	public String toString() {
		return "User [name=" + name + ", age=" + age + "]";
	}
	
	public void exit() {
		System.out.println(name+"退出系统");
	}
	
	public void login(String username,String password) {
		System.out.println("用户名:"+username);
		System.out.println("密码:"+password);
	}
	
	private String CheckInfo() {
		return "年龄:"+age;
	}
}
```

其中包括成员变量，构造方法和一些成员方法。

利用反射机制可以获取类对象（也就是我们前面介绍的类对象，获取类对象之后我们便获取了类的模板，可以对类进行一些操作），有以下三种方法：

    1.类名.class()
    2.对象名.getClass()
    3.Class.forName(具体的类名)

我们通过代码来看一下具体的操作：

```java
package cn.itcast_01;
 
public class Demo {
	
	public static void main(String[] args) throws Exception {
		
		//1.类名.class
		Class clz = User.class;
		System.out.println(clz);
		
		//2.对象名.getClass()
		Class clz1 = new User().getClass();	
		System.out.println(clz==clz1);
		
		//3.Class.forName()
		//Class clz2 = Class.forName("User");
		Class clz2 = Class.forName("cn.itcast_01.User");
		System.out.println(clz==clz2);
	}
}
```

在类加载的三个阶段里都可以获取类对象，其中第三种方法，在源码中获取类对象是最常用的，也是反射机制在框架中的应用，在框架中的应用可以是通过配置文件写入所创建的类名，再利用第三种方法获取类对象。

获取类对象之后就可以对类进行一些创建对象、调用方法、访问成员变量的操作了：

## 2、创建对象

    Object obj = 类对象.newInstance();

实例：

```java
package cn.itcast_01;
 
public class Demo2 {
	public static void main(String[] args) throws Exception {	
		Class clz = Class.forName("cn.itcast_01.User");
		System.out.println(clz);
		
		Object obj = clz.newInstance();
		System.out.println(obj);
	}
}
```

## 3、调用方法

```java
package cn.itcast_05_Method;
 
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import cn.itcast_01.User;
 
public class Demo {
	public static void main(String[] args) throws Exception {
		User u = new User("小巴",18);
		User u2 = new User("老赵",20);
		
		Class clz = Class.forName("cn.itcast_01.User");
		//Method md = 类对象.getMethod("类中的公有方法名");
		//获取公有方法，其中md是Method类型的方法名，
		Method em = clz.getMethod("exit");
		//Object obj1 = lm.invoke(u2,"老赵","aixiaoba");
		//为获取到的方法命名，方便调用。
		Object obj = em.invoke(u2);
		
		System.out.println(obj);
		//Method dm = 类对象.getDeclaredMethod("类中的私有方法名");
		//获取私有方法名，又叫暴力获取，此方法无视方法的访问权限，即使是被private修饰的方法也会被获取到。
        //Method lm = 类对象.getMethod("有参方法方法名",参数的类对象...);
		//获取有参方法，同时要获取参数的类对象，格式为：参数类型.class
		Method lm = clz.getMethod("login",String.class,String.class);
		//Object obj1 = lm.invoke(类的对象,"参数1","参数2");
		//调用获取到的方法，使用invoke关键字，此处表示调用有参方法。
		Object obj1 = lm.invoke(u2,"老赵","aixiaoba");
		
        //以上方法中由于不确定获取到的对象类型，所以用Object接收。
		System.out.println(obj1);
	}
}
```

## 4、访问成员变量

```java
package cn.itcast_02Field;
 
import java.lang.reflect.Field;
 
public class Demo {
	
	public static void main(String[] args) throws Exception {
		
		Class clz = Class.forName("cn.itcast_01.User");
		Object obj = clz.newInstance();
		//最重要的一个关键字：Field
		//Field nf = 类对象.getField("成员变量名");
        //和调用私有方法一样，要访问私有成员变量也要通过暴力获取的的方式，同时也要获取访问私有成员变量的权限。
		//Field af = 类对象.getDeclaredField("私有成员变量名");
		//af.setAccessible(true);
		Field nf = clz.getField("name");
		nf.set(obj, "小巴");
		
		Object object = nf.get(obj);
		System.out.println(object);
	}
}
```

# 二、工厂模式

## 1、简介

工厂模式提供了一种绝佳的创建对象的方法。在工厂模式中，我们并不会直接使用new来创建一个对象，而是使用一个共同的接口类来指定其实现类，这就大大降低了系统的耦合性——**我们无需改变每个调用此接口的类，而直接改变实现此接口的类即可完成软件的更新迭代**。

## 2、分类

- 简单工厂模式（ simple Factory）
 又叫做静态工厂方法（StaticFactory Method） 模式， 但不属于 23 种设计模式之一。
 简单工厂模式的实质是由一个工厂类根据传入的参数， 动态决定应该创建哪一个产品类。

- 工厂方法模式（Factory Method）
 通常由应用程序直接使用 new 创建新的对象， 为了将对象的创建和使用相分离， 采用工厂方法模式方法,即应用程序将对象的创建及初始化职责交给工厂对象

- 抽象工厂模式（abstract Factory）
 主要创建一个产品族，不同工厂继承父类的抽象工厂创建不同的产品族

## 3、简单工厂模式实例

有一个为car的接口，有三种不同类型的车分别为奥迪车、奔驰车、宝马车，为了根据传入不同的参数，就可以得到不同car对象

**接口类car**

```java
//实现车子的接口
public interface Car {
    String getName();
}
```

**三个具体的实现类如下：**

```java
//奥迪车
public class Audi implements Car {
    @Override
    public String getName() {
        return "audi";
    }
}
```

```java
//奔驰车
public class Benz implements Car {
    @Override
    public String getName() {
        return "benz";
    }
}
```

```java
//宝马车
public class Bmw implements Car {
    @Override
    public String getName() {
        return "bmw";
    }
}
```

简单工厂的方法如下：

```java
//简单工厂 根据传入不同的参数得到不同对象
public class SimpleFactory  {
    public Car getCar(String name){
        if("BMW".equalsIgnoreCase(name)){
            return new Bmw();
        }else if("Benz".equalsIgnoreCase(name)){
            return new Benz();
        }else if("Audi".equalsIgnoreCase(name)){
            return new Audi();
        }else{
            System.out.println("这个产品产不出来");
            return null;
        }
    }
}
```

## 4、工厂方法模式实例

为了创建不同的类型的车，需要三个不同的工厂去创建，把创建的任务交给工厂去完成，于是代码就进行下面的演变

- 先引入一张简单工厂的uml图帮助大家理解：

![tx3IaV.png](https://s1.ax1x.com/2020/06/13/tx3IaV.png)

具体代码

```java
//造车工厂的接口
public interface Factory {
     Car getCar();
}
```

```java
//生产奥迪的工厂
public class AudiFactory implements Factory {
    @Override
    public Car getCar() {
        return new Audi();
    }
}
```

```java
//生产奔驰的工厂
public class BenzFactory implements Factory {
    @Override
    public Car getCar() {
        return new Benz();
    }
}
```

```java
//生产宝马的工厂
public class BmwFactory implements Factory {
    @Override
    public Car getCar() {
        return new Bmw();
    }
}
```

工厂方法的测试类

```java
public class FactoryTest {
    public static void main(String[] args) {
        //1.首先先创建一个奥迪工厂出来
        Factory factory = new AudiFactory();
        //2.然后根据工厂得到奥迪车，具体的造车工厂交给工厂来完成
        System.out.println(factory.getCar());
        factory = new BmwFactory();
        System.out.println(factory.getCar());
    }
}
```

## 5、抽象工厂模式实例

假设奥迪和奔驰和宝马属于一个产品族的，那么可以根据抽象工厂即可创建一些的不同的车，此时的造车的接口工厂需要发生改变，需要支持多个产品族的创建

- 先引入一张抽象工厂的uml图帮助理解：

![tx8edP.jpg](https://s1.ax1x.com/2020/06/13/tx8edP.jpg)

具体代码：

```java
public abstract class AbstractFactory {
	//分别得到奥迪、奔驰、宝马车
  	public abstract Car getAudiCar();
  	public abstract Car getBenzCar();
  	public abstract Car getBmwCar();
}
```

测试类

```java
public class AbstractFactoryTest {
    public static void main(String[] args) {
        //1.先创建具体抽象工厂
        AbstractFactory abstractFactory = new CarFactory();
        //2.根据具体的抽象工厂得到车
        Car audi  = abstractFactory.getAudiCar();
        System.out.println(audi.getName());
    }
}
```

# 6、工厂模式在Spring中的体现

`Spring Bean 的创建是典型的工厂模式`， 这一系列的 Bean 工厂， 也即 `IOC 容器`为开发者管理对象间的依赖关系提供了很多便利和基础服务， 在 Spring 中有许多的 IOC 容器的实现供用户选择和使用，其相互关系如下：

![txGG1e.jpg](https://s1.ax1x.com/2020/06/13/txGG1e.jpg)

关于Spring IOC的相关笔记可见{% post_link IOC介绍及其简单实现 IOC介绍及其简单实现 %} 