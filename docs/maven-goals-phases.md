# Maven 目标和阶段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-goals-phases>

## 1。概述

在本教程中，我们将探索不同的 Maven 构建生命周期及其阶段。

我们还将讨论目标和阶段之间的核心关系。

## 2。Maven 构建生命周期

Maven 构建遵循特定的生命周期来部署和分发目标项目。

有三个内置的生命周期:

*   默认:主生命周期，因为它负责项目部署
*   清理:清理项目并移除上一次构建生成的所有文件
*   站点:创建项目的站点文档

**每个生命周期由一系列阶段组成。**构建生命周期由 23 个阶段组成，因为它是主要的构建生命周期。

另一方面，`clean`生命周期由 3 个阶段组成，而`site`生命周期由 4 个阶段组成。

## 3。Maven 阶段

Maven 阶段代表了 Maven 构建生命周期的一个阶段。每个阶段负责一个特定的任务。

以下是`default`构建生命周期中一些最重要的阶段:

*   检查建造所需的所有信息是否可用
*   `compile:`编译源代码
*   `test-compile:`编译测试源代码
*   运行单元测试
*   `package:`将编译好的源代码打包成可分发的格式(jar，war，…)
*   如果需要运行集成测试，处理并部署包
*   `install:`将软件包安装到本地存储库
*   `deploy:`将包复制到远程存储库

要获得每个生命周期阶段的完整列表，请查看 [Maven 参考资料](https://web.archive.org/web/20220827200618/https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)。

各个阶段以特定的顺序执行。这意味着，如果我们使用以下命令运行特定阶段:

```java
mvn <PHASE>
```

它不仅会执行指定的阶段，还会执行之前的所有阶段。

例如，如果我们运行`deploy`阶段，这是`default`构建生命周期的最后一个阶段，它将执行`deploy`阶段之前的所有阶段，这是整个`default`生命周期:

```java
mvn deploy
```

## 4。Maven 目标

**每个阶段都是一个目标序列，每个目标负责一个具体的任务。**

当我们运行一个阶段时，所有绑定到这个阶段的目标都会按顺序执行。

以下是一些阶段和与之相关的默认目标:

*   `compiler:compile`–来自`compiler`插件的`compile`目标被绑定到`compile`阶段
*   `compiler:testCompile`被绑定到`test-compile`阶段
*   `surefire:test`被绑定到`test`阶段
*   `install:install`被绑定到`install`阶段
*   `jar:jar`和`war:war`被绑定到`package`阶段

我们可以使用以下命令列出绑定到特定阶段的所有目标及其插件:

```java
mvn help:describe -Dcmd=PHASENAME
```

例如，要列出绑定到`compile`阶段的所有目标，我们可以运行:

```java
mvn help:describe -Dcmd=compile
```

然后我们会得到示例输出:

```java
compile' is a phase corresponding to this plugin:
org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
```

如上所述，这意味着来自`compiler`插件的`compile`目标被绑定到了`compile`阶段。

## 5。Maven 插件

**一个 Maven 插件是一组目标**；然而，这些目标不一定都绑定到同一个阶段。

例如，下面是 Maven Failsafe 插件的简单配置，它负责运行集成测试:

```java
<build>
    <plugins>
        <plugin>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>${maven.failsafe.version}</version>
            <executions>
                <execution>
                    <goals>
                        <goal>integration-test</goal>
                        <goal>verify</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

正如我们所见，故障保护插件在这里配置了两个主要目标:

*   `integration-test`:运行集成测试
*   `verify`:验证所有集成测试通过

我们可以使用下面的命令**列出特定插件**中的所有目标:

```java
mvn <PLUGIN>:help
```

例如，要列出故障保护插件中的所有目标，我们可以运行:

```java
mvn failsafe:help
```

输出将是:

```java
This plugin has 3 goals:

failsafe:help
  Display help information on maven-failsafe-plugin.
  Call mvn failsafe:help -Ddetail=true -Dgoal=<goal-name> to display parameter
  details.

failsafe:integration-test
  Run integration tests using Surefire.

failsafe:verify
  Verify integration tests ran using Surefire.
```

**要运行一个特定的目标而不执行其整个阶段(以及之前的阶段)，**我们可以使用命令:

```java
mvn <PLUGIN>:<GOAL>
```

例如，要从故障保护插件运行`integration-test`目标，我们需要运行:

```java
mvn failsafe:integration-test
```

## 6。构建 Maven 项目

为了构建 Maven 项目，我们需要通过运行其中一个阶段来执行其中一个生命周期:

```java
mvn deploy
```

这将执行整个`default`生命周期。或者，我们可以在`install`阶段停止:

```java
mvn install
```

但是通常，我们会在新构建之前运行 *清理* 生命周期来清理项目:

```java
mvn clean install
```

我们也可以只运行一个特定目标的插件:

```java
mvn compiler:compile
```

注意，如果我们试图在没有指定阶段或目标的情况下构建 Maven 项目，我们会得到一个错误:

```java
[ERROR] No goals have been specified for this build. You must specify a valid lifecycle phase or a goal
```

## 7。结论

在本文中，我们讨论了 Maven 构建生命周期，以及 Maven 阶段和目标之间的关系。