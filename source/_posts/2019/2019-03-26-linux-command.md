---
layout: post
title: linux命令学习汇总
category: linux
tags: linux
date: 2019-03-26
description: linux命令大全
---

## linux命令学习汇总

### scp命令

* 1、从本地复制到远程
```
scp local_file remote_username@remote_ip:remote_folder 
或者 
scp local_file remote_username@remote_ip:remote_file 
或者 
scp local_file remote_ip:remote_folder 
或者 
scp local_file remote_ip:remote_file
```

* 2.复制目录命令格式
```
scp -r local_folder remote_username@remote_ip:remote_folder 
或者 
scp -r local_folder remote_ip:remote_folder 
```

* 3、从远程复制到本地
```java
scp root@www.runoob.com:/home/root/others/music /home/space/music/1.mp3 
scp -r www.runoob.com:/home/root/others/ /home/space/music/
```
