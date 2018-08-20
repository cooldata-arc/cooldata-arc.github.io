---
layout: post
date:   2018-04-13 17:30:00
title:  "Git hot command"
categories: git
tags:  git gitlab
mathjax: true
---

* content
{:toc}
git常用命令备忘！






#### Git global setup

``` bash
git config --global user.name "xxx"
git config --global user.email "xxx@xxx.com"
```

#### Create a new repository

``` bash
git clone https://github.com/superzhangx/superzhangx.github.io.git

git add README.md
git commit -m "add README"
git push -u origin master
```

#### Existing folder

``` bash
cd existing_folder
git init
git remote add origin git clone https://github.com/superzhangx/superzhangx.github.io.git
git add .
git commit
git push -u origin master
```

#### Existing Git repository

``` bash
cd existing_repo
git remote add origin git clone https://github.com/superzhangx/superzhangx.github.io.git
git push -u origin --all
git push -u origin --tags
```

#### 冲突处理

``` bash
1.git merge --no-ff dev
2.打开文件挨个修改文件
3.git commit -a -m "merge"
4.git push origin master
```

#### CheckOut Branch

git clone 默认会把远程仓库整个给clone下来;但只会在本地默认创建一个master分支。如果最新的代码不在master分支上，该如何拿到呢？

``` bash
git branch -r # 查看远程分支或
git branch -a # 查看远程所有分支

# 用checkout命令来把远程分支取到本地
$ git checkout -t origin/daily/1.4.1

# 也可以使用fetch来做：
$ git fetch origin python_mail.skin:python_mail.skin

#不过通过fetch命令来建立的本地分支不是一个track branch，而且成功后不会自动切换到该分支上- z) 
注意：不要在本地采用如下方法：
$ git branch python_mail.skin
$ git checkout python_mail.skin
$ git pull origin python_mail.skin:python_mail.skin
#因为，这样建立的branch是以master为基础建立的，再pull下来的话，会和master的内容进行合并，有可能会发生冲突... 
```

#### tag

``` bash
## create tag
git tag -a v1.0.0 -m "message"

## delete tag
git tag -d v1.0.0

## show tags
git tag -l

## create new branch from tag
git branch newbranch v1.0.0
```

#### github仓库与原始仓库同步的两种方法

``` bash
## 1、添加原始仓库的路径，这里假设为https://github.com/upstream/master.git 
git remote add upstream https://github.com/upstream/master.git

## 2、commit（提交）本地的变更；如果本地没有修改内容，此步骤可忽略
git commit

## 3、更新远程仓库，从引用 fork 的原仓库地址同步内容，此时原仓库的 master（主干分支）已经可以在本地访问了。
git remote update upstream

## 4、checkout（检出）用于操作的本地分支 ，比如 master；如果此时分支为已检出状态，此步骤可忽略
git checkout master

## 5、直接从远程原始仓库的分支 pull（拉取） 数据
git pull upstream master
PS（第二种方法）：或者本地已检出分支基于远程仓库的分支进行 rebase（变基）操作
git rebase upstream/master

## 6、把本地已检出分支的已提交数据 push（推送） 到自己 fork 的仓库中
git push origin master
```

##### 解决冲突
在使用pull，push方法同步， 或者使用rebase，push同步的过程中，也许会出现冲突(conflict)。在这种情况，需要手动解决冲突。

比如rebase发生冲突时，Git会停止rebase并会让你去解决 冲突；在解决完冲突后，用”git-add”命令去更新这些内容的索引(index), 然后，你无需执行 git-commit,只要执行: 
`git rebase --continue`

这样git会继续应用(apply)余下的补丁。

在任何时候，你可以用–abort参数来终止rebase的行动，并且”mywork” 分支会回到rebase开始前的状态。
`git rebase --abort`

##### fetch，pull和rebase的区别

Git中从远程的分支获取最新的版本到本地有这样2个命令：

* git fetch：相当于是从远程获取最新版本到本地，不会自动merge。
``` bash
git fetch upstream
git checkout master
git merge upstream/master
```
上述命令相当于首先从远程的upstream原始仓库下载最新的版本到origin/master分支上， 然后合并到本地的master分支。 

* git pull：相当于是从远程获取最新版本并merge到本地。

``` bash
git pull upstream master 
```
上述命令其实相当于git fetch 和 git merge。

在实际使用中，git fetch更安全一些，因为在merge前，我们可以查看更新情况，然后再决定是否合并。因此可以用git fetch和git merge代替git pull.