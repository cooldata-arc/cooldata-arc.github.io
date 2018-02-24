---
layout: post
date:   2018-02-24 15:00:00
title:  "Scala 闭包"
categories: Scala
tags:  scala closures 闭包
mathjax: true
---

* content
{:toc}

Scala之闭包！





### Scala 闭包

闭包是引用独立(自由)变量的函数(本地使用的变量，但在封闭范围内定义)。换句话说，在闭包中定义的函数“记住”创建他的环境。

```
Closures are functions that refer to independent (free) variables (variables that are used locally, but defined in an enclosing scope). In other words, the function defined in the closure 'remembers' the environment in which it was created.
```

闭包通常来讲可以简单的理解为可以访问一个函数里局部变量的另外一个函数。闭包的出现是因为lexical scope(静态作用域)，闭包是一种特殊的对象，它结合了两个东西:一个函数和创建该函数的环境。环境由在创建闭包时的范围内的任何本地变量组成；scala支持函数作为参数或返回值，这时如果没有闭包，那么函数的free变量就会出错。

### Example

``` scala
var factor = 5
val multiplier = (i:Int) => i * factor
```

在以上示例中multiplier匿名函数中有俩个变量i和factor，其中i是函数的形参，factor是一个自由变量；这样定义的multiplier函数变量成为一个闭包，因为其引用到函数外面定义的变量，定义这个函数的过程是将这个自由变量捕获而构成一个封闭的函数。