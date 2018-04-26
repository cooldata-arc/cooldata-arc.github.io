---
layout: post
date:   2018-04-25 11:17:00
title:  "Hive Toolbox"
categories: Hive
tags:  Hive Toolbox
mathjax: true
---

* content
{:toc}
工作中常用的Hive操作及优化措施整理总结形成Hive Toolbox，以备忘及提高效率！






### Hive Toolbox

#### 常用函数备忘

** concat_w -- 【多行合并成一行】

``` hql
select
    parent_id,
    concat_ws('_',collect_set(sub_id)) as sub_ids 
from tree_table 
group by parent_id  
```

** lateral view 和 explode 【explode将一行复杂结构拆为多行，然后在用lateral view做各种聚合】

``` hql
/**
 * explode example
 * table col: [1,2,3]
 * result row: 	1
 		   		2
 		   		3
 */
select explode(col) as myrow from mytable;

/**
 * lateral view 和 explode
 * input table row: K	V
 					A	[1,2,3]
 					B	[4,5]
 * output:  A	1
 			A	2
 			A	3
 			B	4
 			B	5
 */
select
    K,
    num
from mytable
lateral view explode(col) test_table as num
```

** to_date/unix_timestamp ... 【日期与时间戳】

``` hql
/**
 * **unix_timestamp**
 * 时间戳：时间戳是指**格林尼治时间**1970年01月01日00时00分00秒（北京时间1970年01月01日时00分00秒）起至现在的总秒数
 * 如果不写第二个参数，默认格式是 yyyy-MM-dd HH:mm:ss
 */
select unix_timestamp('20180425 13:25:30', 'yyyyMMdd HH:mm:ss') from mytable

/**
 * **to_date**
 * 只接受**yyyy-MM-dd HH:mm:ss**格式
 */
select to_date('2018-04-25 13:30:30') from mytable

/**
 * year(), month(), day(), hour(), minute(), second()
 * week() 返回日期所在的星期数
 */
select year('2018-04-25 13:30:30') from mytable

/**
 * **datediff** 日期比较
 * 语法：datediff(string enddate, string startdate)
 */
select datediff('2018-04-25', '2018-01-01') from mytable

/**
 * **date_add** and **date_sub**
 * 语法：date_add(string startdate, int days)
 */
select date_add('2018-04-25', 7) from mytable
select date_sub('2018-04-25', 7) from mytable
```

#### 常用数据导出方式

#### 常用数据导入方式

### Hive Tuning

