---
title: Lexian-mall项目结构搭建
date: 2020-6-28 16:45:08
tags: 
	- Web开发
	- SpringCloud
categories: Spring Cloud
id: lexian-mall-SpringCloud
---

这里记录lexian-mall项目的后端脚手架搭建，仅包含部分配置，在后续的开发中所添加的依赖不再赘述。

<!--more-->

# 系统架构

经过需求分析阶段，在项目初期得到如下的系统架构图：

<img src="https://s1.ax1x.com/2020/06/28/NRGu7j.png" width = 76% />

# 项目结构

基于SpringCloud的电商平台lexian-mall项目的总体框架如下所示：

<img src="https://s1.ax1x.com/2020/06/28/N2IbPs.jpg" width = 50% height = 50% />

其项目结构如下所示：

```
.
├─.idea			
│  └─libraries
├─.mvn
│  └─wrapper
├─lexian-mall-api-dto									//类似于service-api与service对应
│  ├─lexian-mall-api-dto-cart-dto
│  ├─lexian-mall-api-dto-item-dto
│  ├─lexian-mall-api-dto-member-dto
│  ├─lexian-mall-api-dto-order-dto
│  ├─lexian-mall-api-dto-pay-dto
│  ├─lexian-mall-api-dto-search-dto
│  └─lexian-mall-api-dto-sso-dto
├─lexian-mall-basics									//分布式基础配置
│  ├─lexian-mall-basics-apollo-config-server				//阿波罗分布式配置中心			
│  ├─lexian-mall-basics-elk-kafka							//分布式事务解决框架
│  ├─lexian-mall-basics-eureka								//注册中心 8080
│  ├─lexian-mall-basics-lcn									//分布式事务解决框架
│  ├─lexian-mall-basics-xxljob								//分布式任务调度平台
│  ├─lexian-mall-basics-xxlsso-server						//分布式单点登录系统
│  ├─lexian-mall-basics-zipkin								//分布式调用链系统
│  └─lexian-mall-basics-zuul								//统一请求入口 80
├─lexian-mall-common									//通用框架
│  ├─lexian-mall-common-core								//核心工具类
│  └─lexian-mall-common-xxlsso-core							//单点登录系统核心工具类
├─lexian-mall-plugin									//插件类
├─lexian-mall-portal									//门户
│  ├─lexian-mall-porta-cms									//管理系统
│  ├─lexian-mall-portal-cart								//购物车系统
│  ├─lexian-mall-portal-pay									//支付系统
│  ├─lexian-mall-portal-search								//搜索系统
│  └─lexian-mall-portal-sso									//单点登录系统
├─lexian-mall-service									//服务层
│  ├─lexian-mall-service-auth								//OAuth授权验证服务
│  ├─lexian-mall-service-cart								//购物车服务
│  ├─lexian-mall-service-goods								//商品服务
│  ├─lexian-mall-service-integral							//积分服务
│  ├─lexian-mall-service-member								//会员服务
│  ├─lexian-mall-service-order								//订单服务
│  ├─lexian-mall-service-pay								//支付服务
│  ├─lexian-mall-service-search								//搜索服务
│  └─lexian-mall-service-sso								//单点登录服务
└─lexian-mall-service-api							//接口层，这里的接口与service层一一对应，不再赘述
    ├─lexian-mall-service-api-auth
    ├─lexian-mall-service-api-cart
    ├─lexian-mall-service-api-goods
    ├─lexian-mall-service-api-integral
    ├─lexian-mall-service-api-member
    ├─lexian-mall-service-api-order
    ├─lexian-mall-service-api-pay
    ├─lexian-mall-service-api-search
    └─lexian-mall-service-api-sso
```

注意：含有module的maven类型选择为pom类型，每个module类型为jar类型。