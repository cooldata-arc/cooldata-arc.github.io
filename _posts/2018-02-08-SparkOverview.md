---
layout: post
date:   2018-02-08 11:00:00
title:  "Spark Overview"
categories: Spark
tags:  Spark Overview 概述
mathjax: true
---

* content
{:toc}

Spark概述！本文使用与Spark2.x.x版本！





## Spark概述

Apache Spark是一种快速、通用的集群计算系统。它提供了Java、Scala、Python和R的高级API，以及一个支持通用执行图的优化引擎。它还支持丰富的高级工具集，包括用于SQL和结构化数据处理的Spark SQL、用于机器学习的MLlib、用于图像处理的GraphX和Spark Streaming。

> Apache Spark is a fast and general-purpose cluster computing system. It provides high-level APIs in Java, Scala, Python and R, and an optimized engine that supports general execution graphs. It also supports a rich set of higher-level tools including Spark SQL for SQL and structured data processing, MLlib for machine learning, GraphX for graph processing, and Spark Streaming.

## Spark功能架构

![Spark功能架构图](https://superzhangx.github.io/images/spark/20180208-spark-FunctionalArchitecture.png)

## Spark运行在集群上的方式

* Standalone

* Spark On Mesos
	Mesos是一款Apache开源的一个通用的集群管理器，它可以运行Hadoop MapReduce和服务应用程序。
* Spark On YARN
	YARN是Hadoop2中的资源管理器(ResourceManager)
* Spark On Kubernetes
	Spark2.x.x提供的一个实验支持。Kubernetes是一个提供容器为中心的基础设施的开源平台。Kubernetes的支持在一个apache-spark-on-k8s 的Github组织中得到积极开发。

## Spark编程接口
在Spark2.0之前，Spark主要的编程接口是弹性分布式数据集(RDD)。在Spark2.0之后，RDD被Dataset取代，数据集是强类型的，较RDD有更丰富的优化。RDD接口任然支持。但官方强烈建议切换到Dataset，它性能比RDD好。