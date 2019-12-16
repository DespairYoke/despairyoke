---
layout: post
title: linux命令学习汇总
category: linux
tags: linux
date: 2019-03-26
description: linux命令大全
---

## linux命令学习汇总

### 端口查看

* 查看某端口是否开通
```
lsof -i:3306
```

* 查看用户使用的所有端口
```
lsof -i|grep root
```

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

### 文件和目录查找

```
查找目录：find /（查找范围） -name '查找关键字' -type d
查找文件：find /（查找范围） -name 查找关键字 -print

```
### cat命令
```
查看文件所有内容（如果太长一般只显示后面一部分）  cat filename.txt

查看文件前100行  cat filename.txt | head -n 100

查看文件后50行   cat filename.txt | tail -n 50

从1000行开始显示，也就是显示1000行以后的   tail -n +1000

显示1000行到3000行内容   cat filename.txt |head -n 3000 | tail -n +1000
```