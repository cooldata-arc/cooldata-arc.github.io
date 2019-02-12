---
layout: post
date:   2019-02-12 18:04:00
title:  "MacOS中Homebrew管理Python多版本环境"
categories: Python
tags:  homebrew python
mathjax: true
---

* content
{:toc}
MacOS 中利用Homebrew管理多版本Python环境。







#### 安装Homebrew

* 官网地址: http://brew.sh/index_zh-cn.html

``` bash
# 执行以下命令安装
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# 查看版本以检查是否安装成功
brew -v
```

#### 安装pyenv

``` bash
# 运行以下脚本安装pyenv
brew install pyenv

# 执行以下脚本检查是否安装成功
pyenv -v
```

#### 查看可安装的Python版本

``` bash
pyenv install --list
```

#### 安装特定版本的Python

``` bash
pyenv install anaconda3-2018.12
```

#### 查看pyenv已安装的Python版本

``` bash
pyenv versions
```

#### 修改.bash_profile文件切换环境

``` bash
# 编辑.bash_profile文件加入以下内容
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi

# 使用source命令使其生效
source .bash_profile
```

#### 指定目录切换Python版本

``` bash
# 进入python安装目录
cd /Users/zhangx/.pyenv/versions/anaconda3-2018.12

# 切换
pyenv local anaconda3-2018.12
```

#### 设置全局Python版本（慎用）

``` bash
pyenv global <versions>
```

#### Pycharm选择pyenv安装的Python版本

Pycharm -> Preferences -> Project -> Project Interpreter -> Add Local

选择~/.pyenv/versions/下的Python即可。
