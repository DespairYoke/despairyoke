---
layout: post
title: Spring核心注解分类
category: Spring源码阅读笔记
toc: true
tags: annotation
---

### Spring 模式注解

 Spring 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@Repository|数据仓储模式注解|2.0
@Component|通用组件模式注解|2.5
@Service |服务模式注解|2.5
@Controller|Web控制器模式注解|2.5
@Configration|配置类模式注解|3.0
@ImportResource|替换XML元素<import>|3.0
@Import|限定@Autowired依赖注入范围|3.0
@ComponentScan|扫描指定 package 下标注入Spring模式注解的类 |3.1

### 依赖注入注解

 Spring 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@Autowired|Bean依赖注入，优先按类型|2.5
@Qualifer|细粒度的@Autowired依赖查找|2.5

 Java 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@Resoure|Bean依赖注入，优先按名称|1.0

### Bean 定义注解如下表示。

 Spring 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@Bean |替换XML元素<bean>|3.0
@DependsOn|替换XML属性<bean depends-on="..."/>|3.0
@Lazy|替换XML属性<bean lazy-init="true|false"/>|3.0
@Primary|替换XML属性<bean primary="true|false"/>|3.0
@Role|替换XML属性<bean role="..."/>|3.1
@Lookup|替换XML属性<bean lookup-method="..."/>|4.1

### Spring 条件装配注解

 Spring 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@Profile | 配置化条件装配|3.1
@Conditional |编程条件装配|3.1

### 配置属性注解

 Spring 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@PropertySource|配置属性抽象 PropertySource 注解|3.1
@PropertySources|@PropertySource 集合注解|4.0

### 生命周期回调注解

 Java 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@PostConstruct|替换XML属性 <bean init-method="..."/>或者InitializingBean|1.0
@PreDestroy| 替换XML属性 <bean destroy-method="..."/>或者DisposableBean|1.0

### 注解属性注解

 Spring 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@AliaFor|别名注解属性，实现复用的目的 | 4.2

### 性能注解

Spring 注解 | 场景说明 | 起始版本 
-----------|-----|-------
@Indexed | 提升Spring 模式注解的扫描效率|5.o
