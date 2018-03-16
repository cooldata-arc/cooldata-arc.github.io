---
layout: post
date:   2018-03-11 10:50:00
title:  "Hadoop problem summary"
categories: Hadoop
tags:  Hadoop
mathjax: true
---

* content
{:toc}
Hadoop 问题总结！！！






### 001-Hadoop任务提交权限问题

#### 问题描述
``` bash
18/03/16 10:37:33 WARN security.UserGroupInformation: PriviledgedActionException as:username (auth:SIMPLE) cause:org.apache.hadoop.security.AccessControlException: Permission denied: user=username, access=WRITE, inode="/user":hdfs:supergroup:drwxrwxr-x
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.checkAccessAcl(DefaultAuthorizationProvider.java:363)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.check(DefaultAuthorizationProvider.java:256)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.check(DefaultAuthorizationProvider.java:240)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.checkPermission(DefaultAuthorizationProvider.java:162)
	at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:152)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:3530)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:3513)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkAncestorAccess(FSDirectory.java:3495)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkAncestorAccess(FSNamesystem.java:6649)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirsInternal(FSNamesystem.java:4420)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirsInt(FSNamesystem.java:4390)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirs(FSNamesystem.java:4363)
	at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.mkdirs(NameNodeRpcServer.java:873)
```
#### 解决方案

在提交任务的用户下添加HADOOP_USER_NAME环境变量
``` bash
$ vim ~/.bash_profile
export HADOOP_USER_NAME=hdfs ## 添加至最下方

$ source ~/.bash_profile
```

#### 原理详解

Hadoop的用户鉴权是基于JAAS的，其中*hadoop.security.authentication*属性有simple和kerberos等方式。如果*hadoop.security.authentication*的值为*kerberos*，那么是“hadoop-user-kerberos”或者“hadoop-keytab-kerberos”，否则是“hadoop-simple”。 当用户登陆的时候，若org.apache.hadoop.security.User为空，那么说明尚未登录过，调用静态方法getLoginUser()创建org.apache.hadoop.security.UserGroupInformatio实例，在getLoginUser（）中又会调用HadoopLoginModule的login()和commit()方法。
在使用了kerberos的情况下，从javax.security.auth.kerberos.KerberosPrincipal的实例获取username。在没有使用kerberos时，首先读取hadoop 的系统环境变量，如果没有的话，对于windows 从com.sun.security.auth.NTUserPrincipal 获取用户名，对于类unix 从com.sun.security.auth.UnixPrincipal 中获得用户名，然后再看该用户属于哪个group，从而完成登陆认证。
基本理解了问题的根源，那么这个“org.apache.hadoop.security.AccessControlException:Permission denied: user=...”异常信息是怎么产生的呢？远程提交，如果没有hadoop 的系统环境变量，就会读取当前主机的用户名，结果Hadoop集群中没有该用户，就会出现以上错误。

### 02查看MapReduce输出.deflate格式的结果

``` bash
hadoop dfs -text /filename
```