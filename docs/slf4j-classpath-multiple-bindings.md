# SLF4J 警告:类路径包含多个 SLF4J 绑定

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/slf4j-classpath-multiple-bindings>

## 1。概述

当我们在应用程序中使用 SLF4J 时，我们有时会在控制台上看到一条关于类路径中多个绑定的警告消息。

在本教程中，我们将尝试理解为什么我们会看到这个消息，以及如何解决它。

## 2.理解警告

首先，让我们看一个示例警告:

```java
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:.../slf4j-log4j12-1.7.21.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:.../logback-classic-1.1.7.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
```

这个警告告诉我们 SLF4J 发现了两个绑定。一个在`slf4j-log4j12-1.7.21.jar`，一个在`logback-classic-1.1.7.jar`。

现在我们来理解一下为什么会看到这个警告。

Java (SLF4J) 的[简单日志门面作为各种](/web/20220524024034/https://www.baeldung.com/slf4j-with-log4j2-logback)[日志](/web/20220524024034/https://www.baeldung.com/java-logging-intro)框架的简单门面或抽象。它允许我们在部署时插入我们想要的日志框架。

为了实现这一点，SLF4J 在类路径上寻找绑定(又名提供者)。绑定基本上是特定 SLF4J 类的实现，旨在扩展以插入特定的日志记录框架。

按照设计，SLF4J 一次只能绑定一个日志框架。因此，如果类路径中存在多个绑定，它将发出警告。

值得注意的是，像库或框架这样的嵌入式组件不应该声明对任何 SLF4J 绑定的依赖。这是因为当库声明对 SLF4J 绑定的编译时依赖时，它将该绑定强加给了最终用户。显然，这否定了 SLF4J 的基本目的。所以，它们应该只依赖于`slf4j-api`库`.`

同样重要的是**注意这只是一个警告。**如果 SLF4J 找到多个绑定，它将从列表中选择一个日志框架并与之绑定。从警告的最后一行可以看出，SLF4J 通过使用`org.slf4j.impl.Log4jLoggerFactory`选择了 Log4j 进行实际绑定。

## 3。寻找冲突的罐子

该警告列出了它找到的所有绑定的位置。通常，这些信息足以识别不道德的依赖项，这些依赖项将不需要的 SLF4J 绑定转移到我们的项目中。

如果无法从警告中识别出依赖关系，我们可以使用`dependency:tree` maven 目标:

```java
mvn dependency:tree
```

这将显示项目的依赖关系树:

```java
[INFO] +- org.docx4j:docx4j:jar:3.3.5:compile 
[INFO] |  +- org.slf4j:slf4j-log4j12:jar:1.7.21:compile 
[INFO] |  +- log4j:log4j:jar:1.2.17:compile 
[INFO] +- ch.qos.logback:logback-classic:jar:1.1.7:compile 
[INFO] +- ch.qos.logback:logback-core:jar:1.1.7:compile 
```

我们使用 Logback 来登录我们的应用程序。因此，我们特意添加了 Logback 绑定，存在于`logback-classic` JAR 中。但是`docx4j`依赖项还引入了另一个与`slf4j-log4j12` JAR 的绑定。

## 4。分辨率

既然我们知道了有问题的依赖项，我们只需要从`docx4j`依赖项中排除`slf4j-log4j12` JAR:

```java
<dependency>
    <groupId>org.docx4j</groupId>
    <artifactId>docx4j</artifactId>
    <version>${docx4j.version}</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

因为我们不打算使用 Log4j，所以排除它也是一个好主意。

## 5。结论

在本文中，我们看到了如何解决常见的由 SLF4J 发出的关于多重绑定的警告。

本文附带的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220524024034/https://github.com/eugenp/tutorials/tree/master/logging-modules/logback)