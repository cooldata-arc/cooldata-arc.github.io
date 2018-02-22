---
layout: post
date:   2018-02-10 22:00:00
title:  "Spark RDDs"
categories: Spark
tags:  Spark RDD 弹性分布式数据集
mathjax: true
---

* content
{:toc}

弹性分布式数据集(RDD)在Spark中的作用至关重要，可以说在Spark2.0之前一切都围绕RDD概念展开,它是一种可以并行操作的容集合。创建RDDs有两种方法:在驱动程序中并行化现有的集合，或者在外部存储系统中引用数据集，比如共享文件系统、HDFS、HBase或任何提供Hadoop InputFormat的数据源。






下面引用一段官网的描述：
>Spark revolves around the concept of a resilient distributed dataset (RDD), which is a fault-tolerant collection of elements that can be operated on in parallel. There are two ways to create RDDs: parallelizing an existing collection in your driver program, or referencing a dataset in an external storage system, such as a shared filesystem, HDFS, HBase, or any data source offering a Hadoop InputFormat.

## 创建RDDs的俩种方法

### 并行化集合(Parallelized Collection)

并行化集合是通过在驱动程序(a Scala Seq)中的现有集合中调用SparkContext的并行化方法创建的。集合元素被复制以形成可以并行操的分布式数据集。

Exmaple:

``` scala
val data = Array(1,2,3,4,5)
val distData = sc.parallelize(data)
```

并行集合的一个重要参数是将数据集分割成的分区数。Spark将为集群的每个分区运行一个任务。通常，在集群中每个CPU需要2-4个分区。通常，Spark尝试根据集群自动设置分区数量。但是，您也可以手动设置它，将其作为第二个参数传递给并行化(例如，sc.parallelize(数据，10))。

### 外部数据集(External Datasets)

Spark可以从Hadoop支持的任何存储源创建分布式数据集，包括本地文件系统、HDFS、Cassandra、HBase、Amazon S3等。Spark支持文本文件、SequenceFiles和任何其他Hadoop InputFormat。

可以使用SparkContext的textFile方法创建文本文件RDDs。该方法为文件取一个URI(在机器上的本地路径，或hdfs://， s3n://， etc URI)，并将其读取为行集合。下面是一个示例调用:

```
scala> val distFile = sc.textFile("data.txt")
distFile: org.apache.spark.rdd.RDD[String] = data.txt MapPartitionsRDD[10] at textFile at <console>:26
```

* textFile方法还采用可选的第二个参数来控制文件的分区数量。默认情况下，Spark为文件的每个块创建一个分区(默认情况下是在HDFS中为128MB)，但是您也可以通过传递一个较大的值来请求更多的分区。请注意，不能有比块更少的分区。
* SparkContext.wholeTextFiles允许您读取包含多个小文本文件的目录，并将其中的每个文件作为(文件名、内容)对返回。这与textFile形成对比，textFile将在每个文件中返回一行记录。分区是由数据位置决定的，在某些情况下，可能导致分区太少。对于这些情况，wholeTextFiles提供了一个可选的第二个参数，用于控制最小数量的分区。（scala API提供)
* 对于SequenceFiles，使用SparkContext的sequenceFile[K, V]方法，其中K和V是文件中键和值的类型。这些应该是Hadoop的Writable接口的子类，比如IntWritable和Text。此外，Spark允许您为一些常见的可写程序指定本机类型;例如，sequenceFile[Int, String]将自动读取IntWritables和文本。(Scala API 提供)
* 对于其他Hadoop inputformat，您可以使用SparkContext。hadoop方法，它采用任意的JobConf和输入格式类、键类和值类。将这些设置与使用输入源的Hadoop作业相同。您还可以使用SparkContext。newAPIHadoopRDD InputFormats基于“新”MapReduce的API(org.apache.hadoop.mapreduce)。
* RDD.saveAsObjectFile 和 SparkContext.objectFile支持以一种简单的格式(由序列化的Java对象组成)来保存RDD。虽然这并不像Avro那样高效，但它提供了一种简单的方法来保存任何RDD。

## RDD的操作

RDDs支持以下俩种类型的操作：
* transformations
转换，从现有数据集创建新的数据集。例如：map是一个转换，它通过一个函数传递每个元素集元素，并返回一个表示结果的新RDD。
* actions
在数据集上运行计算后返回值给驱动程序操作。例如：reduce是一种操作，它使用一些函数聚合RDD的所有元素，并将最总结果返回给驱动程序。

Spark中的所有转换都是惰性的，因为他们不会立即计算结果；它们只记录应用于基本数据集的转换。只有当操作需要返回到驱动程序时才计算转换。
默认情况下，每次您对它执行操作时，每个转换的RDD都可以重新计算。但是，您也可以使用*persist(或缓存)方法在内存中持久化一个RDD*，在这种情况下，Spark将在您下次查询时使集群周围的元素获得更快的访问。还支持在磁盘上持久化RDDs，或者在多个节点上进行复制。

