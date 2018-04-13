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

git config --global user.name "xxx"
git config --global user.email "xxx@xxx.com"

#### Create a new repository

git clone https://github.com/superzhangx/superzhangx.github.io.git

git add README.md
git commit -m "add README"
git push -u origin master

#### Existing folder

cd existing_folder
git init
git remote add origin git clone https://github.com/superzhangx/superzhangx.github.io.git
git add .
git commit
git push -u origin master

#### Existing Git repository

cd existing_repo
git remote add origin git clone https://github.com/superzhangx/superzhangx.github.io.git
git push -u origin --all
git push -u origin --tags


#### 冲突处理

1.git merge --no-ff dev
2.打开文件挨个修改文件
3.git commit -a -m "merge"
4.git push origin master


11111 test