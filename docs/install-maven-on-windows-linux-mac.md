# 如何在 Windows、Linux 和 Mac 上安装 Maven

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/install-maven-on-windows-linux-mac>

## 1。概述

简单地说，Maven 是一个命令行工具，用于构建和管理任何基于 Java 的项目。

为了方便起见，Maven 项目提供了一个简单的 ZIP 文件，其中包含 Maven 的预编译版本。没有安装程序。由我们来设置运行 Maven 的先决条件和环境。

## 延伸阅读:

## [Apache Maven 教程](/web/20220701024215/https://www.baeldung.com/maven)

A quick and practical guide to building and managing Java projects using Apache Maven.[Read more](/web/20220701024215/https://www.baeldung.com/maven) →

## [Maven 依赖范围](/web/20220701024215/https://www.baeldung.com/maven-dependency-scopes)

A quick and practical guide to dependency scopes in Maven.[Read more](/web/20220701024215/https://www.baeldung.com/maven-dependency-scopes) →

## [如何用 Maven 创建可执行的 JAR](/web/20220701024215/https://www.baeldung.com/executable-jar-with-maven)

A quick and practical guide to creating executable JARs with Maven[Read more](/web/20220701024215/https://www.baeldung.com/executable-jar-with-maven) →

Apache Maven 的安装是一个简单的过程，即提取归档文件，然后配置 Maven，使`mvn`可执行文件在 OS 类路径中可用。

### 1.1。先决条件

Maven 是用 Java 写的。因此，要运行 Maven，我们需要一个安装并正确配置了 Java 的系统。例如，我们可以从甲骨文的下载网站下载兼容操作系统的 Java JDK。建议将其安装到不带空格的路径名中。

一旦安装了 Java，我们需要确保来自 Java JDK 的命令在我们的`PATH`环境变量中。

为此，我们将运行下面的命令来获取当前安装的版本信息:

```java
java -version
```

## 2。在 Windows 上安装 Maven

为了在 Windows 上安装 Maven，我们前往 [Apache Maven 网站](https://web.archive.org/web/20220701024215/https://maven.apache.org/download.cgi)下载最新版本并选择 Maven zip 文件，例如`apache-maven-3.8.4-bin.zip`。

然后，我们将其解压缩到我们希望 Maven 所在的文件夹中。

### 2.1。将 Maven 添加到环境路径

我们使用系统属性将`M2_HOME`和`MAVEN_HOME`变量添加到 Windows 环境中，并将它们指向我们的 Maven 文件夹。

然后，我们通过追加 Maven `bin`文件夹— `%M2_HOME%\bin —`来更新`PATH`变量，这样我们就可以在任何地方运行 Maven 命令。

为了验证这一点，我们运行:

```java
mvn -version
```

上面的命令应该显示 Maven 版本、Java 版本和操作系统信息。就是这样。我们已经在 Windows 系统上安装了 Maven。

## 3。在 Linux 上安装 Maven

为了在 Linux 操作系统上安装 Maven，我们从 [Apache Maven 网站](https://web.archive.org/web/20220701024215/https://maven.apache.org/download.cgi)下载最新版本，并选择 Maven 二进制文件`tar.gz`，例如`apache-maven-3.8.4-bin.tar.gz`。

Redhat、Ubuntu 和许多其他 Linux 发行版都使用 BASH 作为它们的默认 shell。在下一节中，我们将使用 bash 命令。

首先，让我们为 Maven 创建一个位置:

```java
$ mkdir -p /usr/local/apache-maven/apache-maven-3.8.4
```

然后，我们将归档文件提取到我们的 Maven 位置:

```java
$ tar -xvf apache-maven-3.8.4-bin.tar.gz -C /usr/local/apache-maven/apache-maven-3.8.4
```

### 3.1。将 Maven 添加到环境路径

我们打开命令终端，使用下面的命令编辑`.bashrc`文件:

```java
$ nano ~/.bashrc
```

接下来，让我们将特定于 Maven 的行添加到文件中:

```java
export M2_HOME=/usr/local/apache-maven/apache-maven-3.8.4 
export M2=$M2_HOME/bin 
export MAVEN_OPTS=-Xms256m -Xmx512m 
export PATH=$M2:$PATH
```

保存文件后，我们可以重新加载环境配置，而无需重新启动:

```java
$ source ~/.bashrc
```

最后，我们可以验证是否添加了 Maven:

```java
$ mvn -version
```

输出应该类似于以下内容:

```java
Apache Maven 3.8.4 (81a9f75f19aa7275152c262bcea1a77223b93445; 2021-01-07T15:30:30+01:29)
Maven home: /usr/local/apache-maven/apache-maven-3.8.4

Java version: 1.8.0_75, vendor: Oracle Corporation

Java home: /usr/local/java-current/jdk1.8.0_75/jre
```

我们已经成功地在 Linux 系统上安装了 Maven。

### 3.2。在 Ubuntu 上安装 Maven

在终端中，我们运行`apt-cache search maven`来获取所有可用的 Maven 包:

```java
$ apt-cache search maven
....
libxmlbeans-maven-plugin-java-doc - Documentation for Maven XMLBeans Plugin
maven - Java software project management and comprehension tool
maven-debian-helper - Helper tools for building Debian packages with Maven
maven2 - Java software project management and comprehension tool
```

Maven 包总是附带最新的 Apache Maven。

我们运行命令`sudo apt-get install maven`来安装最新的 Maven:

```java
$ sudo apt-get install maven
```

这将需要几分钟来下载。下载完成后，我们可以运行`mvn -version`来验证我们的安装。

## 4。在 Mac OS X 上安装 Maven

为了在 Mac OS X 操作系统上安装 Maven，我们从 [Apache Maven 网站](https://web.archive.org/web/20220701024215/https://maven.apache.org/download.cgi)下载最新版本，并选择 Maven 二进制 tar.gz 文件，例如`apache-maven-3.8.4-bin.tar.gz`。

然后，我们将归档文件提取到我们想要的位置。

### 4.1。将 Maven 添加到环境路径

首先，让我们打开终端，切换到文件解压到的目录，然后以超级用户身份登录。

其次，我们需要删除`tar.gz`档案:

```java
rm Downloads/apache-maven*bin.tar.gz
```

第三，我们必须修复权限并切换 Maven 内容:

```java
chown -R root:wheel Downloads/apache-maven* 
mv Downloads/apache-maven* /opt/apache-maven
```

然后，让我们归档管理会话，并将 Maven 二进制文件添加到路径中，并附加:

```java
exit 
nano $HOME/.profile 
export PATH=$PATH:/opt/apache-maven/bin
```

最后，我们使用`Ctrl+x`保存并从`nano`退出。

要加载新的设置，让我们运行:

```java
bash
```

现在，我们使用下面的命令测试 Maven 是否安装成功:

```java
mvn -version
```

我们现在可以在 Mac OS X 上使用 Maven 了。

### 4.2。将 Maven 添加到 macOS Catalina 或更高版本的环境路径中

macOS 放弃了 Bourne-Again Shell ( `bash`)，这是大多数 GNU / Linux 发行版的命令解释器，转而支持 Z shell ( `zsh`)。这个 shell 可以被认为是 bash 的扩展版本。

Zsh 凭借其先进的命令完成机制、错别字纠正，甚至是添加功能的模块系统脱颖而出。

在 macOS Catalina 或更高版本的情况下，默认 shell 是`zsh,`,我们必须添加到不同的文件中:

```java
nano ~/.zshenv  
export PATH=$PATH:/opt/apache-maven/bin
```

要重新加载环境，我们需要发出:

```java
source ~/.zshenv 
```

其余的操作保持不变。

### 4.3。HighSierra 兼容性

对于 HighSierra，我们需要额外将 Maven 二进制文件添加到路径中，并追加:

```java
nano $HOME/.bashrc
export PATH=$PATH:/opt/apache-maven/bin
```

我们使用`Ctrl+x`来保存并从`nano.` 退出，然后我们运行`bash`来加载新的设置。

## 5。结论

在本文中，我们学习了如何在主流操作系统上安装 Maven 进行开发。

要了解如何开始使用 Spring 和 Maven，请在这里查看教程。