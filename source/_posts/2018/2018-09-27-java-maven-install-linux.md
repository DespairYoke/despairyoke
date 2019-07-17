---
layout: post
title:  java-maven-install

catagory: maven
tags: maven
date: 2018-09-27
excerpt: 在linux上快速搭建java开发环境 java;maven;git;docker;docker-compose,redis,mysql,rabbitmq
---
## 必备环境安装

**执行脚本**

[脚本连接](https://github.com/despairyoke/despairyoke.github.io/blob/master/assets/files/2018/java_install.sh)

！如果执行时，脚本长时间暂停，可能是因为linux版本问题；这时可查看执行进度，手动执行脚本中的命令完成安装。

**配置path**

> Vim /etc/profile

新增行MAVEN_HOME,等于号后面是maven解压的文件夹地址

``` xml
export MAVEN_HOME=/usr/local/maven/apache-maven-3.5.4
```

找到PATH行,追加$MAVEN_HOME/bin<br/>
<strong>例如</strong>
```
PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH
```
## docker基本镜像安装

**mysql,redis,rabbitmq**

* docker pull mysql:5.5.60
* docker pull rabbitmq:3-management    
* docker pull redis:3.2.11

**使用docker-compose**

* step1: 新建一个名为docker-compose.yml的文件
* step2: 复制一下内容放入docker-compose.yml文件中

``` xml

version:  "2"

services:
  mysql:
    container_name: mysql
    image: mysql:5.5.60
    restart: always
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/conf/mysqld.conf:/etc/mysql/mysql.conf.d/mysqld.cnf
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=adminadmin
  redis:
    image: 'redis:3.2.11'
    restart: always
    hostname: redis
    container_name: redis
    ports:
      - '6379:6379'
    command: redis-server --requirepass adminadmin
  rabbitmq:
    image: rabbitmq:3-management
    restart: always
    container_name: rabbitmq
    hostname: rabbitmq
    environment:
      RABBITMQ_DEFAULT_USER: "springcloud"
      RABBITMQ_DEFAULT_PASS: "123456"
    ports:
      - "5672:5672"
      - "15672:15672"
```
* step3: 使用```docker-compose up -d``` 命令启动

如果一切正常的话，恭喜你所有环境搭建完成，密码可在上面代码中找到