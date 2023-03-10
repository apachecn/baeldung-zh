# 如何找到 JAVA_HOME

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/find-java-home>

## 1。简介

在这个快速教程中，我们将学习如何在 Windows、Mac 和 Linux 上找到`[JAVA_HOME](/web/20220629074924/https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux) `。

众所周知，`JAVA_HOME`是我们常用来定位 java 可执行文件的环境变量，比如`java` 和`[javac](/web/20220629074924/https://www.baeldung.com/javac)`。

## 2。Windows 特有的方式找到 **`JAVA_HOME`**

如果我们使用 Windows 作为操作系统，首先我们需要打开命令行(`cmd`)并键入:

```java
echo %JAVA_HOME%
```

**如果`JAVA_HOME`是在我们的环境中定义的，**那么上面的命令会把它打印出来。

或者为了显示`java `可执行文件的位置，我们可以尝试:

```java
where java
```

## 3。macOS 和 Linux 特有的方式找到`JAVA_HOME`

如果我们使用的是 macOS 或 Linux，我们可以打开终端并键入:

```java
echo $JAVA_HOME
```

**如果`JAVA_HOME`是在我们的环境中定义的，**那么上面的命令会把它打印出来。

或者我们可以试试:

```java
which java
```

这可能只是向我们展示了`/usr/bin/java,` 这实际上并没有太大的帮助，因为它是一个符号链接。为了解决这个问题，我们将使用`dirname`和`readlink.`

对于 Linux:

```java
dirname $(dirname $(readlink -f $(which javac)))
```

对于 macOS:

```java
$(dirname $(readlink $(which javac)))/java_home
```

该命令打印当前使用的 java 文件夹。

## 4。用 Java 找到`JAVA_HOME`

如果我们能够自己运行`java`，那么**我们也有一个几乎独立于平台的方法:**

```java
java -XshowSettings:properties -version
```

运行这个命令输出许多属性，其中一个是`java.home.`

但是要解析它，我们仍然需要一个特定于平台的工具。

对于 Linux 和 macOS `,`，我们将使用`grep`:

```java
java -XshowSettings:properties -version 2>&1 > /dev/null | grep 'java.home' 
```

对于 Windows，我们将使用`findstr`:

```java
java -XshowSettings:properties -version 2>&1 | findstr "java.home"
```

## 5。结论

在这篇简短的文章中，我们学习了如何在不同的操作系统上**找到`JAVA_HOME`。**

如果这不起作用，有可能是我们在安装 Java 时没有正确设置`JAVA_HOME`变量。