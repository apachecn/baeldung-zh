# 使用 Log4j2 在日志文件中打印线程信息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/log4j2-print-thread-info>

## 1.概观

本教程将展示使用 [Log4j2 库](/web/20221208143830/https://www.baeldung.com/log4j2-appenders-layouts-filters)记录线程信息的想法和例子。

## 2.日志记录和线程

**日志是一个强大的工具，它提供了当一些错误或流程发生时系统中正在发生的事情的背景**。日志记录有助于我们捕获和保存相关信息，以便随时进行分析。

**线程允许我们的应用程序同时执行多件事情**来处理更多的请求，使我们的工作更有效率。

在这种情况下，许多 Java 应用程序使用日志和线程来控制它们的进程。然而，由于日志通常集中在一个特定的文件上，日志从不同的线程中混乱起来，用户不能识别和理解事件的顺序。我们将使用最流行的 Java 日志框架之一 Log4j2 来显示我们的线程的相关信息，以解决这个问题。

## 3.Log4j2 用法

接下来，我们有一个使用 Log4j2 中的一些参数来显示线程信息的例子:

```java
<Property name="LOG_PATTERN"> %d{yyyy-MM-dd HH:mm:ss.SSS} --- thread_id="%tid" thread_name="%tn" thread_priority="%tp" --- [%p] %m%n </Property>
```

Log4j2 在其模式中使用参数来引用数据。在初学者中，所有参数都以`% `开头。以下是一些线程参数的示例:

*   **`tid`** :线程标识符是创建线程时生成的一个正长数字。
*   **`tn`** :给线程命名的字符序列。
*   `**tp**`:线程优先级是一个 1 到 10 之间的整数，数字越大，优先级越高。

首先，正如建议的那样，**我们添加了关于线程**的 id、名称和优先级的信息。因此，为了可视化它，我们需要创建一个简单的应用程序来创建新线程并记录一些信息:

```java
public class Log4j2ThreadInfo{
    private static final Logger logger = LogManager.getLogger(Log4j2ThreadInfo.class);

    public static void main(String[] args) {
        IntStream.range(0, 5).forEach(i -> {
            Runnable runnable = () -> logger.info("Logging info");
            Thread thread = new Thread(runnable);
            thread.start();
        });
    }
}
```

换句话说，我们只是借助于 [Java Streams](/web/20221208143830/https://www.baeldung.com/java-8-streams) 在 0 到 5 的范围内运行一个 forEach，然后启动一个新线程并进行一些日志记录。因此，我们将拥有:

```java
2022-01-14 23:44:56.893 --- thread_id="22" thread_name="Thread-2" thread_priority="5" --- [INFO] Logging info
2022-01-14 23:44:56.893 --- thread_id="21" thread_name="Thread-1" thread_priority="5" --- [INFO] Logging info
2022-01-14 23:44:56.893 --- thread_id="20" thread_name="Thread-0" thread_priority="5" --- [INFO] Logging info
2022-01-14 23:44:56.893 --- thread_id="24" thread_name="Thread-4" thread_priority="5" --- [INFO] Logging info
2022-01-14 23:44:56.893 --- thread_id="23" thread_name="Thread-3" thread_priority="5" --- [INFO] Logging info
```

## 4.结论

本文展示了一种使用 Log4j2 参数在 Java 项目中添加线程信息的简单方法。如果你想检查代码，可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j2)