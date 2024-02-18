---
title: Spring Transaction 初探
date: 2022-04-03 09:41:34
tags:
  - Spring
  - Interview
  - Transaction
categories:
  - Spring
id: spring-transaction-intro
---

# 事务概念

1. 什么是事务
   
   事务是数据库操作的最基本单元，逻辑上一组操作。要么都成功，要么全部失败（即使只有一个操作失败）。

2. 经典场景：银行转账
   
   - Lucy转账￥100给Mary
   
   - Lucy少100，Mary多100

3. 事务四大特性（ACID）：
   
   - 原子性（Atomic）：全部成功或全部失败
   
   - 一致性（Consistency）：操作前后的总量不变
   
   - 隔离性（Isolation）：多事务操作之间不会产生影响
   
   - 持久性（Duration）：提交后表中数据就会发生变化

<!--more-->

# 搭建事务操作环境

- 以转账过程为例，需要在Dao层（数据库操作层）创建两个方法：扣钱和加钱；并且需要在Service层（业务操作）创建转账的方法来调用Dao层的两个方法。
1. 创建相关数据库以及表

2. 创建Service，搭建Dao，完成对象创建和注入关系
   
   - Service注入Dao，在Dao注入`JdbcTemplate`，在`JdbcTemplate`注入`DataSource`
     
     ```xml
     <context:component-scan base-package="com.shimmerjordan"></context:component-scan>
     
     <bean id="dataSource" class="class.alibaba.druid.pool.DruidDataSource" destroy-method="close">
         <property name="driverClassName" value="${jdbc.driverclass}"></property>
         <property name="url" value="${jdbc.url}"></property>
         <property name="username" value="${jdbc.username}"></property>
         <property name="password" value="${jdbc.password}"></property>
     </bean>
     
     <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
         <property name="dataSource" ref="dataSource"></property>
     </bean>
     ```
     
     ```java
     // UserService.java
     @Service
     public class UserService{
         // 注入Dao
         @Autowired
         private UserDao userDao;
     }
     ```
     
     ```java
     // UserDaoImpl.java
     @Repository
     public class UserDaoImpl implements UserDao{
         @Autowired
         private JdbcTemplate jdbcTemplate;
     }
     ```

3. 在Dao创建两个方法：加钱和扣钱，在service中创建转账方法
   
   ```java
   // UserDaoImpl.java
   @Repository
   public class UserDaoImpl implements UserDao{
       @Autowired
       private JdbcTemplate jdbcTemplate;
       @Override
       public void addMoney(){
           String sql = "update t_account set money=money+? where username=?";
           jdbcTemplate.update(sql, 100, "Mary");
       }
   
       @Override
       public void reduceMoney(){
           String sql = "update t_account set money=money-? where username=?";
           jdbcTemplate.update(sql, 100, "Lucy");
       }
   }
   ```
   
   ```java
   // UserService.java
   @Service
   public class UserService{
       // 注入Dao
       @Autowired
       private UserDao userDao;
   
       public void transferMoney(){
           userDao.reduceMoney();
           userDao.addMoney();
       }
   }
   ```

4. 上面的代码如果正常能够执行完毕是没有问题的，但是倮代码执行过程中出现异常便会有问题。
   
   ```java
   // UserService.java
   @Service
   public class UserService{
       // 注入Dao
       @Autowired
       private UserDao userDao;
   
       public void transferMoney(){
           userDao.reduceMoney();
           // 模拟异常（诸如断电断网等情况）
           int i = 10 / 0;
   
           userDao.addMoney();
       }
   }
   ```
   
   这会导致Lucy钱少了，但是Mary没有收到转账。**使用事务**解决以上问题：
   
   ```java
   // UserService.java
   @Service
   public class UserService{
       // 注入Dao
       @Autowired
       private UserDao userDao;
   
       public void transferMoney(){
           try {
               // 第一步 开启事务
   
               // 第二步 进行业务操作
               userDao.reduceMoney();
               // 模拟异常（诸如断电断网等情况）
               int i = 10 / 0;
   
               userDao.addMoney();
               // 第三步 没有发生异常，提交事务
           } catch(Exception e) {
               // 第四步 出现异常，事务回滚
           }
       }
   }
   ```

# Spring事务管理介绍

1. 事务一般添加到JavaEE三层结构中的Service层（业务逻辑层）

2. 在Spring进行事务管理操作
   
   两种方式：编程式事务管理（一般不采用）以及声明式事务管理（包括基于注解方式以及基于xml配置文件方式）

3. 在Spring进行声明式事务管理，底层使用AOP原理

4. Spring事务管理API
   
   1）提供一个接口，代表**事务管理器**，这个接口针对不同的框架提供不同的实现类。
   
   - `PlatformTransactionManager`接口，实现类有`DataSourceTransactionManager`(JDBC, Mybatis etc.)，`HibernateTransactionManager`(Hibernate etc.)

# 注解声明式事务管理

1. 在Spring配置文件配置事务管理器
   
   ```xml
   <!-- 创建事务管理器 -->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <!-- 注入数据源 -->
       <property name="dataSource" ref="dataSource"></property>
   </bean>
   ```
   
   1）在Spring配置文件中引入名称空间`tx`
   
   2）开启事务注解
   
   ```xml
   <tx:annotation-driven manager="transactionManager"></tx:annotation-driven>
   ```

2. 在Service类（Service类中方法）上面添加事务注解`@Transactional`

# 声明式事务管理参数配置

在Service类上面添加注解`@Transactional`，在这个注解里面可以配置事务相关参数

- `propagation`表示事务传播行为
  
  1）多**事务方法**直接进行调用，这个过程中事务是如何进行管理的
  
  - 事务方法：对数据库表数据进行变化的操作
  
  - `PROPAGATION_REQUIRED` 如果存在一个事务，则支持当前事务（当前的方法就在这个事务内运行）。否则，开启一个新的事务并在自己事务内运行。
  
  - `PROPAGATION_SUPPORTS` 如果存在一个事务，支持当前事务。如果没有事务，则非事务的执行。但是对于事务同步的事务管理器，PROPAGATION_SUPPORTS与不使用事务有少许不同。
  
  - `PROPAGATION_MANDATORY` 如果已经存在一个事务，支持当前事务。如果没有一个活动的事务，则抛出异常。
  
  - `PROPAGATION_REQUIRES_NEW` 总是开启一个新的事务。如果一个事务已经存在，则将这个存在的事务挂起
  
  - `PROPAGATION_NOT_SUPPORTED` 总是非事务地执行，并挂起任何存在的事务。
  
  - `PROPAGATION_NEVER` 总是非事务地执行，如果存在一个活动事务，则抛出异常
  
  - `PROPAGATION_NESTED` 如果一个活动的事务存在，则运行在一个嵌套的事务中. 如果没有活动事务, 则按`TransactionDefinition.PROPAGATION_REQUIRED` 属性执行

- `isolation`事务隔离级别
  
  1）事务有一个特性称为隔离性，多事务操作之间不会产生影响。不考虑隔离性会产生三个读问题：脏读、不可重复读、虚（幻）度：
  
  - **脏读**：是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中。这时，另外一个事务也访问这个数据，然后使用了这个数据。
  
  - **不可以重复读**：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
  
  - **幻读**：是指当事务不是独立执行时发生的一种现象。例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。
  
  - `ISOLATION_DEFAULT` 这是一个`PlatfromTransactionManager`默认的隔离级别，使用数据库默认的事务隔离级别。另外四个与JDBC的隔离级别相对应
  
  - `ISOLATION_READ_UNCOMMITTED` 这是事务最低的隔离级别，它充许别外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读
  
  - `ISOLATION_READ_COMMITTED` 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。可以避免脏读出现，但是可能会出现不可重复读和幻像读。通过**对修改的行进行加锁**，避免脏读。
  
  - `ISOLATION_REPEATABLE_READ` 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。通过**对查询所有的行进行加锁**，避免了不可重复读。
  
  - `ISOLATION_SERIALIZABLE` 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读。通过**对表加锁**实现。
    
    |                              | 脏读  | 不可重复读 | 幻读  |
    |:----------------------------:| --- | ----- | --- |
    | `ISOLATION_READ_UNCOMMITTED` | 有   | 有     | 有   |
    | `ISOLATION_READ_COMMITTED`   | 无   | 有     | 有   |
    | `ISOLATION_REPEATABLE_READ`  | 无   | 无     | 有   |
    | `ISOLATION_SERIALIZABLE`     | 无   | 无     | 无   |

- `timeout`超时时间：事务需要在一定时间（s）内提交，否则回滚，默认`-1`

- `readOnly`是否只读，默认`false`

- `rollbackFor`回滚：设置查询哪些异常进行事务回滚

- `norollbackFor`不回滚：设置出现哪些异常不进行事务回滚

# xml声明式事务管理

1. 在Spring配置文件中进行配置
   
   第一步 配置事务管理器（已在上面的步骤中完成）
   
   第二步 配置通知
   
   ```xml
   <tx:advice id="txadvice">
       <!-- 配置事务参数 -->
       <tx:attributes>
           <!-- 指定哪种规则的方法上面添加事务 -->
           <tx:method name="transferMoney" propagation="REQUIRED"/>
           <!-- <tx:method name="transfer*"/> -->
       </tx:attributes>
   </tx:advice>
   ```
   
   第三步 配置切入点和切面
   
   ```xml
   <aop:config>
       <!-- 配置切入点 -->
       <aop:pointcut id="pt" expression="execution(* com.shimmerjordan.service.UserService.*(..))"/>
       <!-- 配置切面 -->
       <aop:advisor advice-ref="" pointcut-ref="pt"/>
   </aop:config>
   ```

## 完全注解事务管理

在配置类中配置

```java
// TxConfig.java
@Configuration    // 配置类
@ComponentScan(basePackages = "com.shimmerjordan")    // 开启扫描
@EnableTransactionManagement    // 开启事务
public class TxConfig {
    // 创建数据库连接池
    @Bean
    public DruidDataSource getDruidDataSource(){
        DruidDataSource datasource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.cj.Driver");
        dataSource.setUrl("jdbc:mysql:///user_db");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        return dataSource;
    }

    // 创建JdbcTemplate对象
    // 到IOC容器中根据类型找到dataSource
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSouce){
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSouce);
        return jdbcTemplate;
    }

    // 创建事务管理器
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSouce){
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);
        return dataSourceTransactionManager;
    }
}
```
