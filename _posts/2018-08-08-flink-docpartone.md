---
layout: post
date:   2018-08-08 18:43:00
title:  "flink 文档阅读笔记"
categories: flink
tags:  flink stream
mathjax: true
---

* content
{:toc}
Flink文档阅读笔记.







### 1 Flink是什么

#### 1.1 架构-Architectrue

Flink是一个分布式处理引擎框架，用于在无界和有界的数据流上进行有状态的计算。Flink设计用于在所有的常见集群中运行，以内存速度执行任何规模的计算。

##### 1.1.1 处理无界和有界数据

任何类型的数据都是作为事件流产生的。信用卡交易、传感器测量、机器日志或网站或移动应用程序上的用户交互，所有这些数据都以流的形式生成。

* 数据可以作为*无界*或*有界*流来处理。

&ensp;&ensp;&emsp;1、*无界流*有一个开始但没有定义的结束。它们不会在生成数据时终止并提供数据。无界流必须连续地处理，即，事件在摄入后必须及时处理。不可能等待所有输入数据到达，因为输入是无界的，在任何时候都不会完成。处理无界数据通常需要按照特定的顺序处理事件，例如事件发生的顺序，以便能够推断结果的完整性。  

&ensp;&ensp;&emsp;2、*有界流*有一个定义的开始和结束。有界流可以通过在执行任何计算之前摄取所有数据来处理。由于有界数据集总是可以排序的，所以处理有界流不需要有序的摄取。有界流的处理也称为批处理。  

![无界和有界流](https://superzhangx.github.io/images/flink/flink-bounded-unbounded-stream.png)

*Apache Flink擅长处理无界和有界数据集。*对时间和状态的精确控制使Flink的运行时能够在无界流上运行任何类型的应用程序。有界流由专门为固定大小的数据集设计的算法和数据结构内部处理，从而获得优异的性能。

##### 1.1.2 部署(Deploy Application Anywhere)

*Apache Flink是一个分布式系统*，需要计算资源来执行应用程序。Flink集成了所有常见的集群资源管理器，如Hadoop YARN、Apache Mesos和Kubernetes，但也可以设置为作为独立集群运行。

Flink被设计成能够很好地工作于前面列出的每个资源管理器。这是通过特定于资源管理器的部署模式实现的，这种部署模式允许Flink以其惯用的方式与每个资源管理器交互。

在部署Flink应用程序时，Flink根据应用程序的配置并行性自动识别所需的资源，并从资源管理器请求它们。如果发生故障，Flink将通过请求新的资源来替换失败的容器。提交或控制应用程序的所有通信都是通过REST调用进行的。这简化了Flink在许多环境中的集成。


##### 1.1.3 以任何规模运行应用程序

Flink设计用于在任何规模上运行有状态流应用程序。应用程序被并行化成数千个任务，这些任务分布在一个集群中并发执行。因此，应用程序实际上可以利用无限数量的cpu、主内存、磁盘和网络IO。此外，Flink很容易保持非常大的应用状态。它的异步和增量检查点算法确保了对处理延迟的最小影响，同时保证了精确的一次状态一致性。

用户报告了在生产环境中运行的Flink应用程序令人印象深刻的可伸缩性数字，例如：
* 应用程序每天处理数以万亿计的事件。
* 应用程序维护多个tb的状态。
* 在数千个内核上运行的应用程序。

##### 1.1.4 利用内存性能

有状态Flink应用程序针对本地状态访问进行了优化。任务状态总是在内存中维护，如果状态大小超过可用内存，则在具有访问效率的磁盘数据结构中维护。因此，任务通过访问本地(通常是在内存中)状态来执行所有计算，从而产生非常低的处理延迟。Flink通过定期和异步检查本地状态到持久存储，保证了在发生故障时的精确一次状态一致性。

![flink local state](https://superzhangx.github.io/images/flink/flink-local-state.png)

#### 1.2 应用程序

Apache Flink是一个跨无界和有界数据流进行有状态计算的框架。Flink在不同的抽象级别上提供了多个api，并为通用用例提供了专用的库。

##### 1.2.1 流应用程序的构建块

可以使用流处理框架构建和执行的应用程序类型由框架控制流、状态和时间的能力定义。在下面的文章中，我们将描述流处理应用程序的这些构建块，并解释Flink处理它们的方法。

###### 1.2.2 流

显然，流是流处理的一个基本方面。但是，流可能具有不同的特性，这些特性会影响流的处理方式和处理方式。Flink是一个通用的处理框架，可以处理任何类型的流。

&ensp;&ensp;&emsp;* *有界的和无界的流*:流可以是无界的或有界的，即固定大小的数据集。Flink具有处理无界流的复杂特性，但也有专门的操作符来有效地处理有界流。

&ensp;&ensp;&emsp;* *实时流和记录流*:所有数据都作为流生成。有两种处理数据的方法。当流生成或将其持久化到存储系统(例如文件系统或对象存储)时实时处理，并在稍后处理。Flink应用程序可以处理记录或实时流。

###### 1.2.3 状态

每个重要的流应用程序都是有状态的，例如。，只有对单个事件应用转换的应用程序不需要状态。任何运行基本业务逻辑的应用程序都需要记住事件或中间结果，以便在稍后的时间点访问它们，例如在接收下一个事件或在特定的时间后访问它们。

![flink function state](https://superzhangx.github.io/images/flink/flink-function-state.png)

申请状态为Flink一级公民。通过查看Flink在状态处理上下文中提供的所有特性，您可以看到这一点。

&ensp;&ensp;&emsp;* *多重状态原语* :Flink为不同的数据结构(如原子值、列表或映射)提供状态原语。开发人员可以根据函数的访问模式选择最有效的状态原语。

&ensp;&ensp;&emsp;* *可插入状态后端* :应用程序状态由可插入状态后端管理并检查。Flink具有不同的状态后端，它们将状态存储在内存或RocksDB中，RocksDB是一种高效的嵌入式磁盘数据存储。定制的状态后端也可以插入。

&ensp;&ensp;&emsp;* *精确一次状态一致性* :Flink的检查点和恢复算法保证了在出现故障时应用状态的一致性。因此，故障是透明处理的，不会影响应用程序的正确性。

&ensp;&ensp;&emsp;* *非常大的状态* :Flink由于其异步和增量检查点算法，能够维持几个tb大小的应用状态。

&ensp;&ensp;&emsp;* *可伸缩的应用程序* : Flink通过将状态重新分配给更多或更少的工作者来支持有状态应用程序的扩展。

###### 1.2.4 时间

时间是流应用程序的另一个重要组成部分。大多数事件流具有固有的时间语义，因为每个事件都是在特定的时间点生成的。此外，许多常见的流计算基于时间，例如windows聚合、会话、模式检测和基于时间的连接。流处理的一个重要方面是应用程序如何度量时间，即，事件时间和处理时间的差异。

Flink提供了一组丰富的与时间相关的特性。

&ensp;&ensp;&emsp;* Event-time Mode:使用事件时间语义处理流的应用程序根据事件的时间戳计算结果。因此，无论记录事件还是实时事件被处理，事件时间处理都允许得到精确和一致的结果。

&ensp;&ensp;&emsp;* Watermark Support:Flink在事件时间应用程序中使用Watermark来推断时间。Watermark也是一种灵活的机制，可以平衡延迟和结果的完整性。

&ensp;&ensp;&emsp;* Late Data Handling:当以Event-time Mode处理带有Watermark的流时，可能会发生在所有相关事件到达之前已经完成计算。这类事件称为延迟事件。Flink具有多个选项来处理延迟事件，比如通过侧输出重新路由它们，以及更新以前完成的结果。

&ensp;&ensp;&emsp;* Processing-time Mode:除了Event-time Mode之外，Flink还支持Processing-time Mode语义，该语义可以根据处理机器的挂钟时间触发计算。Processing-time Mode可以适用于某些具有严格的低延迟要求、能够容忍近似结果的应用程序。

##### 1.3 API分层

Flink提供了三个分层的api。每种API都在简洁和表达之间提供了不同的权衡，并针对不同的用例。

![flink api stack](https://superzhangx.github.io/images/flink/flink-api-stack.png)

###### 1.3.1 ProcessFunctions example

ProcessFunctions是Flink提供的最具表现力的函数接口。Flink提供processfunction来处理来自一个或两个输入流或在窗口中分组的事件的单个事件。process函数提供细粒度的时间和状态控制。ProcessFunction可以任意修改它的状态和注册计时器，这些计时器将在将来触发回调函数。因此，ProcessFunctions可以根据许多有状态事件驱动应用程序的需要实现复杂的每事件业务逻辑。

下面的示例显示了一个KeyedProcessFunction，它在KeyedStream上操作并匹配开始和结束事件。当接收到启动事件时，函数将记住其处于状态的时间戳，并在4小时内注册一个计时器。如果在计时器触发之前接收到结束事件，函数将计算结束事件和开始事件之间的持续时间，清除状态并返回值。否则，计时器只会触发并清除状态。

``` java
/**
 * Matches keyed START and END events and computes the difference between 
 * both elements' timestamps. The first String field is the key attribute, 
 * the second String attribute marks START and END events.
 */
public static class StartEndDuration
    extends KeyedProcessFunction<String, Tuple2<String, String>, Tuple2<String, Long>> {

  private ValueState<Long> startTime;

  @Override
  public void open(Configuration conf) {
    // obtain state handle
    startTime = getRuntimeContext()
      .getState(new ValueStateDescriptor<Long>("startTime", Long.class));
  }

  /** Called for each processed event. */
  @Override
  public void processElement(
      Tuple2<String, String> in,
      Context ctx,
      Collector<Tuple2<String, Long>> out) throws Exception {

    switch (in.f1) {
      case "START":
        // set the start time if we receive a start event.
        startTime.update(ctx.timestamp());
        // register a timer in four hours from the start event.
        ctx.timerService()
          .registerEventTimeTimer(ctx.timestamp() + 4 * 60 * 60 * 1000);
        break;
      case "END":
        // emit the duration between start and end event
        Long sTime = startTime.value();
        if (sTime != null) {
          out.collect(Tuple2.of(in.f0, ctx.timestamp() - sTime));
          // clear the state
          startTime.clear();
        }
      default:
        // do nothing
    }
  }

  /** Called when a timer fires. */
  @Override
  public void onTimer(
      long timestamp,
      OnTimerContext ctx,
      Collector<Tuple2<String, Long>> out) {

    // Timeout interval exceeded. Cleaning up the state.
    startTime.clear();
  }
}
```

这个例子说明了KeyedProcessFunction的表达能力，但也强调了它是一个相当冗长的接口。

###### 1.3.2 DataStream API

DataStream API为许多常见的流处理操作提供了原语，例如窗口、一次记录转换和通过查询外部数据存储丰富事件。DataStream API可用于Java和Scala，它基于map()、reduce()和aggregate()等函数。函数可以通过扩展接口或Java或Scala lambda函数来定义。

下面的示例展示了如何对clickstream进行会话并计算每个会话的单击次数。
``` java
// a stream of website clicks
DataStream<Click> clicks = ...

DataStream<Tuple2<String, Long>> result = clicks
  // project clicks to userId and add a 1 for counting
  .map(
    // define function by implementing the MapFunction interface.
    new MapFunction<Click, Tuple2<String, Long>>() {
      @Override
      public Tuple2<String, Long> map(Click click) {
        return Tuple2.of(click.userId, 1L);
      }
    })
  // key by userId (field 0)
  .keyBy(0)
  // define session window with 30 minute gap
  .window(EventTimeSessionWindows.withGap(Time.minutes(30L)))
  // count clicks per session. Define function as lambda function.
  .reduce((a, b) -> Tuple2.of(a.f0, a.f1 + b.f1));
```

###### 1.3.3 SQL & Table API

Flink具有两个关系API，Table API和SQL。这两个api都是批处理和流处理的统一api，即，查询在无界流、实时流或有界流上以相同的语义执行，并产生相同的结果。表API和SQL利用Apache方解石进行解析、验证和查询优化。它们可以无缝地与DataStream和DataSet api集成，并支持用户定义的标量、聚合和表值函数。

Flink的关系api旨在简化数据分析、数据流水线和ETL应用程序的定义。

下面的示例显示了用于处理clickstream的SQL查询，并计算每个会话的单击次数。这是与DataStream API示例相同的用例。

``` sql
SELECT userId, COUNT(*)
FROM clicks
GROUP BY SESSION(clicktime, INTERVAL '30' MINUTE), userId
```

###### 1.3.4 Libraries

Flink为通用数据处理用例提供了几个库。这些库通常嵌入在API中，而不是完全独立的。因此，它们可以从API的所有特性中获益，并与其他库集成。

* *Complex Event Processing (CEP)*: 模式检测是事件流处理的一个非常常见的用例。Flink的CEP库提供了一个API来指定事件的模式(考虑正则表达式或状态机)。CEP库与Flink的DataStream API集成，以便在DataStreams上评估模式。CEP库的应用程序包括网络入侵检测、业务流程监视和欺诈检测。

* *DataSet API* : DataSet API是Flink批处理应用程序的核心API。DataSet API的原语包括map、reduce、(外部)join、co-group和iterate。所有操作都由算法和数据结构支持，这些算法和数据结构对内存中的序列化数据进行操作，如果数据大小超过内存预算，则会溢出到磁盘。Flink的DataSet API的数据处理算法受到传统数据库操作符的启发，如混合哈希连接或外部合并排序。

* *Gelly* : Gelly是一个用于可伸缩图形处理和分析的库。Gelly是在数据集API的基础上实现的，并与之集成。因此，它得益于可伸缩和健壮的操作符。Gelly提供了内置的算法，比如标签传播、三角形枚举和页面排名，但也提供了一个图形API，简化了自定义图形算法的实现。

#### 2 操作(Operations)

Apache Flink是一个跨无界和有界数据流进行有状态计算的框架。由于许多流应用程序被设计为连续运行，停机时间最少，流处理器必须提供良好的故障恢复，以及在应用程序运行时监视和维护的工具。

Apache Flink将重点放在流处理的操作方面。在这里，我们解释了Flink的故障恢复机制，并介绍了其管理和监督运行中的应用程序的特性。

##### 2.1 7*24小时不间断运行应用程序

在分布式系统中，机器和过程故障无处不在。像Flink这样的分布式流处理器必须从故障中恢复，才能24/7地运行流应用程序。显然，这不仅意味着在失败后重新启动应用程序，而且还意味着确保其内部状态保持一致，以便应用程序可以继续处理，就好像失败从未发生过一样。

Flink提供了几个特性来确保应用程序保持运行并保持一致:

&ensp;&ensp;&emsp;* Consistent Checkpoints:Flink的恢复机制基于应用程序状态的一致检查点。如果出现故障，应用程序将重新启动，并从最新的检查点加载其状态。与可重新设置的流源相结合，这个特性可以保证一次完全的状态一致性。

&ensp;&ensp;&emsp;* Efficient Checkpoints:如果应用程序保持tb级的状态，那么检查应用程序的状态可能非常昂贵。Flink可以执行异步和增量检查点，以保持检查点对应用程序的延迟sla的影响非常小。

&ensp;&ensp;&emsp;* End-to-End Exactly-Once:Flink为特定存储系统提供了事务性sinks，可以保证数据只写一次，即使在出现故障的情况下也是如此。

&ensp;&ensp;&emsp;* Integration with Cluster Managers:Flink与集群管理器紧密集成，例如Hadoop YARN、Mesos或Kubernetes。当流程失败时，将自动启动一个新流程来接管其工作。

&ensp;&ensp;&emsp;* High-Availability Setup:链接具有高可用性模式，可以消除所有单点故障。ha模式基于Apache ZooKeeper，这是一种经过实战验证的可靠分布式协调服务。

##### 2.2 Update, Migrate, Suspend, and Resume Your Applications

需要维护支持关键业务服务的流应用程序。bug需要修复，改进或新特性需要实现。然而，更新有状态流应用程序并不是件小事。通常，不能简单地停止应用程序并重新启动一个固定的或改进的版本，因为不能失去应用程序的状态。

Flink的保存点是一个独特而强大的特性，它解决了有状态应用程序的更新问题和许多其他相关的挑战。保存点是应用程序状态的一致快照，因此非常类似于检查点。然而，与检查点不同的是，保存点需要手动触发，并且在应用程序停止时不会自动删除。保存点可用于启动状态兼容的应用程序并初始化其状态。保存点支持以下特性:

&ensp;&ensp;&emsp;* Application Evolution:保存点可用于演进应用程序。应用程序的固定或改进版本可以从应用程序的前一个版本的保存点重新启动。也可以从较早的时间点(假设存在这样的保存点)启动应用程序，以修复有缺陷的版本产生的错误结果。

&ensp;&ensp;&emsp;* Cluster Migration:使用保存点，应用程序可以迁移(或克隆)到不同的集群。

&ensp;&ensp;&emsp;* Flink Version Updates:应用程序可以迁移到新的Flink版本上，使用保存点运行。

&ensp;&ensp;&emsp;* Application Scaling:保存点可用于增加或减少应用程序的并行性。

&ensp;&ensp;&emsp;* A/B Tests and What-If Scenarios:应用程序的两个(或多个)不同版本的性能或质量可以通过从相同的保存点开始所有版本进行比较。

&ensp;&ensp;&emsp;* Pause and Resume:应用程序可以通过获取保存点并停止它来暂停。在以后的任何时候，应用程序都可以从保存点恢复。

&ensp;&ensp;&emsp;* Archiving:可以对保存点进行归档，以便能够将应用程序的状态重置到较早的时间点。

#### 3 监控和控制你的应用程序

与任何其他服务一样，持续运行的流应用程序需要被监视并集成到操作基础设施中，即。组织的监视和记录服务。监控有助于预测问题并提前做出反应。日志记录使根源分析能够调查故障。最后，控制运行中的应用程序的容易访问的接口是一个重要的特性。

Flink与许多常见的日志记录和监视服务很好地集成，并提供了一个REST API来控制应用程序和查询信息。

&ensp;&ensp;&emsp;* Web UI:Flink提供了一个web UI来检查、监视和调试运行中的应用程序。它还可以用于提交执行的执行或取消执行。

&ensp;&ensp;&emsp;* Logging:Flink实现了流行的slf4j日志记录接口，并与日志记录框架log4j或logback集成。

&ensp;&ensp;&emsp;* Metrics:Flink具有一个复杂的度量系统，用于收集和报告系统和用户定义的度量。度量可以导出到多个报告器，包括JMX、Ganglia、石墨、普罗米修斯、StatsD、Datadog和Slf4j。

&ensp;&ensp;&emsp;* *REST API*:Flink公开了一个REST API来提交一个新应用程序，获取一个正在运行的应用程序的保存点，或者取消一个应用程序。REST API还公开元数据和收集的运行或完成应用程序的指标。




