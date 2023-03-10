# 用 Bazel 构建 Java 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/bazel-build-tool>

## 1。概述

[Bazel](https://web.archive.org/web/20221101160606/https://bazel.build/) 是一款用于构建和测试源代码的开源工具，类似于 Maven 和 Gradle。**它支持多种语言的项目并为多种平台构建输出**。

在本教程中，我们将介绍使用 Bazel 构建一个简单的 Java 应用程序所需的步骤。举例来说，我们将从一个多模块 Maven 项目开始，然后使用 Bazel 构建源代码。

我们将从[安装 Bazel](https://web.archive.org/web/20221101160606/https://docs.bazel.build/versions/master/install.html) 开始。

## 2。项目结构

让我们创建一个多模块 Maven 项目:

```java
bazel (root)
    pom.xml
    WORKSPACE (bazel workspace)
    |— bazelapp
        pom.xml
        BUILD (bazel build file)
        |— src
            |— main
                |— java
            |— test
                |— java
    |— bazelgreeting
        pom.xml
        BUILD (bazel build file)
        |— src
            |— main
                |— java
            |— test
                |— java
```

**`WORKSPACE`文件的存在为巴泽尔**建立了工作空间。一个项目中可能有一个或多个这样的人。对于我们的例子，我们将只在顶层项目目录中保存一个文件。

下一个重要的文件是 **`BUILD`文件，它包含了构建规则**。它用唯一的`target` `name`来标识每个规则。

`Bazel`提供了灵活性，让**拥有我们需要的**那么多的`BUILD`文件，配置成任何粒度级别。这意味着我们可以通过相应地配置`BUILD`规则来构建更少的 Java 类。为了简单起见，我们将在示例中保留最少的`BUILD`文件。

由于 Bazel `BUILD`配置的输出通常是一个 jar 文件，我们将包含`BUILD`文件的每个目录称为一个构建包。

## 3。构建文件

### 3.1。规则配置

是时候配置我们的第一个构建规则来构建 Java 二进制文件了。让我们在属于`bazelapp`模块的`BUILD`文件中配置一个:

```java
java_binary (
    name = "BazelApp",
    srcs = glob(["src/main/java/com/baeldung/*.java"]),
    main_class = "com.baeldung.BazelApp"
) 
```

让我们逐一了解配置设置:

*   `java_binary`–规则的名称；构建二进制文件需要额外的属性
*   `name`–构建目标的名称
*   `srcs`–文件位置模式的数组，它告诉我们要构建哪些 Java 文件
*   `main_class`–应用程序主类的名称(可选)

### 3.2。构建执行

我们现在可以构建应用程序了。从包含`WORKSPACE` 文件`,` 的目录中，让我们在 shell 中执行`bazel build`命令来构建我们的目标:

```java
$ bazel build //bazelapp:BazelApp
```

最后一个参数是在一个`BUILD` 文件中配置的目标名称。它有图案“`//<path_to_build>:<target_name>`”。

模式的第一部分“//”表示我们从工作区目录开始。下一个是`“bazelapp”`，是从工作区目录到`BUILD`文件的相对路径。最后，`“BazelApp”`是要构建的目标名称。

### 3.3。构建输出

我们现在应该注意到上一步中的两个二进制输出文件:

```java
bazel-bin/bazelapp/BazelApp.jar
bazel-bin/bazelapp/BazelApp
```

`BazelApp.jar`包含所有的类，而`BazelApp`是执行 jar 文件的包装器脚本。

### 3.4。可展开的罐子

我们可能需要将 jar 及其依赖项运送到不同的位置进行部署。

上一节中的包装器脚本将所有依赖项(jar 文件)指定为`BazelApp.jar`的启动命令的一部分。

然而，我们也可以制作一个包含所有依赖项的胖罐子:

```java
$ bazel build //bazelapp:BazelApp_deploy.jar
```

在目标名称后面加上`“_deploy”` **指示 Bazel 将所有依赖项打包到 jar** 中，并准备好进行部署。

## 4。依赖性

到目前为止，我们只使用了`bazelapp.` 中的文件，但是几乎每个应用都有依赖关系。

在这一节中，我们将看到如何将依赖项与 jar 文件打包在一起。

### 4.1。建筑图书馆

不过，在此之前，我们需要一个`bazelapp`可以使用的依赖项。

让我们创建另一个名为`bazelgreeting`的 Maven 模块，并用`java_library`规则为新模块配置`BUILD`文件。我们将这个目标命名为“`greeter”`:

```java
java_library (
    name = "greeter",
    srcs = glob(["src/main/java/com/baeldung/*.java"])
)
```

这里，我们使用了`java_library`规则来创建库。构建这个目标后，我们将得到`libgreetings.jar`文件:

```java
INFO: Found 1 target...
Target //bazelgreeting:greetings up-to-date:
  bazel-bin/bazelgreeting/libgreetings.jar
```

### 4.2。配置依赖关系

为了在`bazelapp`中使用`greeter`，我们需要一些额外的配置。首先，我们需要让包对`bazelapp`可见。我们可以通过在`greeter`包的`java_library`规则中添加`visibility`属性来实现这一点:

```java
java_library (
    name = "greeter",
    srcs = glob(["src/main/java/com/baeldung/*.java"]),
    visibility = ["//bazelapp:__pkg__"]
)
```

属性使当前包对数组中列出的那些人可见。

现在在`bazelapp`包中，我们必须配置对`greeter`包的依赖。让我们用`deps`属性来做这件事:

```java
java_binary (
    name = "BazelApp",
    srcs = glob(["src/main/java/com/baeldung/*.java"]),
    main_class = "com.baeldung.BazelApp",
    deps = ["//bazelgreeting:greeter"]
)
```

属性使当前包依赖于数组中列出的包。

## 5。外部依赖性

我们可以从事有多个工作空间并且相互依赖的项目。或者，我们可以从远程位置导入库。我们可以将这种外部依赖归类为:

*   **本地依赖关系**:我们在同一个工作区内管理它们，就像我们在前面章节中看到的那样，或者跨越多个工作区
*   HTTP 存档:我们通过 HTTP 从远程位置导入库

有许多 Bazel 规则可用于管理[外部依赖关系](https://web.archive.org/web/20221101160606/https://docs.bazel.build/versions/master/external.html)。我们将在后面的小节中看到如何从远程位置导入 jar 文件。

### 5.1。HTTP URL 位置

对于我们的例子，让我们将 [Apache Commons Lang](https://web.archive.org/web/20221101160606/https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.9/commons-lang3-3.9.jar) 导入到我们的应用程序中。因为我们必须从 HTTP 位置导入这个 jar，所以我们将使用 [`http_jar`](https://web.archive.org/web/20221101160606/https://docs.bazel.build/versions/master/repo/http.html#http_jar) 规则。我们将首先从 Bazel HTTP 构建定义中加载规则，并使用 Apache Commons 的位置在`WORKSPACE`文件中配置它:

```java
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_jar")

http_jar (
    name = "apache-commons-lang",
    url = "https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar"
)
```

我们必须进一步在"`bazelapp”`包的`BUILD`文件中添加依赖项:

```java
deps = ["//bazelgreeting:greeter", "@apache-commons-lang//jar"]
```

注意，我们需要从`WORKSPACE`文件中指定在`http_jar`规则中使用的相同名称。

### 5.2。Maven 依赖关系

管理单个 jar 文件变成了一项单调乏味的任务。或者，我们可以使用我们的`WORKSPACE`文件中的 [`rules_jvm_external`](https://web.archive.org/web/20221101160606/https://github.com/bazelbuild/rules_jvm_external) 规则来**配置 Maven 存储库。这将使我们能够从存储库中获取尽可能多的依赖项。**

首先，我们必须使用`WORKSPACE`文件中的`http_archive`规则从远程位置导入`rules_jvm_external`规则:

```java
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

RULES_JVM_EXTERNAL_TAG = "2.0.1"
RULES_JVM_EXTERNAL_SHA = "55e8d3951647ae3dffde22b4f7f8dee11b3f70f3f89424713debd7076197eaca"

http_archive(
    name = "rules_jvm_external",
    strip_prefix = "rules_jvm_external-%s" % RULES_JVM_EXTERNAL_TAG,
    sha256 = RULES_JVM_EXTERNAL_SHA,
    url = "https://github.com/bazelbuild/rules_jvm_external/archive/%s.zip" % RULES_JVM_EXTERNAL_TAG,
)
```

接下来，我们将使用`maven_install`规则并配置 Maven 存储库 URL 和所需的工件:

```java
load("@rules_jvm_external//:defs.bzl", "maven_install")

maven_install(
    artifacts = [
        "org.apache.commons:commons-lang3:3.12.0" ], 
    repositories = [ 
        "https://repo1.maven.org/maven2", 
    ] )
```

最后，我们将在`BUILD`文件中添加依赖关系:

```java
deps = ["//bazelgreeting:greeter", "@maven//:org_apache_commons_commons_lang3"]
```

**它使用下划线(_)字符解析工件的名称。**

## 6.结论

在本教程中，我们学习了使用 Bazel 构建工具构建 Maven 风格的 Java 项目的基本配置。

源代码也可以通过 [GitHub](https://web.archive.org/web/20221101160606/https://github.com/eugenp/tutorials/tree/master/bazel) 获得。它作为一个 Maven 项目存在，也是用 Bazel `WORKSPACE`和`BUILD`文件配置的。