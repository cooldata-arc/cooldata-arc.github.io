---
layout: post
date:   2018-02-01 16:00:00
title:  "Spark On Yarn Pseudo Distributed"
categories: Spark
tags:  Spark Yarn Pseudo Distributed
excerpt: 新版本的Spark On Yarn伪分布搭建!
mathjax: true
---

* content
{:toc}

Spark2.0+版本已经出来很长时间了，较之前的版本有了较大的变化；由于生活琐事不断，一直没时间好好研究一番；分布式集群不利于调试研究，就以新版本的Spark On Yarn伪分布搭建开启研究之旅吧！！！






## 环境与材料

OS	|	Ubuntu16.04
Java	|	jdk1.8.0_162
Scala	|	scala-2.11.12
Hadoop	|	hadoop-2.7.5
Spark	|	spark-2.2.1-bin-hadoop2.7

## 准备工作
### 安装Java及Scala

安装Java及Scala，并配置好环境变量；基础操作就不细说了！

### 安装Hadoop的准备工作
* 创建用户

``` bash
$ sudo useradd -m hadoop -s /bin/bash # 创建Hadoop用户且用bash作为shell
$ sudo passwd hadoop # 设置密码
$ sudo adduser hadoop sudo # hadoop用户增加管理员权限
$ su - hadoop
```

* 设置SSH无密码登录

``` bash
$ cd ~/.ssh/
$ ssh-keygen -t rsa # 一路回车，成功生成秘钥
$ cat ./id_rsa.pub >> ./authorized_keys
$ ssh localhost # 正常情况这时就不需要密码了，如有问题Google好了
```

## 安装Hadoop

* 解压hadoop包到安装目录并授权

``` bash
$ sudo tar -zxvcf hadoop-2.7.5.tar.gz -C /deploy # 解压到想要安装的目录
$ cd /deploy
$ sudo ln -s hadoop-2.7.5 hadoop # 创建软连
$ sudo chown -R hadoop ./hadoop # 设置权限
```

* 设置环境变量

``` bash
$ vim ~/.bashrc # 将一下代码添加至末尾

export HADOOP_HOME=/deploy/hadoop
export CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath):$CLASSPATH
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HADOOP_COMMON_LIB_NATIVE_DIR

$ source ~/.bashrc # 使以上配置生效
```

* 将JDK路径添加至hadoop-env.sh

``` bash
$ cd /deploy/hadoop/etc/hadoop
$ vim hadoop-env.sh # 添加JDK路径

export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162
```

* 添加以下代码至core-site.xml

``` xml
<configuration>
    <property>
         <name>hadoop.tmp.dir</name>
         <value>file:/deploy/hadoop/tmp</value>
         <description>Abase for other temporary directories.</description>
    </property>
    <property>
         <name>fs.defaultFS</name>
         <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

* 添加以下代码至hdfs-site.xml 

``` xml
<configuration>
    <property>
         <name>dfs.replication</name>
         <value>1</value>
    </property>
    <property>
         <name>dfs.namenode.name.dir</name>
         <value>file:/deploy/hadoop/tmp/dfs/name</value>
    </property>
    <property>
         <name>dfs.datanode.data.dir</name>
         <value>file:/deploy/hadoop/tmp/dfs/data</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
</configuration>
```

* 添加以下代码至mapred-site.xml

``` xml
<configuration>
	<property>
	    <name>mapreduce.framework.name</name>
	    <value>yarn</value>
	</property>
	<property>
		<name>mapreduce.jobhistory.address </name>
		<value>localhost:10020</value>
	</property>
	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>localhost:19888</value>
	</property>
</configuration>
```

* 添加以下代码至yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
         <name>yarn.log.server.url</name>
         <value>http://localhost:19888/jobhistory/logs</value>
    </property>
    <property>
       <name>yarn.nodemanager.delete.debug-delay-sec</name>
       <value>1200</value>
    </property>
</configuration>
```
OK,配置部分已经修改完毕，接下来就可以格式化NameNode，启动了！！！

* NameNode格式化

``` bash
$ ./bin/hdfs namenode -format
```

* 启动Hadoop

``` bash
$ ./sbin/start-all.sh
$ jps # 如正常启动会出现一下进程
4049 DataNode
4274 SecondaryNameNode
4563 NodeManager
3926 NameNode
4439 ResourceManager
345 Jps
```

## 安装Spark
介个比Hadoop简单多了^_^

* 解压spark-2.2.1-bin-hadoop2.7.tar.gz到安装目录

``` bash
$ sudo tar -zxvf spark-2.2.1-bin-hadoop2.7.tar.gz -C /deploy
$ sudo ln -s spark-2.2.1-bin-hadoop2.7 spark221 # 创建软连，用软连的名称，比用那长串舒服多了
```

* 设置环境变量

``` bash
vim ~/.bashrc # 以下代码至末尾
export SPARK_HOME=/deploy/spark221
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
```

* 添加以下代码至spark-env.sh

``` bash
$ cd /deploy/spark221/conf
$ cp spark-env.sh.template spark-env.sh
$ vim spark-env.sh # 添加以下代码
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162
export SCALA_HOME=/deploy/scala-2.11.12
export SPARK_HOME=/deploy/spark221
export HADOOP_CONF_DIR=/deploy/hadoop/etc/hadoop
export SPARK_MASTER_HOST=localhost
export SPARK_LOCAL_IP=IP
```

* 添加以下代码至spark-defaults.conf

``` bash
$ vim spark-defaults.conf # 添加以下代码
spark.master=spark://localhost:7077
spark.eventLog.enabled=true
spark.eventLog.dir=hdfs://localhost:9000/spark/eventLogs
spark.history.fs.logDirectory=hdfs://localhost:9000/spark/eventLogs
spark.yarn.historyServer.address=localhost:18080
spark.history.ui.port=18080
```

** spark.eventLog.enabled  spark.eventLog.dir spark.history.fs.logDirectory spark.yarn.historyServer.address 这些配置决定了你是否可以通过Spark UI看到历史应用哦！！


* 配置slaves

``` bash
$ cp slaves.template slaves # 在这里拷贝一个就可以了，里面默认的就是localhost
```

* OK，启动Spark
``` bash
$ ./sbin/start-all.sh
$ ./sbin/start-history-server.sh
$ jps
29105 Master
29323 HistoryServer
29228 Worker
```


