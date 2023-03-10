# Log4j 2 和 Lambda 表达式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/log4j-2-lazy-logging>

## 1。概述

从 2.4 版本开始，`Log4j 2`库增加了对 Java 8 lambda 表达式的支持。**这些表达式可以被`Logger`接口用来启用惰性日志记录。**

让我们看一个简单的例子来说明如何利用这个特性。

想了解更多关于`Log4j 2`的信息，也可以看看我们的[介绍文章](/web/20220727063741/https://www.baeldung.com/log4j2-appenders-layouts-filters)。

## 2。使用 Lambda 表达式的惰性日志记录

如果没有启用相应的日志级别，避免计算日志消息可能会提高使用日志记录的应用程序的性能。

首先，让我们看看跟踪级别的简单日志语句:

```java
logger.trace("Number is {}", getRandomNumber());
```

在本例中，无论是否显示跟踪语句，都会调用`getRandomNumber()`方法来替换日志消息参数。例如，如果日志级别设置为 DEBUG，`log4j 2`将不会记录消息，但是`getRandomNumber()`方法仍然会运行。

换句话说，这个方法的执行可能是不必要的。

在添加对 lambda 表达式的支持之前，我们可以通过在执行 log 语句之前显式检查日志级别来避免构造不被记录的消息:

```java
if (logger.isTraceEnabled()) {
    logger.trace("Number is {}", getRandomNumer());
}
```

在这种情况下，只有启用了跟踪日志级别，才会调用 `getRandomNumber()`方法。**根据用于替代参数的方法的执行成本，这可以提高性能。**

通过使用 lambda 表达式，我们可以进一步简化上面的代码:

```java
logger.trace("Number is {}", () -> getRandomNumber());
```

**lambda 表达式仅在相应的日志级别被启用时才被评估。**这被称为惰性日志记录。

我们还可以对一条日志消息使用多个 lambda 表达式:

```java
logger.trace("Name is {} and age is {}", () -> getName(), () -> getRandomNumber());
```

## 3。结论

在这个快速教程中，我们已经看到了如何使用 lambda 表达式和`Log4j 2`记录器。

和往常一样，这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220727063741/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)