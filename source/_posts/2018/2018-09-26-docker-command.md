---
layout:     post
title:      docker基础命令使用
category: Docker
tags: docker
date: 2018-09-26
description: 记录docker平常使用的命令
---

## 容器

```
    docker run --name "contain_name" -d  #启动容器
    
    docker run -d -p 5000:5000 -v ./mydirctory:/var/registry container_name:version 
    
    -d #后台启动
    
    -p 端口映射
    
    -v 文件夹映射
    
    docker stop containerid #停止容器
    
    docker rm containerid  #移除容器
    
    docker stop $(docker ps -a -q) #停止所有容器
    
    docker rm $(docker ps -a -q) #移除所有容器
```

## 镜像

```     
    docker rmi imageid  #移除镜像
    
    docker rmi $(docker images -a) #移除所有镜像
    
```


