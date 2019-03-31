---
title: git修改分支名称
date: 2019-03-31 13:05:50
tags: git
categories: Git
---
### 情景
在使用`git`命令操作仓库代码时，建议大家不要心急，敲完命令后先别急着敲回车(虽然命令+回车连着敲起来很爽)，检查一下命令是否正确，名称（分支）是否是正确的单词，待检查完没有问题，便可以狠狠的砸下`Enter`键。

如果检查不够仔细，创建的分支名称不想要，当然也有对应的修改方法，下面将针对两种情况分别作出处理：
  * 分支只在本地，没有推送到远程
  * 分支已经推送到远程

假设分支名称为`oldName`，想要修改为 `newName`

#### 本地分支重命名(还没有推送到远程)

```
 // -m 重命名分支
git branch -m oldName newName
```

#### 远程分支重命名
> 已经推送远程-假设本地分支和远程对应分支名称相同
-  重命名远程分支对应的本地分支
 ```
 git branch -m oldName newName
 ```
 - 删除远程分支
 ```
 git push --delete origin oldName
 ```
 - 上传新命名的本地分支
 ```
 git push origin newName
 ```
 - 把修改后的本地分支与远程分支关联
 ```
git branch --set-upstream-to origin/newName
 ```