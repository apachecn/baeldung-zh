# Gradle 中的依赖管理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-dependency-management>

## 1.概观

在本教程中，我们将看看在 Gradle 构建脚本中声明依赖关系。对于我们的例子，我们将使用 [Gradle 6.7](https://web.archive.org/web/20221022154512/https://gradle.org/) 。

## 2.典型结构

让我们从一个简单的 Java 项目脚本开始:

```
plugins {
    id 'java'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter:2.3.4.RELEASE'
    testImplementation 'org.springframework.boot:spring-boot-starter-test:2.3.4.RELEASE'
}
```

从上面可以看出，我们有三个代码块:`plugins`、`repositories,`和`dependencies`。

首先，`plugins`块告诉我们这是一个 Java 项目。其次，`dependencies`块声明了编译项目生产源代码所需的`spring-boot-starter`依赖项的`2.3.4.`版本。此外，它还声明该项目的测试套件需要`spring-boot-starter-test`来编译。

Gradle build 从 Maven 中央存储库中取出所有依赖项，如`repositories`块所定义的。

让我们关注如何定义依赖关系。

## 3.依赖关系配置

我们可以在不同的配置中声明依赖关系。在这方面，我们可以选择更精确或更不精确，稍后我们会看到。

### 3.1.如何声明依赖关系

首先，该配置有 4 个部分:

*   `**group**`–组织、公司或项目的标识符
*   `**name**` –依赖性标识符
*   `**version**` –我们想要导入的那个
*   `**classifier**` –用于区分具有相同`group`、`name`和`version`的依赖关系

我们可以用两种格式声明依赖关系。契约格式允许我们将依赖项声明为`String`:

`implementation 'org.springframework.boot:spring-boot-starter:2.3.4.RELEASE'`

相反，扩展格式允许我们将它写成一个`Map`:

`implementation group:` `'org.springframework.boot', name: 'spring-boot-starter', version: '2.3.4.RELEASE'`

### 3.2.配置类型

此外，Gradle 提供了许多依赖配置类型:

*   `**api**`–用于使依赖关系显式化，并在类路径中公开它们。例如，当实现一个对库消费者透明的库时
*   `**implementation**` –编译生产源代码所必需的，完全是内部的。它们不会暴露在包装之外
*   `**compileOnly**` –仅在编译时需要声明时使用，如仅源代码注释或注释处理器。它们不会出现在运行时类路径或测试类路径中
*   **`compileOnlyApi`**–在编译时需要时使用，当它们需要在类路径中对消费者可见时使用
*   `**runtimeOnly**` –用于声明仅在运行时需要而在编译时不可用的依赖项
*   `**testImplementation**` –编译测试所需
*   `**testCompileOnly**` –仅在测试编译时需要
*   `**testRuntimeOnly**` –仅在测试运行时需要

我们应该注意到，在撰写本文时，Gradle 的最新版本弃用了一些配置，如`compile`、`testCompile`、`runtime,`和`testRuntime.`，它们仍然可用。

## 4.外部依赖的类型

让我们深入研究我们在 Gradle 构建脚本中遇到的外部依赖类型。

### 4.1.模块依赖性

基本上，声明依赖关系的最常见方式是引用存储库。一个梯度库是由`group`、`name`和`version`组织的模块的集合。

事实上，Gradle 从`repository`块中的指定存储库中提取依赖项:

```
repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter:2.3.4.RELEASE'
}
```

### 4.2.文件依赖关系

鉴于项目并不总是使用自动化的依赖关系管理，一些项目将依赖关系组织为源代码或本地文件系统的一部分。因此，我们需要指定依赖项的确切位置。

为此，我们可以使用`files`来包含一个依赖集合:

```
dependencies {
    runtimeOnly files('libs/lib1.jar', 'libs/lib2.jar')
}
```

类似地，我们可以使用`filetree`在一个目录中包含一个层次结构的`jar`文件:

```
dependencies {
    runtimeOnly fileTree('libs') { include '*.jar' }
}
```

### 4.3.项目相关性

由于一个项目可以依赖另一个项目来重用代码，Gradle 为我们提供了这样做的机会。

假设我们想要声明我们的项目依赖于`shared`项目:

```
dependencies { 
    implementation project(':shared') 
}
```

### 4.4.梯度依赖性

在某些情况下，例如开发一个任务或插件，我们可以定义属于我们正在使用的 Gradle 版本的依赖关系:

```
dependencies {
    implementation gradleApi()
}
```

## 5.`buildScript`

正如我们之前看到的，我们可以在`dependencies`块中声明源代码和测试的外部依赖关系。类似地，`buildScript`块允许我们声明 Gradle build 的依赖项，比如第三方插件和任务类。特别是，如果没有`buildScript`模块，我们只能使用 Gradle 的开箱即用特性。

下面我们声明我们想通过从 Maven Central 下载来使用 [Spring Boot 插件](/web/20221022154512/https://www.baeldung.com/spring-boot-gradle-plugin):

```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.springframework.boot:spring-boot-gradle-plugin:2.3.4.RELEASE' 
    }
}
apply plugin: 'org.springframework.boot'
```

因此，我们需要指定下载外部依赖项的来源，因为没有默认的来源。

上面所描述的是与旧版本的 Gradle 相关的。相反，在新版本中，可以使用更简洁的形式:

```
plugins {
    id 'org.springframework.boot' version '2.3.4.RELEASE'
}
```

## 6.结论

在本文中，我们研究了 Gradle 依赖项，如何声明它们，以及不同的配置类型。

鉴于这几点，本文的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221022154512/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle/gradle-dependency-management)