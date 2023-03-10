# 核心 Maven 插件指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/core-maven-plugins>

[This article is part of a series:](javascript:void(0);)[• Maven Resources Plugin](/web/20221206105618/https://www.baeldung.com/maven-resources-plugin)
[• Maven Compiler Plugin](/web/20221206105618/https://www.baeldung.com/maven-compiler-plugin)
[• Quick Guide to the Maven Install Plugin](/web/20221206105618/https://www.baeldung.com/maven-install-plugin)
[• The Maven Failsafe Plugin](/web/20221206105618/https://www.baeldung.com/maven-failsafe-plugin)
[• Quick Guide to the Maven Surefire Plugin](/web/20221206105618/https://www.baeldung.com/maven-surefire-plugin)
[• The Maven Deploy Plugin](/web/20221206105618/https://www.baeldung.com/maven-deploy-plugin)
[• The Maven Clean Plugin](/web/20221206105618/https://www.baeldung.com/maven-clean-plugin)
[• The Maven Verifier Plugin](/web/20221206105618/https://www.baeldung.com/maven-verifier-plugin)
[• The Maven Site Plugin](/web/20221206105618/https://www.baeldung.com/maven-site-plugin)
• Guide to the Core Maven Plugins (current article)

## 1。概述

Maven 是 Java 世界中最常用的构建工具。主要是，它只是一个插件执行框架，其中所有的作业都由插件实现。

在本教程中，我们将介绍核心 Maven 插件，提供其他教程的链接，重点是这些插件可以做什么，以及它们的目标如何与构建生命周期绑定。

## 2。Maven 构建生命周期

核心插件与构建生命周期密切相关。

Maven 定义了三个构建生命周期:`default`、`site`和`clean`。每个生命周期由多个阶段组成，按顺序运行到`mvn`命令中指定的阶段。

**最重要的生命周期是`default`，负责构建过程**中的所有步骤，从项目验证到包部署。

`site`生命周期负责构建一个站点，显示项目的 Maven 相关信息，而`clean`生命周期负责删除在先前构建中生成的文件。

所有三个生命周期中的许多阶段都自动绑定到核心插件的目标上。参考文章将详细介绍这些目标和内置绑定。

所有插件都包含在 POM 的一个`build`元素中:

```java
<build>
    <plugins>
        <!-- plugins go here -->
    </plugins>
</build>
```

## 3。绑定到默认生命周期的插件

默认生命周期的内置绑定依赖于 POM 的`packaging`元素的值。为了简洁起见，我们将讨论最常见的打包类型的绑定:`jar`和`war`。

以下是与`default`生命周期的每个阶段相关的目标列表，格式为"`phase`->:`goal”`:

*   `process-resources`->-`resources:resources`
*   `compile`->-`compiler:compile`
*   `process-test-resources`->-`resources:testResources`
*   `test-compile`->-`compiler:testCompile`
*   `test`->-`surefire:test`
*   `package`->-`ejb:ejb`或`ejb3:ejb3`或`jar:jar`或`par:par`或`rar:rar`或`war:war`
*   `install`->-`install:install`
*   `deploy`->-`deploy:deploy`

以上目标包含在以下插件中。点击每个插件上的文章链接:

*   **[资源插件](/web/20221206105618/https://www.baeldung.com/maven-resources-plugin)**
*   [**编译器插件**](/web/20221206105618/https://www.baeldung.com/maven-compiler-plugin)
*   **[万全外挂](/web/20221206105618/https://www.baeldung.com/maven-surefire-plugin)**
*   **[故障保护插件](/web/20221206105618/https://www.baeldung.com/maven-failsafe-plugin)**
*   [**验证器插件**](/web/20221206105618/https://www.baeldung.com/maven-verifier-plugin)
*   [**安装插件**](/web/20221206105618/https://www.baeldung.com/maven-install-plugin)
*   [**部署插件**](/web/20221206105618/https://www.baeldung.com/maven-deploy-plugin)

## 4。其他插件

除了上一节提到的插件，还有另外两个核心插件，它们的目标与`site`和`clean`生命周期的阶段相关联:

*   [**站点插件**](/web/20221206105618/https://www.baeldung.com/maven-site-plugin)
*   [**清理插件**](/web/20221206105618/https://www.baeldung.com/maven-clean-plugin)

## 5。结论

在本文中，我们回顾了 Maven 构建生命周期，并提供了教程参考，详细介绍了 Maven 构建工具的核心插件。

大多数参考文章的代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20221206105618/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-plugins)

**«** Previous[The Maven Site Plugin](/web/20221206105618/https://www.baeldung.com/maven-site-plugin)