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