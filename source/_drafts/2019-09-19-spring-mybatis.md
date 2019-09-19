---
layout: post
title: spring到mybatis的加载过程
category: Mybatis
date: 2019-09-19
tags: mybatis
---

### Mapper文件何时加载？
无论是基于xml或注解的spring framework 还是springboot的自动注入，mybatis并没有进行什么改变。相反spring为了使用mybatis而进行了相应改变。
基于xml的配置
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 映射数据源 -->
    <property name="dataSource" ref="dataSource"/>
    <!-- 映射mybatis核心配置文件 -->
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <!-- 映射mapper文件 -->
    <property name="mapperLocations">
        <list>
            <value>classpath:com/bdqn/dao/**/*.xml</value>
        </list>
    </property>
</bean>
```
springboot
```java
@org.springframework.context.annotation.Configuration
@ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
@ConditionalOnBean(DataSource.class)
@EnableConfigurationProperties(MybatisProperties.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class MybatisAutoConfiguration {
  
}
```
比较共同点，看来重点应该是 sqlSessionFactory SqlSessionFactoryBean
看下这两个类
```java
public interface SqlSessionFactory {
    SqlSession openSession();

    SqlSession openSession(boolean var1);

    SqlSession openSession(Connection var1);

    SqlSession openSession(TransactionIsolationLevel var1);

    SqlSession openSession(ExecutorType var1);

    SqlSession openSession(ExecutorType var1, boolean var2);

    SqlSession openSession(ExecutorType var1, TransactionIsolationLevel var2);

    SqlSession openSession(ExecutorType var1, Connection var2);

    Configuration getConfiguration();
}
```
功能如其名，是一个创建SqlSession的Factory
```java
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
    //内容过多
}
```
内容过多，不展示，不过还是能看到几个熟悉的身影
1、private Resource[] mapperLocations; //mapper文件集合
  
2、private DataSource dataSource; //数据源

从xml的配置方式是能最直接看到spring创建了一个bean（SqlSessionFactoryBean）并进行了赋值操作。而还有一个重点是这个类继承了 InitializingBean，了解bean生命
周期的程序员都应该知道继承 InitializingBean 的类在创建bean的同时会调用 afterPropertiesSet 方法，哪SqlSessionFactoryBean中必有此方法。
```java
  public void afterPropertiesSet() throws Exception {
    notNull(dataSource, "Property 'dataSource' is required");
    notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
    state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
              "Property 'configuration' and 'configLocation' can not specified with together");

    this.sqlSessionFactory = buildSqlSessionFactory();
  }
```
跟踪下 buildSqlSessionFactory
```java
  protected SqlSessionFactory buildSqlSessionFactory() throws IOException {

    Configuration configuration;

    XMLConfigBuilder xmlConfigBuilder = null;
    //... 内容过多 请自己浏览
    
    XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),
                  configuration, mapperLocation.toString(), configuration.getSqlFragments());
              xmlMapperBuilder.parse();
    }
```
发现有我们熟悉的xml解析。这应该就是用来解析xml;
里面有个 XMLMapperBuilder 应该是解析我们熟知的mapper.xml

XMLMapperBuilder#parse
```java
    public void parse() {
        if (!this.configuration.isResourceLoaded(this.resource)) {
            this.configurationElement(this.parser.evalNode("/mapper"));
            this.configuration.addLoadedResource(this.resource);
            this.bindMapperForNamespace();
        }

        this.parsePendingResultMaps();
        this.parsePendingCacheRefs();
        this.parsePendingStatements();
    }
```
可见是对xml内容进行一步一步解析，这里不展开，有兴趣的可以自己去解读。