---
layout: post
date:   2019-04-24 10:30:00
title:  "Elasticsearch update doc by script"
categories: elasticsearch
tags:  elasticsearch update script
mathjax: true
---

* content
{:toc}

Elasticsearch通过脚本更新文档只，可实现字段累加功能。




### Elasticsearch实现字段自增

```java
POST /index/type/xxx/_update
{
  "script":{
    "lang": "painless",
    "inline": "ctx._source.fieldname += 1;"
  }
}
```
