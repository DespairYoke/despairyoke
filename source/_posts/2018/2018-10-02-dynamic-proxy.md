---
layout: post
title: java动态代理

category: Other
tags: [other]
date: 2018-10-02
---
``` java
public class LogicClassFir {
    /**
     * 逻辑方法A
     */
    public void logicMethodFir() {
        System.out.println("我是第一个逻辑方法的内容........");
    }
}


import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class ProxyCreator implements InvocationHandler {


    private Object proxy;

    private LogicClassFir logicObj;

    public ProxyCreator(Object proxy, LogicClassFir logicObj) {
        this.proxy = proxy;
        this.logicObj = logicObj;
    }


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        logicObj.logicMethodFir();
        return method.invoke(this.proxy,args);
    }
}

public interface ProxyService {
    /**
     * 测试方法
     */
    void testProxy();
}


public class ProxyServiceImpl implements ProxyService {
    /**
     * 测试方法
     */
    @Override
    public void testProxy() {
        System.out.println("我是ProxyService中的测试方法......");
    }
}


import java.lang.reflect.Proxy;

public class SpringApplication {

    public static void main(String[] args) {
        LogicClassFir logicClassFir = new LogicClassFir();
        ProxyService targetService = new ProxyServiceImpl();
        ProxyService proxyService = (ProxyService) Proxy.newProxyInstance(ProxyCreator.class.getClassLoader(),
                new Class[]{ProxyService.class},new ProxyCreator(targetService,logicClassFir));
    proxyService.testProxy();
    }
}
```

输入结果：
```
我是第一个逻辑方法的内容........
我是ProxyService中的测试方法......
```
[项目地址](https://github.com/despairyoke/spring-examples/tree/master/dynamic-proxy)
