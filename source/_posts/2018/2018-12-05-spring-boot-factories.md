---
layout: post
title: springboot2.0 自动注入文件spring.factories如何加载详解
category: springboot
tags: [springboot]
date: 2018-12-05
---

### 1.首先看下启动类：

``` java

@SpringBootApplication

public class Application {

public static void main(String[] args) {

SpringApplication.run(Application.class);

    }

}

```

启动类使用`@SpringBootApplication`注解，再看下这个注解内容

```java

@Target(ElementType.TYPE)

@Retention(RetentionPolicy.RUNTIME)

@Documented

@Inherited

@SpringBootConfiguration

@EnableAutoConfiguration

@ComponentScan(excludeFilters = {

@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),

@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })

public @interface SpringBootApplication {



```

再进入`@EnableAutoConfiguration`中查看

```java

@Target(ElementType.TYPE)

@Retention(RetentionPolicy.RUNTIME)

@Documented

@Inherited

@AutoConfigurationPackage

@Import(AutoConfigurationImportSelector.class)

public @interface EnableAutoConfiguration {

```

发现加载了一个`AutoConfigurationImportSelector.class`类

`温馨提示`：所有实现ImportSelector的类，都会在启动时被`ConfigurationClassParser`中的`processImports`进行实例化，并执行`selectImports`方法。

### 2.查看`AutoConfigurationImportSelector`中的`selectImports`

```java

@Override

public String[]selectImports(AnnotationMetadata annotationMetadata) {

if (!isEnabled(annotationMetadata)) {

return NO_IMPORTS;

  }

AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader

.loadMetadata(this.beanClassLoader);

  AnnotationAttributes attributes = getAttributes(annotationMetadata);

  List configurations = getCandidateConfigurations(annotationMetadata,

        attributes);

  configurations = removeDuplicates(configurations);

  Set exclusions = getExclusions(annotationMetadata, attributes);

  checkExcludedClasses(configurations, exclusions);

  configurations.removeAll(exclusions);

  configurations = filter(configurations, autoConfigurationMetadata);

  fireAutoConfigurationImportEvents(configurations, exclusions);

  return StringUtils.toStringArray(configurations);

}

```

通过方法可知是为了选出想要加载的import类，而如何获取这些类呢？其实是通过一个SpringFactoriesLoader去加载对应的spring.factories

下面展示如何和此类建立关系。

我们进入`selectImports`方法中的`getCandidateConfigurations`
```java

protected ListgetCandidateConfigurations(AnnotationMetadata metadata,

      AnnotationAttributes attributes) {

List configurations = SpringFactoriesLoader.loadFactoryNames(

getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());

  Assert.notEmpty(configurations,

        "No auto configuration classes found in META-INF/spring.factories. If you "

              +"are using a custom packaging, make sure that file is correct.");

  return configurations;

}
```

仔细看`SpringFactoriesLoader.loadFactoryNames`所需的参数，里面有个`classLoader`。这个classLoader是`AutoConfigurationImportSelector`类中的变量，这个变量起到了`关键`作用；这个变量是决定了我们去那里加载spring.factories配置文件。

许多人看到springFactories这个类加载配置文件时，以为是在spirngFactories所在包对应下的资源文件中，但是很不幸的是，并没有这个文件的存在；springFactories只是起到了一个加载工具的形式，想要找到具体的文件，还是要根据classLoader的来源决定路径。而上述中，classLoader是来源`AutoConfigurationImportSelector`所在的包下，所以加载的是`org.springframework.boot.autoconfigure`下的资源文件。

### 3. SpringFactoriesLoader如何加载

此类中只有三个方法，我们使用了其中二个

* loadFactoryNames

```java

public static ListloadFactoryNames(Class factoryClass, @Nullable ClassLoader classLoader) {

String factoryClassName = factoryClass.getName();

  return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());

}

```

```java

private static Map>loadSpringFactories(@Nullable ClassLoader classLoader) {

MultiValueMap result =cache.get(classLoader);

  if (result !=null) {

return result;

  }

try {

Enumeration urls = (classLoader !=null ?

classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :

ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));

      result =new LinkedMultiValueMap<>();

      while (urls.hasMoreElements()) {

URL url = urls.nextElement();

        UrlResource resource =new UrlResource(url);

        Properties properties = PropertiesLoaderUtils.loadProperties(resource);

        for (Map.Entry entry : properties.entrySet()) {

List factoryClassNames = Arrays.asList(

StringUtils.commaDelimitedListToStringArray((String) entry.getValue()));

            result.addAll((String) entry.getKey(), factoryClassNames);

        }

}

cache.put(classLoader, result);

      return result;

  }

catch (IOException ex) {

throw new IllegalArgumentException("Unable to load factories from location [" +

FACTORIES_RESOURCE_LOCATION +"]", ex);

  }

}

```