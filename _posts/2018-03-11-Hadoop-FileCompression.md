---
layout: post
date:   2018-03-11 13:50:00
title:  "Hadoop file compression"
categories: Hadoop
tags:  Hadoop File Compression 文件压缩
mathjax: true
---

* content
{:toc}
MapReduce压缩解压！！！






### Hadoop对于压缩文件的支持

hadoop对于压缩格式是透明识别的，MapReduce任务的执行同样是透明的，如果输入的压缩文件具有相应的压缩格式的扩展名（比如lzo、gz、bzip2等），hadoop就会根据扩展名去选择解码器减压。

Hadoop对每个压缩格式的支持如下：

压缩格式	|	工具	|	算法	|	文件扩展名	|	多文件	|	可分割行
------------|-----------|-----------|---------------|-----------|-------------
DEFLATE	|无	|DEFLATE	|.DEFLATE|不	|不
gzip	|gzip	|DEFLATE	|.gz 	|不	|不
ZIP	|zip 	|DEFLATE	|.zip 	|是	|是，在文件范围内
bzip2	|bzip2	|bzip2	|.bz2 	|不	|是
LZO	|lzop	|LZO 	|.lzo 	|不	|是

### MapReduce作业输出的压缩

如果要压缩MapReduce作业的输出，需要将配置文件中的*mapred.output.compress*属性设置为true；将*mapred.output.compression.codec*属性设置为要使用的压缩编码/解码器的类名。
如果为输出使用了一系列文件，可以设置*mapred.output.compression.type*属性来控制压缩类型，默认为*RECORD*，其压缩单独的记录，将其改为*BLOCK*,则可压缩一组记录；可以所有更好的性能。


### map作业输出结果压缩

即使MapReduce应用使用非压缩的数据来读取和写入，我们也可以受益于压缩map阶段的中间输出。因为map作业的输出会被写入磁盘并通过网络传输到reducer节点，所以如果使用LZO之类的快速压缩，能得到更好的性能，因为传输的数据量大大减少了。以下代码显示了启用rnap输出压缩和设置压缩格式的配置属性。

``` java
conf.setCompressMapOutput(true);
conf.setMapOutputCompressorClass(GzipCodec.class);
```