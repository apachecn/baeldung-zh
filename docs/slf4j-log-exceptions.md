# 使用 SLF4J 记录异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/slf4j-log-exceptions>

## 1。概述

在这个快速教程中，我们将展示如何使用 [SLF4J](https://web.archive.org/web/20221026123202/https://www.slf4j.org/) API 在 Java 中记录异常。我们将使用`slf4j-simple` API 作为日志实现。

您可以在我们之前的文章中探索不同的日志技术。

## 2。Maven 依赖关系

首先，我们需要向我们的`pom.xml`添加以下依赖项:

```java
<dependency>                             
    <groupId>org.slf4j</groupId>         
    <artifactId>slf4j-api</artifactId>   
    <version>1.7.30</version>  
</dependency> 

<dependency>                             
    <groupId>org.slf4j</groupId>         
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.30</version>  
</dependency>
```

这些库的最新版本可以在 [Maven Central](https://web.archive.org/web/20221026123202/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.slf4j%22%20AND%20(a%3A%22slf4j-api%22%20OR%20a%3A%22slf4j-simple%22)) 上找到。

## 3。示例

通常，使用 [`Logger`](https://web.archive.org/web/20221026123202/https://static.javadoc.io/org.slf4j/slf4j-api/1.7.25/org/slf4j/Logger.html) 类中可用的`error()`方法记录所有异常。这种方法有相当多的变化。我们将探索:

```java
void error(String msg);
void error(String format, Object... arguments);
void error(String msg, Throwable t);
```

让我们首先初始化我们将要使用的`Logger`:

```java
Logger logger = LoggerFactory.getLogger(NameOfTheClass.class);
```

如果我们只需要显示错误消息，那么我们可以简单地添加:

```java
logger.error("An exception occurred!");
```

上述代码的输出将是:

```java
ERROR packageName.NameOfTheClass - An exception occurred!
```

这很简单。但是要添加关于异常的更多相关信息(包括堆栈跟踪),我们可以写:

```java
logger.error("An exception occurred!", new Exception("Custom exception"));
```

输出将是:

```java
ERROR packageName.NameOfTheClass - An exception occurred!
java.lang.Exception: Custom exception
  at packageName.NameOfTheClass.methodName(NameOfTheClass.java:lineNo)
```

在存在多个参数的情况下，如果日志记录语句中的最后一个参数是异常，则 SLF4J 将假定用户希望将最后一个参数视为异常，而不是简单的参数:

```java
logger.error("{}, {}! An exception occurred!", 
  "Hello", 
  "World", 
  new Exception("Custom exception"));
```

在上面的代码片段中，`String`消息将根据传递的对象细节进行格式化。我们使用花括号作为传递给方法的`String`参数的占位符。

在这种情况下，输出将是:

```java
ERROR packageName.NameOfTheClass - Hello, World! An exception occurred!
java.lang.Exception: Custom exception 
  at packageName.NameOfTheClass.methodName(NameOfTheClass.java:lineNo)
```

## 4。结论

在这个快速教程中，我们了解了如何使用 SLF4J API 记录异常。

代码片段可以在 [GitHub 仓库](https://web.archive.org/web/20221026123202/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j)中找到。