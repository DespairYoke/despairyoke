---
layout: post
title: BeanPostProcessor使用示例和原理
category: Spring源码阅读笔记
date: 2019-09-05
---

BeanPostProcessor主要用于在bean初始化的时候执行
### BeanPostProcessor 的来源
在执行 refresh时，通过add的方式添加
如
```java
@Override
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    beanFactory.addBeanPostProcessor(new ServletContextAwareProcessor(this.servletContext, this.servletConfig));
    beanFactory.ignoreDependencyInterface(ServletContextAware.class);
    beanFactory.ignoreDependencyInterface(ServletConfigAware.class);

    WebApplicationContextUtils.registerWebApplicationScopes(beanFactory, this.servletContext);
    WebApplicationContextUtils.registerEnvironmentBeans(beanFactory, this.servletContext, this.servletConfig);
}
```
也可以自己定义在xml中
```java
public class ChangeTeacherBeanPostProcessor implements BeanPostProcessor {

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		if (bean instanceof Teacher){
			Teacher teacher = (Teacher)bean;
			teacher.teach();
		}
		return bean;

	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return null;
	}
}
```

```xml
<bean id="changesmoking" class="com.zwd.bean.domain.ChangeTeacherBeanPostProcessor"/>
```

### BeanPostProcessor 何时使用

AbstractApplicationContext#finishBeanFactoryInitialization
```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
  
    // Instantiate all remaining (non-lazy-init) singletons.
    beanFactory.preInstantiateSingletons();
}
```
DefaultListableBeanFactory#preInstantiateSingletons
```java
public void preInstantiateSingletons() throws BeansException {
    getBean(beanName);
}
```
AbstractBeanFactory#getBean
```java
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}
```
AbstractBeanFactory#doGetBean
```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
        @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
    
    //创建bean
    if (mbd.isSingleton()) {
        sharedInstance = getSingleton(beanName, () -> {
            try {
                return createBean(beanName, mbd, args);
            }
            catch (BeansException ex) {
                // Explicitly remove instance from singleton cache: It might have been put there
                // eagerly by the creation process, to allow for circular reference resolution.
                // Also remove any beans that received a temporary reference to the bean.
                destroySingleton(beanName);
                throw ex;
            }
        });
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
    }
}
```

AbstractAutowireCapableBeanFactory#createBean
```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
        throws BeanCreationException {
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
}
```

AbstractAutowireCapableBeanFactory#doCreateBean
```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
    exposedObject = initializeBean(beanName, exposedObject, mbd); 
}
```

AbstractAutowireCapableBeanFactory#initializeBean
```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
}
```

AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
```java
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
    			Object current = processor.postProcessAfterInitialization(result, beanName);
    			if (current == null) {
    				return result;
    			}
    			result = current;
    		}
}
```