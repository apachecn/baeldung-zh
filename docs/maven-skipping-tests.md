# 用 Maven 跳过测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-skipping-tests>

## 1.介绍

跳过测试通常是个坏主意。然而，在某些情况下，它可能是有用的——也许当我们开发新代码，并且想要运行测试没有通过或编译的中间版本时。

只有在这种情况下，我们可能会跳过测试，以避免编译和运行它们的开销。当然，考虑到不运行测试会导致糟糕的编码实践。

在这个快速教程中，我们将探索使用 Maven 跳过测试的所有可能的命令和选项。

## 2.Maven 生命周期

在进入如何跳过测试的细节之前，**我们必须理解测试何时被编译或运行**。在关于 [Maven 目标和阶段](/web/20221206042547/https://www.baeldung.com/maven-goals-phases)的文章中，我们深入探讨了 Maven 生命周期的概念，但是对于本文的目的来说，了解 Maven 可以:

1.  忽略测试
2.  编译测试
3.  运行测试

在我们的例子中，我们将使用`package`阶段，它包括编译和运行测试。贯穿本教程的选项属于 [Maven Surefire 插件](https://web.archive.org/web/20221206042547/https://maven.apache.org/surefire/maven-surefire-plugin/)。

## 3.使用命令行标志

### 3.1.跳过测试编译

首先，让我们来看一个无法编译的测试示例:

```java
@Test
public void thisDoesntCompile() {
    baeldung;
}
```

运行命令行命令时:

```java
mvn package
```

我们会得到一个错误:

```java
[INFO] -------------------------------------------------------------
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] /Users/baeldung/skip-tests/src/test/java/com/antmordel/skiptests/PowServiceTest.java:[11,9] not a statement
[INFO] 1 error
```

因此，让我们探索一下**如何跳过测试源代码的编译阶段**。在 Maven 中，我们可以使用`maven.test.skip` 标志:

```java
mvn -Dmaven.test.skip package
```

因此，测试源不会被编译，因此也不会被执行。

### 3.2.跳过测试执行

作为第二个选择，让我们看看如何**我们可以编译测试文件夹，但跳过运行过程**。这对于那些我们没有改变方法或类的签名，但是我们改变了业务逻辑，并因此破坏了测试的情况是很有用的。让我们考虑一个像下面这样的人为测试用例，它总是会失败:

```java
@Test
public void thisTestFails() {
    fail("This is a failed test case");
}
```

由于我们包含了语句`fail()`，如果我们运行`package`阶段，构建将会失败，并出现错误:

```java
[ERROR] Failures:
[ERROR]   PowServiceTest.thisTestFails:16 This is a failed test case
[INFO]
[ERROR] Tests run: 2, Failures: 1, Errors: 0, Skipped: 0
```

**让我们假设我们想要跳过运行测试，但是我们仍然想要编译它们。**在这种情况下，我们可以使用`-DskipTests`标志:

```java
mvn -DskipTests package
```

打包阶段将会成功。同样，在 Maven 中，有一个运行集成测试的专用插件叫做 [maven 故障保护插件](https://web.archive.org/web/20221206042547/https://maven.apache.org/surefire-archives/surefire-2.17/maven-failsafe-plugin/examples/skipping-test.html)。`-DskipTests` 将跳过单元测试(surefire)和集成测试(failsafe)的执行。**为了跳过集成测试，我们可以通过`-DskipITs `系统属性。**

最后，值得一提的是，现在已被否决的标志`-Dmaven.test.skip.exec`也将编译测试类，但不会运行它们。

## 4.使用 Maven 配置

如果我们需要在更长的时间内排除编译或运行测试，我们可以修改`pom.xml`文件，以便包含正确的配置。

### 4.1.跳过测试编译

正如我们在上一节所做的，让我们检查一下如何避免编译测试文件夹。在这种情况下，我们将使用`pom.xml`文件。让我们添加以下属性:

```java
<properties>
    <maven.test.skip>true</maven.test.skip>
</properties>
```

请记住，**我们可以通过在命令行**中添加相反的标志来覆盖该值:

```java
mvn -Dmaven.test.skip=false package
```

### 4.2.跳过测试执行

再次，作为第二步，让我们探索如何使用 Maven 配置**构建测试文件夹，但是跳过测试执行**。为此，我们必须为 Maven Surefire 插件配置一个属性:

```java
<properties>
    <tests.skip>true</tests.skip>
</properties>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <skipTests>${tests.skip}</skipTests>
    </configuration>
</plugin>
```

Maven 属性`tests.skip`是我们之前定义的自定义属性。因此，如果我们想要执行测试，我们可以覆盖它:

```java
mvn -Dtests.skip=false package
```

## 4.结论

在这篇快速教程中，我们探索了 Maven 提供的所有替代方法，以便跳过编译和/或运行测试。

我们浏览了 Maven 命令行选项和 Maven 配置选项。