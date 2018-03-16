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
*---------------------------------------------------------------------------*
DEFLATE	|无	|DEFLATE	|.DEFLATE|不	|不


gzip	|gzip	|DEFLATE	|.gz 	|不	|不
ZIP	|zip 	|DEFLATE	|.zip 	|是	|是，在文件范围内
bzip2	|bzip2	|bzip2	|.bz2 	|不	|是
LZO	|lzop	|LZO 	|.lzo 	|不	|是