---
layout: post
title: spring事务传播属性
category: Spring源码阅读笔记
date: 2019-12-18
---


### 7种传播行为
```java
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);
}
```
* `REQUIRED`：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。大部分数据库默认为此传播行为。
* `SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
* `MANDATORY`：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
* `REQUIRES_NEW`：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
* `NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
* `NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。
* `NESTED`：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于REQUIRED。

### 示例演示
建立两个service 

- UserService
```java
@Service
public class UserService  {


    @Autowired
    private UserMapper userMapper;

    @Autowired
    private UserAccountService userAccountService;
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void insertUser() {

        User user = new User();
        user.setName("cccc");
        userMapper.insertUser(user);
        userAccountService.insertSelective();
        int i = 3 / 0;

    }
    
}
```

- UserAccountService

```java
@Service
public class UserAccountService {

    @Autowired
    private UserInfoMapper userInfoMapper;

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void insertSelective() {
        UserInfo userInfo = new UserInfo();
        userInfo.setPost("hhhhh");
        userInfoMapper.insertSelective(userInfo);
    }
}
```
当 UserService 使用 REQUIRED ，UserAccountService 使用 REQUIRES_NEW，UserService的异常，并不影响UserAccountService的事务。应用场景，如日志记录，并不
能因为逻辑错误，而失去了日志记录等。
但`UserAccountService` 抛出异常时，UserService 会回滚。
UserAccountService 使用 `NOT_SUPPORTED`时，发生异常时，父事务UserService不会回滚。可以理解成毫无关联。
UserAccountService 使用 `NESTED`时，无论逻辑是否正常，只要父事务UserService有事务存在，就会抛出异常。
