# Spring Boot 首发父母

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-starter-parent>

## 1.介绍

在本教程中，我们将学习`spring-boot-starter-parent.` ，我们将讨论如何从中受益，以获得更好的依赖性管理、插件的默认配置，并快速构建我们的`Spring Boot`应用程序。

我们还将看到如何覆盖由`starter-parent.`提供的现有依赖项和属性的版本

## 2.Spring Boot 首发父母

*spring-boot-starter-parent*项目是一个特殊的 starter 项目，它为我们的应用程序提供了默认配置和一个完整的依赖树来快速构建我们的`Spring Boot`项目。它还提供了 Maven 插件的默认配置，比如`maven-failsafe-plugin`、`maven-jar-plugin`、`maven-surefire-plugin`和`maven-war-plugin`。

除此之外，它还从 s `pring-boot-starter-parent`的父节点`spring-boot-dependencies,` 继承了依赖关系管理。

我们可以开始在我们的项目中使用它，方法是将它作为父项添加到我们项目的`pom.xml`:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
```

我们总能从 Maven Central 获得 [`spring-boot-starter-parent`](https://web.archive.org/web/20221206084508/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-parent%22) 的最新版本。

## 3.管理依赖关系

一旦我们在项目中声明了起始父项，我们就可以通过在我们的`dependencies`标签中声明它来从父项中提取任何依赖项。我们也不需要定义依赖关系的版本；Maven 将根据在 parent 标记中为 starter parent 定义的版本下载 jar 文件。

例如，如果我们正在构建一个 web 项目，我们可以直接添加`spring-boot-starter-web`，而不需要指定版本:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

## 4.依赖性管理标签

为了管理 starter parent 提供的依赖项的不同版本，我们可以在`dependencyManagement`部分显式声明依赖项及其版本:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>2.4.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 5.性能

要更改 starter parent 中定义的任何属性值，我们可以在 properties 部分重新声明它。

`spring-boot-starter-parent` 通过其父`spring-boot-dependencies`使用属性来配置所有依赖版本、Java 版本和 Maven 插件版本。因此，我们只需更改相应的属性，就可以轻松控制这些配置。

如果我们想要改变任何我们想要从起始父节点获取的依赖项的版本，我们可以在依赖项标记中添加依赖项，并直接配置它的属性:

```java
<properties>
    <junit.version>4.11</junit.version>
</properties>
```

## 6.其他属性覆盖

我们还可以将属性用于其他配置，比如管理插件版本，甚至一些基本配置，比如管理 Java 版本和源代码。我们只需要用新的值重新声明属性。

例如，要更改 Java 版本，我们可以在`java.version` 属性中指明:

```java
<properties>
    <java.version>1.8</java.version>
</properties>
```

## 7.没有起始父项的 Spring Boot 项目

有时我们有一个定制的 Maven 父节点，或者我们更喜欢手工声明我们所有的 Maven 配置。

在这种情况下，我们可以选择不使用*spring-boot-starter-parent*项目。但是我们仍然可以通过在我们的项目中的`import`范围内添加一个依赖项`spring-boot-dependencies,`来受益于它的依赖树。

让我们用一个简单的例子来说明这一点，在这个例子中，我们希望使用不同于起始父级的另一个父级:

```java
<parent>
    <groupId>com.baeldung</groupId>
    <artifactId>spring-boot-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</parent>
```

这里我们使用了`parent-modules,` 一个不同的项目，作为我们的父依赖项。

现在，在这种情况下，我们仍然可以通过将依赖关系管理添加到`import`范围和`pom`类型中来获得同样的好处:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

此外，我们可以通过在`dependencies,`中声明来引入任何依赖关系，就像我们在前面的例子中所做的那样。这些依赖项不需要版本号。

## 8.结论

在本文中，我们概述了`spring-boot-starter-parent,` 以及将其作为父项目添加到任何子项目中的好处。

接下来，我们学习了如何管理依赖关系。我们可以在`dependencyManagement`中或者通过属性覆盖依赖关系。

本文中使用的代码片段的源代码可以在 [Github](https://web.archive.org/web/20221206084508/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-parent) 上获得，一个使用 starter 父代码，另一个使用自定义父代码。