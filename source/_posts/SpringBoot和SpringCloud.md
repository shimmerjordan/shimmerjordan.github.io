---
title: SpringBoot和SpringCloud
date: 2020-06-25 13:47:51
tags:
	- Spring
	- SpringBoot
	- SpringCloud
categories: Spring
id: SpringBoot-SpringCloud
---

1、SpringBoot：一个快速开发框架，通过用MAVEN依赖的继承方式，帮助我们快速整合第三方常用框架，完全采用注解化（使用注解方式启动SpringMVC），简化XML配置，内置HTTP服务器（Tomcat，Jetty），最终以Java应用程序进行执行。

2、SpringCloud：一套目前完整的微服务框架，它是是一系列框架的有序集合。它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用SpringBoot的开发风格做到一键启动和部署。

这篇文章简单记录关于SpringBoot以及SpringCloud的关系与区别：

<!--more-->

# SpringBoot介绍

SpringBoot简化了基于Spring的应用开发，通过少量的代码就能创建一个独立的，产品级别的Spring应用。SpringBoot为Spring平台及第三方库提供开箱即用的设置，这样你就可以有条不紊的开始，多数Spring应用只需要很少的Spring配置。

SpringBoot是由Pivotal团队提供的全新框架，其设计目的是用来简化Spring应用的初始搭建以及开发过程。该框架使用了特定的方式进行配置，从而使开发人员不再需要定义样板化的配置。用我的话来理解，就是SpringBoot其实并不是什么新的框架，它默认配置了很多框架的使用方式，就像maven，整合了所有的jar包，SpringBoot整合了所有框架。

SpringBoot的核心思想就是约定大于配置，一切自动完成。采用SpringBoot可以大大的简化你的开发模式，所有你想集成的常用框架，它都有对应的组件支持。

简而言之，SpringBoot是Spring的一套快速配置脚手架，可以基于SpringBoot快速开发单个微服务。而SpringBoot是Spring的引导，也就是用于启动Spring的，使得Spring的学习和使用快速无痛。不仅适合工程结构的替换，更适合微服务的开发。

众所周知，Spring开发有非常头疼的三点：

以启动一个带Hibernate的Spring MVC为例。

1. 依赖太多了，而且要注意版本兼容。这个应用，要添加10-20个依赖，Spring相关的包10多个，然后是Hibernate包，Spring与Hibernate整合包，日志包，json包一堆，而且要注意版本兼容性。
2. 配置太多了，要配置注解驱动，要配置数据库连接池，要配置Hibernate，要配置事务管理器，要配置Spring MVC的资源映射，要在web.xml中配置启动Spring和Spring MVC等
3. 部署和运行麻烦。要部署到tomcat里面。不能直接用java命令运行。

而大部分的配置在各工程间是通用的，SpringBoot的哲学就是约定大于配置。既然很多东西都是一样的，为什么还要去配置——

1. 通过starter和依赖管理解决依赖问题。
2. 通过自动配置，解决配置复杂问题。
3.  通过内嵌web容器，由应用启动tomcat，而不是tomcat启动应用，来解决部署运行问题。

# SpringCloud介绍

SpringCloud是一系列框架的有序集合。它利用SpringBoot的开发便利性巧妙的简化了分布式系统基础设置的开发，如服务发现注册，配置中心，消息总线，负载均衡，断路器，数据监控等，都可以用SpringBoot的开发风格做到一键启动和部署。Spring 并没有重复制造轮子，它只是将目前各家公司开发的比较成熟，经得起实际考验的服务框架组合起来，通过SpringBoot风格进行封装屏蔽掉了复杂的配置和实现原理，最终结合开发者留出了一套简单易懂，易部署和易维护的分布式系统开发工具包。

微服务是可以独立部署，水平扩展。独立访问（或有独立的数据库）的服务单元，SpringCloud就是这些微服务的大管家，采用了微服务这种架构后，项目的数量会非常多，SpringCloud作为大管家就需要提供各种方案来维护整个生态。

SpringCloud就是一套分布式服务治理的框架，既然它是一条分布式治理的框架，那么它本身不会提供身体功能性的操作，更专注于服务之间的通讯，熔断，监控等。因此需要很多的组件来支持一套功能。

SpringCloud架构大致可以用下图来描述：

![NBgPu8.jpg](https://s1.ax1x.com/2020/06/25/NBgPu8.jpg)

这里从整体来看一下Spring Cloud主要组件，以及它的访问流程：

1. 外部或内部的非Spring Cloud项目统一通过API网管（Zuul）来访问内部服务。
2. 网关接收到请求后，从注册中心（Eureka）获取可用服务。
3. 由Ribbon进行负载均衡后，分发到后端的具体实例。
4. 微服务之间通过Feign进行通信处理业务。
5. Hystrix负责处理服务超时熔断。
6. Turbine监控服务间的调用和熔断相关指标。

# SpringBoot与SpringCloud的关系

SpringBoot是Spring的一套快速配置手架，可以通过SpringBoot快速开发单个微服务，SpringCloud是一个教育SpringBoot实现的云应用开发工具；SpringBoot用于快速，方便集成的单个微服务个体，SpringCloud关注全局的服务治理框架；SpringBoot使用了默认大于配置的理念，很多集成方案已经帮你选择好了，能不配置就不配置，SpringCloud 很大一部分是基于SpringBoot来实现，属于依赖关系，但是SpringBoot是可以离开SpringCloud独立使用开发项目。

简言之，结合Spring，三者的关系应为：`Spring –>Spring Boot–>Spring Cloud`


