---
title: Mybatis注解开发入门
date: 2020-06-12 22:18:43
tags:
	- Web开发
	- MyBaits
	- 框架
categories: Web框架
mathjax: true
id: MybatisAnnotationDevelopment
---

前面介绍了Mybatis基于xml文件的入门，这里以maven环境为例，对Mybatis注解开发做简单札记。

<!--more-->

# 一、环境搭建

## 1、pom.xml配置

新建一个maven项目（不使用脚手架），命名为`mybatis_annotationTest`，pom.xml文件配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>

   <groupId>com.stevensam</groupId>
   <artifactId>mybatis_annotationTest</artifactId>
   <version>1.0-SNAPSHOT</version>
   <packaging>jar</packaging>
   <dependencies>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.12</version>
           <scope>test</scope>
       </dependency>
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>5.1.6</version>
           <scope>runtime</scope>
       </dependency>
       <dependency>
           <groupId>log4j</groupId>
           <artifactId>log4j</artifactId>
           <version>1.2.17</version>
       </dependency>
       <dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis</artifactId>
           <version>3.4.5</version>
       </dependency>
   </dependencies>
   <build>
       <plugins>
           <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-compiler-plugin</artifactId>
             <configuration>
               <source>1.8</source>
               <target>1.8</target>
               <encoding>utf-8</encoding>
             </configuration>
           </plugin>
       </plugins>
   </build>
</project>
```

# 2、dao以及domain建立

在main-java目录中建立com.stevensam.dao以及com.stevensam.domain两个包，在dao包中新建一个IStudentDao接口，可以先不写代码。

# 3、实体类创建

domain包中建立实体类，可以将之前的项目的实体类复制粘贴过来，各个类的内容如下：

```java
package com.stevensam.domain;
import java.io.Serializable;
import java.util.Date;
import java.util.List;
/**
* description:学生类
**/
public class Student implements Serializable {
   private int sid;
   private String sname;
   private String sex;
   private Date birthday;
   private int cno;
   //学生所在班级
   private Classes cla;
   //学生所选课程
   private List<Course> courseList;
   //学生所选的课程分数类
   private List<StuCourse> stuCourseList;
   @Override
   public String toString() {
       return "Student{" +
               "sid=" + sid +
               ", sname='" + sname + '\'' +
               ", sex='" + sex + '\'' +
               ", birthday=" + birthday +
               ", cno=" + cno +
               '}';
   }
/***************get和set方法此处省略，请自行加载*********************/
}

```

```java
package com.stevensam.domain;
import java.util.List;
/**
* description:班级实体
**/
public class Classes {
   private int cid;
   private String cname;
   private int cnum;
   private List<Student> students;//班级的学生
   @Override
   public String toString() {
       return "Classes{" +
               "cid=" + cid +
               ", cname='" + cname + '\'' +
               ", cnum=" + cnum +
               '}';
   }
/***************get和set方法此处省略，请自行加载********************/
}

```

# 4、resources相关配置

这里的配置包括jdbcConfig.properties、log4j.properties、SqlMapConfig.xml等，前两者这里不再赘述，此处的SqlMapConfig.xml配置内容如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
   <!--如果用resource属性则可以直接写文件名jdbcConfig.properties-->
   <properties resource="jdbcConfig.properties"></properties>
   <!--只能给实体类区别名-->
   <typeAliases>
       <package name="com.stevensam.domain"></package>
   </typeAliases>
   <!--配置环境-->
   <environments default="mysql">
       <!--配置mysql的环境-->
       <environment id="mysql">
           <!--配置事务的类型-->
           <transactionManager type="JDBC"></transactionManager>
           <!--配置数据源（连接池）-->
           <dataSource type="POOLED">
               <!--配置连接数据库的4个基本信息-->
               <property name="driver" value="${jdbc.driver}"></property>
               <property name="url" value="${jdbc.url}"></property>
               <property name="username" value="${jdbc.username}"></property>
               <property name="password" value="${jdbc.password}"></property>
           </dataSource>
       </environment>
   </environments>
   <mappers>
       <!--用别名更加简洁方便-->
       <package name="com.stevensam.dao"></package>
   </mappers>
</configuration>
```

# 5、测试类编写

test目录下，在java文件夹中添加com.stevensam.test文件夹，再建立AnnotationTest测试类进行测试，代码如下：

```java
package com.stevensam.test;

import com.stevensam.dao.IStudentDao;
import com.stevensam.domain.Student;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

/**
* author:seven lin
* date:2018/8/2414:33
* description:
**/
public class AnnotationTest {

   private InputStream in;
   private SqlSession session;
   private IStudentDao iStudentDao;

   @Before//在测试方法之前执行
   public void init() throws Exception {
       //1.读取配置文件
       in = Resources.getResourceAsStream("SqlMapConfig.xml");
       //2.创建SqlSessionFactory工厂
       SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
       SqlSessionFactory factory = builder.build(in);
       //3.工厂生产SqlSession对象
       session = factory.openSession();
       //4.使用SqlSession创建Dao接口的代理对象
       iStudentDao = session.getMapper(IStudentDao.class);
   }
   @After//在测试方法之后执行
   public void destroy() throws Exception {
       //6.释放资源
       in.close();
       session.close();
   }
   /**后续增加测试方法*/
}
```

# 二、注解开发——CURD操作

## 1、查询所有学生

在IStudentDao接口中增加一个查询所有学生的方法

```java
/**
* 查询所有操作
* 注解开发和xml文件开发最大的区别就是把查询语句用注解写在方法的上面
* @return
*/
@Select("select * from student")
List<Student> findAll();
```

在测试类AnnotationTest中增加测试查询所有学生的方法

```java
/**
* 测试添加学生
*/
@Test
public void testFindAll(){
   List<Student> studentList = iStudentDao.findAll();
   for(Student student:studentList){
       System.out.println(student);
   }
}
```

## 2、增加一个学生

在IStudentDao接口中增加一个增加学生的方法

```java
/**
* 添加一个学生
* @param student
*/
@Insert("insert into student(sname,sex,birthday,cno) " +
       "values(#{sname},#{sex},#{birthday},#{cno})")
void addStudent(Student student);
```

在测试类AnnotationTest中增加测试查询所有学生的方法

```java
/**
* 测试查询所有学生
*/
@Test
public void testAddStudent() throws Exception {
   Student student = new Student();
   student.setSname("徐盛");
   student.setSex("男");
   student.setBirthday(new SimpleDateFormat("yyyy-MM-dd").parse("2000-02-03"));
   student.setCno(3);
   iStudentDao.addStudent(student);
}
```

## 3、更改学生信息

在IStudentDao中添加一个修改学生信息的方法

```java
/**
* 修改学生信息
* @param student
*/
@Update("update student set sname=#{sname},sex=#{sex},birthday=#{birthday},cno=#{cno} " +
       "where sid=#{sid}")
void updateStudent(Student student);
```

在测试类AnnotationTest中添加测试修改学生信息的方法

```java
/**
* 测试修改学生
*/
@Test
public void testUpdate() throws Exception {
   Student student = new Student();
   student.setSid(18);
   student.setSname("徐盛");
   student.setSex("女");
   student.setBirthday(new SimpleDateFormat("yyyy-MM-dd").parse("2001-12-03"));
   student.setCno(1);
   iStudentDao.updateStudent(student);
   session.commit();
}
```

## 4、删除学生信息

在IStudentDao中添加一个删除学生信息的方法

```java
/**
    * 根据id删除学生
    * @param student
    */
@Delete("delete from student where sid=#{sid}")
void deleteStudent(Student student);
```

在测试类AnnotationTest中添加测试删除学生的方法

```java
/**
    * 测试删除学生
    */
@Test
public void testDelete(){
   Student student = new Student();
   student.setSid(18);
   iStudentDao.deleteStudent(student);
   session.commit();
}
```

## 5、模糊查询学生信息

在IStudentDao中添加一个根据姓名模糊查询学生信息的方法

```java
/**
* 根据姓名模糊查询学生
* @param name
* @return
*/
@Select("select * from student where sname like #{sname}")
List<Student> findByName(String name);
```

在测试类AnnotationTest中添加测试根据姓名模糊查询学生信息的方法

```java
/**
* 测试根据姓名模糊查询学生
*/
@Test
public void testFindByName(){
   List<Student> studentList = iStudentDao.findByName("%张%");
   for(Student student:studentList){
       System.out.println(student);
   }
}
```

## 6、注意事项

由于设置了事务管理，所以在操作的时候除了查询之外的语句，我们都需要添加提交的代码`session.commit();`

# 三、如何设置返回值

在实体类中的成员变量名和数据库中的列名不一致的时候，我们需要一一对应列名，在之前的文章中已经用了XML配置文件设置过，这次用注解开发又如何编写呢？请看下面的例子：

## 1、查询所有学生的时候设置返回值对应列表

假设学生类中的成员变量名不同，这里将变量名改一下，并重新生成get和set方法，还有`toString`方法，如下代码：

```java
private int id;//之前为sid
private String name;//之前为sname
/*******************省略其他成员变量********************/
@Override
public String toString() {
   return "Student{" +
       "id=" + id +
       ", name='" + name + '\'' +
       ", sex='" + sex + '\'' +
       ", birthday=" + birthday +
       ", cno=" + cno +
       '}';
}
```

接下来，在IStudentDao接口中修改`findAll()`方法，增加结果返回集注解，如下：

```java
/**
    * 查询所有操作.
    * Results可以用id命名，这里命名为studentMap，在其他方法中可以直接调用
    * @return
    */
   @Select("select * from student")
   @Results(id = "studentMap",value = {
           @Result(id=true,column = "sid",property = "sid"),
           @Result(column = "sname",property = "sname"),
           @Result(column = "sex",property = "sex"),
           @Result(column = "birthday",property = "birthday"),
           @Result(column = "cno",property = "cno")
   })
   List<Student> findAll();
```

## 2、查询一个学生

第一例子中的结果集可以在其他方法使用，比如查询一个学生的方法

在IStudentDao中添加方法

```java
/**
* 根据id查询学生
* @param integer
* @return
*/
@Select("select * from student where sid=#{id}")
@ResultMap("studentMap")
Student findById(Integer integer);
```

AnnotationTest添加测试方法

```java
/**
* 测试根据id查询学生
*/
@Test
public void testFindById(){
   System.out.println("学生："+iStudentDao.findById(16));
}
```

## 3、插入学生信息后返回该学生id

这里的返回和上面的有点不同的是，方法是没有没有返回值的，依靠的是注解将值返回。而且有两种方式实现。

第一种：options，添加表的列名`sid`，指定成员变量的名`id`，关键的一步是设置`useGeneratedKeys = true`，直译过来就是要接代传递变量。

第二种：selectkey，基本的思路和xml配置一样，代码编写如下：

```java
/**
     * 添加一个学生
     * @param student
     */
@Insert("insert into student(sname,sex,birthday,cno) " +
        "values(#{name},#{sex},#{birthday},#{cno})")
//@Options(useGeneratedKeys = true,keyColumn = "sid",keyProperty = "id")
@SelectKey(statement = "select last_insert_id()",keyProperty = "id",
           keyColumn = "sid",resultType = int.class,before = false)
void addStudent(Student student);
```

## 4、一对一关系映射one注解
在xml开发中，一对一关系映射查询，用的是`association`标签解决，而在注解开发用的是`one`注解。
以第一个例子来修改，在IStudentDao修改查询所有的操作：

```java
/**
   * 查询所有操作
   * @return
   */
@Select("select * from student")
@Results(id = "studentMap",value = {
  @Result(id=true,column = "sid",property = "id"),
  @Result(column = "sname",property = "name"),
  @Result(column = "sex",property = "sex"),
  @Result(column = "birthday",property = "birthday"),
  @Result(column = "cno",property = "cno"),
  @Result(column = "cno",property = "cla",
/*select属性中填写执行方法的全限定类名加方法名，这里还开启了延迟加载（懒加载），在第四节会讲到
*/
one = @One(select = "com.stevensam.dao.IClasses.findByCId",
                     fetchType = FetchType.LAZY))
})
List<Student> findAll();
```

建立一个班级Dao接口

```java
package com.stevensam.dao;
import com.stevensam.domain.Classes;
import org.apache.ibatis.annotations.Select;
/**
 * author:seven lin
 * date:2018/8/3010:59
 * description:班级实体类
 **/
public interface IClasses {

    /**
     * 根据id查询班级
     * @param i
     * @return
     */
    @Select("select * from classes where cid=#{cid}")
    Classes findByCId(int i);
}
```

在测试类中的测试方法添加打印该学生的班级信息

```java
/**
 * 测试查询所有学生
 */
@Test
public void testFindAll(){
    List<Student> studentList = iStudentDao.findAll();
    for(Student student:studentList){
        System.out.println(student);
        System.out.println(student.getCla());//打印该学生对应的班级信息
    }
}
```

执行结果：

![tXLszV.png](https://s1.ax1x.com/2020/06/13/tXLszV.png)

为什么打印出来会不连续呢？而且查询班级的语句怎么只有三条，不应该是每次查询学生的时候就会触发查询班级的语句吗？
这里需要给大家讲的是缓存的知识，Mybatis中默认开启一级缓存。一级缓存是存在于sqlsession中的，当同一个语句同一个条件要查询的时候，系统会缓存第一次查询执行对象，而第二次查询的时候就不需要再创建查询对象。比如，当执行`select * from classes where cid=3`时，那么第一次会执行，第二次不会再去创建对象执行，所以才会有以下的结果：

![tXOGk9.png](https://s1.ax1x.com/2020/06/13/tXOGk9.png)

## 5、一对多关系映射many注解
内容和one是差不多的，只是根据班级去查询学生，思路相反，没有很多不同。这里暂做留空，不再赘述。





