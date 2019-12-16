---
layout:     post
title:      docker基础命令使用
category: Docker
tags: docker
toc: true
date: 2018-09-26
description: 记录docker平常使用的命令
---

### 镜像和容器的生成
#### 镜像生成：

```
cd bus-eureka

mvn clean package -Dmaven.test.skip=true docker:build
```


#### 镜像查看：
```
docker images
```

#### 容器生成：

```
docker run [imageID]

docker run -p 9090:3306 --name mymysql [imageID]
```
-p 9090:3306：将容器的 3306 端口映射到主机的 9090 端口。

#### 后台生成容器：
```
docker run --name [containername] -d -p 80:80 [imagename]:[version]
```

#### 容器查看：
```
docker ps #是查看所有运行中的容器
docker ps -a  #是查看所有的容器
```

#### 启动指定容器：

```
docker start [containerID]
```

#### 停止指定容器：
```
docker stop [containerID]
```

#### 停止所有容器：

```
docker stop $(docker ps -a -q)
```


### 想要删除镜像，必须停止对应的容器

#### 删除指定容器：
```
docker rm [containerID]
```


#### 删除所有容器：
```
docker rm $(docker ps -a -q)
```

#### 删除指定镜像：
```
docker rmi [imageID]
```

#### 删除所有镜像：
```
docker rmi $(docker images -q)
```


#### 进入容器内部：
```
docker exec -it [containName] /bin/bash #这里是containername不是id
```


#### 查看容器内部ip:
```
cat /etc/hosts
```


#### 查看network列表
```
docker network ls
```


#### 查看network内部信息
```
docker network inspect [networkname]

```








