---
title: Spring系列(一) Spring框架总览.md
comments: false
date: 2018-10-16 00:08:17
categories: interview
tags: spring
img: 
---

# Spring框架

<!-- TOC -->

- [Spring框架](#spring框架)
    - [基本框架列表](#基本框架列表)
    - [Spring的核心](#spring的核心)
        - [Spring之旅](#spring之旅)
            - [简化java开发](#简化java开发)
            - [容纳你的bean](#容纳你的bean)
            - [俯翰Spring风景线](#俯翰spring风景线)
            - [Spring新功能](#spring新功能)
        - [装配Bean](#装配bean)
            - [声明Bean](#声明bean)
            - [注入Bean属性](#注入bean属性)
            - [使用表达式装配](#使用表达式装配)
        - [最小化SpringXML配置](#最小化springxml配置)
            - [自动装配bean属性](#自动装配bean属性)
            - [使用注解装配](#使用注解装配)
            - [自动检测Bean](#自动检测bean)
            - [使用Spring基于Java的配置](#使用spring基于java的配置)
        - [面向切面的Spring](#面向切面的spring)
            - [什么是面向切面编程](#什么是面向切面编程)
            - [使用切点选择连接点](#使用切点选择连接点)
            - [在XML中声明切面](#在xml中声明切面)
            - [注解切面](#注解切面)
            - [注入AspectJ切面](#注入aspectj切面)
    - [Spring应用程序的核心组件](#spring应用程序的核心组件)
        - [征服数据库](#征服数据库)
            - [Spring的数据访问哲学](#spring的数据访问哲学)
            - [配置数据源](#配置数据源)
            - [在Spring中使用JDBC](#在spring中使用jdbc)
            - [在Spring中集成Hibernate](#在spring中集成hibernate)
            - [Spring与Java持久化API](#spring与java持久化api)
        - [事务管理](#事务管理)
            - [理解事务](#理解事务)
            - [选择事务管理器](#选择事务管理器)
            - [在Spring中的编码事务](#在spring中的编码事务)
            - [声明式事务](#声明式事务)
        - [使用Spring MVC构建Web应用程序](#使用spring-mvc构建web应用程序)
            - [SpringMVC起步](#springmvc起步)
            - [编写基本的控制器](#编写基本的控制器)
            - [处理控制器的输入](#处理控制器的输入)
            - [处理表单](#处理表单)
            - [处理文件上传](#处理文件上传)
        - [使用SpringWebFlow](#使用springwebflow)
        - [保护Spring应用](#保护spring应用)
    - [Spring集成](#spring集成)
        - [使用远程服务](#使用远程服务)
        - [为Spring添加REST功能](#为spring添加rest功能)
        - [Spring消息](#spring消息)
        - [使用JMX管理](#使用jmx管理)
        - [其他Spring技巧](#其他spring技巧)

<!-- /TOC -->

## 基本框架列表
- 核心容器
- Spring上下文
- SpringAOP
- SpringDAO
- SpringORM
- SpringWeb模块
- SpringMVC框架

## Spring的核心  

### Spring之旅  

#### 简化java开发
- 激发pojo的潜能
- 依赖注入
- 应用切面
- 使用模版消除样板式代码  

#### 容纳你的bean
- 与应用上下文共事
- bean的生命周期

#### 俯翰Spring风景线
- spring模版
- spring portfolio

#### Spring新功能
- spring2.5新特性
- spring3.0新特性
- spring portfollo新特性

### 装配Bean
#### 声明Bean
- 创建spring配置
- 声明一个简单的bean
- 通过构造器注入
- bean的作用域
- 初始化和销毁bean

#### 注入Bean属性
- 引入简单值
- 引入其他bean
- 使用spring的命名空间
- 装配属性
- 装配集合
- 装配空值

#### 使用表达式装配
- spEL的基本原理
- 在spEL值上执行操作
- 在spEL中筛选集合

### 最小化SpringXML配置
#### 自动装配bean属性
- 4种类型的自动装配
- 默认的自动装配
- 混合使用自动装配和显示装配

#### 使用注解装配
- 使用@Autowired
- 借助@Inject实现基于标准的自动装配
- 在注解中使用表达式

#### 自动检测Bean
- 为自动检测标注bean
- 过滤组件扫描

#### 使用Spring基于Java的配置
- 创建基于java的配置
- 定义一个配置类
- 声明一个简单的bean
- 使用spring的基于java的配置进行注入

### 面向切面的Spring
#### 什么是面向切面编程
- 定义AOP术语
- Spring对AOP的支持

#### 使用切点选择连接点
- 编写切点
- 使用spring的bean()指示器

#### 在XML中声明切面
- 声明前置和后置通知
- 声明环绕通知
- 为通知传递参数
- 通过切面引入新功能

#### 注解切面
- 注解环绕通知
- 传递参数给所标注的通知
- 标注引入

#### 注入AspectJ切面

## Spring应用程序的核心组件
### 征服数据库
#### Spring的数据访问哲学
- 了解spring的数据库访问异常体系
- 数据访问模块化
- 使用DAO支持类

#### 配置数据源
- 使用JNDI数据源
- 使用数据源连接池
- 基于JDBC驱动的数据源

#### 在Spring中使用JDBC
- 应对失控的jdbc代码
- 使用jdbc模版

#### 在Spring中集成Hibernate
- hibernate预览
- 声明hibernate的session工厂
- 构建不依赖于spring的hibernate代码

#### Spring与Java持久化API
- 配置实体管理器工厂
- 编写基于JPA的DAO

### 事务管理
#### 理解事务
- 用4个词来表示事务
- 理解spring对事务管理的支持  

#### 选择事务管理器
- JDBC事务
- Hibernate事务
- Java持久化API事务
- JTA(Java Transaction API)事务

#### 在Spring中的编码事务

#### 声明式事务
- 定义事务属性
- 在XML中定义事务
- 定义注解驱动的事务

### 使用Spring MVC构建Web应用程序
#### SpringMVC起步
- 跟踪SpringMVC的请求
- 搭建SpringMVC

#### 编写基本的控制器
- 配置注解驱动的SpringMVC
- 定义首页的控制器
- 解析视图
- 定义首页的视图
- 完成spring应用上下文

#### 处理控制器的输入
- 编写处理输入的控制器
- 渲染视图

#### 处理表单
- 展现注册表单
- 处理表单输入
- 校验输入

#### 处理文件上传
- 在表单上添加文件上传域
- 接收上传文件
- 配置spring支持文件上传

### 使用SpringWebFlow
- 安装SpringWebFlow
- 流程的组件
- 组合起来：比萨流程
- 保护Web流程

### 保护Spring应用
- Spring Security介绍
- 保护Web请求
- 保护视图级别的元素
- 认证用户
- 保护方法的调用

## Spring集成
### 使用远程服务
- Spring远程调用概览
- 使用RMI
- 使用Hession和Burlap发布远程服务
- 使用Spring的HttpInvoker
- 发布和使用Web服务

### 为Spring添加REST功能
- 了解REST
- 编写面向资源的控制器
- 表达资源
- 编写REST客户端
- 提交RESTful表单

### Spring消息
- JMS简介
- 在Spring中搭建消息代理
- 使用Spring的JMS模版
- 创建消息驱动的POJO
- 使用基于消息的RPC

### 使用JMX管理
- 将SpringBean导出为MBean
- 远程MBean
- 处理通知

### 其他Spring技巧
- 外部化配置
- 装配JNDI对象
- 发送邮件
- 调度和后台任务
