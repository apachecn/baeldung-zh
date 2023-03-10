# Java 9 平台日志 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-logging-api>

## 1。简介

在本教程中，我们将探索 Java 9 中新引入的日志 API，并实现一些例子来涵盖最常见的情况。

这个 API 已经在 Java 中引入，以**提供一个公共机制来处理所有平台日志，并公开一个可由库和应用程序定制的服务接口。**通过这种方式，JDK 平台日志可以使用与应用程序相同的日志框架，并且可以减少项目依赖性。

## 2。创建自定义实现

在这一节中，我们将展示日志 API 的主要类，我们必须实现这些类来创建一个新的日志记录器。我们将通过实现一个简单的记录器来实现这一点，该记录器将所有日志打印到控制台。

### 2.1。创造`Logger`

我们必须创建的主要类是`Logger`。这个类至少要实现`System.Logger`接口和这四个方法:

*   `getName()`:返回记录器的名称。JDK 将使用它来按名称创建伐木工
*   `isLoggable()`:表示记录器启用的级别
*   这个方法将日志打印到应用程序正在使用的底层系统——在我们的例子中是控制台。有 2 个`log()`方法要实现，每个方法接收不同的参数

让我们看看我们的实现是什么样子的:

```java
public class ConsoleLogger implements System.Logger {

    @Override
    public String getName() {
        return "ConsoleLogger";
    }

    @Override
    public boolean isLoggable(Level level) {
        return true;
    }

    @Override
    public void log(Level level, ResourceBundle bundle, String msg, Throwable thrown) {
        System.out.printf("ConsoleLogger [%s]: %s - %s%n", level, msg, thrown);
    }

    @Override
    public void log(Level level, ResourceBundle bundle, String format, Object... params) {
        System.out.printf("ConsoleLogger [%s]: %s%n", level, 
          MessageFormat.format(format, params));
    }
}
```

我们的`ConsoleLogger`类覆盖了前面提到的四个方法。在所有情况下，`getName()`方法返回一个`String,`，而`isLoggable()`方法返回`true`。最后，我们有 2 个`log()`方法输出到控制台。

### 2.2.创建`LoggerFinder`

一旦我们创建了日志记录器，**，我们需要实现一个`LoggerFinder`来创建我们的`ConsoleLogger`的实例。**

为此，我们必须扩展抽象类`System.LoggerFinder`并实现`getLogger()`方法:

```java
public class CustomLoggerFinder extends System.LoggerFinder {

    @Override
    public System.Logger getLogger(String name, Module module) {
        return new ConsoleLogger();
    }
}
```

在这种情况下，我们总是返回我们的`ConsoleLogger`。

**最后，我们需要将我们的`LoggerFinder`注册为服务，这样它就可以被 JDK** 发现。如果我们不提供实现，默认情况下将使用`SimpleConsoleLogger`。

JDK 用来加载实现的机制是`ServiceLoader`。你可以在[本教程](/web/20221205165046/https://www.baeldung.com/java-spi)中找到更多相关信息。

由于我们使用的是 Java 9，我们将把我们的类打包到一个模块中，并在`module-info.java` 文件中注册我们的服务:

```java
module com.baeldung.logging {
    provides java.lang.System.LoggerFinder
      with com.baeldung.logging.CustomLoggerFinder;
    exports com.baeldung.logging;
}
```

关于 Java 模块的更多信息，请查看另一个教程。

### 2.3。测试我们的示例

为了测试我们的示例，让我们创建另一个模块作为应用程序。这将只包含使用我们服务实现的`Main`类。

这个类将通过调用`System.getLogger()`方法获得我们的`ConsoleLogger`的一个实例:

```java
public class MainApp {

    private static System.Logger LOGGER = System.getLogger("MainApp");

    public static void main(String[] args) {
        LOGGER.log(Level.ERROR, "error test");
        LOGGER.log(Level.INFO, "info test");
    }
}
```

**在内部，JDK 将获取我们的`CustomLoggerFinder`实现，并创建我们的`ConsoleLogger.`** 的实例

之后，让我们为这个模块创建`module-info`文件:

```java
module com.baeldung.logging.app {
}
```

此时，我们的项目结构将如下所示:

```java
├── src
│   ├── modules
│   │   ├── com.baeldung.logging
│   │   │   ├── com
│   │   │   │   └── baeldung
│   │   │   │       └── logging
│   │   │   │           ├── ConsoleLogger.java
│   │   │   │           └── CustomLoggerFinder.java
│   │   │   └── module-info.java
│   │   ├── com.baeldung.logging.app
│   │   │   ├── com
│   │   │   │   └── baeldung
│   │   │   │       └── logging
│   │   │   │           └── app
│   │   │   │               └── MainApp.java
│   │   │   └── module-info.java
└──
```

最后，我们将编译我们的两个模块，并将它们放在一个`mods`目录中:

```java
javac --module-path mods -d mods/com.baeldung.logging \
  src/modules/com.baeldung.logging/module-info.java \
  src/modules/com.baeldung.logging/com/baeldung/logging/*.java

javac --module-path mods -d mods/com.baeldung.logging.app \
  src/modules/com.baeldung.logging.app/module-info.java \
  src/modules/com.baeldung.logging.app/com/baeldung/logging/app/*.java
```

最后，让我们运行`app`模块的`Main`类:

```java
java --module-path mods \
  -m com.baeldung.logging.app/com.baeldung.logging.app.MainApp
```

如果我们看一下控制台输出，我们可以看到我们的日志是使用我们的`ConsoleLogger`打印的:

```java
ConsoleLogger [ERROR]: error test
ConsoleLogger [INFO]: info test
```

## 3。添加外部日志框架

在我们之前的例子中，我们将所有消息记录到控制台，这与默认记录器所做的相同。**Java 9 中日志 API 最有用的用途之一是让应用程序将 JDK 日志路由到应用程序正在使用的同一个日志框架**，这就是我们在本节将要做的。

我们将创建一个新模块，它使用 SLF4J 作为日志门面，使用 Logback 作为日志框架。

由于我们已经在前一节解释了基础知识，现在我们可以关注如何添加外部日志框架。

### 3.1。使用 SLF4J 的定制实现

首先，我们将实现另一个`Logger` ，它将为每个实例创建一个新的 SLF4J 记录器:

```java
public class Slf4jLogger implements System.Logger {

    private final String name;
    private final Logger logger;

    public Slf4jLogger(String name) {
        this.name = name;
        logger = LoggerFactory.getLogger(name);
    }

    @Override
    public String getName() {
        return name;
    }

    //...
}
```

**注意这个`Logger `是一个`org.slf4j.Logger`。**

对于其余的方法，**我们将依赖于 SLF4J 记录器实例**上的实现。因此，如果 SLF4J 记录器被启用，我们的`Logger`将被启用:

```java
@Override
public boolean isLoggable(Level level) {
    switch (level) {
        case OFF:
            return false;
        case TRACE:
            return logger.isTraceEnabled();
        case DEBUG:
            return logger.isDebugEnabled();
        case INFO:
            return logger.isInfoEnabled();
        case WARNING:
            return logger.isWarnEnabled();
        case ERROR:
            return logger.isErrorEnabled();
        case ALL:
        default:
            return true;
    }
}
```

并且日志方法将根据所使用的日志级别调用适当的 SLF4J 记录器方法:

```java
@Override
public void log(Level level, ResourceBundle bundle, String msg, Throwable thrown) {
    if (!isLoggable(level)) {
        return;
    }

    switch (level) {
        case TRACE:
            logger.trace(msg, thrown);
            break;
        case DEBUG:
            logger.debug(msg, thrown);
            break;
        case INFO:
            logger.info(msg, thrown);
            break;
        case WARNING:
            logger.warn(msg, thrown);
            break;
        case ERROR:
            logger.error(msg, thrown);
            break;
        case ALL:
        default:
            logger.info(msg, thrown);
    }
}

@Override
public void log(Level level, ResourceBundle bundle, String format, Object... params) {
    if (!isLoggable(level)) {
        return;
    }
    String message = MessageFormat.format(format, params);

    switch (level) {
        case TRACE:
            logger.trace(message);
            break;
        // ...
        // same as the previous switch
    }
}
```

最后，让我们创建一个新的使用我们的`Slf4jLogger`的`LoggerFinder`:

```java
public class Slf4jLoggerFinder extends System.LoggerFinder {
    @Override
    public System.Logger getLogger(String name, Module module) {
        return new Slf4jLogger(name);
    }
}
```

### 3.2。模块配置

一旦我们实现了所有的类，让我们在模块中注册我们的服务，并添加 SLF4J 模块的依赖项:

```java
module com.baeldung.logging.slf4j {
    requires org.slf4j;
    provides java.lang.System.LoggerFinder
      with com.baeldung.logging.slf4j.Slf4jLoggerFinder;
    exports com.baeldung.logging.slf4j;
}
```

该模块将具有以下结构:

```java
├── src
│   ├── modules
│   │   ├── com.baeldung.logging.slf4j
│   │   │   ├── com
│   │   │   │   └── baeldung
│   │   │   │       └── logging
│   │   │   │           └── slf4j
│   │   │   │               ├── Slf4jLoggerFinder.java
│   │   │   │               └── Slf4jLogger.java
│   │   │   └── module-info.java
└──
```

现在我们可以像在上一节中所做的那样，将这个模块编译到`mods`目录中。

注意，我们必须将 slf4j-api jar 放在 mods 目录中来编译这个模块。此外，请记住使用模块化版本的库。最新版本可以在 [Maven Central](https://web.archive.org/web/20221205165046/https://search.maven.org/search?q=g:org.slf4j%20AND%20a:slf4j-api) 找到。

### 3.3。添加回退

我们差不多完成了，但是我们仍然需要添加回退依赖项和配置。为此，将`logback-classic`和`logback-core` jar 放在`mods`目录中。

和以前一样，我们必须确保我们使用的是模块化版本的库。同样，最新版本可以在 [Maven Central](https://web.archive.org/web/20221205165046/https://search.maven.org/search?q=g:ch.qos.logback) 找到。

最后，让我们创建一个回退配置文件，并将其放在我们的`mods`目录中:

```java
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} -- %msg%n
            </pattern>
        </encoder>
    </appender>

    <root>
        <appender-ref ref="STDOUT"/>
    </root>

</configuration>
```

### 3.4。运行我们的应用程序

此时，我们可以使用我们的 SLF4J 模块运行我们的`app`。

在这种情况下，**我们还需要指定我们的回退配置文件**:

```java
java --module-path mods \
  -Dlogback.configurationFile=mods/logback.xml \
  -m com.baeldung.logging.app/com.baeldung.logging.app.MainApp
```

最后，如果我们检查输出，我们可以看到我们的日志是使用我们的日志回溯配置打印的:

```java
2018-08-25 14:02:40 [main] ERROR MainApp -- error test
2018-08-25 14:02:40 [main] INFO  MainApp -- info test
```

## 4。结论

在本文中，我们展示了如何使用新的平台日志 API 在 Java 9 中创建自定义日志记录器。此外，我们使用外部日志框架实现了一个示例，这是这个新 API 最有用的用例之一。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221205165046/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)