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
2. **spring-boot-actuator**  
3. **spring-boot-autoconfigure**  
4. **spring-boot-dependencies**  
5. **spring-boot-parent**  
6. **spring-boot-starters**  



