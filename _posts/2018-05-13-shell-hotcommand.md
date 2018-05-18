---
layout: post
date:   2018-05-18 11:09:00
title:  "Shell Hot Command"
categories: shell
tags:  shell shellcommand
mathjax: true
---

* content
{:toc}
工作中常用的Shell脚本备忘及提高效率！






### shell.while

``` bash
#!/bin/bash

startdate=`date -d "$1" +%Y%m%d`
enddate=`date -d "$2" +%Y%m%d`

while [[ $startdate < $enddate  ]]  
do
    echo "########$startdate#########"  

    startdate=`date -d "+1 day $startdate" +%Y%m%d`
done
```
