---
layout: post
title: BeanFactory还是ApplicationContext及Aware的使用
category: SpringMvc系列

tags: SpringMvc
date: 2018-12-17
---


|Feature|BeanFactory|ApplicationContext|
|--|--|--------------|
|bean实例化/注入|<center>支持</center>|<center>支持</center>|
|实例生命周期管理|<center>不支持</center>|<center>支持</center>|
|自动BeanPostProcessor注册|<center>不支持</center>|<center>支持</center>|
|自动BeanFactoryPostProcessor注册|<center>不支持</center>|<center>支持</center>|
|方便MessageSource接收|<center>不支持</center>|<center>支持</center>|
|内置的ApplicationEvent发布机制|<center>不支持</center>|<center>支持</center>|

使用BeanFactroy加载时，要手动附加所需的功能。


实例：
```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

----------

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertyPlaceholderConfigurer cfg = new PropertyPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```
--------

#### 深入源码，探索AppliContext是如何和BeanFactory建立关系

-------------

* 1 先从ClassPathXmlApplicationContext走起
```java
    public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent) throws BeansException {
        super(parent);
        this.setConfigLocations(configLocations);
        if (refresh) {
            this.refresh();  //beanFactory的关联函数
        }

    }
```
* 2 进入refresh()
```java
    public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory =             this.obtainFreshBeanFactory(); //此方法是和beanFactory关联，进行xml解析和groovy解析生成bean的函数
    this.postProcessBeanFactory(beanFactory);//这里就是postProcessBeanFactory的自动注册，所以beanFactory不具备自动注册功能；注册成功后，后续bean的生命周期中会进行检查并执行相应的方法。
    this.invokeBeanFactoryPostProcessors(beanFactory);
    this.registerBeanPostProcessors(beanFactory);
    this.initMessageSource();
    this.initApplicationEventMulticaster();
    this.onRefresh();
    this.registerListeners();
    this.finishBeanFactoryInitialization(beanFactory);
    this.finishRefresh();
          ...
        }
    }
```

* 细看obtainFreshBeanFactory方法
```java
    protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
        this.refreshBeanFactory();
        ConfigurableListableBeanFactory beanFactory = this.getBeanFactory();
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Bean factory for " + this.getDisplayName() + ": " + beanFactory);
        }

        return beanFactory;
    }
```

### Aware的作用
--------------
  说到Aware必须要了解bean的生命周期(不了解的直接百度)，在bean的生命周期介绍中都会提到5大接口`BeanNameAware`,`BeanFactoryAware`,`ApplicationContextAware`,`InitializingBean`,`DisposableBean`。

  可以看出去`BeanNameAware`等都是`Aware`的继承；下面我们就分析这些接口的作用。
  
#### BeanNameAware
* 实现BeanNameAware的类都会实现一个`setBeanName`方法，而这个方法所给的值就是`beanName`;也就是所传递是容器的本身。同理ApplicationContextAware传递的参数是bean自己所在的上下文。

#### 实例
* User
```java
public class User implements BeanNameAware{

    private String id;

    private String name;

    public void setBeanName(String beanName) {
        id = beanName;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}
```
* User1
```java
public class User1 {

    private String id;

    private String name;

    public void setBeanName(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User1{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}
```
```xml
    <bean id="user" class="com.zwd.example.test.zzz.User">

    </bean>

    <bean id="user1" class="com.zwd.example.test.zzz.User1">

    </bean>
```
* Test
```java
public class ApplicationTest {

    @Test
    public void test1() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("tinyioc.xml");
        System.out.println("user: "+applicationContext.getBean("user"));
        System.out.println("user1: "+applicationContext.getBean("user1"));
    }
}
```
* 输出
```java
user: User{id='user', name='null'}
user1: User1{id='null', name='null'}
```


