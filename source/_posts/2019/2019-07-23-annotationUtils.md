---
layout: post
title: 从AnnotationConfigUtils看beanFactory的初始化
category: Springboot编程思想读书笔记
tags: annotation
---

无论是XML配置，还是注解驱动，都会调用AnnotationConfigUtils进行bean的注册。那我们分别启动XMl项目和注解项目，把debug打在AnnotationConfigUtils#registerPostProcessor上。

### XML如何调用AnnotationConfigUtils？
`提醒：`：这里我是使用maven搭建的能debug的springmvc项目

都知道SpringMvc默认是XmlWebApplicationContext做为入口，去加载xml。
XmlWebApplicationContext会调用`AbstractApplicationContext`中的refresh()方法，也是springMvc的核心。

```java
public void refresh() throws BeansException, IllegalStateException {
        ...
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        ...
}
```
refresh()中调用了obtainFreshBeanFactory()去获取beanFactory。

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    if (logger.isDebugEnabled()) {
        
        logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
    }
    return beanFactory;
}
```
这里refreshBeanFactory()和getBeanFactory()都是抽象方法，都有两个实现类分别实现了这两种抽象方法，`AbstractRefreshableApplicationContext`和`GenericApplicationContext`。
AbstractRefreshableApplicationContext对应的是`XmlWebApplicationContext`,而GenericApplicationContext对应的就是后续要讲述的`AnnotationConfigApplicationContext`。两者的
getBeanFactory都只是简单的get方法，所需beanFactory从断点可以看出是从refreshBeanFactory()初始化。
```java
@Override
	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
			DefaultListableBeanFactory beanFactory = createBeanFactory(); //创建一个空的 beanFactory
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
			loadBeanDefinitions(beanFactory); //初始化 beanFactory
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}

```
从断点可以看出后续是一系列xml解析过程，直接跳到这次的重点`AnnotationConfigUtils`.

解析后会调用AnnotationConfigUtils#registerAnnotationConfigProcessors，在此方法中进行了需多if判断如下：
```java
if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));//注册到beanFactory
		}
```
而这里的`CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME`是
```java
public static final String CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME =
			"org.springframework.context.annotation.internalConfigurationAnnotationProcessor";
```
这里共初始化了下列bean:
```java
org.springframework.context.annotation.internalConfigurationAnnotationProcessor

org.springframework.context.annotation.internalAutowiredAnnotationProcessor

org.springframework.context.annotation.internalRequiredAnnotationProcessor

### jsr250Present存在时加载
org.springframework.context.annotation.internalCommonAnnotationProcessor

### jpaPresent存在时加载
org.springframework.context.annotation.internalPersistenceAnnotationProcessor

org.springframework.context.event.internalEventListenerProcessor

org.springframework.context.event.internalEventListenerFactory
```
`registerPostProcessor()`则是注册到beanFactory中
```java
private static BeanDefinitionHolder registerPostProcessor(
			BeanDefinitionRegistry registry, RootBeanDefinition definition, String beanName) {

		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(beanName, definition); //生成相应的 bean
		return new BeanDefinitionHolder(definition, beanName);
	}
```
当方法调用完时，返回上一级ComponentScanBeanDefinitionParser#registerComponents
此方法中注意这几行代码
```java
Set<BeanDefinitionHolder> processorDefinitions =
                AnnotationConfigUtils.registerAnnotationConfigProcessors(readerContext.getRegistry(), source);
        for (BeanDefinitionHolder processorDefinition : processorDefinitions) {
            compositeDef.addNestedComponent(new BeanComponentDefinition(processorDefinition));
        }
```
processorDefinitions是AnnotationConfigUtils#registerAnnotationConfigProcessors的返回值集合。
这里的`compositeDef`其实就是`context:component-scan`，有兴趣的爱好者可以研究下，这里不深究。
在我执行for循环时，发现compositeDef是有值的，经过查看是对@Controller @Service 的等注解的扫描生成的实体类，再加上for循环添加的bean，构建了完
正的beanFactory#beandefinition。

### 注解如何调用AnnotationConfigUtils？
`注意：`我这里使用为springboot注解启动项目做为演示项目
```java
@Configuration
public class SpringBootDemoApplication {
	public static void main(String[] args) {
		AnnotationConfigApplicationContext context = new
				AnnotationConfigApplicationContext();
		context.register(SpringBootDemoApplication.class);
		context.refresh();
		context.close();
	}

}
```
和xml一样，断点打在 AnnotationConfigUtils
```java
AnnotationConfigApplicationContext context = new
				AnnotationConfigApplicationContext();
```
会初始化调用 AnnotationConfigUtils#registerAnnotationConfigProcessors注册
```java
org.springframework.context.annotation.internalConfigurationAnnotationProcessor

org.springframework.context.annotation.internalAutowiredAnnotationProcessor

org.springframework.context.annotation.internalRequiredAnnotationProcessor

### jsr250Present存在时加载
org.springframework.context.annotation.internalCommonAnnotationProcessor

### jpaPresent存在时加载
org.springframework.context.annotation.internalPersistenceAnnotationProcessor

org.springframework.context.event.internalEventListenerProcessor

org.springframework.context.event.internalEventListenerFactory
```
然后调用注册到beanFactory中
```java
his.beanDefinitionMap.put(beanName, beanDefinition);
                this.beanDefinitionNames.add(beanName);
```
再后期AbstractApplicationContext#finishBeanFactoryInitialization进行bean的加载。