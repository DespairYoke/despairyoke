---
layout: post
title: springboot2.0实战中如何使用事件发布？
category: springboot
tags: springboot
date: 2019-08-14
---

示例：模拟商家订单。当商家接收到下单请求时，触发订单事件，进行订单派送。
### 定义事件监听器ApplicationListener
```java
public class SendListener implements ApplicationListener<OrderEvent> {


	public void onApplicationEvent(OrderEvent event) {
		Object source = event.getSource();
		System.out.println("收到订单号"+ source +"，立即派送");
	}
}
```

### 定义事件ApplicationEvent
```java
public class OrderEvent extends ApplicationEvent {

	public OrderEvent(Object source) {
		super(source);
	}
}
```
注意：`super(source)`一定要加。

### 编写controller层模拟下单
```java
@RestController
public class HelloController {

	@Autowired
	private ApplicationEventPublisher eventPublisher;
	
	
	@GetMapping(value = "/")
	public void index() {
		eventPublisher.publishEvent(new OrderEvent("34324234234"));
	}
}

```

### 编写启动类
```java
@SpringBootApplication
public class BootStrap {

	public static void main(String[] args)   {
		SpringApplication application = new SpringApplication(BootStrap.class);
		application.addListeners(new SendListener());
		application.run(args);

	}
}

```

请求接口，查看控制台输出
```
2019-08-14 18:45:19.491  INFO 1131 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-08-14 18:45:19.747  INFO 1131 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-08-14 18:45:19.762  INFO 1131 --- [           main] com.zwd.BootStrap                        : Started BootStrap in 7.309 seconds (JVM running for 9.329)
2019-08-14 18:46:08.803  INFO 1131 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2019-08-14 18:46:08.803  INFO 1131 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2019-08-14 18:46:08.876  INFO 1131 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 73 ms
收到订单号34324234234，立即派送
收到订单号34324234234，立即派送
```