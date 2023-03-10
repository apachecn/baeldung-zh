# 为 WildFly 增加堆内存

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/wildfly-increase-heap>

## 1.概观

在这个快速教程中，我们将看看**如何为[野](https://web.archive.org/web/20220628153822/http://wildfly.org/)增加堆内存大小。**

自然，为了处理服务器上运行的大量应用程序，增加内存大小是很有用的

## 2.使用启动文件

在独立模式下，我们可以更新启动文件中的配置来增加堆内存。

在 WildFly 安装主目录中，我们可以在`bin`文件夹中找到`standalone.conf`(对于基于 Unix 的系统)或`standalone.conf.bat`(对于 Windows 系统)文件。

我们可以简单地打开它，更新下面一行中的`-Xmx`选项(根据需要更改数字):

```java
JAVA_OPTS="-Xms64m -Xmx512m ..."
```

这将在启动服务器时设置堆内存。

## 3.使用环境变量

我们还可以通过设置`JAVA_OPTS`环境变量来设置默认堆内存大小，这将覆盖启动文件中的值。我们可以从命令提示符/终端设置环境变量。

**对于 Windows:**

```java
set JAVA_OPTS=-Xms256m -Xmx1024m ...
```

**对于 Unix/Linux:**

```java
export JAVA_OPTS=-Xms256m -Xmx1024m ...
```

一旦设置完成，我们可以在启动 WildFly 服务器时在日志中看到结果:

```java
JAVA_OPTS already set in environment; overriding default settings with values ...
```

## 4.结论

在这个快速教程中，我们发现了如何在 WildFly 中很容易地改变堆内存大小。

您可以在我们的[上一篇文章](/web/20220628153822/https://www.baeldung.com/java-servers)中探索可用于 Java 开发的不同流行服务器。