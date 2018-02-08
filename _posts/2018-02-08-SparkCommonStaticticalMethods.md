---
layout: post
date:   2018-02-08 11:00:00
title:  "Spark常用的统计方法"
categories: Spark
tags:  Spark 统计方法 
mathjax: true
---

* content
{:toc}

总结Spark基于scala API的常用统计方法，将持续更新。





* 找到出现次数最多的词

``` scala
textFile.map(line => line.split(" ").size).reduce((a,b) => if(a>b) a else b)
```