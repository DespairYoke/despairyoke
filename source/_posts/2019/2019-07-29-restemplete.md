---
layout: post
title: springboot中restTemplate的用法
category: springboot
tags: springboot
toc: true
---

post提交有 FormData和Payload  两种形式：
* 第一种是formdata形式，在header参数里可以直接看到
* payload则封装成json格式post过去，获取以后需要再解析成实体。

`注意`： restTemplate为http请求，不能请求本地局域网ip。

使用之前要注入bean
```java
@Bean
RestTemplate restTemplate() {
    return new RestTemplate();
}
```
## POST

### 发送表单请求
一般不需要设置消息头
#### 不设置消息头
```java
MultiValueMap<String, Object> formData = new LinkedMultiValueMap<String, Object>();
formData.add("id",123456);
formData.add("name","张三");
String result = restTemplate.postForObject("https://xxx.ccc.com",
        formData, String.class);
```
#### 设置消息头
```java
MultiValueMap<String, Object> formData = new LinkedMultiValueMap<String, Object>();

HttpHeaders headers = new HttpHeaders();
headers.set("Content-Type", "multipart/form-data");
headers.set("Accept", "text/plain");
formData.add("id",123456);
formData.add("name","张三");

HttpEntity<MultiValueMap<String, Object>> requestEntity = new HttpEntity<MultiValueMap<String, Object>>(formData, headers);
String result = restTemplate.postForObject("https://xxx.ccc.com",
        requestEntity, String.class);
```

#### 当需要传递文件
```java
MultiValueMap<String, Object> formData = new LinkedMultiValueMap<String, Object>();

HttpHeaders headers = new HttpHeaders();
headers.set("Content-Type", "multipart/form-data");
headers.set("Accept", "text/plain");

Iterator<String> itr = request.getFileNames();
MultipartFile file = request.getFile(itr.next());
formData.add("file",new ByteArrayResource(file.getBytes()));

HttpEntity<MultiValueMap<String, Object>> requestEntity = new HttpEntity<MultiValueMap<String, Object>>(formData, headers);
String result = restTemplate.postForObject("https://xxx.ccc.com",
        requestEntity, String.class);
```
### 发送json请求

#### 使用jsonObject
使用阿里巴巴的json包  com.alibaba.fastjson

* postForEntity
```java
JSONObject postData = new JSONObject();
postData.put("phone","134434344332");
JSONObject body = restTemplate.postForEntity("http://www.xxx.ooo.com", postData, JSONObject.class).getBody();
System.out.println(body);
return body.toJSONString();
```

* postForObject
```java
JSONObject postData = new JSONObject();
postData.put("phone","134434344332");
JSONObject body = restTemplate.postForObject("http://www.xxx.ooo.com", postData, JSONObject.class);
System.out.println(body);
return body.toJSONString();
```
#### 使用对象

* postForEntity
```java
User user = new User();
user.setUsername("141423433434");
String body = restTemplate.postForEntity("url",
        user,
        String.class).getBody();
System.out.println(body);
return body;
```

* postForObject
```java
User user = new User();
user.setUsername("141423433434");
String body = restTemplate.postForObject("url",
        user,
        String.class);
System.out.println(body);
return body;
```
## GET
### 普通url

* getForObject
```java
 String body = restTemplate.getForObject("url",
              String.class);
System.out.println(body);
return body;
```

* getForEntity
```java
 String body = restTemplate.getForEntity("url",
              String.class).getBody();
System.out.println(body);
return body;
```

### 动态url参数填充

* getForObject
```java
String body = restTemplate.getForObject("http://www.zzz.com/{xxx}/{ccc}",
      String.class,"zzz","cccc");
System.out.println(body);
return body;
```
* getForEntity
```java
String body = restTemplate.getForEntity("http://www.zzz.com/{xxx}/{ccc}",
      String.class,"zzz","cccc").getBody();
System.out.println(body);
return body;
```
