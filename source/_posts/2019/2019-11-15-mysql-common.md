---
layout: post
title: mysql基础命令
category: Other
tags: mysql
date: 2019-11-15
---

#### 显示不同引擎支持功能
show ENGINES;

#### 显示当前表使用的引擎
show create table `user`;

#### 修改表引擎
alter table user engine = innodb;

