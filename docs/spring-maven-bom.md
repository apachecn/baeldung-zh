# 春天与 Maven BOM

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-maven-bom>

## 1.概观

在这个快速教程中，我们将了解 Maven，一个基于项目对象模型(POM)概念的工具，如何利用 BOM 或“物料清单”。

关于 Maven 的更多细节，可以查看我们的文章 [Apache Maven 教程](/web/20220701010503/https://www.baeldung.com/maven)。

## 2.依赖性管理概念

为了理解 BOM 是什么以及我们可以用它来做什么，我们首先需要学习一些基本概念。

### 2.1.什么是 Maven POM？

Maven POM 是一个 XML 文件，包含 Maven 用来导入依赖项和构建项目的信息和配置(关于项目)。

### 2.2.什么是 Maven BOM？

BOM 代表材料清单。BOM 是一种特殊的 POM，用于控制项目依赖项的版本，并提供一个定义和更新这些版本的中心位置。

BOM 提供了向我们的模块添加依赖项的灵活性，而不用担心我们应该依赖哪个版本。

### 2.3.传递依赖性

Maven 可以在我们的`pom.xml`中发现我们自己的依赖项所需要的库，并自动包含它们。从哪个依赖级别收集库是没有限制的。

当两个依赖关系引用一个特定工件的不同版本时，冲突就出现了。哪个会被 Maven 收录？

**这里的答案是“最接近的定义”。这意味着所使用的版本将是依赖关系树中最接近我们项目的版本。这被称为依赖中介。**

让我们看下面的例子来阐明依赖中介:

```java
A -> B -> C -> D 1.4  and  A -> E -> D 1.0
```

这个例子显示了项目`A`依赖于`B`，`E.`，`B`和`E`有它们自己的依赖，它们遇到了不同版本的`D`工件。工件`D` 1.0 将用于`A`项目的构建，因为通过`E`的路径更短。

有不同的技术来确定应该包含哪个版本的工件:

*   我们总是可以通过在项目的 POM 中显式声明来保证一个版本。例如，为了保证使用了`D` 1.4，我们应该在`pom.xml`文件中明确地添加它作为一个依赖项。
*   我们可以使用`Dependency Management`部分来控制工件版本，我们将在本文后面解释。

### 2.4.依赖性管理

简单地说，依赖性管理是一种集中依赖性信息的机制。

当我们有一组继承一个公共父项目的项目时，我们可以将所有的依赖信息放在一个名为 BOM 的共享 POM 文件中。

以下是如何编写 BOM 文件的示例:

```java
<project ...>

    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Baeldung-BOM</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>BaelDung-BOM</name>
    <description>parent pom</description>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>test</groupId>
                <artifactId>a</artifactId>
                <version>1.2</version>
            </dependency>
            <dependency>
                <groupId>test</groupId>
                <artifactId>b</artifactId>
                <version>1.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>test</groupId>
                <artifactId>c</artifactId>
                <version>1.0</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

正如我们所看到的，BOM 是一个普通的 POM 文件，带有一个`dependencyManagement`部分，我们可以在其中包含所有工件的信息和版本。

### 2.5.使用 BOM 文件

有两种方法可以在我们的项目中使用以前的 BOM 文件，然后我们将准备好声明我们的依赖关系，而不必担心版本号。

我们可以从父母那里继承:

```java
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Test</name>
    <parent>
        <groupId>baeldung</groupId>
        <artifactId>Baeldung-BOM</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
</project>
```

正如我们所看到的，我们的项目测试继承了 Baeldung-BOM。

我们也可以导入 BOM。

在较大的项目中，继承的方法效率不高，因为项目只能继承一个父项目。导入是另一种选择，因为我们可以根据需要导入任意数量的 BOM。

让我们看看如何将 BOM 文件导入到项目 POM 中:

```java
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Test</name>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>baeldung</groupId>
                <artifactId>Baeldung-BOM</artifactId>
                <version>0.0.1-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 2.6.覆盖物料清单相关性

工件版本的优先顺序是:

1.  我们的项目 pom 中工件的直接声明版本
2.  父项目中工件的版本
3.  考虑到导入文件的顺序，导入的 pom 中的版本
4.  依赖中介

*   我们可以通过在项目的 pom 中明确定义工件的期望版本来覆盖工件的版本
*   如果同一个工件在两个导入的 BOM 中定义了不同的版本，那么 BOM 文件中最先声明的版本将胜出

### 3\. Spring BOM

我们可能会发现一个第三方库，或者另一个 Spring 项目，把一个可传递的依赖拉进了一个旧版本。如果我们忘记显式声明一个直接依赖项，就会出现意想不到的问题。

为了克服这些问题，Maven 支持 BOM 依赖的概念。

我们可以在我们的`dependencyManagement`部分导入`spring-framework-bom`,以确保所有的 Spring 依赖项都是相同的版本:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>4.3.8.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

当我们像下面的例子一样使用 Spring 工件时，我们不需要指定`version`属性:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
<dependencies>
```

### 4.结论

在这篇简短的文章中，我们展示了 Maven 的物料清单概念，以及如何将工件的信息和版本集中在一个公共的 POM 中。

简而言之，我们可以继承或导入它来利用 BOM 的好处。

文章中的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220701010503/https://github.com/eugenp/tutorials/tree/master/spring-bom)