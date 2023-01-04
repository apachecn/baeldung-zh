# Linux 与 Windows 中的 Java 类路径语法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classpath-syntax>

## 1.概观

[类路径](https://web.archive.org/web/20221208143917/https://en.wikipedia.org/wiki/Classpath)是 Java 世界中的一个基本概念。当我们编译或启动一个 Java 应用程序时，JVM 会在类路径中找到并加载这些类。

我们可以通过`java/` j `avac`命令的`-cp`选项或者通过`CLASSPATH`环境变量来定义类路径中的元素。无论我们采用哪种方法来设置类路径，我们都需要遵循类路径语法。

在这个快速教程中，我们将讨论类路径语法，特别是 Windows 和 Linux 操作系统上的类路径分隔符。

## 2.类路径分隔符

类路径语法实际上非常简单:由路径分隔符分隔的路径列表。然而，[路径分隔符](/web/20221208143917/https://www.baeldung.com/java-file-vs-file-path-separator#path-separator)本身是系统相关的。

**而分号(；)在 Microsoft Windows 系统上用作分隔符，冒号(`:`)在类似 Unix 的系统上使用:**

```
# On Windows system:
CLASSPATH="PATH1;PATH2;PATH3"

# On Linux system:
CLASSPATH="PATH1:PATH2:PATH3"
```

## 3.令人误解的 Linux 手册页

我们已经知道，类路径分隔符可以根据操作系统的不同而不同。

然而，如果我们仔细看看 Linux 上的 Java `man`页面，它说类路径分隔符是分号(`;`)。

例如，来自最新(版本 17) OpenJDK 的`java`命令的`man`页面显示:

> `–class-path`类路径、`-classpath`类路径或`-cp`类路径
> 分号(`;`)分隔的目录、JAR 档案和 ZIP 档案列表，用于搜索类文件。
> …

此外，我们可以在甲骨文 JDK 公司的手册中找到确切的文本。

这是因为 Java 目前对不同的系统使用相同的手册内容。一个相应的 [bug 问题](https://web.archive.org/web/20221208143917/https://bugs.openjdk.java.net/browse/JDK-8262004)已经在今年早些时候被创建。

此外，Java 清楚地证明了路径分隔符依赖于`File`类的`[pathSeparatorChar](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#pathSeparatorChar)`字段。

## 4.结论

在这篇短文中，我们讨论了不同操作系统上的类路径语法。

此外，我们在 Linux 上的 Java 手册页中讨论了一个关于路径分隔符的错误。

我们应该记住，路径分隔符是系统相关的。在类似 Unix 的系统上使用冒号，而在 Microsoft Windows 系统上使用分号。