---
layout: post
title:  git基础命令记录
category: Other
tags: git
date: 2019-04-18
description: git基础命令记录
---

### .gitignore不起作用
```xml
    git rm -r --cached .
    git add .
    git commit -m 'update .gitignore'
```

### tag
#### tag创建
- git tag v1.0
- git tag -a v0.1 -m "version 0.1 released"

### tag提交

- git push origin v1.0   提交具体的tag
- git push origin --tags   提交所有的tag
    
### 删除远程tag
```xml
git push origin --delete tag <tagname>
```

### 撤销指令
当我们编写的内容发现有问题，这时我们想回到原来的内容，就可以使用此命令撤回。
```xml
git checkout <file>
```
注意事项：指令该是先从缓存区中拉取版本还原，如果没有再到版本库中拉取还原。
缓存区：git add 
版本库：git commit 

### git提交到远程仓库
```xml
git remote add origin 你的远程库地址  // 把本地库与远程库关联

git push -u origin master    // 第一次推送时

git push origin master  // 第一次推送后，直接使用该命令即可推送修改
```

### GitHub Pages
GitHub Pages 的静态资源支持下面 3 个来源：

`master` 分支
`master` 分支的 `/docs` 目录
`gh-pages` 分支
执行下面命令，将 _book 目录推送到 GitHub 仓库的 gh-pages 分支。
```xml
git subtree push --prefix=_book origin gh-pages
```