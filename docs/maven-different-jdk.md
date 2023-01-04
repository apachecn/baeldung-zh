# 为什么 Maven 使用不同的 JDK

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-different-jdk>

## 1.概观

在本教程中，我们将解释为什么 Maven 可能使用不同于系统中默认设置的 Java 版本。此外，我们将展示 Maven 的配置文件的位置。然后，我们将解释如何在 Maven 中配置 Java 版本。

## 2.Maven 配置

首先，让我们来看看一个可能的系统配置，其中 [Maven](/web/20220628090731/https://www.baeldung.com/maven) 使用了不同于系统中默认设置的 Java 版本。Maven 配置返回:

```
$ mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T17:41:47+01:00)
Maven home: C:\Users\test\apps\maven\3.3.9
Java version: 11.0.10, vendor: Oracle Corporation
Java home: C:\my\java\jdk-11.0.10
Default locale: pl_PL, platform encoding: Cp1250
OS name: "windows 10", version: "10.0", arch: "amd64", family: "dos" 
```

正如我们所看到的，它返回 Maven 版本、Java 版本和 OS 信息。Maven 工具使用 JDK 版本 11.0.10。

现在让我们看看我们系统中设置的 Java 版本:

```
$ java -version
java version "13.0.2" 2020-01-14
Java(TM) SE Runtime Environment (build 13.0.2+8)
Java HotSpot(TM) 64-Bit Server VM (build 13.0.2+8, mixed mode, sharing) 
```

默认 JDK 设置为 13.0.2。在下面几节中，我们将解释为什么它与 Maven 使用的不匹配。

## 3.全局设置`JAVA_HOME`

让我们来看看默认设置。综上所述，****[`JAVA_HOME`](/web/20220628090731/https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux) 变量是一个强制的 Maven 配置**。此外，如果没有设置，`mvn`命令会返回一条错误消息:**

```
$ mvn
Error: JAVA_HOME not found in your environment.
Please set the JAVA_HOME variable in your environment to match the
location of your Java installation.
```

在 Linux 上，我们用[命令`export`和](/web/20220628090731/https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux#linux)来设置系统变量。Windows 有专门的[系统变量设置](/web/20220628090731/https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux#windows)用于此目的。当它被全局设置时，Maven 使用系统中设置的默认 Java 版本。

## 4.Maven 配置文件

现在让我们快速看一下在哪里可以找到 Maven 配置文件。有几个地方可以提供配置:

*   `.mavenrc/mavenrc_pre.cmd`–位于用户主目录中的用户定义脚本
*   `settings.xml`–位于`~/.m2`目录中的文件，包含跨项目的配置
*   `.mvn`–包含项目内配置的目录

另外，我们可以使用`MAVEN_OPTS` 环境变量来设置 JVM 启动参数。

## 5.仅为 Maven 设置`JAVA_HOME`

我们看到 Java 版本不同于 Maven 使用的版本。换句话说， **Maven 覆盖了由`JAVA_HOME`变量**提供的默认配置。

它可以在`mvn`命令开始时执行的用户定义脚本中设置。**在 Windows 中，我们将其设置在`%HOME%\mavenrc_pre.bat`或`%HOME%\mavenrc_pre.cmd`文件**中。Maven 两者都支持。蝙蝠和。cmd '文件。在文件中，我们简单地设置了`JAVA_HOME`变量:

```
set JAVA_HOME="C:\my\java\jdk-11.0.10"
```

另一方面， **Linux 有用于相同目的的`$HOME/.mavenrc` 文件**。这里，我们以几乎相同的方式设置变量:

```
JAVA_HOME=C:/my/java/jdk-11.0.10
```

由于这种设置，Maven 使用 JDK 11，尽管系统中默认的是 JDK 13。

**我们可以用`MAVEN_SKIP_RC`标志**跳过用户自定义脚本的执行。

此外，我们可以直接在 Maven 的可执行文件中设置变量。然而，不推荐这种方法，因为如果我们将 Maven 升级到更高的版本，它不会自动应用。

## 6.结论

在这篇短文中，我们解释了 Maven 如何使用不同于默认版本的 Java 版本。然后，我们展示了 Maven 的配置文件所在的位置。最后，我们解释了如何为 Maven 设置 Java 版本。**