# 在 Gradle 创建一个胖罐子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-fat-jar>

[This article is part of a series:](javascript:void(0);)[• Introduction to Gradle](/web/20220910153856/https://www.baeldung.com/gradle)
[• Ant vs Maven vs Gradle](/web/20220910153856/https://www.baeldung.com/ant-maven-gradle)
[• Writing Custom Gradle Plugins](/web/20220910153856/https://www.baeldung.com/gradle-create-plugin)
• Creating a Fat Jar in Gradle (current article)

## 1。概述

在这篇简短的文章中，我们将介绍如何在 Gradle 中创建一个“胖罐子”。

基本上，fat jar(也称为 uber-jar)是一个自给自足的档案，包含运行应用程序所需的类和依赖项。

## 2。初始设置

让我们从一个简单的 Java 项目的`build.gradle`文件开始，它有两个依赖项:

```java
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    implementation group: 'org.slf4j', name: 'slf4j-api', version: '1.7.25'
    implementation group: 'org.slf4j', name: 'slf4j-simple', version: '1.7.25'
}
```

## 3。使用 Java 插件中的 Jar 任务

让我们从修改 Java Gradle 插件中的`jar`任务开始。默认情况下，该任务生成没有任何依赖关系的 jar。

我们可以通过添加几行代码来覆盖这种行为。我们需要两样东西来实现它:

*   清单文件中的一个`Main-Class`属性
*   包括依赖关系 jar

让我们给 Gradle 任务添加一些修改:

```java
jar {
    manifest {
        attributes "Main-Class": "com.baeldung.fatjar.Application"
    }

    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
```

## 4。创建单独的任务

如果我们想让原来的 jar 任务保持原样，我们可以创建一个单独的任务来完成同样的工作。

下面的代码将添加一个名为`customFatJar:`的新任务

```java
task customFatJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'com.baeldung.fatjar.Application'
    }
    baseName = 'all-in-one-jar'
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
    from { configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) } }
    with jar
}
```

## 5。使用专用插件

我们也可以使用现有的 Gradle 插件来构建一个胖罐子。

在这个例子中，我们将使用 [Shadow](https://web.archive.org/web/20220910153856/https://github.com/johnrengelman/shadow) 插件:

```java
buildscript {
    repositories {
        mavenCentral()
        gradlePluginPortal()
    }
    dependencies {
        classpath "gradle.plugin.com.github.johnrengelman:shadow:7.1.2"
    }
}

apply plugin: 'java'
apply plugin: 'com.github.johnrengelman.shadow'
```

一旦我们应用了影子插件，`shadowJar`任务就可以使用了。

## 6。结论

在本教程中，我们介绍了在 Gradle 中创建 fat jars 的几种不同方法。我们覆盖了默认的 jar 任务，创建了一个独立的任务，并使用了影子插件。

推荐哪种方法？答案是——视情况而定。

在简单的项目中，覆盖默认的 jar 任务或创建一个新任务就足够了。但是随着项目的发展，我们强烈建议使用插件，因为它们已经解决了更困难的问题，比如与外部 META-INF 文件的冲突。

一如既往，本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220910153856/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle/gradle-fat-jar)

**«** Previous[Writing Custom Gradle Plugins](/web/20220910153856/https://www.baeldung.com/gradle-create-plugin)