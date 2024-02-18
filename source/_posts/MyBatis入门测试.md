---
title: MyBatis入门测试
date: 2020-06-12 17:51:09
tags:
	- Web开发
	- MyBaits
	- 框架
categories: Web框架
id: MyBatisBeginning
---
此篇对于Mybatis的入门学习，现进行总结。现提供主要基于xml的Mybatis完整测试代码。主要知识内容包括配置文件、映射文件的编写、日志文件的编写、sqlSession的使用以及封装成工具类。基于注解的Mybatis开发见{% post_link Mybatis注解开发入门 Mybatis注解开发入门 %} 
<!--more-->  

# 1、MyBatis概述

1. 它是基于Java的持久层框架，内部封装了JDBC，简单点就是用来执行sql语句与数据库进行交互的框架。  
2. 与Hibernate的区别是，Mybatis还是要写sql语句的，Hibernate封装得更加全面，不需要写sql语句。

# 2、测试代码

## 2.1、工程路径图

![tXpgTU.jpg](https://s1.ax1x.com/2020/06/12/tXpgTU.jpg)

## 2.2、mapper文件编写
   mapper文件的作用是程序用以查找具体执行的sql语句：

   值得注意的是：

   A、`#{name}`，`#{age}`，`#{sore} `这个写法，后面讲到DAO层的时候会提。

   B、namespace是命名空间，用来限定范围的，它有两个注意点：

    1）后面开发，一般一个Dao对应一个mapper，在这种情况下，我们的insert语句的id可能出现重复，这个时候就可以通过  使用namespace.id来限定是那个mapper文件的操作
    2）通过限定namespace可以设定日志文件显示的内容

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<!--test是命名空间  -->
<mapper namespace="test">
	<insert id="insertStudent" parameterType="Student">
		insert into student(sname,age,score) value(#{name},#{age},#{sore})
	</insert>		
</mapper>
```

# 3、Mybatis配置文件
## 3.1、映射文件的注册
**方法一：**mapper标签class属性，针对于sql注册文件在类中，并该注册文件名必须和接口名称同名
```xml
<mappers>
		<mapper class="com.stone.mybatis.mapper.StudentDao"></mapper>
</mappers>
```
方法一如果是maven工程需要添加以下代码：
```xml
 <!-- 扫描除了resources之外的其他xml包 -->
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```
**方法二：**mapper标签url属性，能读取硬盘上的注册文件或者网络上的注册文件 （前面file不可省略）
```xml
<mappers>
    <mapper url="file:\D:\Testenv\idea\Java_Mybatis\src\main\resources\mapper\StudentMapper.xml"></mapper>	
</mappers>
```
**方法三：**mapper标签resource属性，针对于sql注册文件在类路径下
```xml
<mappers>
        <mapper resource="mapper/StudentMapper.xml"></mapper>
</mappers>
```
**方法四：**package标签name属性，映射该包下所有的sql映射文件 ，这种方式能够批量注册，并且每一对接口和xml名称必须相同
```xml
<!-- 注册mapper -->
<mappers>
    <package name="com.stone.mybatis.mapper"/>
</mappers>
```
方法四如果是maven工程需要在pom中添加以下代码：
```xml
 <!-- 扫描除了resources之外的其他xml包 -->
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```
## 3.2、运行环境的配置
1. environmentals下可以配置多个environment，通过id来进行选择，这里我们使用默认的JDBC的mysql环境
2. datasource用来选择数据连接池。作用是存放连接数据库的连接。因为每一次连接到数据库是件非常耗时的事件，增加一个连接池，用来存放连接对象，它可以接受发送连接请求，也可以回收连接进行保存。数据连接池有很多参数，包括最大，最小缓存等等，我们后台的调优，有一部就是这些参数的选择。另外，这里我们使用Mybatis默认的连接池，后续我们会使用到c3p0/dbcp等数据池。
3. 数据库连接四要素，采用properties文件进行配置，采用${}方式进行获取（SpringBoot也可以通过yml进行配置）

> 正常的情况是先加载yml，接下来加载properties文件。如果相同的配置存在于两个文件中。最后会使用properties中的配置。最后读取的优先集最高。
> 两个配置文件中的端口号不一样会读取properties中的端口号

## Mybaits.xml文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="mysql.properties"></properties>
	<!--给dao取类名  -->
	<typeAliases>
		<package name="beans"/>
	</typeAliases>
	<!-- 配置运行环境  -->
	<environments default="mysqlEM">
		<environment id="mysqlEM">
			<!--JDBC表示使用JDBC的事务管理器  -->
			<transactionManager type="JDBC"></transactionManager>			
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
	</environments>
	
	<!--映射文件的注册  -->
	<mappers>
		<mapper resource="dao/mapper.xml"/>
	</mappers>
	
</configuration>
```
## mysql.properties文件代码
```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost/bjpowernode?serverTimezone=GMT%2B8
jdbc.username=root
jdbc.password=123
```

# 4、Dao层测试代码

1. 这里我们把获取`sqlsession`方法提取出来，封装成一个类在`utils.myUtils`文件中。

2. `sqlsession.insert("insertStudent", student);`这里的第二个参数,是一个Student对象，因此我们在Mapper文件中的sql语句采用获取的方式,进行编写

```java
package dao;
 
import org.apache.ibatis.session.SqlSession;
 
import beans.Student;
import utils.myUtils;
 
public class IStudentDaoImpl implements IStudentDao {
 
	private SqlSession sqlsession;
 
	@Override
	public void insertStu(Student student) {
		// TODO Auto-generated method stub
		try {
			sqlsession = myUtils.getsqlsession();
            //具体操作代码
			sqlsession.insert("insertStudent", student);
            //提交数据代码
			sqlsession.commit();
		} 
		finally {
			if(sqlsession!=null) {
			sqlsession.close();
			}
		}	
	}
}
```

## myUtils.java文件代码

这里我们使用了单例模式，因为考虑到创建factory耗时很长，因此就让他在初始化的时候，就创建好一个对象，并且每次使用的是同一个对象。当然这样是线程不安全的，因为当我进行数据操作的时候，其他线程用的是同一个factory对象，他会读取到我数据，类似mysql的dirtyread。具体情况具体使用。

```java
package utils;
 
import java.io.IOException;
import java.io.InputStream;
 
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
 
public class myUtils {
	private static SqlSessionFactory sqlsessionfactory;
 
	private myUtils() {};
 
	public static SqlSession getsqlsession() {
		InputStream inputstream;
		try {
            //获取Mybatis容器内容
			inputstream = Resources.getResourceAsStream("myBatis.xml");
			if (sqlsessionfactory == null) {
             	//新建一个factory，他的作用用来生产sqlSeesion
				sqlsessionfactory = new SqlSessionFactoryBuilder().build(inputstream);
			}
            //使用factory的open方法创建一个sqlSeesion
			SqlSession sqlsession = sqlsessionfactory.openSession();
			return sqlsession;
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
	}
}
```

# 5、日志文件

添加一个日志配置log4j.properties文件：

注意到最后一句：`log4j.logger.test`这个test就是我们的命名空间，最后他之显现和test有关的日志信息到控制台console

```properties
### \u8F93\u51FA\u5230\u63A7\u5236\u53F0 ###
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.consoleTarget = System.out
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern =[%5-5p][%d{yyyy-MM-dd HH:mm:ss}] %c %L %m%n   //格式信息
 
### set log levels ###
//debug输出到控制台
//test是我们mapper的命名空间，一下设置可以使得只输出有关test的信息
log4j.logger.test = debug,console 
```

# 6、其他代码

## bean对象Student

```java
package beans;
 
public class Student {
	private Integer id;
	private String name;
	private int age;
	private double sore;
	public Student(String name, int age, double sore) {
		super();
		this.name = name;
		this.age = age;
		this.sore = sore;
	}
	public Student() {
		super();
	}
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
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public double getSore() {
		return sore;
	}
	public void setSore(double sore) {
		this.sore = sore;
	}
	@Override
	public String toString() {
		return "Student [id=" + id + ", name=" + name + ", age=" + age + ", sore=" + sore + "]";
	}
 
}
```

## 测试类mytest

```java
package test;
 
import org.junit.Before;
import org.junit.Test;
 
import beans.Student;
import dao.IStudentDao;
import dao.IStudentDaoImpl;
 
public class myTest {
	
	private IStudentDao sdao;
 
	@Before
	public void before() {
		sdao = new IStudentDaoImpl();
	}
	
	@Test
	public void test1() {
		Student student = new Student("张si",23,93.2);
		sdao.insertStu(student);
	}
}
```

## Mysql数据库建表

```sql
use bjpowernode;
drop table if EXISTS student;
CREATE table student(
    id INT(12) PRIMARY KEY auto_increment,
		sname VARCHAR(32),
		age INT(12),
		score INT(12)
)
```

