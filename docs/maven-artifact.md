# 什么是阿帕奇 Maven 神器？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-artifact>

## 1。概述

手动构建一个复杂的项目相当麻烦。使用构建工具有更简单的方法来做到这一点。正如我们所知，Java 项目的主要构建工具之一是 [Maven](/web/20220810024515/https://www.baeldung.com/maven) 。Maven 帮助标准化应用程序的构建和部署。

在本教程中，我们将讨论什么是 Maven 工件以及它的关键元素是什么。我们还将研究 Maven 坐标、依赖性管理，最后是 Maven 仓库。

## 2。Maven 是什么？

我们可以使用 Maven 来构建和管理任何基于 Java 的项目。它提供了许多功能，例如:

*   构建和编译
*   文件和报告
*   依赖性管理
*   来源管理
*   项目更新
*   部署

每个 Maven 项目都有自己的 [POM](/web/20220810024515/https://www.baeldung.com/maven-super-simplest-effective-pom) 文件。我们可以配置 Maven 来构建一个项目或多个项目。

多项目通常在根目录中定义一个 POM 文件，并在`modules`部分列出各个项目。简而言之，一个 **Maven 构建产生一个或多个工件**。

## 3。什么是 Maven 神器？

工件是一个项目可以使用或者产生的元素。在 Maven 术语中，**工件** **是在 Maven 项目构建之后生成的输出。**例如，它可以是`jar`、`war`或任何其他可执行文件。

同样，Maven 工件包括五个关键元素，`groupId`、`artifactId`、`version`、`packaging`和`classifier`。这些是我们用来识别藏物的元素，被称为 Maven 坐标。

## 4。Maven 坐标

**Maven 坐标是给定工件的`groupId`、`artifactId`和`version`值的组合。**此外，Maven 使用坐标来查找任何与`groupId, artifactId,` 和`version`的值相匹配的组件。

在坐标元素中，我们必须定义`groupId`、`artifactId`、`version`、T3`packaging`元素是可选的，我们不能直接定义`classifier`。

例如，下面的`pom.xml`配置文件显示了一个 Maven 坐标的例子:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.baeldung</groupId>
    <artifactId>org.baeldung.java</artifactId>
    <packaging>jar</packaging>
    <version>1.0.0-SNAPSHOT</version>
    <name>org.baeldung.java</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.1.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

让我们详细看看每个 Maven 坐标。

### 4.1.`groupId` 元素

**`groupId`元素是位于项目**起点的组的标识符。该键使组织和查找项目变得更加容易和快捷。

另外，`groupId`遵循与 Java 包相同的命名规则，我们通常选择项目顶层包的名称作为`groupId`。

例如，`org.apache.commons`的一个`groupId`对应于`${repository_home}/org/apache/commons.`

### 4.2.`artifactId` 元素

**`artifactId`元素是组内项目的标识符。**默认情况下，它用于构建工件的最终名称。因此，这个名字有一些规格，因为它应该理想地长度很小。命名`artifactId`的最佳实践是使用实际的项目名称作为前缀。这样做的好处是可以更容易地找到藏物。

和`groupId`一样，`artifactId`显示为代表`groupId`的目录树下的子目录。

例如，在`org.apache.commons,`的`groupId`下的`commons-lang3` 的`artifactId`将确定工件在`${repository_home}/org/apache/commons/commons-lang3/`下。

### 4.3.`version` 元素

`version`被用作工件标识符的一部分。**它定义了 Maven 项目**的当前版本。我们应该注意，Maven 定义了一组版本规范，以及发布和快照的概念，我们将在后面介绍。

一个`version`表示为目录树中的一个子目录，由`groupId`和`artifactId`组成。

例如，`org.apache.commons`的`groupId `下的`artifactId commons-lang3`的`3.1.1`版本将确定工件位于:`${repository_home}/org/apache/commons/commons-lang3/3.1.1/.`下

### 4.4.`packaging` 元素

该元素用于指定项目生成的工件的类型。**`packaging`可以是描述任何二进制软件格式**的任何东西，包括 `ZIP, EAR, WAR, SWC, NAR, SWF, SAR.`

另外，`packaging`定义了在项目的默认生命周期中要执行的不同目标。例如，打包阶段为 jar 类型的工件执行`jar:jar`目标，为 war 类型的工件执行 *war:war* 目标。

### 4.5.`classifier` 元素

由于技术原因，当交付相同的代码时，我们通常使用一个`classifier`作为几个独立的工件。

例如，如果我们想用不同的 Java 编译器构建一个`JAR` 的两个工件，我们可以很容易地使用`classifier` 来完成，因为它允许用相同的组合为`groupId:artifactId:version`生成两个不同的工件。

此外，在打包源代码、工件的 JavaDoc 或汇编二进制文件时，我们可以使用`classifier` s。

对于我们上面的`commons-lang3`的例子，要寻找的工件是:`commons-lang3-3.10-javadoc.jar`或`commons-lang3-3.10-sources.jar`，在`${repository_home}/org/apache/commons/commons-lang3/3.1.0/.`下面

## 5.发布与快照工件

现在，让我们看看快照工件和发布工件之间的区别`.`

### 5.1.发布工件

`A` **发布** **工件表明版本是稳定的** **并且可以在开发过程**之外使用，如集成测试、客户资格、预生产等。

还有，一个**释放神器是独一无二的。**运行`mvn` `deploy`命令将我们的项目部署到存储库。但是，随后对相同版本的相同项目执行相同的命令将会导致失败。

### 5.2.快照工件

**快照工件表明项目正在开发中**。当我们安装或发布一个组件时，Maven 会检查`version`属性。如果它包含字符串“SNAPSHOT”，Maven 会将这个键转换为 UTC(协调世界时)格式的日期和时间值。

例如，如果我们的项目是版本 1.0-SNAPSHOT，并且我们在 Maven 存储库上部署它的工件，Maven 将这个版本转换为“1.0-202202019-230800”，假设我们在 2022 年 2 月 19 日 23:08 UTC 部署。

换句话说，当我们部署一个快照时，我们并没有交付一个软件组件，而只是交付了它的一个快照。

## 6。依赖性管理

依赖性管理在 Maven 世界中是必不可少的。例如，当一个项目在其操作过程(编译、执行)中依赖于其他类时，就有必要识别这些依赖关系，并将其从远程存储库导入到本地存储库中。因此，项目将依赖于这些库，这些库最终将被添加到项目的类路径中。

此外，Maven 的依赖性管理基于几个概念:

*   存储库:存储工件的基本要素
*   范围:允许我们指定在哪个上下文中使用依赖关系
*   传递性:允许我们管理依赖关系的依赖性
*   继承:从父节点继承的 POM 可以通过只提供依赖项的`groupId`和`artifactId`而不提供`version`属性来设置它们的依赖项。Maven 从父 POM 文件中获取适当的版本。

使用 Maven，**的依赖关系管理是通过`pom.xml`** 来完成的。例如，Maven 项目中的依赖声明如下所示:

```java
<dependencies>
    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-plugin-api</artifactId>
        <version>3.8.4</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.15</version>
    </dependency>
</dependencies>
```

## 7.Maven 知识库

Maven 使用存储库来存储构建项目所需的依赖项和插件等元素。这使得集中这些通常在几个项目中使用的元素成为可能。

正如我们前面提到的，存储库使用一组坐标来存储工件:`groupId`、`artifactId`、`version`和`packaging`。此外，Maven 使用特定的目录结构来组织存储库的内容，并允许它找到所需的元素:`${repository_home}/groupId/artifactId/version`。

例如，让我们考虑一下这个 POM 配置。

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.baeldung</groupId>
    <artifactId>myApp</artifactId>
    <version>1.0.0</version>
</project>
```

使用上面的配置，我们的项目将被存储在一个位于`${repository_home}/com/baeldung/myApp/1.0.0/` 路径的存储库中。

## 8。 **结论**

在本文中，我们讨论了 Maven 工件及其坐标系的概念。此外，我们还学习了相关概念，如依赖关系和存储库。