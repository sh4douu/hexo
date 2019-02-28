---
title: Git基本命令及使用
date: 2019-02-22 14:04:26
tags: 必会技能
categories: Misc
---

## 0x00 Git概述

*Git*是一个开源的分布式版本控制系统,用于敏捷高效地处理任何或小或大的项目。

<!-- more -->

## 0x01 Git安装

到官网下载安装程序安装即可：https://git-scm.com/

## 0x02 基本操作

> **`git   init`  **     #初始化当前目录，表示对当前目录进行版本控制。该步骤是对某一目录进行版本控制的第一步。

> **`git status`**       #查看当前目录文件状态。（红色文件表示未加入版本控制，绿色表示已加入但未提交版本）
>
> - PS：对文件进行修改变动后，git会自动发现并变为红色，此时又需要git add变为绿色

> **`git add  <filename> `**    #将指定文件加入版本控制。（加入后git status查看红色变为绿色）
> **`git add  .  `**          #将当前目录下的所有文件加入版本控制。

> **`git commit -m`** "详细描述信息"       #提交一个版本。即生成一次版本快照。版本号是随意生成的MD5值。

> **`git log `**         #版本创建/变更日志。
> **`git reflog`**     #同上，但更详细。
>
> > 可通过这两条命令看到变更的记录，更直白一点就是可以获取版本号用于变更版本。

> **`git reset --hard <版本号>` **    #切换状态到指定的版本（快照）

> **`git branch`  ** #查看分支列表。一个项目可以用众多分支，分支之间拷贝的，即互相独立不影响。默认是master
>
> **`git branch <分支名称>`**         #创建一个新分支。（创建的分支是当前所在分支的拷贝，本质是两份独立的拷贝）
> **`git branch -d <分支名称>`**     #删除指定分支
>
> **`git checkout <分支名称>`**     #进入指定分支
>
> **`git merge <分支名称>`**          #将指定分支与当前所在分支进行合并。

## 0x03 连接Github

> **`git remote add  <github仓库地址别名>    <github仓库地址>`**        
>
> > 别名是为了避免记网址，对应关系会被记录在.git目录的config文件中。
>
>
>
> **`git clone  <github项目网址>`**      #克隆下载项目
>
> **`git pull <github网址别名>  <分支名称>`**    #从github拉取指定分支代码。
>
> > 上面条等价于下面两条：
> > ​		**`git fetch   <github网址别名>    <分支名称> `**
> > ​		**`git merge   <github网址别名>   <分支名称>`**
>
> **`git push <github网址别名>   <分支名称>`**  #将当前所在分支推到github上。

## 0x04 将某些文件隔离出版本控制

- 在工作目录下创建**.gitignore**文件
- 文件的编写格式内容

```
*.sql    #表示所有后缀为.sql的文件都被隔离
*.pyc	 #表示所有后缀为.pyc的文件都被隔离
a.txt	 #表示隔离a.txt
```

#### 教程博客与参考链接

- https://www.cnblogs.com/wupeiqi/p/7295372.html
- http://www.ruanyifeng.com/blog/2018/10/git-internals.html
- http://www.ruanyifeng.com/blog/2015/12/git-workflow.html
- http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html