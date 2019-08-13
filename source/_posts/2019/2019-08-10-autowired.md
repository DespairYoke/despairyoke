---
layout: post
title: Springboot中@Autowired的原理解析
category: Springboot编程思想读书笔记
tags: annotation
toc: true
---
@Autowired是spring项目中经常使用到的注解，但大多数人都是简单的使用，并不知道原理，下面记录下个人在研究`@Autowired`时的收获和理解。
### @Autowired的用法
Autowired能使用在什么地方，参数含义。
#### @Autowired在何处使用
```
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
```
* 构造
* 方法
* 参数
* 字段
* 注解

#### @Autowired参数的意义
Autowired注解，只有一个required元素，默认是true，也是就是说这个值能改为false。true和false的意义不同。
```
require=ture 时，表示解析被标记的字段或方法，一定有对应的bean存在。
require=false 时，表示解析被标记的字段或方法，没有对应的bean存在不会报错。
```
```
public @interface Autowired {

	/**
	 * Declares whether the annotated dependency is required.
	 * <p>Defaults to {@code true}.
	 */
	boolean required() default true;

}
```
### @Autowired的原理
首先通过查看@Autowired注解注释文档，可以了解到加载放在了`AutowiredAnnotationBeanPostProcessor`。
而`AutowiredAnnotationBeanPostProcessor`是何时来的又是怎么执行去加载`@Autowired`呢？
#### 添加AutowiredAnnotationBeanPostProcessor
首先要知道再执行`AbstractApplicationContext#refresh()`方法时会执行`obtainFreshBeanFactory()`方法，而这个方法执行时，会在
`DefaultListableBeanFactory#beanDefinitionNames`数组中添加`internalAutowiredAnnotationProcessor `。而`internalAutowiredAnnotationProcessor `是和`AutowiredAnnotationBeanPostProcessor `被一起注册到registerPostProcessor中
```
if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
	RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
	def.setSource(source);
	beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
}
```
不知道可以参考[BeanFactory的加载](https://www.despairyoke.com/2019/07/24/2019/2019-07-23-annotationUtils/)
项目启动会自动执行refresh方法中的`registerBeanPostProcessors(beanFactory)`方法
```
	protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
		PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
	}
```
此方法使用委派模式进行委派。
```
	public static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

		String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
         ...
		registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);
         ...
	}
```
 getBeanNamesForType 是对前面加载的`internalAutowiredAnnotationProcessor`进行转换成`AutowiredAnnotationBeanPostProcessor`然后把返回值`postProcessorNames `转为`priorityOrderedPostProcessors`然后注册到registerBeanPostProcessors中
```
	private static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanPostProcessor> postProcessors) {

		for (BeanPostProcessor postProcessor : postProcessors) {
			beanFactory.addBeanPostProcessor(postProcessor);
		}
	}
```
AbstractBeanFactory#beanPostProcessors
```
	public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor) {
		Assert.notNull(beanPostProcessor, "BeanPostProcessor must not be null");
		// Remove from old position, if any
		this.beanPostProcessors.remove(beanPostProcessor);
		// Track whether it is instantiation/destruction aware
		if (beanPostProcessor instanceof InstantiationAwareBeanPostProcessor) {
			this.hasInstantiationAwareBeanPostProcessors = true;
		}
		if (beanPostProcessor instanceof DestructionAwareBeanPostProcessor) {
			this.hasDestructionAwareBeanPostProcessors = true;
		}
		// Add to end of list
		this.beanPostProcessors.add(beanPostProcessor);
	}
```
到此知道了`AutowiredAnnotationBeanPostProcessor`的来龙去脉。下面就开下AutowiredAnnotationBeanPostProcessor如何去解析@Autowired。

#### AutowiredAnnotationBeanPostProcessor解析@Autowired
refresh()执行完registerBeanPostProcessors 方法后，继续执行finishBeanFactoryInitialization(beanFactory);
```
	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
        ...
		// Instantiate all remaining (non-lazy-init) singletons.
		beanFactory.preInstantiateSingletons();
	}
```
这里spring会创建所需要的bean，一般在controller层中引入的service也会在此时依赖加载和创建。
```
	public void preInstantiateSingletons() throws BeansException {
		if (logger.isTraceEnabled()) {
			logger.trace("Pre-instantiating singletons in " + this);
		}
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
                       ...
			getBean(beanName);
                      ...
		}
	...
```
当执行到我们自定义的controller层时，会在getBean中进行执行autowire解析。
```
	public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
	}
```
```
	protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
              return createBean(beanName, mbd, args);
}
```
```
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {
      ...
      doCreateBean(beanName, mbdToUse, args);
      ...
}
```
而doCreateBean是真正的创建bean实现，在创建的时候调用了`applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);`
```
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
              ...
			applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            ...
	}
```
终于在applyMergedBeanDefinitionPostProcessors中看到了我们熟悉的身影`BeanPostProcessor`
```
	protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof MergedBeanDefinitionPostProcessor) {
				MergedBeanDefinitionPostProcessor bdp = (MergedBeanDefinitionPostProcessor) bp;
				bdp.postProcessMergedBeanDefinition(mbd, beanType, beanName);
			}
		}
	}
```
`getBeanPostProcessors`方法获取的就是上文中添加的`AutowiredAnnotationBeanPostProcessor `的集合`beanPostProcessors`。
则执行我们的想看到的`AutowiredAnnotationBeanPostProcessor#postProcessMergedBeanDefinition`。
```
	public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
		InjectionMetadata metadata = findAutowiringMetadata(beanName, beanType, null);
		metadata.checkConfigMembers(beanDefinition);
	}
```
findAutowiringMetadata查询这个beanName中，是否有
```
	private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, @Nullable PropertyValues pvs) {
         ...
	  	metadata = buildAutowiringMetadata(clazz);
         ...
	}
```
```
	private InjectionMetadata buildAutowiringMetadata(final Class<?> clazz) {
		List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
		Class<?> targetClass = clazz;
				AnnotationAttributes ann = findAutowiredAnnotation(bridgedMethod);
				if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
					boolean required = determineRequiredStatus(ann);
					PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
					currElements.add(new AutowiredMethodElement(method, required, pd));
				}
			});

			elements.addAll(0, currElements);
			targetClass = targetClass.getSuperclass();
		}
		return new InjectionMetadata(clazz, elements);
	}
```
`findAutowiredAnnotation(bridgedMethod)`找到这个类的autowire注解的类，添加到InjectionMetadata对象中。然后在checkConfigMembers方法中又注册到beanDefinition中。
```
	@Override
	public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
		InjectionMetadata metadata = findAutowiringMetadata(beanName, beanType, null);
		metadata.checkConfigMembers(beanDefinition);
	}
```
```
	public void checkConfigMembers(RootBeanDefinition beanDefinition) {
		Set<InjectedElement> checkedElements = new LinkedHashSet<>(this.injectedElements.size());
		for (InjectedElement element : this.injectedElements) {
			Member member = element.getMember();
			if (!beanDefinition.isExternallyManagedConfigMember(member)) {
				beanDefinition.registerExternallyManagedConfigMember(member);
				checkedElements.add(element);
				if (logger.isTraceEnabled()) {
					logger.trace("Registered injected element on class [" + this.targetClass.getName() + "]: " + element);
				}
			}
		}
		this.checkedElements = checkedElements;
	}
```
