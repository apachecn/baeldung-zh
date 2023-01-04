# Apache Camel 异常处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-camel-exception-handling>

## 1。概述

Apache Camel 是一个强大的开源集成框架，实现了几种已知的 T2 企业集成模式 T3。

通常，当使用 Camel 处理消息路由时，我们需要一种有效处理错误的方法。为此，Camel 提供了一些处理异常的策略。

在本教程中，我们将看看在 Camel 应用程序中用于异常处理的两种方法。

## 2。依赖性

我们需要做的就是将`[camel-spring-boot-starter](https://web.archive.org/web/20221109115315/https://search.maven.org/search?q=a:camel-spring-boot-starter)`添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.camel.springboot</groupId>
    <artifactId>camel-spring-boot-starter</artifactId>
    <version>3.19.0</version>
</dependency>
```

## 3。创建路线

让我们从定义一个相当基本的故意抛出异常的路由开始:

```java
@Component
public class ExceptionThrowingRoute extends RouteBuilder {

    private static final Logger LOGGER = LoggerFactory.getLogger(ExceptionThrowingRoute.class);

    @Override
    public void configure() throws Exception {

        from("direct:start-exception")
          .routeId("exception-handling-route")
          .process(new Processor() {

              @Override
              public void process(Exchange exchange) throws Exception {
                  LOGGER.error("Exception Thrown");
                  throw new IllegalArgumentException("An exception happened on purpose");

              }
          }).to("mock:received");
    }
}
```

简单回顾一下，Apache Camel 中的 [route](https://web.archive.org/web/20221109115315/https://camel.apache.org/manual/routes.html) 是一个基本的构建块，通常由一系列步骤组成，由 Camel 按顺序执行，消费和处理消息。

正如我们在这个小例子中看到的，我们配置我们的路由来消费来自名为 start 的直接端点的消息。

然后，**我们从一个新的`Processor,`中抛出一个`IllegalArgumentException`，这个新的`Processor,`是我们使用 Java DSL 在我们的路由中内联创建的。**

目前，我们的路由不包含任何类型的异常处理，所以当我们运行它时，我们会在应用程序的输出中看到一些难看的东西:

```java
...
10:21:57.087 [main] ERROR c.b.c.e.ExceptionThrowingRoute - Exception Thrown
10:21:57.094 [main] ERROR o.a.c.p.e.DefaultErrorHandler - Failed delivery for (MessageId: 50979CFF47E7816-0000000000000000 on ExchangeId: 50979CFF47E7816-0000000000000000). 
Exhausted after delivery attempt: 1 caught: java.lang.IllegalArgumentException: An exception happened on purpose

Message History (source location and message history is disabled)
---------------------------------------------------------------------------------------------------------------------------------------
Source                                   ID                             Processor                                          Elapsed (ms)
                                         exception-handling-route/excep from[direct://start-exception]                               11
	...
                                         exception-handling-route/proce [[email protected]](/web/20221109115315/https://www.baeldung.com/cdn-cgi/l/email-protection)                                          0

Stacktrace
---------------------------------------------------------------------------------------------------------------------------------------
java.lang.IllegalArgumentException: An exception happened on purpose
... 
```

## 4。使用`doTry()`块

现在让我们继续向我们的路由添加一些异常处理。在这一节，**我们将看看 Camel 的`doTry()`块，我们可以把它看作是 Java 中的`[try​ catch finally](/web/20221109115315/https://www.baeldung.com/java-exceptions)`的等价物，但是它直接嵌入在 DSL 中。**

但首先，为了帮助简化我们的代码，我们将定义一个抛出`IllegalArgumentException –` 的专用处理器类。这将使我们的代码更具可读性，并且我们可以在以后的其他路径中重用我们的处理器:

```java
@Component
public class IllegalArgumentExceptionThrowingProcessor implements Processor {

    private static final Logger LOGGER = LoggerFactory.getLogger(ExceptionLoggingProcessor.class);

    @Override
    public void process(Exchange exchange) throws Exception {
        LOGGER.error("Exception Thrown");
        throw new IllegalArgumentException("An exception happened on purpose");
    }
}
```

有了新的处理器，让我们在第一个异常处理路径中使用它:

```java
@Component
public class ExceptionHandlingWithDoTryRoute extends RouteBuilder {

    @Override
    public void configure() throws Exception {

        from("direct:start-handling-exception")
          .routeId("exception-handling-route")
          .doTry()
            .process(new IllegalArgumentExceptionThrowingProcessor())
            .to("mock:received")
          .doCatch(IOException.class, IllegalArgumentException.class)
            .to("mock:caught")
          .doFinally()
            .to("mock:finally")
          .end();
    }
}
```

正如我们所看到的，我们的路由中的代码是不言自明的。我们基本上是在使用 Camel 等价物模仿一个普通的 Java `try catch finally`语句。

但是，让我们来看看路线的关键部分:

*   首先，我们使用`doTry()`方法将我们希望抛出的异常立即被捕获的路径部分包围起来
*   接下来，我们使用`doCatch`方法关闭这个块。**注意，我们可以传递我们希望捕捉的不同异常类型的列表**
*   最后，我们调用`doFinally(),`，它定义了总是在`doTry()`和任何`doCatch()`块之后运行的代码

此外，我们应该注意，在 Java DSL 中调用`end()`方法来表示块的结束是很重要的。

Camel 还提供了另一个强大的特性，允许我们在使用`doCatch()`块时使用[谓词](https://web.archive.org/web/20221109115315/https://camel.apache.org/manual/predicate.html):

```java
...
.doCatch(IOException.class, IllegalArgumentException.class).onWhen(exceptionMessage().contains("Hello"))
   .to("mock:catch")
...
```

这里我们添加了一个运行时谓词来确定 catch 块是否应该被触发。**在这种情况下，我们只想在导致异常的消息包含单词 Hello** 时触发它。相当酷！

## 5。使用例外条款

不幸的是，以前的方法的一个限制是它只适用于单个路由。

通常，随着应用程序的增长，我们会添加越来越多的路由，我们可能不希望在一个路由接一个路由的基础上处理异常。这可能会导致重复的代码，我们可能需要一个通用的应用程序错误处理策略。

幸运的是，Camel 通过 Java DSL 提供了一个异常子句机制，以指定我们需要的基于每个异常类型或全局的错误处理:

假设我们想要为我们的应用程序实现一个异常处理策略。对于我们的简单示例，我们假设只有一条路线:

```java
@Component
public class ExceptionHandlingWithExceptionClauseRoute extends RouteBuilder {

    @Autowired
    private ExceptionLoggingProcessor exceptionLogger;

    @Override
    public void configure() throws Exception {
        onException(IllegalArgumentException.class).process(exceptionLogger)
          .handled(true)
          .to("mock:handled")

        from("direct:start-exception-clause")
          .routeId("exception-clause-route")
          .process(new IllegalArgumentExceptionThrowingProcessor())
          .to("mock:received");
    }
}
```

**正如我们所见，我们使用`onException`方法来处理`IllegalArgumentException`发生的时间，并应用一些特定的处理。**

对于我们的例子，我们将处理传递给一个定制的`ExceptionLoggingProcessor`类，该类只记录消息头。最后，在将结果发送到一个名为`handled`的模拟端点之前，我们使用`handled(true)`方法将消息交换标记为已处理。

然而，我们应该注意，在 Camel 中，我们代码的全局范围是每个`RouteBuilder`实例。因此，如果我们想通过多个`RouteBuilder`类共享这个异常处理代码，我们可以使用下面的技术。

**简单地创建一个基础抽象`RouteBuilder`类，并将错误处理逻辑放在它的`configure`方法**中。

随后，我们可以简单地扩展这个类，并确保我们调用了`super.configure()` 方法。本质上，我们只是使用了 Java 继承技术。

## 6。结论

在本文中，我们学习了如何处理路由中的异常。首先，我们创建了一个简单的 Camel 应用程序，用几条路线来了解异常。

然后我们学习了使用`doTry()`和`doCatch()`块语法的两种具体方法，以及后来的`onException()`子句。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221109115315/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-camel)