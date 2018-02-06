---
layout: post
date:   2018-02-06 09:45:00
title:  "Java exception best practice"
categories: Java
tags:  Java Exception 异常
excerpt: Java异常的最佳实践，遵循最佳的异常处理规则，可以避免很多的异常处理的坑!
mathjax: true
---

* content
{:toc}

在Java中，提供了比较完善的异常处理机制，但异常处理任然是个很麻烦的事情；尤其对于新手来说，异常处理很难理解，甚至经验丰富的开放者也会不小心掉入异常处理的陷阱。

最近在[dzone.com](https://dzone.com)上看到一篇关于异常处理的总结[9 Best Practices to Handle Exceptions in Java](https://dzone.com/articles/9-best-practices-to-handle-exceptions-in-java)很不错。团队可以纳入开发规范中，个人开发者也可以遵循此规则来提高编码素养，以避免不必要的异常处理麻烦。一下大部分为翻译内容，其中有些个人见解！






### 用Try-With-Resource特性或者在Finally中释放资源
通常情况下，在try块中使用一个资源，需要在之后关闭它。在这些情况下一个常见的错误是在try块末尾释放资源。
``` java
public void doNotCloseResourceInTry{
	FileInputStream inputStream = null;
	try{
		File file = new File("./tmp.txt");
		inputStream = new FileInputStream(file);

		// use the inputStream to read a file

		// do NOT do this
		inputStream.close();
	} catch (FileNotFoundException e){
        log.error(e);
    } catch (IOException e) {
        log.error(e);
    }
}
```
以上异常处理方式的问题在于，只要不抛出异常就可以正常工作，try块中的所有语句都将被执行；但在try块中发生异常，这意味着无法到达try块的末尾；因此，资源将不会释放。

因此，应该将所有的清理代码放入finally块中，或者使用try-to-resource语句。

* 在Finally块中释放资源
   与在try块末尾释放资源相比，finally块总是会被执行的（在try块成功执行之后或者在catch中处理一个异常之后）。
   ``` java
   public void closeResourceInFinally() {
       FileInputStream inputStream = null;
	    try {
	        File file = new File("./tmp.txt");
	        inputStream = new FileInputStream(file);
	        // use the inputStream to read a file
	    } catch (FileNotFoundException e) {
	        log.error(e);
	    } finally {
	        if (inputStream != null) {
	            try {
	                inputStream.close();
	            } catch (IOException e) {
	                log.error(e);
	            }
	        }
	    }
	}
   ```
* 使用Java7的Try-With-Resource特性
   如果您的资源实现了自动关闭接口，您可以使用它。这就是大多数Java标准资源所做的。当您在try子句中打开资源时，当try块被执行或异常处理后，它会自动关闭。
   ``` Java
	public void automaticallyCloseResource() {
	    File file = new File("./tmp.txt");
	    try (FileInputStream inputStream = new FileInputStream(file);) {
	        // use the inputStream to read a file
	    } catch (FileNotFoundException e) {
	        log.error(e);
	    } catch (IOException e) {
	        log.error(e);
	    }
	}
   ```
### 优先具体异常
抛出的异常越具体越明确越好，要记住，一个不了解你代码的同事，或将在几个月后，需要调用你的方法来处理这个异常。因此需要保证提供给他们尽可能多的信息；这使你的API更容易理解，从而使方法调用者能够更好地处理异常或避免使用额外的检查。例如：用NumberFormatException代替一个IllegalArgumentException,应该避免抛出这样不明确的异常。
``` java
public void doNotDoThis() throws Exception {
    ...
}

public void doThis() throws NumberFormatException {
    ...
}
```
### 记录制定的异常
每当在方法签名中指定一个异常时，也应该将其记录在Javadoc中。这与之前的最佳实践有相同的目标:尽可能多地向调用者提供信息，这样他就可以避免或处理异常。
所以，一定要向Javadoc添加一个@抛出声明，并描述可能导致异常的情况。
``` java
/**
 * This method does something extremely useful ...
 *
 * @param input
 * @throws MyBusinessException if ... happens
 */
public void doSomething(String input) throws MyBusinessException {
    ...
}
```
### 使用描述性消息抛出异常
这种最佳做法与前两种做法相似。但是这一次，您不会向您的方法的调用者提供信息。当日志文件或监视工具中报告异常时，每个必须理解发生了什么事情的人都会读取异常的消息。
因此，它应该尽可能准确地描述问题，并提供最相关的信息来理解异常事件。
别误会我;你不应该写一段文字。但是你应该用1-2个短句子来解释这个例外的原因。这样可以帮助您的操作团队了解问题的严重性，同时也使您更容易分析任何服务事件。
如果抛出一个特定的异常，它的类名很可能已经描述了这种错误。所以，你不需要提供很多额外的信息。一个很好的例子就是NumberFormatException。它由类java.lang的构造函数抛出。当您以错误的格式提供字符串时。
``` java
try {
    new Long("xyz");
} catch (NumberFormatException e) {
    log.error(e);
}
```
NumberFormatException类的名称已经告诉您问题的类型。它的消息只需要提供导致问题的输入字符串。如果异常类的名称不具有表达性，则需要在消息中提供所需的信息。
``` java
17:17:26,386 ERROR TestExceptionHandling:52 - java.lang.NumberFormatException: For input string: "xyz"
```
### 优先捕获具体的异常
大多数IDE都可以帮助你实现这个最佳实践。当你尝试首先捕获较不具体的异常时，它们会报告无法访问的代码块。

但问题在于，只有匹配异常的第一个catch块会被执行。因此，如果首先捕获IllegalArgumentException，则永远不会到达应该处理更具体的 NumberFormatException 的catch块，因为它是IllegalArgumentException的子类。

总是优先捕获最具体的异常类，并将不太具体的 catch 块添加到列表的末尾。

你可以在下面的代码片断中看到这样一个 try-catch 语句的例子。 第一个 catch 块处理所有NumberFormatException异常，第二个处理所有非NumberFormatException异常的IllegalArgumentException 异常。
``` java
public void catchMostSpecificExceptionFirst() {
    try {
        doSomething("A message");
    } catch (NumberFormatException e) {
        log.error(e);
    } catch (IllegalArgumentException e) {
        log.error(e)
    }
}
```
### 不要捕获Throwable异常
Throwable 是所有Exceptions和Errors的超类，我们可以在catch子句中捕获它，但不应该这样做！

如果在catch子句中使用Throwable，它将不仅捕获所有异常;它还将捕获所有错误。JVM抛出错误，表示应用程序不打算处理的严重问题。典型的例子是OutOfMemoryError或StackOverflowError。两者都是由应用程序控制之外的情况造成的，无法处理。所以，除非你绝对确信自己处在一个可以或需要处理错误的特殊情况下，否则最好不要捕获Throwable。

``` java
public void doNotCatchThrowable() {
    try {
        // do something
    } catch (Throwable t) {
        // don't do this!
    }
}
```

### 不要忽略异常
在任何时候都不应该忽略一个异常；要么不捕获，要么处理异常；捕获而不处理即“忽略”异常，将给后续的工作带来巨大的麻烦。这将隐藏Bug，在出现问题时定位问题造成困难。

``` java
public void doNotIgnoreExceptions() {
    try {
        // do something
    } catch (NumberFormatException e) {
        // this will never happen
    }
}
```

所以，请不要忽略一个例外。您不知道代码在将来会如何变化。有人可能会删除阻止异常事件的验证，而认识不到这会造成问题。或者抛出异常的代码被更改，现在抛出同一个类的多个异常，而调用代码并不能阻止所有这些异常。
你至少应该写一份日志，告诉每个人，意想不到的事情发生了，有人需要检查一下。

``` java
public void logAnException() {
    try {
        // do something
    } catch (NumberFormatException e) {
        log.error("This should never happen: " + e);
    }
}
```

### 不要记录日志后再抛出异常

这可能是该文章中最常被忽略的最佳实践。你可以找到很多的其中有一个异常被捕获的代码片段，甚至是一些代码库，被记录和重新抛出。

``` java
try {
    new Long("xyz");
} catch (NumberFormatException e) {
    log.error(e);
    throw e;
}
```

如果在发生异常时记录异常，然后重新抛出异常，以便调用方能够恰当地处理它，可能会感觉很直观。但是它也会为相同的异常写入多个错误消息。

``` java
17:44:28,945 ERROR TestExceptionHandling:65 - java.lang.NumberFormatException: For input string: "xyz"
Exception in thread "main" java.lang.NumberFormatException: For input string: "xyz"
at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
at java.lang.Long.parseLong(Long.java:589)
at java.lang.Long.(Long.java:965)
at com.stackify.example.TestExceptionHandling.logAndThrowException(TestExceptionHandling.java:63)
at com.stackify.example.TestExceptionHandling.main(TestExceptionHandling.java:58)
```

附加的消息也不会添加任何信息。正如在最佳实践#4中所解释的那样，异常消息应该描述异常事件。堆栈跟踪告诉您在哪个类、方法和行中抛出异常。
如果您需要添加其他信息，您应该捕获异常并将其封装到一个定制的异常中。但是要确保遵循最佳实践9。

``` java
public void wrapException(String input) throws MyBusinessException {
    try {
        // do something
    } catch (NumberFormatException e) {
        throw new MyBusinessException("A message that describes the error.", e);
    }
}
```

所以，如果您想处理它，只捕获一个异常。否则，在方法签名中指定它，并让调用者处理它。

### 封装异常而不适用它

有时候，最好是捕获一个标准异常并将其封装成一定制的异常。一个典型的例子是应用程序或框架特定的业务异常。允许你添加些额外的信息，并且你也可以为你的异常类实现一个特殊的处理。

在你这样做时，请确保将原始异常设置为原因（注：参考下方代码 NumberFormatException e 中的原始异常 e ）。Exception 类提供了特殊的构造函数方法，它接受一个 Throwable 作为参数。

另外，你将会丢失堆栈跟踪和原始异常的消息，这将会使分析导致异常的异常事件变得困难。

``` java
public void wrapException(String input) throws MyBusinessException {
    try {
        // do something
    } catch (NumberFormatException e) {
        throw new MyBusinessException("A message that describes the error.", e);
    }
}
```

### 总结
正如您所看到的，当您抛出或捕获异常时，您应该考虑许多不同的事情。它们中的大多数都有提高代码可读性或API可用性的目标。
异常通常是错误处理机制和通信媒介。因此，您应该确保与您的同事讨论您想要应用的最佳实践和规则，以便每个人都理解通用概念并以相同的方式使用它们。

***

[1]: https://dzone.com/articles/9-best-practices-to-handle-exceptions-in-java	"9 Best Practices to Handle Exceptions in Java"