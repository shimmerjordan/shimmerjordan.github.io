---
title: ApplicationContext之getBean方法详解
date: 2020-06-14 17:12:43
tags: 
	- Spring
	- AOP
	- bean对象方法
categories: Spring
id: ApplicationContextGetBean
---

我们知道可以通过ApplicationContext的getBean方法来获取Spring容器中已初始化的bean。getBean一共有以下四种方法原型：

- getBean(String name)
- getBean(Class<T> type)
- getBean(String name,Class<T> type)
- getBean(String name,Object[] args)

下来我们分别来探讨以上四种方式获取bean的区别。

<!--more-->

# 其中实体类Person定义如下：

```java
public class Person {
    private String name;
    private int age;
    public Person(){}

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	@Override
    public String toString() {
    return "Person [name=" + name + ", age=" + age + "]";
	}
}
```

applicationContext.xml注册有id为p的bean，配置如下：

```jsx
<bean id="p" class="com.bean.Person">
    <property name="name" value="张三"/>
    <property name="age" value="18"/>
</bean>
```

## 1. getBean(String name)

​		参数name表示IOC容器中已经实例化的bean的id或者name,且无论是id还是name都要求在IOC容器中是唯一的不能重名。那么这种方法就是通过id或name去查找获取bean.获取bean的参考代码如下：

```java
@Test
public void testPerson() {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    Person p = (Person) ctx.getBean("p");
    System.out.println(p);
}
```

## 2. getBean(Class<T> type)

​		参数Class<T> type表示要加载的Bean的类型。如果该类型没有继承任何父类(Object类除外)和实现接口的话，那么要求该类型的bean在IOC容器中也必须是唯一的。比如applicationContext.xml配置两个类型完全一致的bean,且都没有配置id和name属性。

```jsx
<bean class="com.bean.Person">
    <property name="name" value="张三"/>
    <property name="age" value="18"/>
</bean>

<bean class="com.bean.Person">
    <property name="name" value="李四"/>
    <property name="age" value="20"/>
</bean>
```

​		那么通过com.bean.Person这种类型来查找bean,参考代码如下：

```java
@Test
public void testPerson() {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    Person p = ctx.getBean(Person.class);
    System.out.println(p);
}
```

​		但是由于属于com.bean.Person的bean在IOC容器中不唯一，所以这里会抛出NoUniqueBeanDefinitionException异常。

​		由此我们可以总结getBean(String name)和getBean(Class<T> type)的异同点。

- ##### 相同点：都要求id或者name或者类型在容器中的唯一性。

- ##### 不同点：getBean(String name)获得的对象需要类型转换而getBean(Class<T> type)获得的对象无需类型转换。

## 3.  getBean(String name,Class<T> type)

​		这种方式比较适合当类型不唯一时，再通过id或者name来获取bean。

​		例如applicationContext.xml配置有如下bean:

```jsx
<bean id="p1" class="com.bean.Person">
    <property name="name" value="张三"/>
    <property name="age" value="18"/>
</bean>
<bean name="p2" class="com.bean.Person">
    <property name="name" value="李四"/>
    <property name="age" value="20"/>
</bean>
```

参考代码如下：

```java
@Test
public void testPerson() {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    Person p = ctx.getBean("p2",Person.class);
    System.out.println(p);
}
```

​		这样可以获取到名字叫”李四”的对象。测试结果如下：

## 4.  getBean(String name,Object[] args)

​		这种方式本质还是通过bean的id或者name来获取bean,通过第二个参数Object[] args可以给bean的属性赋值，赋值的方式有两种：构造方法和工厂方法。但是通过这种方式获取的bean必须把scope属性设置为prototype，也就是非单例模式。

​		先在com.factory包下设计有如下的工厂类：

```java
public class PersonFactory {
	//静态工厂注入
    public static Person getPersonInstance(String name,int age)throws Exception {
        Person p = (Person)Class.forName("com.bean.Person").newInstance();
        Method m = p.getClass().getMethod("setName", java.lang.String.class);
        m.invoke(p, name);
        m = p.getClass().getMethod("setAge", int.class);
        m.invoke(p, age);
        return p;
	}
}
```

在applicationContext.xml中配置有如下bean:

```jsx
<bean name="p3" class="com.bean.Person" scope="prototype"/>
```

获取bean的参考代码如下：

```java
@Test
public void testPerson()
{
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
Person p = (Person) ctx.getBean("p3",new Object[]{"王五",35});
System.out.println(p);
}
```

如果想通过工厂注入属性，在applicationContext.xml配置如下bean:

```jsx
<bean name="p3" class="com.factory.PersonFactory" factory-method="getPersonInstance" scope="prototype">
<constructor-arg name="name">
<null/>
</constructor-arg>
<constructor-arg name="age" value="0"/>
</bean>
```

> 转自：https://www.jianshu.com/p/01bee649a0c9