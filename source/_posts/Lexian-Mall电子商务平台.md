---

title: Lexian-Mall电子商务平台
date: 2020-07-24 11:19:00
tags: 
	- 电商平台
	- SpringBoot
	- SpringCloud
	- Redis
	- uni-app
	- Vue
	- CMS管理
categories: Spring Cloud
id: LexianMallIntro
---

# 介绍

Neu中软国际实训 Lexian-Mall电子商务平台，包括基于SpringBoot的后端服务器、基于SpringCloud的后端服务器、基于Vue的CMS后台管理系统以及基于Uni-app的客户端。

<!--more-->

## 相关技术栈

<table>
	<tr >
		<th>技术栈</th>
		<th>说明</th>
	</tr>
 	<tr>
 		<td>Mysql </td>
 		<td>Mysql版本号区别，操作系统对应的不同版本，Mysql相对其他数据库的优势，Mysql服务介绍、启动服务的方式,配置客户端服务命名，配置文件的位置</td>
 	</tr>
 	<tr>
 		<td>html5&css3</td><td>Html5新特性、Html5新增块标签、Html5新增表单元素、CSS3新特性</td>
 	</tr>
	<tr>
		<td>VUE技术</td>
		<td>Vue技术</td>
	</tr>
     <tr><td>VueX</td><td>Vue数据管理技术</td></tr>
     <tr><td>ElementUI</td><td>用于Vue界面的组件化技术</td></tr>
     <tr><td>Echarts</td><td>用于电子图表的实现</td></tr>
     <tr><td  rowspan="2">SpringMVC</td><td>环境搭建、Controller控制器、传参、视图解析器</td></tr>
     <tr><td>文件上传下载、表单验证</td></tr>
     <tr><td rowspan="2">Mybatis</td><td>生成工具、CURD操作、映射文件</td></tr>
     <tr><td>动态SQL、分页操作、Dao设计</td></tr>
     <tr><td rowspan="2">Spring</td><td>环境搭建、IOC与AOP</td></tr>
     <tr><td>SSM整合，共同处理封装</td></tr>
     <tr><td>Redis</td><td>高速缓存和消息队列代理的应用</td></tr>
     <tr><td>SpringBoot</td><td>SpringBoot框架自动配置</td></tr>
     <tr><td>Log4j</td><td>日志系统</td></tr>
     <tr><td>Hikari</td><td>快速的数据库连接池</td></tr>
     <tr><td>pagehelper</td><td>分页插件</td></tr>
     <tr><td>NATAPP</td><td>内网穿透工具</td></tr>
     <tr><td rowspan="2">Spring Cloud</td><td>Ribbon、feign、getWay、eureka、Zuul等</td></tr>
     <tr><td>Hytrix、config、用户中心、事务中心。</td></tr>
 </table>
## 项目开发方法

全阶段采用模块化分工与全组合作的方式进行，将瀑布式的生命周期模型结合原型法加速了项目的进展。项目后期利用单元测试与逻辑测试，使得项目有更强健的稳定性。

## 开发工具

| **工具名**          | **用途**                             |
| ------------------- | ------------------------------------ |
| HBuilerX            | Uni-app端的开发IDE                   |
| Navicat             | 数据库连接可视化工具                 |
| SQLyogEnt           | 数据库可视化工具                     |
| Ezdml               | 数据库文档生成工具                   |
| IntelliJ IDEA       | SpringCloud以及SpringBoot后端开发IDE |
| VSCode              | CMS后台管理端开发IDE                 |
| NATAPP              | 内网穿透工具                         |
| SourceTree          | Git管理可视化工具                    |
| RedisDesktopManager | Redis可视化工具                      |
| Pycharm             | Python可视化工具                     |
| 微信开发者工具      | 用于Uni-app小程序端测试              |
| Processon           | 流程图、状态图等作图工具             |
| katalon recorder    | 自动化测试脚本生成插件               |
| mybatis-generator   | MyBaits生成工具                      |

## 开发环境

| **本地环境**           | **远端环境**                           |
| ---------------------- | -------------------------------------- |
| Windows10              | 滴滴云：Windows Server 2019 DataCenter |
| JDK1.8                 | MySQL8.0                               |
| Tomcat9                | 百度接口：OCR证件识别                  |
| MySQL8.0               | 七牛云：图片上传接口                   |
| SpringBoot2.3          |                                        |
| SpringCloud Hoxton.SR1 |                                        |
| Redis-x64-3.0.504      |                                        |

# 软件架构

## 架构设计

这里采用了SpringBoot以及SpringCloud双后端服务器的并行开发方法。由于受机器性能的限制。在本地测试时大部分采用基于SpringBoot的后端服务器，而SpringCloud环境服务器需要部署在机器条件更好的组员或者远端服务器上。这里的架构展示是基于SpringCloud的服务器架构，其详情如下图所示：

![Ujp1I0.png](https://s1.ax1x.com/2020/07/24/Ujp1I0.png)

对于SpringBoot后端，其主要的前置框架如下图所示，

![UjC2VA.png](https://s1.ax1x.com/2020/07/24/UjC2VA.png)

## 功能模块划分

在需求分析与设计阶段，我们已经根据实际情况对项目的功能模块进行了初步的分析与设计。在进行项目编码的时候一方面严格按照模块划分规则来实现项目，另一方面也对初期生成的模块划分进行了优化。其最终的高一致性版本如下表所示：
<table>
<tr><th>角色</th><th>功能类别</th><th>子功能</th></tr>
 <tr><td rowspan="21">顾客</td><td rowspan="3">登陆</td><td>登陆（手机号登录、登录名登录）</td></tr>
 <tr><td>注册</td></tr>
 <tr><td>找回密码</td></tr>
 <tr><td rowspan="3">购物车</td><td>查看购物车</td></tr>
 <tr><td>添加购物车</td></tr>
 <tr><td>移除/清空购物车</td></tr>
 <tr><td rowspan="5">个人中心</td><td>实名认证</td></tr>
 <tr><td>修改个人信息</td></tr>
 <tr><td>系统设置</td></tr>
 <tr><td>地址管理</td></tr>
 <tr><td>浏览记录管理</td></tr>
 <tr><td rowspan="4">订单管理</td><td>下单操作</td></tr>
 <tr><td>订单查看</td></tr>
 <tr><td>订单状态管理</td></tr>
 <tr><td>申诉、评价订单</td></tr>
 <tr><td rowspan="6">个人账户商品管理</td><td>商品搜索</td></tr>
 <tr><td>查看商品详情</td></tr>
 <tr><td>查看商品评价</td></tr>
 <tr><td>商品浏览记录</td></tr>
 <tr><td>商品收藏夹</td></tr>
 <tr><td>推广分享</td></tr>
 <tr><td rowspan="18">店铺管理员</td><td rowspan="6">店铺订单管理</td><td>浏览订单记录</td></tr>
 <tr><td>查询订单</td></tr>
 <tr><td>修改订单信息</td></tr>
 <tr><td>新增订单</td></tr>
 <tr><td>处理客户申请</td></tr>
 <tr><td>统计店铺订单信息</td></tr>
 <tr><td rowspan="3">店铺活动管理</td><td>查看活动列表</td></tr>
 <tr><td>提交活动申请</td></tr>
 <tr><td>修改活动申请</td></tr>
 <tr><td rowspan="3">店铺信息管理</td><td>查看门店信息</td></tr>
 <tr><td>门店信息管理</td></tr>
 <tr><td>门店营业管理</td></tr>
 <tr><td rowspan="4">店铺商品管理</td><td>查看商品信息列表</td></tr>
 <tr><td>新增商品</td></tr>
 <tr><td>修改商品信息</td></tr>
 <tr><td>上架/下架商品</td></tr>
 <tr><td rowspan="2">店铺活动商品管理</td><td>活动商品信息展示</td></tr>
 <tr><td>活动商品管理</td></tr>
 <tr><td rowspan="7">系统管理员</td><td>平台商品管理</td><td>下架违规商品</td></tr>
 <tr><td rowspan="4">平台店铺管理</td><td>新增门店审核</td></tr>
 <tr><td>暂停门店运营</td></tr>
 <tr><td>修改门店信息</td></tr>
 <tr><td>移除门店</td></tr>
 <tr><td rowspan="2">平台权限管理</td><td>查看后端人员信息</td></tr>
 <tr><td>权限管理</td></tr>
</table>
## SSM框架下的类调用关系

主要编写的类和对象的详细设计主要涉及到的包括表现层（前端），controller层，service层，DAO层四层，在下述的调用图中已用蓝色背景标识。

![UjZAmT.png](https://s1.ax1x.com/2020/07/24/UjZAmT.png)

# 安装教程

1. cmd进入`lexian_admin/vue-element-admin-master`进行Vue项目的npm/cnpm安装

2. 运行`数据库文件/lexian.sql`建立数据库并插入数据

3. 启用Redis、MySQL服务

4. （H5测试环境）右击浏览器快捷方式/属性/目标：在末尾添加以下代码，注意开头有空格

   ```
    --test-type --disable-web-security
   ```

# 使用说明

1. 配置后端application.yaml

   调整MySQL数据库地址、密码以及Redis密码、后端运行端口号

   ```yaml
   spring:
     datasource:
       username: root
       password: password
       url: "jdbc:mysql://localhost:3306/lexian?useUnicode=true&characterEncoding=utf8&serverTimezone=UTC"
       driver-class-name: com.mysql.cj.jdbc.Driver
       hikari:
         # 连接只读数据库时配置为true， 保证安全
         read-only: false
         # 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 缺省:30秒
         connection-timeout: 30000
         # 一个连接idle状态的最大时长（毫秒），超时则被释放（retired），缺省:10分钟
         idle-timeout: 600000
         # 一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒
         max-lifetime: 450000
         # 连接池中允许的最大连接数。缺省值：10
         maximum-pool-size: 60
         minimum-idle: 10
   
     redis:
       database:
         host: 127.0.0.1
         port: 6379
         password:
         jedis:
           pool:
             max-active: 100
             max-idle: 10
             max-wait: 100000
         timeout: 5000
   server:
     port: 8088
     servlet:
       #context-path: /ocp
       encoding:
         charset: UTF-8
         force: true
         enabled: true
   ```

2. 修改前端（包括CMS以及Uni-app）连接后端API地址，使其端口号一致

# 参与贡献

- 辛毅

- 鞠桥丹

- 曾德歆

- 杨佳玉

- 易忱

- 金卓然

# 项目地址

github：https://github.com/shimmerjordan/Lexian-Mall