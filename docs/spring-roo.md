# Spring Roo 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-roo>

## 1。概述

Spring Roo 是一个快速应用程序开发(RAD)工具，旨在交付快速、即时的结果，重点是 Spring web 应用程序和更新的 Spring 技术。它允许我们用简单易用的命令为 Spring 应用程序生成样板代码和项目结构。

Roo 可以用作从操作系统命令行运行的独立应用程序。不需要使用 Eclipse、Spring Tool Suite (STS)或任何其他 IDE 事实上，我们可以使用任何文本编辑器来编写代码！

然而，为了简单起见，我们将使用带有 Roo 扩展的 STS IDE。

## 2。安装弹簧顶

### 2.1。要求

要学习本教程，必须安装以下软件:

1.  [爪哇 JDK 8](https://web.archive.org/web/20221129011945/http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
2.  [STS](https://web.archive.org/web/20221129011945/https://spring.io/tools)
3.  [弹簧屋顶](https://web.archive.org/web/20221129011945/https://spring.io/projects/spring-roo)

### 2.2。安装

一旦我们下载并安装了 Java JDK 和 STS，我们需要解压缩 Spring Roo 并将其添加到系统路径中。

让我们创建`ROO_HOME`环境变量并将`%ROO_HOME%\bin`添加到路径中。

为了验证 Roo 安装是否正确，我们可以打开命令行并执行以下命令:

```java
mkdir baeldung
cd baeldung
roo quit
```

几秒钟后我们会看到:

```java
 _
 ___ _ __  _ __(_)_ __   __ _   _ __ ___   ___
/ __| '_ \| '__| | '_ \ / _` | | '__/ _ \ / _ \
\__ \ |_) | |  | | | | | (_| | | | | (_) | (_) |
|___/ .__/|_|  |_|_| |_|\__, | |_|  \___/ \___/
    |_|                 |___/          2.0.0.RC1

Welcome to Spring Roo. For assistance press TAB or type "hint" then hit ENTER.
```

Roo 已经安装，并且正在运行。请注意，Spring Roo 版本会有所不同，步骤和说明可能取决于实际使用的版本。

重要:Spring Roo 2.0 不向后兼容 1.x。

### 2.3。添加和配置 STS 扩展

开箱即用，STS 支持 Spring 应用程序的开发，并包括现成的扩展。但是，Spring Roo 扩展不包括在内。因此，我们需要手动添加它。

在 STS 中，让我们转到`Install New Software`并将书签导入到`Available Software Sites`。目前，书签在`%ROO_HOME%\conf`文件夹中。一旦我们导入了书签，我们就可以简单地搜索`roo`并安装最新版本的`Spring IDE Roo Support`。最后，我们会被要求重启 STS。

要了解详细的最新步骤，我们可以随时查看 [Spring Roo 入门](https://web.archive.org/web/20221129011945/https://docs.spring.io/spring-roo/docs/2.0.x/reference/html/#getting-started-requirements)文档。

一旦我们在 STS 中安装了 Roo 支持，我们就需要设置扩展。这就像将 Roo 支持指向`%ROO_HOME%`文件夹一样简单。再次， [Spring Roo 入门](https://web.archive.org/web/20221129011945/https://docs.spring.io/spring-roo/docs/2.0.x/reference/html/#getting-started-requirements)给出了具体的操作步骤。

现在我们可以再次进入“窗口”应用菜单，选择`Show View > Roo Shell.`

## 3。第一个项目

### 3.1。在 STS 中设置项目

在 STS 中，让我们打开`Roo Shell`窗口，点击`Create New Roo Project`图标。这将打开一个`New Roo Project`窗口。

我们将这个项目命名为`roo`，并使用`com.baeldung`作为我们的顶层包名。我们可以保留所有其他默认值，并继续到最后使用 Roo 创建一个新项目。

在 STS 中，这将为我们运行以下命令:

```java
project setup --topLevelPackage com.baeldung --projectName "roo" --java 8 --packaging JAR
```

如前所述，我们不需要 IDE，我们可以自己从 Roo Shell 中运行该命令！为了简单起见，我们使用 STS 的内置特性。

如果我们得到以下错误:

```java
Could not calculate build plan: Plugin org.codehaus.mojo:aspectj-maven-plugin:1.8 
  or one of its dependencies could not be resolved: 
  Failed to read artifact descriptor for org.codehaus.mojo:aspectj-maven-plugin:jar:1.8
```

最简单的解决方法是手动编辑`pom.xml`文件，将`aspectj.plugin.version`从`1.8`更新为`1.9`:

```java
<aspectj.plugin.version>1.9</aspectj.plugin.version>
```

在这个阶段，项目中不应该有任何错误，并且会有一些自动生成的文件。

### 3.2。鲁迈拉运营公司外壳

现在是时候熟悉 Roo Shell 了。Spring Roo 的主要用户界面实际上是命令提示符！

因此，让我们回到 Roo Shell 窗口。在其中，让我们通过键入“h”并按 CTRL+SPACE 来运行第一个命令:

```java
roo> h

help    hint
```

Roo 会自动为我们建议和自动完成命令。我们可以输入‘hi’，按 CTRL+SPACE，Roo 会自动建议`hint`命令。

Roo Shell 的另一个伟大特性是**上下文感知**。例如，`hint`命令的输出将根据之前的输入而改变。

现在让我们执行`hint`命令，看看会发生什么:

```java
roo> hint 
Roo requires the installation of a persistence configuration.

Type 'jpa setup' and then hit CTRL+SPACE. We suggest you type 'H'
then CTRL+SPACE to complete "HIBERNATE".

After the --provider, press CTRL+SPACE for database choices.
For testing purposes, type (or CTRL+SPACE) HYPERSONIC_IN_MEMORY.
If you press CTRL+SPACE again, you'll see there are no more options.
As such, you're ready to press ENTER to execute the command.

Once JPA is installed, type 'hint' and ENTER for the next suggestion.
```

它为我们提供了需要执行的后续步骤。现在让我们添加一个数据库:

```java
roo> jpa setup --provider HIBERNATE --database HYPERSONIC_IN_MEMORY 
Created SRC_MAIN_RESOURCES\application.properties
Updated SRC_MAIN_RESOURCES\application.properties
Updated SRC_MAIN_RESOURCES\application-dev.properties
Updated ROOT\pom.xml [added dependencies org.springframework.boot:spring-boot-starter-data-jpa:, org.springframework.boot:spring-boot-starter-jdbc:, org.hsqldb:hsqldb:; added property 'springlets.version' = '1.2.0.RC1'; added dependencies io.springlets:springlets-data-jpa:${springlets.version}, io.springlets:springlets-data-jpa:${springlets.version}; added dependencies io.springlets:springlets-data-commons:${springlets.version}, io.springlets:springlets-data-commons:${springlets.version}]
```

在这个阶段，我们需要执行一些命令。在它们之间，我们总是可以运行`hint`命令来看看 Roo 给出了什么建议。这是一个非常有用的功能。

**让我们先运行命令**，然后我们将仔细检查它们:

```java
roo> 
entity jpa --class ~.domain.Book
field string --fieldName title --notNull 
field string --fieldName author --notNull 
field string --fieldName isbn --notNull 
repository jpa --entity ~.domain.Book
service --all 
web mvc setup
web mvc view setup --type THYMELEAF 
web mvc controller --entity ~.domain.Book --responseType THYMELEAF
```

我们现在准备运行我们的应用程序。但是，让我们回顾一下这些命令，看看我们都做了些什么。

首先，我们在`src/main/java`文件夹中创建了一个新的 JPA 持久实体。接下来，我们在`Book`类中创建了三个`String`字段，给它们起了个名字，并设置为 not `null`。

之后，我们为指定的实体生成了 Spring 数据仓库，并创建了一个新的服务接口。

最后，我们包含了 Spring MVC 配置，安装了百里香并创建了一个新的控制器来管理我们的实体。因为我们已经将百里香叶作为响应类型传递，所以生成的方法和视图将反映这一点。

### 3.3。运行应用程序

让我们刷新项目，右键单击`roo`项目并选择`Run As > Spring Boot App`。

一旦应用程序启动，我们就可以打开网页浏览器，进入 [http://localhost:8080](https://web.archive.org/web/20221129011945/http://localhost:8080/) 。接下来，到 Roo 图标，我们会看到`Book`菜单和下面的两个选项:`Create Book`和`List Books`。我们可以用它向我们的应用程序添加一本书，并查看添加的书的列表。

### 3.4。其他功能

当我们打开`Book.java`类文件时，我们会注意到该类用`@Roo`注释进行了注释。这些是由 Roo Shell 添加的，用于控制和定制 AspectJ 内部类型声明(ITD)文件的内容。我们可以通过在查看菜单中取消选择“隐藏生成的 Spring Roo ITDs”过滤器，在 STS 的包资源管理器中查看文件，或者我们可以直接从文件系统中打开文件。

Roo 注释有`SOURCE`保留策略。这意味着**注释不会出现在编译后的类字节码**中，并且在部署的应用程序中不会有对 Roo 的任何依赖。

另一个在`Book.java`类中明显缺失的部分是`getters`和`setters`。如前所述，它们存储在单独的 AspectJ ITDs 文件中。鲁迈拉运营公司将积极为我们维护这些样板代码。因此，对任何类中字段的更改都会自动反映在 AspectJ ITDs 中，因为 Roo 正在“监控”所有更改——要么通过 Roo Shell，要么直接由 IDE 中的开发人员来完成。

Roo 也会处理类似`toString()`或`equals()`方法的重复代码。

此外，通过移除注释并将 AspectJ ITD 推入标准 java 代码，可以很容易地将框架从项目中移除，从而避免供应商锁定。

## 4。结论

在这个简单的例子中，我们设法在 STS 中安装和配置了 Spring Roo，并创建了一个小项目。

我们使用 Roo Shell 来设置它，不需要编写一行实际的 Java 代码！我们能够在几分钟内得到一个可用的应用程序原型，Roo 为我们处理了所有的样板代码。

和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129011945/https://github.com/eugenp/tutorials/tree/master/spring-roo)