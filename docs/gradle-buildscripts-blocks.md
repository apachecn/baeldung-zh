# Gradle 中的 BuildScripts 块

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-buildscripts-blocks>

## 1.概观

在本教程中，我们将学习 [Gradle](/web/20221124122810/https://www.baeldung.com/gradle) 中的构建脚本块(在`[build.gradle](/web/20221124122810/https://www.baeldung.com/gradle-build-settings-properties)`文件中的脚本),并详细了解`buildScript`块的用途。

## 2.介绍

### 2.1.格拉德是什么？

它是一个构建自动化工具，执行诸如编译、打包、测试、部署、发布、[依赖解析](/web/20221124122810/https://www.baeldung.com/gradle)等任务。如果没有这个，我们将不得不手动完成这些任务，这是非常复杂和耗时的。在今天的软件开发中，没有这样的构建工具很难工作。

### 2.2.Gradle 的公共构建脚本块

在这一节中，我们将简要了解最常见的构建脚本块。`**allProjects**`、**、`subProjects`、**、**、`plugins`、`dependencies`、`repositories`、`publishing`、`buildScript`、**是最常见的构建脚本块。给定的列表介绍了这些块的概念:

*   [`allProjects`](https://web.archive.org/web/20221124122810/https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:allprojects(groovy.lang.Closure)) 块配置根项目和每个子项目。
*   [`subProjects`](https://web.archive.org/web/20221124122810/https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:subprojects(groovy.lang.Closure)) 块，不像`allProjects,`只配置子项目。
*   通过引入一组有用的功能来扩展 Gradle 的功能。例如，`java `插件添加了像汇编、构建、清理、jar、文档等任务。，等等。
*   顾名思义，这是一个声明我们项目所需的所有 jar 的地方。
*   `[repositories](https://web.archive.org/web/20221124122810/https://docs.gradle.org/current/userguide/declaring_repositories.html#header) `块包含 Gradle 下载在`dependencies`块中声明的 jar 的位置。一个人可以声明按声明顺序执行的多个位置。
*   当我们开发一个库并希望发布它时，block 被声明。这个块包含详细信息，如库 jar 的坐标和包含发布位置的`repositories` 块。

现在，让我们考虑一个用例，我们想在构建脚本中使用一个库。在这种情况下，我们不能使用 **`dependencies` 块，因为它包含项目类路径**所需的 jar。

由于我们要在构建脚本本身中使用这个库，因此**需要** **在脚本类路径中添加这个库。**而这就是`buildScript is for`。下一节将通过这个用例深入讨论`buildScript`块。

## 3.构建脚本块的目的

考虑到上面定义的用例，假设在一个 [Spring Boot](/web/20221124122810/https://www.baeldung.com/spring-boot) 应用程序中，我们想要读取构建脚本中 application.yml 文件的定义属性。为了实现这一点，我们可以使用一个名为`[snakeyaml](/web/20221124122810/https://www.baeldung.com/java-snake-yaml) `的库来解析 YAML 文件并轻松读取属性。

正如我们在上一节中所讨论的，我们需要在脚本类路径中有这个库。**解决方案是将其作为依赖项添加到`buildScript` 块中。**

该脚本显示了如何读取 application.yml 文件的属性`temp.files.path`。`buildScript`块包含了 [`snakeyaml`](/web/20221124122810/https://www.baeldung.com/java-snake-yaml) 的依赖库和储存库位置下载它:

```java
import org.yaml.snakeyaml.Yaml

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.yaml', name: 'snakeyaml', version: '1.19'
    }
}

plugins {
    //plugins
}

def prop = new Yaml().loadAll(new File("$projectDir/src/main/resources/application.yml")
  .newInputStream()).first()
var path = prop.temp.files.path
```

path 变量包含`temp.files.path. `的值

关于`buildScript`区块的更多信息:

*   它可以包含除项目类型依赖项之外的任何类型的依赖项。
*   对于多项目构建，所有子项目`.`的构建脚本都可以使用声明的依赖项
*   要将作为外部 jar 可用的二进制插件添加到项目中，我们应该将它们添加到构建脚本类路径中，然后应用插件。

## 4.结论

在本教程中，我们学习了 Gradle 的用法，构建脚本中最常见的块的用途，并通过一个用例深入研究了`buildScript`块。