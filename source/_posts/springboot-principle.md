---
title: SpringBoot工作原理
date: 2017-09-17 22:22:19
tags: Spring,SpringBoot,Microservices
---

本文主要通过阅读源码的方式来分析学习Spring Boot的工作原理，以期望在阅读完本文后可以明晰以下几个问题：  

1. Spring Boot 的项目结构，及主要模块的作用  
2. Spring Boot 如何做到约定优于配置  
3. Spring Boot 如何自动配置Bean的  
4. Spring Boot 如何‘抛弃’外部容器的  

<!-- more -->

## Spring Boot的项目结构及主要模块的作用  

如下图所示便是SpringBoot的源码结构（基于SpringBoot 1.5.x版本的分支）  
![SpringBoot Structure](/images/springboot-structure.PNG)  

其中主要模块的作用如下：  
1. **spring-boot**  
Spring Boot的主要模块，它的任务主要是创建并初始化Spring的应用上下文（ApplicationContext），启动嵌入式Servlet容器，如Tomcat或Jetty等。  

2. **spring-boot-autoconfigure**  
该模块主要任务就是根据我们的应用所依赖的类库，自动推断并配置我们可能所需要的Beans。  

3. **spring-boot-dependencies**  
该模块是一个Maven的依赖管理模块，它的主要功能是为SpringBoot项目提供依赖管理，它包含了几乎大部分我们常用的第三方依赖，包括数据库，缓存，消息，邮件，安全等相关类库的依赖。

4. **spring-boot-starter-parent**  
我们通常只需创建一个该模块的Maven子模块，便可快速创建SpringBoot项目，因为它是spring-boot-dependencies的Maven子模块，继承了父模块的管理的所有依赖外，还还包含了很多常用的Maven插件，如spring-boot-maven-plugin插件，可以将我们的Java项目打包成可以直接运行的Jar包。  

5. **spring-boot-starters**  
该模块提供了诸多我们可能会需要的依赖集合，每个starter都包含自身所需的其他依赖，只需将相应的starter加入到我们的依赖中，SpringBoot便会自动创建，并配置相应的Beans，如spring-boot-starter-data-redis，maven的依赖传递性，会添加相应依赖，然后SpringBoot便会自动创建redisTemplate等Bean。并且我们也可以编写自定义的starter。  

来看一个最简单的SpringBoot项目的例子，我们从这个实例中来探讨其中的原理细节：  


