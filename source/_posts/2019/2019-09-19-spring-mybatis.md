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

### Spring使用mybatis
在使用的过程中，我们创建的Mapper 都是接口，并没有实现类，而我们调用方法时，也没有报错，这很容易联想到再学习动态代理时，接口代理必须要用到JDKProxy。所以这里的Mapper 会生成一个代理对象去执行。
![](/image/spring-mybatis-01.png)
由图可知，userMapper与MapperProxy是有关联关系。（本人理解: 从编写过原始的mybatis示例中，见到过如下代码）
```java
 ClassPathXmlApplicationContext  ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

    SqlSession sqlSession = (SqlSession) ctx.getBean("sqlSession");


    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    User user = userMapper.selectUserById(1);
```
而sqlSession.getMapper(UserMapper.class);的代码如下
```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
if (mapperProxyFactory == null) {
  throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
}
try {
  return mapperProxyFactory.newInstance(sqlSession);
} catch (Exception e) {
  throw new BindingException("Error getting mapper instance. Cause: " + e, e);
}
}
```
mapperProxyFactory.newInstance(sqlSession);
```java
public T newInstance(SqlSession sqlSession) {
final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
return newInstance(mapperProxy);
}
```
所以我猜此可能是spring也应该是如此 debug一下 newInstance 看下调用栈
![](/image/spring-mybatis-02.png)

![](/image/spring-mybatis-03.png)

果然在创建Mapper的时候调用了ibatis的getMapper 而 MapperRegistry 已经放置好了xml加载后的集合。此时，只需要创建一个MapperProxy。
当我们请求时，MapperProxy的拦截方法会执行。
```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
try {
  if (Object.class.equals(method.getDeclaringClass())) {
    return method.invoke(this, args);
  } else if (isDefaultMethod(method)) {
    return invokeDefaultMethod(proxy, method, args);
  }
} catch (Throwable t) {
  throw ExceptionUtil.unwrapThrowable(t);
}
final MapperMethod mapperMethod = cachedMapperMethod(method);
return mapperMethod.execute(sqlSession, args);
}
```
根据不同的请求语句进行不同的拦截。
```java
  public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
      case INSERT: {
      Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.insert(command.getName(), param));
        break;
      }
      case UPDATE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.update(command.getName(), param));
        break;
      }
      case DELETE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.delete(command.getName(), param));
        break;
      }
      case SELECT:
        if (method.returnsVoid() && method.hasResultHandler()) {
          executeWithResultHandler(sqlSession, args);
          result = null;
        } else if (method.returnsMany()) {
          result = executeForMany(sqlSession, args);
        } else if (method.returnsMap()) {
          result = executeForMap(sqlSession, args);
        } else if (method.returnsCursor()) {
          result = executeForCursor(sqlSession, args);
        } else {
          Object param = method.convertArgsToSqlCommandParam(args);
          result = sqlSession.selectOne(command.getName(), param);
        }
        break;
      case FLUSH:
        result = sqlSession.flushStatements();
        break;
      default:
        throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
      throw new BindingException("Mapper method '" + command.getName() 
          + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
  }

```
当Service执行完成后，会执行提交或回滚操作。
```java
if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
    // Standard transaction demarcation with getTransaction and commit/rollback calls.
    TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);

    Object retVal;
    try {
        // This is an around advice: Invoke the next interceptor in the chain.
        // This will normally result in a target object being invoked.
        retVal = invocation.proceedWithInvocation();
    }
    catch (Throwable ex) {
        // target invocation exception
        completeTransactionAfterThrowing(txInfo, ex);
        throw ex;
    }
    finally {
        cleanupTransactionInfo(txInfo);
    }
    commitTransactionAfterReturning(txInfo);
    return retVal;
}
```
不知道为什么会执行到这里的可以看下
[spring事务处理流程](https://www.despairyoke.com/2019/09/17/2019/2019-09-17-spring-transaction/)

commitTransactionAfterReturning(txInfo);
```java
protected void commitTransactionAfterReturning(@Nullable TransactionInfo txInfo) {
    if (txInfo != null && txInfo.getTransactionStatus() != null) {
        if (logger.isTraceEnabled()) {
            logger.trace("Completing transaction for [" + txInfo.getJoinpointIdentification() + "]");
        }
        txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
    }
}
```
一系列调用栈后会执行
 ```java
  public void beforeCommit(boolean readOnly) {
          this.holder.getSqlSession().commit();
        
    }
```
这个sqlSeesion 没有配置，则取默认的 DefaultSqlSession
然后执行commit
```java
  @Override
  public void commit() {
    commit(false);
  }

  @Override
  public void commit(boolean force) {
    try {
      executor.commit(isCommitOrRollbackRequired(force));
      dirty = false;
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```
baseExector#commit
![](/image/spring-mybatis-04.png)

最终执行完Mybatis事务
```java
  public void commit() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Committing JDBC Connection [" + connection + "]");
      }
      connection.commit();
    }
  }
```