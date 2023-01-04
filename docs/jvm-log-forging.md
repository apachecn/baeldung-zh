# JVM 日志伪造

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-log-forging>

## 1。概述

在这篇简短的文章中，我们将探讨世界上最常见的安全问题之一——日志伪造。我们还将展示一种示例技术，它可以保护我们免受这种安全问题的影响。

## 2。什么是原木锻造？

据`[OWASP](https://web.archive.org/web/20220831223237/https://www.owasp.org/)`介绍，日志伪造是最常见的攻击技术之一。

当数据从不受信任的来源进入应用程序，或者数据被某些外部实体写入应用程序/系统日志文件时，就会出现日志伪造漏洞。

根据 [`OWASP`指南](https://web.archive.org/web/20220831223237/https://owasp.org/www-community/attacks/Log_Injection)日志伪造或注入是一种将未经验证的用户输入写入日志文件的技术，因此它可以允许攻击者伪造日志条目或向日志中注入恶意内容。

简单来说，通过日志伪造，攻击者试图通过探索应用程序中的安全漏洞来添加/修改记录内容。

## 3。示例

考虑一个用户从 web 提交支付请求的例子。在应用程序级别，一旦该请求得到处理，将记录一个条目，其金额为:

```java
private final Logger logger 
  = LoggerFactory.getLogger(LogForgingDemo.class);

public void addLog( String amount ) {
    logger.info( "Amount credited = {}" , amount );
}

public static void main( String[] args ) {
    LogForgingDemo demo = new LogForgingDemo();
    demo.addLog( "300" );
}
```

如果我们查看控制台，我们会看到类似这样的内容:

```java
web - 2017-04-12 17:45:29,978 [main] 
  INFO  com.baeldung.logforging.LogForgingDemo - Amount credited = 300
```

现在，假设攻击者以`“\n\nweb – 2017-04-12 17:47:08,957 [main] INFO Amount reversed successfully”,` 的形式提供输入，那么日志将会是:

```java
web - 2017-04-12 17:52:14,124 [main] INFO  com.baeldung.logforging.
  LogForgingDemo - Amount credited = 300

web - 2017-04-12 17:47:08,957 [main] INFO Amount reversed successfully
```

攻击者故意在应用程序日志中创建了一个伪造的条目，这破坏了日志的值，并在将来混淆了任何审计类型的活动。这就是原木锻造的精髓。

## 4。预防

最明显的解决方案是不要将任何用户输入写入日志文件。

但是，这不可能在所有情况下都实现，因为用户给定的数据是将来调试或审计应用程序活动所必需的。

我们必须使用其他替代方案来处理这种情况。

### 4.1。引入验证

最简单的解决方案之一是在记录之前验证输入。这种方法的一个问题是，我们必须在运行时验证大量数据，这将影响整个系统的性能。

此外，如果验证失败，数据将不会被记录并永远丢失，这通常是不可接受的情况。

### 4.2。数据库记录

另一种选择是将数据记录到数据库中。这比另一种方法更安全，因为`‘\n'`或 newline 对这个上下文没有任何意义。然而，这将引发另一个性能问题，因为大量的数据库连接将用于记录用户数据。

更重要的是，这种技术引入了另一个安全漏洞——即 [`SQL Injection`](https://web.archive.org/web/20220831223237/https://owasp.org/www-community/attacks/SQL_Injection) 。为了解决这个问题，我们可能会写很多额外的代码。

### 4.3。ESAPI

在这种情况下，使用`ESAPI`是最常见和最可取的技术。在这里，每个用户数据在写入日志之前都要进行编码。`ESAPI` 是来自`OWASP`的开源 API:

```java
<dependency>
    <groupId>org.owasp.esapi</groupId>
    <artifactId>esapi</artifactId>
    <version>2.5.0.0</version>
</dependency>
```

它可以在[中央 Maven 资源库](https://web.archive.org/web/20220831223237/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.owasp.esapi%22%20AND%20a%3A%22esapi%22)中获得。

我们可以使用`ESAPI`的 [`Encoder`](https://web.archive.org/web/20220831223237/https://static.javadoc.io/org.owasp.esapi/esapi/2.0.1/org/owasp/esapi/Encoder.html) 接口对数据进行编码:

```java
public String encode(String message) {
    message = message.replace( '\n' ,  '_' ).replace( '\r' , '_' )
      .replace( '\t' , '_' );
    message = ESAPI.encoder().encodeForHTML( message );
    return message;
}
```

这里，我们创建了一个包装器方法，它用下划线替换所有回车和换行符，并对修改后的消息进行编码。

在前面的示例中，如果我们使用此包装函数对消息进行编码，日志应该类似于以下内容:

```java
web - 2017-04-12 18:15:58,528 [main] INFO  com.baeldung.logforging.
  LogForgingDemo - Amount credited = 300
__web - 2017-04-12 17:47:08,957 [main] INFO Amount reversed successfully
```

在这里，损坏的字符串片段被编码，可以很容易地识别。

需要注意的重要一点是，要使用`ESAPI`，我们需要在类路径中包含`ESAPI.properties`文件，否则`ESAPI` API 将在运行时抛出异常。这里有[。](https://web.archive.org/web/20220831223237/https://github.com/OWASP/EJSF/blob/master/esapi_master_FULL/WebContent/ESAPI.properties)

## 5。结论

在这个快速教程中，我们学习了日志伪造和克服这个安全问题的技术。

像往常一样，完整的源代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220831223237/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm)