# JAVA_HOME 应该指向 JDK 而不是 JRE

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-java-home-jdk-jre>

## 1。简介

在本教程中，我们将讨论 Maven 在配置错误时抛出的一个异常:`JAVA_HOME should point to a JDK, not a JRE.`

Maven 是构建代码的强大工具。我们将深入了解这个错误发生的原因，并看看如何解决它。

## 2。`JAVA_HOME`问题

安装 Maven 之后，**我们必须设置`JAVA_HOME`环境变量，这样工具就知道在哪里可以找到要执行的 JDK 命令**。Maven 目标针对项目的源代码运行适当的 Java 命令。

例如，最常见的场景是通过执行`javac`命令来编译代码。

如果 **`JAVA_HOME`没有指向有效的 JDK 安装**，Maven 将在每次执行时抛出一个错误:

```java
mvn -version

# Output... 
The JAVA_HOME environment variable is not defined correctly
This environment variable is needed to run this program
NB: JAVA_HOME should point to a JDK, not a JRE
```

## 3。JDK 或 JRE

Maven 如何验证`JAVA_HOME`路径？

在运行任何目标之前， **Maven 检查由`JAVA_HOME`指定的路径中是否存在`java`命令**，或者向操作系统请求默认的 JDK 安装。**如果没有找到可执行文件，Maven 会终止并显示错误。**

下面是针对 Linux 的`mvn`可执行文件检查(Apache Maven v3.5.4):

```java
if [ -z "$JAVA_HOME" ] ; then
    JAVACMD=`which java`
else
    JAVACMD="$JAVA_HOME/bin/java"
fi

if [ ! -x "$JAVACMD" ] ; then
    echo "The JAVA_HOME environment variable is not defined correctly" >&2
    echo "This environment variable is needed to run this program" >&2
    echo "NB: JAVA_HOME should point to a JDK not a JRE" >&2
    exit 1
fi
```

乍一看，这个检查似乎很合理，但是我们必须考虑到**JDK 和 JRE 都有一个`bin`文件夹，并且都包含一个可执行的`java`文件。**

因此，**可以配置`JAVA_HOME`指向一个 JRE 安装，隐藏这个特定的错误，并在以后的阶段引起问题。**虽然 JRE 的主要目的是运行 Java 代码，**JDK 也可以编译和调试** [以及其他重要的区别](/web/20220627090522/https://www.baeldung.com/jvm-vs-jre-vs-jdk)。

由于这个原因，`mvn compile`命令会失败——我们将在下一小节中看到。

### 3.1。编译失败是因为 JRE 而不是 JDK

和通常的默认设置一样，如果我们有一个“标准”配置，它们会很有帮助。

例如，如果我们在 Ubuntu 18.04 系统上安装 Java 11，并且不设置`JAVA_HOME`环境变量，Maven 仍然会很高兴地找到我们的 JDK，并将其用于不同的目标，包括编译。

但是如果我们设法在系统上建立了一个非标准的配置(更不用说搞得一团糟)，Maven 的帮助就不再足够了。它甚至会误导人。

假设我们在 Ubuntu 18.04 上有以下配置:

*   JDK 11
*   JRE 8
*   `JAVA_HOME`设置 JRE 8 安装的路径

如果我们做基本检查:

```java
mvn --version
```

我们将得到如下有意义的输出:

```java
Apache Maven 3.6.0 (97c98ec64a1fdfee7767ce5ffb20918da4f719f3; 2018-10-24T18:41:47Z)
Maven home: /opt/apache-maven-3.6.0
Java version: 1.8.0_191, vendor: Oracle Corporation, runtime: /usr/lib/jvm/java-8-openjdk-amd64/jre
Default locale: en, platform encoding: UTF-8
OS name: "linux", version: "4.15.0-42-generic", arch: "amd64", family: "unix"
```

让我们看看如果我们试图编译一个项目会发生什么:

```java
mvn clean compile
```

现在，我们得到一个错误:

```java
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.7.0:compile (default-compile) on project my-app: Compilation failure
[ERROR] No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?
```

### 3.2。在项目级别修复编译错误

与 Maven 的许多其他情况一样，建议设置有意义的系统级设置——在这种情况下，我们将如第 5 节所述更改`JAVA_HOME`变量的值，以指向 JDK 而不是 JRE。

然而，如果我们因为某种原因不能设置默认值，我们仍然可以覆盖项目级别的设置。让我们看看如何做到这一点。

首先，我们将打开我们项目的`pom.xml`，转到`build / pluginManagement / plugins`部分，看看`maven-compiler-plugin`的条目:

```java
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.7.0</version>
</plugin>
```

然后，我们将向它添加一个配置，以便它使用一个定制的可执行文件，并跳过在`JAVA_HOME/bin`目录中搜索`javac`:

```java
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.7.0</version>
    <configuration>
        <fork>true</fork>
        <executable>/usr/lib/jvm/java-11-openjdk-amd64/bin/javac</executable>
    </configuration>
</plugin>
```

在 Unix 系统上，这个可执行文件应该是一个具有足够权限的脚本。在 Windows 系统上，它应该是一个`.exe`文件。

接下来，我们将再次尝试编译该项目:

```java
mvn clean compile
```

现在，构建——包括使用`maven-compiler-plugin`的编译阶段——成功了。

## 4。检查`JAVA_HOME`配置

检查**是否指向一个真实的 JDK 非常简单。**我们既可以在终端中打印它的内容，也可以运行以下 shell 命令之一:

### 4.1。在 Linux 上检查`JAVA_HOME`

只需打开一个终端并键入:

```java
> $JAVA_HOME/bin/javac -version
```

如果`JAVA_HOME`指向 JDK，输出应该是这样的:

```java
> javac 1.X.0_XX
```

如果`JAVA_HOME`没有指向 JDK，操作系统将抛出一条错误消息:

```java
> bash: /bin/javac: No such file or directory
```

### 4.2。在视窗上检查`JAVA_HOME`

打开命令提示符并键入:

```java
%JAVA_HOME%\bin\javac -version
```

如果`JAVA_HOME`指向 JDK，输出应该是这样的:

```java
> javac 1.X.0_XX
```

如果`JAVA_HOME`没有指向 JDK，操作系统将抛出一条错误消息:

```java
> The system cannot find the path specified.
```

## 5。如何解决问题

首先，我们需要知道在哪里可以找到我们的 JDK:

*   如果我们使用软件包安装程序安装了我们的 JDK 发行版，我们应该能够使用操作系统搜索工具找到路径
*   如果发行版是可移植的，让我们检查一下我们解压它的文件夹

**一旦我们知道了到 JDK 的路径，我们就可以设置我们的`JAVA_HOME`环境变量**，使用[我们特定操作系统](/web/20220627090522/https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux)的适当结果。

## 6。结论

在这个简短的教程中，我们讨论了"`JAVA_HOME should point to a JDK not a JRE” ` Maven 错误并分析了其根本原因。

最后，我们讨论了如何检查您的`JAVA_HOME`环境变量，以及如何确保它指向一个 JDK。