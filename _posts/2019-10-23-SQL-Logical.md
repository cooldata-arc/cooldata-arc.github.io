---
layout: post
date:   2019-10-23 19:30:00
title:  "SQLlogical plan"
categories: SQL
tags:  sql logical plan
mathjax: true
---

* content
{:toc}

SQL执行计划优化。






### SQL执行计划优化基础理论

#### 幂等

```
# 数学
20 * 10 * 1 * 1 = 20 * 10 = 200

# SQL
SELECT * FROM (SELECT name, age FROM User) = SELECT name, age FROM User
```

#### 交换律

```
# 数学
20 * 10 = 10 * 20 = 200

# SQL
table_a union all table_b = table_b union all table_a
```

#### 结合律

```
# 数学
(20 * 5) * 2 = 20 * (5 * 2) = 200

20 * (8 + 2) = 20 * 8 + 20 * 2 = 200

# SQL
SELECT * FROM (SELECT name, age FROM User) WHERE age > 20  =  SELECT * FROM (SELECT name, age FROM User WHERE age > 20)
```
