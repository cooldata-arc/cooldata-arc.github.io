---
layout: post
date:   2018-08-09 18:04:00
title:  "Markdwon 常用语法积累备忘"
categories: Markdown
tags:  markdown
mathjax: true
---

* content
{:toc}
Markdwon常用语法积累备忘，以备经常温习，熟练的使用Markdwon记录所学所用。







### 1.1 Markdown常用语法

#### 1.1.1 换行

* 方法1: 连续两个以上空格+回车
* 方法2：使用html语言换行标签：\<br>

#### 1.1.2 缩进

* &ensp; 半角的空格
* &emsp; 全角的空格

#### 1.1.3 字体-字号-颜色

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过类似HTML的标记语法，它可以使普通文本内容具有一定的格式。但是它本身是不支持修改字体、字号与颜色等功能的！ 
  CSDN-markdown编辑器是其衍生版本，扩展了Markdown的功能（如表格、脚注、内嵌HTML等等）！对，就是内嵌HTML，接下来要讲的功能就需要使用内嵌HTML的方法来实现。 
字体，字号和颜色编辑如下代码

``` html
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=#0099ff size=7 face="黑体">color=#0099ff size=72 face="黑体"</font>
<font color=#00ffff size=7>color=#00ffff</font>
<font color=gray size=7>color=gray</font>
 
Size：规定文本的尺寸大小。可能的值：从 1 到 7 的数字。浏览器默认值是 
```

#### 1.1.4 背景色

Markdown本身不支持背景色设置，需要采用内置html的方式实现：借助 table, tr, td 等表格标签的 bgcolor 属性来实现背景色的功能。举例如下：

``` html
<table><tr><td bgcolor=#FF4500>这里的背景色是：OrangeRed，  十六进制颜色值：#FF4500， rgb(255, 69, 0)</td></tr></table>
```

Example:<br>

<table>
<tr>
<td bgcolor=#FF4500>这里的背景色是：OrangeRed，  十六进制颜色值：#FF4500， rgb(255, 69, 0)
</td>
</tr>
</table>

#### 1.1.5 分割线

可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。以上每种写法都可以建立分隔线

#### 1.1.6 链接

``` html
[This link](http://example.NET/) has no title attribute.
```

#### 1.1.7 代码块

* 代码块：用2个以上TAB键起始的段落，会被认为是代码块（效果如下）：

			
struct {
  int year;
  int month;
  int day;
 }bdate;


* 如果在一个行内需要引用代码，只要用反引号`引起来就好(Esc健）

* 代码块与语法高亮：在需要高亮的代码块的前一行及后一行使用三个反引号“`”，同时第一行反引号后面表面代码块所使用的语言

#### 1.1.8 插入图片

```
![这里写图片描述](http://img3.douban.com/mpic/s1108264.jpg)
```

### 1.2 Markdown数学公式书写

#### 1.2.1 LaTex是什么

LaTeX（LATEX，音译“拉泰赫”）是一种基于ΤΕΧ的排版系统，由美国计算机学家莱斯利·兰伯特（Leslie Lamport）在20世纪80年代初期开发，利用这种格式，即使使用者没有排版和程序设计的知识也可以充分发挥由TeX所提供的强大功能，能在几天，甚至几小时内生成很多具有书籍质量的印刷品。对于生成复杂表格和数学公式，这一点表现得尤为突出。因此它非常适用于生成高印刷质量的科技和数学类文档。这个系统同样适用于生成从简单的信件到完整书籍的所有其他种类的文档。

#### 1.2.2 规则

* 空格：LaTeX中空格用来隔开单词(英语一类字母文字)，多个空格等效于一个空格；对中文没有作用。 
* 换行：用控制命令“\”,或“ \newline”. 
* 分段：用控制命令“\par” 或空出一行。 
* 换页：用控制命令“\newpage”或“\clearpage” 
* 特殊控制字符：#，$, %, &, - ,{, }, ^, ~ 

要想输出这些控制符用下列命令：

```
\# \$ \% \& \- \{ \} \^{} \~{} $\backslash$表示“ \”.。
```

#### 1.2.3 常用数学符号的 LaTeX 表示方法

* 指数和下标可以用^和_后加相应字符来实现

```
$a_{1}$ \qquad $x^{2}$ \qquad
$e^{-\alpha t}$ \qquad
$a^{3}_{ij}$\\
$e^{x^2} \neq {e^x}^2$ 
```

Example:<br>


$a_{1}$ \qquad $x^{2}$ \qquad
$e^{-\alpha t}$ \qquad
$a^{3}_{ij}$\\
$e^{x^2} \neq {e^x}^2$ 

* 平方根（square root）的输入命令为：\sqrt，n 次方根相应地为:\sqrt[n]。方根符号的大小由 LATEX自动加以调整。也可用 \surd 仅给出符号

```
$\sqrt{x}$ \qquad
$\sqrt{ x^{2}+\sqrt{y} }$
\qquad $\sqrt[3]{2}$\\[3pt]
$\surd[x^2 + y^2]$
```
Example:<br>

$\sqrt{x}$ \qquad
$\sqrt{ x^{2}+\sqrt{y} }$
\qquad $\sqrt[3]{2}$\\[3pt]
$\surd[x^2 + y^2]$

* 命令\overline 和\underline 在表达式的上、下方画出水平线

```
$\overline{m+n}$ \qquad
$\underline{m+n}$
```

Example:<br>

$\overline{m+n}$ \qquad
$\underline{m+n}$

* 命令\overbrace 和\underbrace 在表达式的上、下方给出一水平的大括号

```
$\underbrace{ a+b+\cdots+z }_{26}$
```

Example:<br>

$\underbrace{ a+b+\cdots+z }_{26}$