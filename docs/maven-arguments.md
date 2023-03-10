# 作为 Maven 属性的命令行参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-arguments>

## 1。概述

在这个简短的教程中，我们将看看如何使用命令行向 Maven 传递参数。

## 2。Maven 属性

Maven 属性是值占位符。首先，**我们需要在我们的`pom.xml`** 文件中的`properties`标签下定义它们:

```java
<properties>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <start-class>com.example.Application</start-class>
    <commons.version>2.5</commons.version>
</properties>
```

然后，我们可以在其他标签中使用它们。例如，在这种情况下，我们将在`commons-io`依赖关系中使用“`commons.version`”值:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>{commons.version}</version>
</dependency>
```

事实上，**我们可以在`pom.xml`、**的任何地方使用这些属性，比如在`build`、`package`或`plugin`部分。

## 3。为属性定义占位符

有时，我们在开发时不知道属性值。在这种情况下，我们可以使用语法`${some_property}`留下一个占位符而不是值，并且 **Maven 将在运行时**覆盖占位符值。让我们为`COMMON_VERSION_CMD`设置一个占位符:

```java
<properties>
    <maven.compiler.source>1.7</maven.compiler.source>
    <commons.version>2.5</commons.version>
    <version>${COMMON_VERSION_CMD}</version>
</properties>
```

## 4。将参数传递给 Maven

现在，让我们像往常一样从终端运行 Maven，例如使用`package`命令。但是在这种情况下，让我们也添加符号`-D`,后跟一个属性名:

```java
mvn package -DCOMMON_VERSION_CMD=2.5
```

Maven 将使用作为参数传递的值(2.5)来替换在我们的`pom.xml`中设置的`COMMON_VERSION_CMD`属性。这不仅限于`package`命令— **我们可以和任何 Maven 命令**一起传递参数，比如`install`、`test`或`build`。

## 5。结论

在本文中，我们研究了如何从命令行向 Maven 传递参数。通过使用这种方法，我们可以动态地提供属性，而不是修改`pom.xml`或任何静态配置。