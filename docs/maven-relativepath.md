# 理解 Maven 对父 POM 的“relativePath”标记

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-relativepath>

## 1.概观

在本教程中，我们将了解 [Maven](/web/20220628123205/https://www.baeldung.com/maven-guide) 的父 POM 分辨率。首先，我们将发现默认行为。然后，我们将讨论定制它的可能性。

## 2.默认父 POM 分辨率

如果我们想指定一个父 POM，我们可以通过命名`groupId`、`artifactId,`和`version`，即所谓的`GAV coordinate`来实现。 **Maven 不通过首先在存储库中搜索来解析父 POM**。我们可以在 [Maven 模型文档](https://web.archive.org/web/20220628123205/https://maven.apache.org/ref/3.0/maven-model/maven.html)中找到细节，并总结其行为:

1.  如果父文件夹中有一个`pom.xml`文件，并且该文件具有匹配的 GAV 坐标，则该文件被分类为项目的父 POM
2.  如果没有，Maven 返回到存储库

管理多模块项目时，将一个 Maven 项目放入另一个项目是最佳实践。例如，我们有一个聚合器项目，具有以下 GAV 坐标:

```java
<groupId>com.baeldung.maven-parent-pom-resolution</groupId>
<artifactId>aggregator</artifactId>
<version>1.0.0-SNAPSHOT</version>
```

然后，我们可以将模块放入子文件夹中，并将聚合器称为父级:

[![](img/beebdc0a5a17fe3de08e2a50d58d4669.png)](/web/20220628123205/https://www.baeldung.com/wp-content/uploads/2021/09/module1.svg)

因此，模块 1 POM 可以包括以下部分:

```java
<artifactId>module1</artifactId>
<parent>
    <groupId>com.baeldung.maven-parent-pom-resolution</groupId>
    <artifactId>aggregator</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</parent>
```

不需要将聚合器 POM 安装到存储库中。甚至不需要在聚合器 POM 中声明`module1`。但是我们必须意识到，这仅适用于项目的本地检出(例如，在构建项目时)。如果项目被解析为来自 Maven 存储库的依赖项，那么父 POM 也应该在存储库中可用。

我们必须确保聚合器 POM 具有匹配的 GAV 坐标。否则，我们会得到一个构建错误:

```java
[ERROR]     Non-resolvable parent POM for com.baeldung.maven-parent-pom-resolution:module1:1.0.0-SNAPSHOT:
  Could not find artifact com.baeldung.maven-parent-pom-resolution:aggregator:pom:1.0-SNAPSHOT
  and 'parent.relativePath' points at wrong local POM @ line 7, column 13
```

## 3。定制父 POM 的位置

如果父 POM 不在父文件夹中，我们需要使用`relativePath`标签来引用这个位置。例如，如果我们有第二个模块应该继承来自`module1`的设置，而不是来自聚合器，我们必须将兄弟文件夹命名为:

[![](img/0ddb8f46c532d64efcabfcd26fd76ca5.png)](/web/20220628123205/https://www.baeldung.com/wp-content/uploads/2021/09/module2.svg)

```java
<artifactId>module2</artifactId>
<parent>
    <groupId>com.baeldung.maven-parent-pom-resolution</groupId>
    <artifactId>module1</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <relativePath>../module1/pom.xml</relativePath>
</parent>
```

当然，我们应该只使用在每个环境中都可用的相对路径(主要是同一个 Git 存储库中的路径),以确保我们构建的可移植性。

## 4.禁用本地文件解析

为了跳过本地文件搜索，直接在 Maven 存储库中搜索父 POM，我们需要显式地将`relativePath`设置为空值:

```java
<parent>
    <groupId>com.baeldung</groupId>
    <artifactId>external-project</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <relativePath/>
</parent>
```

[![](img/063f4940a8657c6546308631bc890a1e.png)](/web/20220628123205/https://www.baeldung.com/wp-content/uploads/2021/09/external.svg)

每当我们从像 Spring Boot 这样的外部项目继承时，这应该是一个最佳实践。

## 5.5 月

有趣的是，IntelliJ IDEA(当前版本:2021.1.3)附带了一个 Maven 插件，该插件在父 POM 解析方面不同于外部 Maven 运行时。与 [Maven 的 POM 模式](https://web.archive.org/web/20220628123205/https://maven.apache.org/xsd/maven-4.0.0.xsd)不同，它这样解释`relativePath`标签:

> [……]Maven 首先在当前构建项目的反应器中寻找父 POM[……]

这意味着，对于 IDE 内部解析，只要父项目注册为 IntelliJ Maven 项目，父 POM 的位置就无关紧要。这可能有助于简单地开发项目，而不需要显式地构建它们(如果它们不在同一个 Git 存储库的范围内)。但是如果我们试图用外部 Maven 运行时来构建项目，它将会失败。

## 6.结论

在本文中，我们了解到 Maven 不会通过首先搜索 Maven 库来解析父 POM。而是在本地搜索它，当从外部项目继承时，我们必须显式地停用这种行为。此外，ide 可能会额外解析工作区中的项目，这可能会在我们使用外部 Maven 运行时时导致错误。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220628123205/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-parent-pom-resolution)