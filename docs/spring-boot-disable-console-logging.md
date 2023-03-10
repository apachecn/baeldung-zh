# 如何在 Spring Boot 禁用控制台日志记录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-disable-console-logging>

## 1.概观

通常，控制台日志让我们有机会以简单直观的方式调试我们的系统。然而，有时我们不想在系统中启用这个特性。

在这个快速教程中，**我们将看到在运行 Spring Boot 应用程序**时如何避免登录到控制台。

无论我们使用的是 Logback、Log4js2，还是 Java Util 日志框架，我们都会用简单的例子来说明如何实现这一点。

要了解更多关于 Spring Boot 的日志框架，我们建议看看我们的[Spring Boot 日志](/web/20221205170113/https://www.baeldung.com/spring-boot-logging)教程。

## 2.如何禁用日志回退的控制台输出

如果我们的项目使用 [Spring Boot 启动器](/web/20221205170113/https://www.baeldung.com/spring-boot-starters)，那么`spring-boot-starter-logging` 依赖项也将被包含。

这个特殊的启动器将 [Logback](/web/20221205170113/https://www.baeldung.com/logback) 配置为默认框架，并且默认情况下最初只登录到控制台。

**这个配置可以通过向我们的资源添加一个`logback-spring.xml`文件来定制** **。**

例如，让我们设置 XML，以便禁用控制台输出并只记录到一个文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource=
      "org/springframework/boot/logging/logback/defaults.xml" />
    <include resource=
      "org/springframework/boot/logging/logback/file-appender.xml" />
    <root level="INFO">
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

此外，我们将需要在我们的`application.properties `文件中的`logging.file`配置属性:

```java
logging.file=baeldung-disabled-console.log
```

注意:实际上禁用控制台输出的原因是我们没有在 XML 文件中包含`console-appender.xml `，所以一个空的`configuration`标签也可以做到这一点。

**或者，** **我们可以通过用应用程序属性**覆盖默认配置来避免创建 XML 文件。

例如，我们可以潜在地利用`logging.pattern.console `属性:

```java
logging.pattern.console=
```

这个属性被转换成系统属性`CONSOLE_LOG_PATTERN`，然后被 Spring 默认控制台配置使用。

**当然，这种方法不如前一种方法**干净、可靠。这不是该属性的预期目的，因此这种“入侵”在某些时候可能不被 Logback 支持。

此外，我们可以通过将根日志记录器级别的值设置为`OFF`来禁用所有日志记录活动:

```java
logging.level.root=OFF
```

## 3.如何避免用 Log4j2 登录控制台

正如我们所知， [Log4j2](/web/20221205170113/https://www.baeldung.com/log4j2-appenders-layouts-filters) 支持 XML、JSON、YAML 或属性格式来配置其日志行为。

为了简单起见，我们这次只展示一个简单的`log4j2.xml`文件的例子。

其他格式遵循相同的配置结构:

```java
<Configuration status="INFO">
    <Appenders>
        <File name="MyFile" fileName="baeldung.log"
          immediateFlush="true" append="false">
        <PatternLayout pattern=
          "%d{yyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </File>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="MyFile"/>
        </Root>
    </Loggers>
</Configuration>
```

与回退设置一样，框架避免记录到控制台的原因不是配置本身，而是根记录器不包含对控制台 Appender 的引用。

## 4.如何禁用 Java Util 日志记录的控制台日志记录

Java Util 日志(或简称为“JUL”)可能不是当今 Spring Boot 应用程序最流行的日志解决方案。

无论如何，我们将分析如何摆脱控制台日志，以防框架出现在我们的项目中。

我们所要做的就是将以下值添加到资源文件夹的默认`logging.properties `中:

```java
handlers=java.util.logging.FileHandler
java.util.logging.FileHandler.pattern=baeldung.log
java.util.logging.FileHandler.formatter=java.util.logging.SimpleFormatter
```

并将`logging.file` 属性包含在我们的 `application.properties`文件中。任何值都可以:

```java
logging.file=true
```

## 5.结论

有了这些简短的例子，我们现在可以在应用程序中轻松地禁用控制台日志，不管我们使用的是什么日志框架。

和往常一样，我们可以在 Github 上找到示例的实现[。](https://web.archive.org/web/20221205170113/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-disable-logging)