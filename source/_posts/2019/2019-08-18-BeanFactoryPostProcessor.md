---
layout: post
title: BeanFactoryPostProcessor使用示例和原理
category: Spring源码阅读笔记
date: 2019-08-18
---

### BeanFactoryPostProcessor 和 BeanPostProcessor 的区别
看代码说话：
```java
    //允许自定义BeanFactoryPostProcessors改变beanDifinition
    invokeBeanFactoryPostProcessors(beanFactory);
    
    // Register bean processors that intercept bean creation.
    registerBeanPostProcessors(beanFactory);
```
从代码的执行顺序可以看出，BeanFactoryPostProcessor要在BeanPostProcessor之前使用,所以二者同时使用时，要注意加载顺序。
BeanFactoryPostProcessor是通过获取到beanDefinition，改变值；beanPostProcessor是通过初始化bean时，改变bean的值。
### 如何使用BeanFactoryPostProcessor

示例：
定义一个Teacher类
```java
public class Teacher {


	/**
	 * 老师的姓名
	 */
	private String name;

	/**
	 * 年龄
	 */
	private int age;

	/**
	 * 是否抽烟
	 */
	private boolean smoking;

	/**
	 * 老师教授的课程
	 */
	private String language;

	public Teacher() {
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public boolean isSmoking() {
		return smoking;
	}

	public void setSmoking(boolean smoking) {
		this.smoking = smoking;
	}

	public String getLanguage() {
		return language;
	}

	public void setLanguage(String language) {
		this.language = language;
	}

	public void teach(){
		System.out.println("I am :"+name+" and I will teach you :"+language + " and I "+(smoking?"will":"will not")+" smoking");
	}

}

```

定义一个BeanFactoryPostProcessor的实现类
```java
public class ChangeTeacherBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

		BeanDefinition teacher = beanFactory.getBeanDefinition("teacher");
		MutablePropertyValues propertyValues = teacher.getPropertyValues();
		if (propertyValues.contains("smoking")){
			propertyValues.add("smoking",false);
		}

	}
}

```
把smoking属性改为false

编写bean.xml
```xml
<bean id="teacher" class="com.zwd.bean.domain.Teacher">
    <property name="name" value="张三"/>
    <property name="age" value="32"/>
    <property name="language" value="java"/>
    <property name="smoking" value="true"/>
</bean>
<bean id="changesmoking" class="com.zwd.bean.domain.ChangeTeacherBeanFactoryPostProcessor"/>
```

运行测试用例
```java
	@Test
	public void test1() {

		ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");

		Teacher teacher = (Teacher) context.getBean("teacher");

		teacher.teach();

	}
```

输出结果如下：
```
I am :张三 and I will teach you :java and I will not smoking
```
说明`ChangeTeacherBeanFactoryPostProcessor`生效。

### 原理分析
每个bean都会对应一个beanDifinition，相当于bean的名片，beanfactoryPostProcessor的修改，相当于重新生成名片。
启动上下文
AbstractApplicationContext#refresh()
```java
public void refresh(){
    //允许自定义BeanFactoryPostProcessors改变beanDifinition
    invokeBeanFactoryPostProcessors(beanFactory);
}
```
AbstractApplicationContext#invokeBeanFactoryPostProcessors()
```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
}
```
PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors
进行对beanFactoryPostProcessors过滤，挑选符合条件的beanFactoryPostProcessor
```java
public static void invokeBeanFactoryPostProcessors(
         ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
    String[] postProcessorNames =
            beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false); //获取BeanFactoryPostProcessor的实现类
    invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
 
 }
```
PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors
执行上述挑选过的BeanFactoryPostProcessor，及示例中的ChangeTeacherBeanFactoryPostProcessor也会被执行
```java
	private static void invokeBeanFactoryPostProcessors(
			Collection<? extends BeanFactoryPostProcessor> postProcessors, ConfigurableListableBeanFactory beanFactory) {

		for (BeanFactoryPostProcessor postProcessor : postProcessors) {
			postProcessor.postProcessBeanFactory(beanFactory);
		}
	}
```
