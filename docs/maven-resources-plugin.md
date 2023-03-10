# Maven 资源插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-resources-plugin>

[This article is part of a series:](javascript:void(0);)• Maven Resources Plugin (current article)[• Maven Compiler Plugin](/web/20220628143042/https://www.baeldung.com/maven-compiler-plugin)
[• Quick Guide to the Maven Install Plugin](/web/20220628143042/https://www.baeldung.com/maven-install-plugin)
[• The Maven Failsafe Plugin](/web/20220628143042/https://www.baeldung.com/maven-failsafe-plugin)
[• Quick Guide to the Maven Surefire Plugin](/web/20220628143042/https://www.baeldung.com/maven-surefire-plugin)
[• The Maven Deploy Plugin](/web/20220628143042/https://www.baeldung.com/maven-deploy-plugin)
[• The Maven Clean Plugin](/web/20220628143042/https://www.baeldung.com/maven-clean-plugin)
[• The Maven Verifier Plugin](/web/20220628143042/https://www.baeldung.com/maven-verifier-plugin)
[• The Maven Site Plugin](/web/20220628143042/https://www.baeldung.com/maven-site-plugin)
[• Guide to the Core Maven Plugins](/web/20220628143042/https://www.baeldung.com/core-maven-plugins)

## 1。概述

本教程描述了`resources`插件，Maven 构建工具的核心插件之一。

关于其他核心插件的概述，请参考本文。

## 2。插件目标

**`resources`插件将文件从输入资源目录复制到输出目录。**这个插件有三个目标，不同之处仅在于如何指定资源和输出目录。

这个插件的三个目标是:

*   **`resources`–**将主源代码中的资源复制到主输出目录中
*   **`testResources`–**将测试源代码中的资源复制到测试输出目录中
*   **`copy-resources`–**将任意资源文件复制到输出目录，需要我们指定输入文件和输出目录

我们来看看`pom.xml`中的`resources`插件:

```java
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>
        ...
    </configuration>
</plugin>
```

我们可以在这里找到这个插件的最新版本。

## 3。示例

假设我们想要将资源文件从目录`input-resources`复制到目录`output-resources `，并且我们想要排除所有以扩展名`.png`结尾的文件。

这种配置满足了这些要求:

```java
<configuration>
    <outputDirectory>output-resources</outputDirectory>
    <resources>
        <resource>
            <directory>input-resources</directory>
            <excludes>
                <exclude>*.png</exclude>
            </excludes>
            <filtering>true</filtering>
        </resource>
    </resources>
</configuration>
```

该配置适用于`resources`插件的所有执行。

例如，当这个插件的`resources`目标被命令`mvn resources:resources`执行时，`input-resources`目录下的所有资源，除了 PNG 文件，都会被复制到`output-resources`。

因为默认情况下，`resources`目标被绑定到 Maven `default`生命周期的`process-resources`阶段，所以我们可以通过运行命令`mvn process-resources`来执行这个目标和所有前面的阶段。

在给定的配置中，有一个名为`filtering`的参数，其值为`true`。**`filtering`参数用于替换资源文件**中的占位符变量。

例如，如果我们在 POM 中有一个属性:

```java
<properties>
    <resources.name>Baeldung</resources.name>
</properties>
```

其中一个资源文件包含:

```java
Welcome to ${resources.name}!
```

然后，将在输出资源中计算该变量，结果文件将包含:

```java
Welcome to Baeldung!
```

## 4。结论

在这篇简短的文章中，我们回顾了`resources`插件，并给出了使用和定制它的说明。

本教程的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628143042/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-plugins)

Next **»**[Maven Compiler Plugin](/web/20220628143042/https://www.baeldung.com/maven-compiler-plugin)