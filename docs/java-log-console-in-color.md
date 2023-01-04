# 如何以彩色方式登录控制台

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-log-console-in-color>

## 1.介绍

添加一些颜色可以使日志更容易阅读。

在本文中，我们将了解如何为控制台(如 Visual Studio 代码终端、Linux 和 Windows 命令提示符)的日志添加颜色。

在开始之前，我们要注意，不幸的是，Eclipse IDE 控制台中只有有限的颜色设置。Eclipse IDE 中的控制台不支持由 Java 代码决定的颜色，所以本文中介绍的解决方案在 Eclipse IDE 控制台中无法工作。

## 2.如何使用 ANSI 代码给日志着色

**实现多彩日志记录的最简单方法是使用 [ANSI 转义序列，](https://web.archive.org/web/20221208143830/https://en.wikipedia.org/wiki/ANSI_escape_code)** 通常称为 ANSI 代码。

ANSI 代码是特殊的字节序列，一些终端将其解释为命令。

让我们用一个 ANSI 代码注销:

```java
System.out.println("Here's some text");
System.out.println("\u001B[31m" + "and now the text is red");
```

在输出中，我们看到 ANSI 代码没有打印出来，字体的颜色变成了红色:

```java
Here's some text
and now the text is red
```

请注意，我们需要确保在完成登录后重置字体颜色。

幸运的是，这很容易。我们可以简单的打印`\u001B[31m`，这是 ANSI reset 命令。

reset 命令会将控制台重置为默认颜色。请注意，这不一定是黑色，也可以是白色或控制台配置的任何其他颜色。例如:

```java
System.out.println("Here's some text");
System.out.println("\u001B[31m" + "and now the text is red" + "\u001B[0m");
System.out.println("and now back to the default");
```

给出输出:

```java
Here's some text
and now the text is red
and now back to the default
```

大多数日志库将遵循 ANSI 代码，这允许我们构建一些丰富多彩的日志程序。

例如，我们可以快速构建一个日志记录器，为不同的日志级别使用不同的颜色。

```java
public class ColorLogger {

    private static final Logger LOGGER = LoggerFactory.getLogger(ColorLogger.class);

    public void logDebug(String logging) {
        LOGGER.debug("\u001B[34m" + logging + "\u001B[0m");
    }
    public void logInfo(String logging) {
        LOGGER.info("\u001B[32m" + logging + "\u001B[0m");
    }

    public void logError(String logging) {
        LOGGER.error("\u001B[31m" + logging + "\u001B[0m");
    }
}
```

让我们用它来打印一些颜色到控制台:

```java
ColorLogger colorLogger = new ColorLogger();
colorLogger.logDebug("Some debug logging");
colorLogger.logInfo("Some info logging");
colorLogger.logError("Some error logging");
```

```java
[main] DEBUG com.baeldung.color.ColorLogger - Some debug logging
[main] INFO com.baeldung.color.ColorLogger - Some info logging
[main] ERROR com.baeldung.color.ColorLogger - Some error logging
```

我们可以看到，每个日志级别都有不同的颜色，这使得我们的日志可读性更好。

最后，ANSI 代码不仅可以用来控制字体颜色，还可以控制背景颜色、字体粗细和样式。在示例项目中选择了这些 ANSI 代码。

## 3.如何在 Windows 命令提示符下为日志着色

不幸的是，有些终端不支持 ANSI 代码。一个主要的例子是 Windows 命令提示符，上面的不会工作。因此，我们需要一个更复杂的解决方案。

然而，我们可以利用 pom.xml 的一个名为 [JANSI](https://web.archive.org/web/20221208143830/https://github.com/fusesource/jansi) 的已有库，而不是尝试自己实现它:

```java
<dependency>
    <groupId>org.fusesource.jansi</groupId>
    <artifactId>jansi</artifactId>
    <version>2.4.0</version>
</dependency>
```

现在要登录 color，我们只需调用 JANSI 提供的 ANSI API:

```java
private static void logColorUsingJANSI() {
    AnsiConsole.systemInstall();

    System.out.println(ansi()
        .fgRed()
        .a("Some red text")
        .fgBlue()
        .a(" and some blue text")
        .reset());

    AnsiConsole.systemUninstall();
}
```

这会产生以下文本:

```java
Some red text and some blue text
```

请注意，**我们必须先安装`AnsiConsole`，完成后卸载它。**

与 ANSI 代码一样，JANSI 也提供了大量的日志格式。

JANSI 通过检测正在使用的终端并调用所需的适当的特定于平台的 API 来实现这一功能。这意味着当 JANSI 检测到 Windows 命令提示符时，它不会使用不起作用的 ANSI 代码，而是调用使用 [Java 本地接口(JNI)](https://web.archive.org/web/20221208143830/https://baeldung-cn.com/java-native) 方法的库。

此外，JANSI 不仅仅在 Windows 命令提示符下工作——它能够覆盖大多数终端(尽管 Eclipse IDE 控制台不在其中，因为 Eclipse 中对彩色文本的设置有限)。

最后，JANSI 还将确保在环境不需要时删除不需要的 ANSI 代码，帮助保持我们的日志整洁。

总的来说，JANSI 为我们提供了一种强大而方便的方法，可以在大多数环境和终端中使用彩色登录。

## 4.结论

在本文中，我们学习了如何使用 ANSI 代码来控制控制台字体的颜色，并看到了如何使用颜色来区分日志级别的示例。

最后，我们发现并非所有的控制台都支持 ANSI 代码，并强调了一个这样的库，JANSI，它提供了更复杂的支持。

与往常一样，GitHub 上的示例项目[可用。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-console)