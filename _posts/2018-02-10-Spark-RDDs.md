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

### 基础操作

``` scala
val lines = sc.textFile("data.txt")
val lineLengths = lines.map(s => s.length)
val totalLength = lineLengths.reduce((a, b) => a + b)
```

第一行从外部文件定义了一个基本RDD。该数据集没有加载到内存中，或者在其他情况下执行:行仅仅是指向该文件的指针。第二行定义了lineLengths作为映射转换的结果。同样，由于懒惰，lineLengths并没有立即得到计算。最后，我们运行reduce，这是一个操作。在这一点上，Spark将计算分解为在独立的机器上运行的任务，并且每台机器都运行map的一部分和局部还原，只返回其对驱动程序的响应。

如果我们以后还想使用lineLengths，我们可以加上：

``` scala
lineLengths.persist()
```

在reduce之前，这将导致lineLengths在第一次计算之后保持到内存中。

### 闭包

Spark的难点之一是了解在集群中执行代码时变量和方法的范围和生命周期。在其范围之外修改变量的RDD操作可能是混淆的常见来源。在下面的示例中，我们将查看使用foreach()来增加计数器的代码，但是类似的问题也可能发生在其他操作中。

考虑下面的简单的RDD元素求和，它可能会有不同的表现，这取决于执行是否在同一个JVM中发生。一个常见的例子是在本地模式中运行Spark (--master = local[n])，而不是将Spark应用程序部署到集群(例如，通过Spark -submit to YARN):

``` scala
var counter = 0
var rdd = sc.parallelize(data)

// Wrong: Don't do this!!
rdd.foreach(x => counter += x)

println("Counter value: " + counter)
```

#### Local vs. cluster模式

上述代码的行为是未定义的，可能不像预期的那样工作。为了执行作业，Spark将RDD操作的处理分解为任务，每个任务由执行器执行。在执行之前，Spark计算任务的闭包。闭包是执行器在RDD上执行其计算时必须可见的变量和方法(在本例中foreach())。这个闭包被序列化并发送给每个执行器。

闭包发送给每个executor的变量现在是副本，因此，当在foreach函数中引用计数器时，它不再是驱动节点上的计数器。在驱动节点的内存中仍然有一个计数器，但是这对执行器来说已经不可见了!执行程序只从序列化的闭包中看到副本。因此，计数器的最终值仍然为零，因为计数器上的所有操作都引用了序列化闭包中的值。

在本地模式中，在某些情况下，foreach函数实际上会在同一JVM中执行驱动程序，并引用相同的原始计数器，并可能实际更新它。

为了确保在这些场景中定义良好的行为，应该使用累加器。Spark中的累加器专门用于提供一种机制，用于在集群中跨工作节点的执行时安全地更新一个变量。

一般来说，闭包——像循环或局部定义的方法——不应该被用来改变某些全局状态。Spark没有定义或保证从闭包外部引用的对象的突变行为。有些代码可以在本地模式下工作，但这只是偶然，这种代码在分布式模式下不会像预期那样运行。如果需要一些全局聚合，则使用累加器。

#### 打印RDD的元素

另一个常见的用法是尝试用rdd.foreach(println)或rdd.map(println)来打印RDD的元素。在一台机器上，这将生成预期的输出并打印所有RDD的元素。然而，在集群模式中，执行程序调用stdout的输出现在写入执行者的stdout，而不是驱动程序中的stdout，所以stdout在驱动程序上不会显示这些!要在驱动程序上打印所有元素，可以使用collect()方法首先将RDD带到驱动节点:rdd.collect().foreach(println)。这可能导致驱动程序耗尽内存，因为collect()将整个RDD提取到一台机器;如果您只需要打印RDD的一些元素，那么使用take(): rdd.take(100).foreach(println)就更安全了。

### 使用键值对

虽然大多数Spark操作都在包含任何类型对象的RDDs上工作，但是一些特殊操作只在键值对的RDDs上可用。最常见的是分布式的“shuffle”操作，例如按一个键分组或聚合元素。

在Scala中，这些操作可以在包含Tuple2对象的RDDs上自动获得(通过简单的编写(a, b)创建的语言中的内置元组)。键值对操作可在PairRDDFunctions类中使用，它可以自动地封装元组的RDD。

例如，下面的代码使用键-值对上的reduceByKey操作来计算文件中每一行文本的数量。

``` scala
val lines = sc.textFile("data.txt")
val pairs = lines.map(s => (s, 1))
val counts = pairs.reduceByKey((a, b) => a + b)
```

我们还可以使用counts.sortByKey()来按字母顺序对它们进行排序，最后是counts.collect()将它们作为对象数组返回给驱动程序。

注意:在键-值对操作中使用自定义对象时，必须确保自定义equals()方法附带了一个匹配的hashCode()方法。有关详细信息，请参见Object.hashCode()文档中概述的契约。

### Transformations

下表列出了Spark支持的一些常见转换。

Transformations |   描述
map(func)           |   返回一个新的分布式数据集，该数据集通过函数func传递源的每个元素。


filter(func)        |   返回一个新的数据集，它选择了func返回true的源元素。
flatMap(func)       |   类似于map，但是每个输入项都可以映射到0或更多的输出项(因此func应该返回一个Seq而不是单个项目)。
mapPartitions(func) |   类似于map，但是在RDD的每个分区(块)上分别运行，所以func必须在类型为T的RDD上运行时，Iterator<T> => Iterator<U> 。
mapPartitionsWithIndex(func)    |   类似于mapPartitions，但也提供了表示分区索引的func，因此func必须是类型 (Int, Iterator<T>) => Iterator<U>,当运行在T类型的RDD上时。
sample(withReplacement, fraction, seed) |   使用给定的随机数生成器种子，使用给定的随机数生成器种子，对数据的一小部分进行或不进行替换。
union(otherDataset) |   返回一个新的数据集，该数据集包含源数据集和参数中的元素的联合。
intersection(otherDataset)  |   返回一个新的RDD，它包含源数据集中元素和参数的交集。
distinct([numTasks]))   |   返回包含源数据集的不同元素的新数据集。
groupByKey([numTasks])  |   当调用(K, V)对的数据集时，返回一个数据集(K, Iterable)对。
注意:如果要对每个键执行聚合(比如求和或平均值)，使用reduceByKey或aggregateByKey会获得更好的性能。
注意:默认情况下，输出的并行度取决于父RDD分区的数量。您可以通过一个可选的numTasks参数来设置不同数量的任务。
reduceByKey(func, [numTasks])   |   当调用(K、V)的数据集对,返回一个数据集(K、V)对每个键的值在哪里聚合使用给定减少函数func,必须(V,V)= > V型像groupByKey,减少任务的数量通过一个可选的第二个参数是可配置的。
aggregateByKey(zeroValue)(seqOp, combOp, [numTasks])    |   当调用(K, V)对的数据集时，返回一个(K, U)对的数据集，其中每个键的值都使用给定的组合函数和一个中立的“零”值进行聚合。允许与输入值类型不同的聚合值类型，同时避免不必要的分配。和groupByKey一样，通过一个可选的第二个参数可配置减少任务的数量。
sortByKey([ascending], [numTasks])  |   当调用(K, V)的数据集时，K实现了排序，返回一个(K, V)的数据集，按照布尔提升参数中指定的升序或降序排序。
join(otherDataset, [numTasks])  |   当调用类型(K, V)和(K, W)的数据集时，返回一个(K， (V, W))对每个键的所有对元素的数据集。外部连接通过leftOuterJoin、右端连接和fullOuterJoin来支持。
cogroup(otherDataset, [numTasks])   |   当调用类型(K, V)和(K, W)的数据集时，返回一个数据集(K， (Iterable， Iterable))元组。这个操作也称为groupWith。
cartesian(otherDataset) |   当调用类型T和U的数据集时，返回(T, U)对的数据集(所有对元素)。
pipe(command, [envVars])    |   通过shell命令(例如:Perl或bash脚本)将RDD的每个分区连接起来。RDD元素被写入到进程的stdin中，输出到它的stdout的行被作为字符串的RDD返回。
coalesce(numPartitions) |   将RDD中的分区数量减少到num分区。在过滤大数据集之后更有效地运行操作。
repartition(numPartitions)  |   在RDD中随机地重新调整数据，以创建更多或更少的分区，并在它们之间进行平衡。这总是会打乱网络上的所有数据。
repartitionAndSortWithinPartitions(partitioner) |   根据给定的分区器重新划分RDD，并在每个结果分区中按其键对记录进行排序。这比调用repartition并在每个分区中进行排序更有效，因为它可以将排序分解到shuffle机器中。


### Actions

下表列出了Spark支持的一些常见操作。

Actions |   描述
reduce(func)    |   使用函数func聚合数据集的元素(它接受两个参数并返回一个参数)。函数应该是可交换的和关联的，这样可以并行计算。
collect()   |   将数据集的所有元素作为一个数组返回到驱动程序。在过滤器或其他操作返回足够小的数据子集之后，这通常是有用的。
count() |   返回数据集中的元素个数。
first() |   返回数据集的第一个元素(类似于take(1))。
take(n) |   返回数据集的前n个元素的数组。
takeSample(withReplacement, num, [seed])    |   返回一个包含数据集的num元素的随机样本的数组，有或没有替换，可选地预先指定一个随机数生成器种子。
takeOrdered(n, [ordering])  |   使用它们的自然顺序或自定义比较器返回RDD的第一个n个元素。
saveAsTextFile(path)    |   将数据集的元素作为文本文件(或文本文件集)写入本地文件系统、HDFS或任何其他hadoop支持的文件系统中。Spark将在每个元素上调用toString，将其转换为文件中的一行文本。
saveAsSequenceFile(path) 
(Java and Scala)    |   在本地文件系统、HDFS或任何其他Hadoop支持的文件系统中，将数据集的元素写入到给定路径中的Hadoop SequenceFile中。这可以在实现Hadoop可写界面的键值对的RDDs上获得。在Scala中，它也可用于隐式可转换的类型(Spark包括用于基本类型的转换，例如Int、Double、String等)。
saveAsObjectFile(path) 
(Java and Scala)    |   使用Java序列化以简单的格式编写数据集的元素，然后使用SparkContext.objectFile()加载。
countByKey()    |   只有在类型(K, V)类型的RDDs上才可用。返回一个(K, Int)对的hashmap (K, Int)对每个键的计数。
foreach(func)   |   在数据集的每个元素上运行函数func。这通常用于诸如更新累加器或与外部存储系统交互等副作用。

注意:除了在foreach()之外的累加器之外修改变量可能会导致未定义的行为。有关更多细节，请参见了解闭包。

Spark RDD API还公开了一些操作的异步版本，例如foreach的foreachAsync，它会立即向调用者返回一个futuresponse，而不是阻塞完成操作。这可以用于管理或等待操作的异步执行。

### Shuffle操作

Spark中的某些操作会触发一个名为shuffle的事件。shuffle是Spark的重新分配数据的机制，以便在不同的分区之间进行不同的分组。这通常涉及在执行器和机器之间复制数据，从而使操作变得复杂且代价高昂。

#### 背景

为了了解在洗牌过程中发生了什么，我们可以考虑简化操作的例子。reduceByKey操作生成一个新的RDD，其中一个键的所有值都被合并到一个元组中——关键和执行reduce函数的结果与该键关联的所有值。挑战在于，对于单个键的所有值不一定都驻留在同一个分区上，甚至是同一台机器上，但它们必须是共存的，以计算结果。

在Spark中，数据一般不会跨分区分布到特定操作的必要位置。在计算过程中，单个任务将在单个分区上操作——因此，为了组织所有的数据，以减少单个的减少任务的执行，Spark需要执行一个全面的操作。它必须从所有分区读取所有键的所有值，然后将各个分区的值集合起来，以计算每个键的最终结果——这称为shuffle。

虽然新打乱的数据的每个分区中的元素集是确定的，但是分区本身的排序也是确定的，但是这些元素的顺序是不确定的。如果一个人想要在洗牌之后得到可以预见的有序数据，那么就有可能使用：

* 要对mapPartitions的每个分区进行排序 ,例如： .sorted
* repartitionAndSortWithinPartitions 对有效分区进行重新分区的同时，对分区进行排序。
* sortBy 产生一个全局排序的RDD

可以引起混乱的操作包括重新划分操作，比如重新分区和合并，“ByKey操作(除了计数)，比如groupByKey和reduceByKey，并加入cogroup和join之类的操作。

#### 性能影响

Shuffle是一个昂贵的操作，因为它涉及磁盘I/O、数据序列化和网络I/O。为了组织shuffle的数据，Spark生成任务集——map tasks来组织数据，以及一组reduce tasks来聚合数据。这种命名法来自MapReduce，它与Spark的map和reduce操作没有直接关系。

在内部，单个map task的结果被保存在内存中，直到它们不适合。然后，根据目标分区对它们进行排序，并将它们写入单个文件。在reduce方面，任务读取相关的排序块。

某些shuffle操作可以消耗大量的堆内存，因为它们使用内存中的数据结构来组织传输之前或之后的记录。具体来说，reduceByKey和aggregateByKey在map上创建这些结构，而“ByKey操作”在reduce方面生成这些结构。当数据不适合内存时，Spark会将这些表序列化到磁盘，从而导致磁盘I/O的额外开销和增加的垃圾收集。

Shuffle还会在磁盘上生成大量的中间文件。在Spark 1.3中，这些文件被保存，直到相应的RDDs不再被使用并被垃圾收集。这是这样做的，如果重新计算了沿袭，就不需要重新创建shuffle文件。垃圾收集可能在很长一段时间之后才会发生，如果应用程序保留对这些RDDs的引用，或者GC不经常启动。这意味着长时间运行的Spark作业可能消耗大量磁盘空间。临时存储目录由spark.local指定。在配置Spark上下文时使用dir配置参数。

通过调整各种配置参数可以调整Shuffle行为。在Spark配置指南中看到“Shuffle行为”部分。

### RDD 的持久性

Spark最重要的功能之一是在内存中持久化(或缓存)数据集。当您持久化一个RDD时，每个节点都会存储它在内存中计算的任何分区，并在该数据集(或来自它的数据集)的其他操作中重用它们。这允许未来的动作更快(通常超过10x)。缓存是迭代算法和快速交互使用的关键工具。

您可以使用persist()或cache()方法来标记一个RDD。第一次在操作中计算它时，它将保存在节点上的内存中。Spark的缓存是容错的——如果丢失了RDD的任何分区，它将使用最初创建的转换自动重新计算。

另外，每个持久化的RDD都可以使用不同的存储级别来存储，例如，允许您在磁盘上保存数据集，将其保存在内存中，但是作为序列化的Java对象(为了节省空间)，可以跨节点复制它。这些级别是通过传递一个StorageLevel对象(Scala、Java、Python)来持persist()的。cache()方法是使用默认存储级别(即StorageLevel)的简写。
MEMORY_ONLY(存储在内存中的反序列化对象)。完整的存储级别为:

Storage Level |   描述
MEMORY_ONLY |   将RDD存储为JVM中反序列化的Java对象。如果RDD不适合内存，一些分区将不会被缓存，并且在每次需要时都将被重新计算。这是默认的级别。
MEMORY_AND_DISK |   将RDD存储为JVM中反序列化的Java对象。如果RDD不适合内存，则存储不适合磁盘的分区，并在需要时从那里读取它们。
MEMORY_ONLY_SER 
(Java and Scala)    |   将RDD存储为序列化的Java对象(每个分区一个字节数组)。这通常比反序列化的对象更具有空间效率，特别是在使用快速序列化器时，但是要读取更多的cpu。
MEMORY_AND_DISK_SER 
(Java and Scala)    |   与MEMORY_ONLY_SER类似，但是泄漏分区在内存中不适合磁盘，而不是在每次需要时重新计算它们。
DISK_ONLY   |   仅将RDD分区存储在磁盘上。
MEMORY_ONLY_2, MEMORY_AND_DISK_2, etc.  |   与上面的级别相同，但是在两个集群节点上复制每个分区。
OFF_HEAP (experimental) |   与MEMORY_ONLY_SER类似，但将数据存储在非堆内存中。这需要启用非堆内存。

注意:在Python中，存储对象将始终与Pickle库进行序列化，因此是否选择序列化级别无关紧要。Python中可用的存储级别包括MEMORY_ONLY、MEMORY_ONLY_2、MEMORY_AND_DISK、MEMORY_AND_DISK_2、DISK_ONLY和DISK_ONLY_2。

Spark还会自动地在shuffle操作中(例如reduceByKey)中保留一些中间数据，即使没有用户调用。这样做是为了避免在洗牌过程中节点出现故障时重新计算整个输入。我们仍然建议用户在计划重用时仍然坚持使用RDD。

#### 选择那个存储级别

Spark的存储级别意味着在内存使用和CPU效率之间提供不同的权衡。我们建议通过以下过程来选择一个:

* 如果您的RDDs与默认的存储级别(MEMORY_ONLY)相适应，那么将它们保留下来。这是最高效的选项，允许RDDs上的操作尽可能快地运行。
* 如果没有，尝试使用MEMORY_ONLY_SER并选择一个快速序列化库，以使对象的空间效率更高，但仍然可以快速访问。(Java和Scala)
* 不要溢出到磁盘，除非计算您的数据集的函数是昂贵的，或者它们过滤大量的数据。否则，重新计算分区可能与从磁盘读取分区一样快
* 如果您希望快速故障恢复(例如，如果使用Spark来服务来自web应用程序的请求)，则使用复制的存储级别。所有的存储级别都通过重新计算丢失的数据来提供完全的容错，但是复制的级别允许您继续在RDD上运行任务，而无需等待重新计算丢失的分区。

#### 删除数据

Spark自动监视每个节点上的缓存使用情况，并在最近使用的(LRU)方式中删除旧的数据分区。如果您想手动删除一个RDD，而不是等待它从缓存中掉出来，请使用RDD.unpersist()方法。

### 共享变量

通常，当传递给Spark操作(如map或reduce)的函数在远程集群节点上执行时，它会在函数中使用的所有变量的单独副本上工作。这些变量被复制到每台机器上，而对远程机器上的变量的更新不会被传播回驱动程序。支持一般的、读写的跨任务的共享变量将是低效的。不过，Spark确实为两个常见的使用模式提供了两种有限类型的共享变量、广播变量和累加器。

### 广播变量

广播变量允许程序员将只读变量保存在每台机器上，而不是将其复制到任务中。例如，可以使用它们以有效的方式为每个节点提供一个大型输入数据集的副本。Spark还尝试使用高效的广播算法来分配广播变量，以减少通信成本。

Spark动作通过一组stages执行，由分布式的“shuffle”操作分隔。Spark会自动地在每个stages中广播任务所需的公共数据。通过这种方式广播的数据以序列化形式缓存，并在运行每个任务之前进行反序列化。这意味着，当跨多个stages的任务需要相同的数据或以反序列化形式缓存数据时，显式创建广播变量只会有用。

广播变量通过调用SparkContext.broadcast(v)从变量v中创建。广播变量是一个围绕v的包装器，它的值可以通过调用value方法来访问。下面的代码显示了以下内容:

``` scala
scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar: org.apache.spark.broadcast.Broadcast[Array[Int]] = Broadcast(0)

scala> broadcastVar.value
res0: Array[Int] = Array(1, 2, 3)
```

在创建了广播变量之后，应该在集群中运行的任何函数中使用它，而不是值v，这样就不会不止一次地将v发送到节点。此外，在广播之后，对象v不应该被修改，以确保所有节点都获得广播变量的相同值(例如，如果该变量稍后被发送到一个新节点)。

### 累加器(Accumulators)

累加器是只通过关联和交换操作“添加”的变量，因此可以并行地得到有效支持。它们可以用于实现计数器(如在MapReduce中)或求和。Spark本机支持数字类型的累加器，程序员可以添加对新类型的支持。

作为用户，您可以创建命名的或未命名的累加器。如下图所示，一个名为累加器(在此实例计数器中)将在web UI中显示用于修改该累加器的阶段。Spark显示了在“任务”表中由任务修改的每个累加器的值。

在UI中跟踪累加器对于理解运行阶段的进展非常有用(注意:这在Python中还没有得到支持)。

数字累加器可以通过调用创建SparkContext.longAccumulator()或SparkContext.doubleAccumulator()积累值类型分别为Long或Double。在集群上运行的任务可以使用add方法添加到它。然而，他们无法解读其价值。只有驱动程序可以使用它的值方法读取累加器的值。

下面的代码显示了一个用于添加数组元素的累加器:

``` scala
scala> val accum = sc.longAccumulator("My Accumulator")
accum: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 0, name: Some(My Accumulator), value: 0)

scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s

scala> accum.value
res2: Long = 10
```

虽然这段代码使用了对Long类型累加器的内置支持，程序员也可以通过子类AccumulatorV2来创建自己的类型。AccumulatorV2抽象类有几个方法，一个必须重写:重置accumulator到零，添加另一个值到累加器中，合并另一个同类型累加器合并到这个累加器中。其他必须重写的方法包含在API文档中。例如，假设我们有一个表示数学向量的MyVector类，我们可以这样写:

``` scala
class VectorAccumulatorV2 extends AccumulatorV2[MyVector, MyVector] {

  private val myVector: MyVector = MyVector.createZeroVector

  def reset(): Unit = {
    myVector.reset()
  }

  def add(v: MyVector): Unit = {
    myVector.add(v)
  }
  ...
}

// Then, create an Accumulator of this type:
val myVectorAcc = new VectorAccumulatorV2
// Then, register it into spark context:
sc.register(myVectorAcc, "MyVectorAcc1")
```


请注意，当程序员定义自己的类型的AccumulatorV2时，结果类型可能与添加的元素不同。

对于仅在操作中执行的累加器更新，Spark保证每个任务对累加器的更新只会被应用一次，即重新启动的任务不会更新该值。在转换中，用户应该意识到，如果重新执行任务或工作阶段，每个任务的更新可能不止一次应用。

累加器不会改变Spark的延迟评估模型。如果它们在RDD的操作中被更新，它们的值只会在RDD被计算为动作的一部分时更新。因此，在像map()这样的惰性转换中，不保证会执行累加器更新。下面的代码片段演示了该属性:

``` scala
val accum = sc.longAccumulator
data.map { x => accum.add(x); x }
// Here, accum is still 0 because no actions have caused the map operation to be computed.
```

