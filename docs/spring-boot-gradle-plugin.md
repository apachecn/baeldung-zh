# Spring Boot Gradle Plugin

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-gradle-plugin>

## 1。概述

Spring Boot Gradle 插件帮助我们管理 Spring Boot 的依赖关系，并在使用 Gradle 作为构建工具时打包和运行我们的应用程序。

在本教程中，我们将讨论如何添加和配置插件，然后我们将看到如何建立和运行一个 Spring Boot 项目。

## 2。构建文件配置

首先，**我们需要将 Spring Boot 插件添加到我们的`build.gradle`** 文件中，方法是将它包含在我们的`plugins`部分:

```java
plugins {
    id "org.springframework.boot" version "2.0.1.RELEASE"
}
```

如果我们使用的是 2.1 之前的 Gradle 版本，或者我们需要动态配置，我们可以这样添加它:

```java
buildscript {
    ext {
        springBootVersion = '2.0.1.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath(
          "org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'org.springframework.boot'
```

## 3。打包我们的应用程序

**我们可以使用`build`命令:**将我们的应用程序打包成一个可执行的归档文件(jar 或 war 文件)

```java
./gradlew build
```

因此，生成的可执行文件将被放在`build/libs`目录中。

如果我们想生成一个可执行的`jar`文件，那么我们还需要应用`java`插件:

```java
apply plugin: 'java'
```

另一方面，如果我们需要一个`war`文件，我们将应用`war`插件:

```java
apply plugin: 'war'
```

构建应用程序将为 Spring Boot 1.x 和 2.x 生成可执行文件。然而，对于每个版本，Gradle 触发不同的任务。

接下来，让我们仔细看看每个引导版本的构建过程。

### 3.1。Spring Boot 2.x

**在 Boot 2.x 中，`bootJar`和`bootWar`任务负责打包应用程序。**

任务 `bootJar`负责创建可执行文件`jar`。这是在应用了`java`插件后自动创建的。

让我们看看如何直接执行`bootJar`任务:

```java
./gradlew bootJar
```

类似地，`bootWar`生成一个可执行的 war 文件，并在应用了`war`插件后被创建。

我们可以使用以下命令来执行`bootWar`任务:

```java
./gradlew bootWar
```

**注意，对于 Spring Boot 2.x，我们需要使用 Gradle 4.0 或更高版本。**

我们也可以配置这两项任务。例如，让我们使用`mainClassName`属性来设置主类:

```java
bootJar {
    mainClassName = 'com.baeldung.Application'
}
```

或者，我们可以使用 Spring Boot DSL 中的相同属性:

```java
springBoot {
    mainClassName = 'com.baeldung.Application'
}
```

### 3.2。 Spring Boot 1.x

**用 Spring Boot 1.x，`bootRepackage`负责根据配置创建可执行归档文件** `(jar`或`war`文件。

我们可以使用以下命令直接执行`bootRepackage`任务:

```java
./gradlew bootRepackage
```

类似于 Boot 2.x 版本，我们可以在我们的`build.gradle:`中向`bootRepackage`任务添加配置

```java
bootRepackage {
    mainClass = 'com.example.demo.Application'
}
```

我们也可以通过将`enabled`选项设置为`false:`来禁用`bootRepackage`任务

```java
bootRepackage {
    enabled = false
}
```

## 4。运行我们的应用程序

构建完应用程序后，**我们可以在生成的可执行 jar 文件上使用`java -jar`命令**来运行它:

```java
java -jar build/libs/demo.jar
```

**Spring Boot·格拉德插件还为我们提供了`bootRun`任务**，使我们无需先构建应用程序就能运行它:

```java
./gradlew bootRun
```

`bootRun`任务可以在`build.gradle.`中简单配置

例如，我们可以定义主类:

```java
bootRun {
    main = 'com.example.demo.Application'
}
```

## 5。与其他插件的关系

### 5.1。依赖管理插件

对于 Spring Boot 1.x，它曾经自动应用依赖管理插件。这将导入 Spring Boot 依赖项 BOM，其行为类似于 Maven 的依赖项管理。

但是从 Spring Boot 2.x 开始，如果我们需要这个功能，我们需要在我们的`build.gradle`中显式地应用它:

```java
apply plugin: 'io.spring.dependency-management'
```

### 5.2。Java 插件

当我们应用`java`插件时，Spring Boot·格拉德插件采取多种动作，如:

*   创建`a bootJar`任务，我们可以用它来生成一个可执行的 jar 文件
*   创建`a bootRun`任务，我们可以用它来直接运行我们的应用程序
*   禁用`jar`任务

### 5.3。战争插件

类似地，当我们应用`war`插件时，结果是:

*   创建 `bootWar`任务，我们可以用它来生成可执行的 war 文件
*   禁用 `war`任务

## 6。结论

在这个快速教程中，我们了解了 Spring Boot Gradle 插件及其不同的任务。

此外，我们讨论了它如何与其他插件交互。