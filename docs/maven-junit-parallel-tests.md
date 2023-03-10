# 用 Maven 并行运行 JUnit 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-junit-parallel-tests>

## 1.介绍

尽管串行执行测试在大多数情况下工作得很好，但是我们可能想要并行化它们以加快速度。

在本教程中，我们将介绍如何使用 JUnit 和 Maven 的 Surefire 插件并行测试。首先，我们将在单个 JVM 进程中运行所有测试，然后我们将在一个多模块项目中尝试它。

## 2.Maven 依赖性

让我们从导入所需的依赖项开始。**我们需要使用 [JUnit 4.7 或更高版本](https://web.archive.org/web/20221118191751/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22junit%22%20AND%20a%3A%22junit%22)以及 [Surefire 2.16 或更高版本:](https://web.archive.org/web/20221118191751/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.surefire%22%20AND%20a%3A%22surefire%22)**

```java
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.21.0</version>
</plugin>
```

简单地说，Surefire 提供了两种并行执行测试的方法:

*   单个 JVM 进程中的多线程
*   运行多个 jvm 进程

## 3.运行并行测试

**要并行运行测试，我们应该使用扩展了`org.junit.runners.ParentRunner`的测试运行器。**

然而，即使没有显式声明测试运行器的测试也可以工作，因为默认运行器扩展了这个类。

接下来，为了演示并行测试执行，我们将使用一个包含两个测试类的测试套件，每个测试类都有一些方法。事实上，JUnit 测试套件的任何标准实现都可以。

### 3.1.使用并行参数

首先，让我们使用`parallel `参数在 Surefire 中启用并行行为。**它陈述了我们希望应用并行性的粒度级别。**

可能的值有:

*   `methods – `在单独的线程中运行测试方法
*   在单独的线程中运行测试类
*   在单独的线程中运行类和方法
*   `suites –` 并行运行套件
*   在单独的线程中运行套件和类
*   为类和方法创建单独的线程
*   在单独的线程中运行套件、类以及方法

在我们的例子中，我们使用`all`:

```java
<configuration>
    <parallel>all</parallel>
</configuration>
```

其次，让我们定义我们希望 Surefire 创建的线程总数。我们可以通过两种方式做到这一点:

使用定义 Surefire 将创建的最大线程数的`threadCount`:

```java
<threadCount>10</threadCount>
```

或者使用`useUnlimitedThreads`参数，每个 CPU 内核创建一个线程:

```java
<useUnlimitedThreads>true</useUnlimitedThreads>
```

默认情况下，`threadCount`是每个 CPU 内核。我们可以使用参数`perCoreThreadCount`来启用或禁用这种行为:

```java
<perCoreThreadCount>true</perCoreThreadCount>
```

### 3.2.使用线程数限制

现在，假设我们想要定义在方法、类和套件级别创建的线程数量。**我们可以通过`threadCountMethods`、`threadCountClasses`和`threadCountSuites` 参数来实现。**

让我们将这些参数与前面配置中的`threadCount`:结合起来

```java
<threadCountSuites>2</threadCountSuites>
<threadCountClasses>2</threadCountClasses>
<threadCountMethods>6</threadCountMethods>
```

由于我们在`parallel, `中使用了`all` ，我们已经为方法、套件和类定义了线程数。然而，定义叶参数并不是强制性的。Surefire 推断出在叶参数被省略的情况下要使用的线程数量。

例如，如果省略了`threadCountMethods` ，那么我们只需要确定`threadCount` > `threadCountClasses ` + `threadCountSuites.`

有时，我们可能希望限制为类、套件或方法创建的线程数量，即使我们使用的线程数量没有限制。

在这种情况下，我们也可以应用线程数限制:

```java
<useUnlimitedThreads>true</useUnlimitedThreads>
<threadCountClasses>2</threadCountClasses>
```

### 3.3.设置超时

有时我们可能需要确保测试执行是有时间限制的。

**为此，我们可以使用`parallelTestTimeoutForcedInSeconds `参数。**这将中断当前正在运行的线程，并且在超时后不会执行任何排队的线程:

```java
<parallelTestTimeoutForcedInSeconds>5</parallelTestTimeoutForcedInSeconds>
```

**另一个选择是使用`parallelTestTimeoutInSeconds`。**

在这种情况下，只有排队的线程将停止执行:

```java
<parallelTestTimeoutInSeconds>3.5</parallelTestTimeoutInSeconds>
```

然而，对于这两个选项，当超时结束时，测试将以错误消息结束。

### 3.4.警告

Surefire 在父线程中调用用`@Parameters`、`@BeforeClass`和 `@AfterClass`注释的静态方法。因此，在并行运行测试之前，一定要检查潜在的内存不一致或竞争情况。

此外，改变共享状态的测试绝对不适合并行运行。

## 4.多模块 Maven 项目中的测试执行

到目前为止，我们一直专注于在 Maven 模块中并行运行测试。

但是假设我们在一个 Maven 项目中有多个模块。因为这些模块是按顺序构建的，所以每个模块的测试也是按顺序执行的。

**我们可以通过使用 Maven 的`-T`参数来改变这种默认行为，该参数并行构建模块**。这可以通过两种方式实现。

我们可以指定构建项目时要使用的确切线程数:

```java
mvn -T 4 surefire:test
```

或者使用可移植版本，并指定每个 CPU 内核要创建的线程数量:

```java
mvn -T 1C surefire:test
```

无论哪种方式，我们都可以加速测试和构建执行时间。

## 5.派生 JVM

**通过`parallel `选项的并行测试执行，并发发生在使用线程**的 JVM 进程内部。

因为线程共享相同的内存空间，所以这在内存和速度方面是高效的。然而，我们可能会遇到意想不到的竞争情况或其他微妙的与并发性相关的测试失败。事实证明，共享同一个内存空间可能是福也可能是祸。

为了防止线程级并发问题，Surefire 提供了另一种并行测试执行模式: **forking 和进程级并发**。分叉流程的想法实际上非常简单。surefire 没有产生多个线程并在它们之间分配测试方法，而是创建新的进程并进行相同的分配。

由于不同进程之间没有共享内存，我们不会受到那些微妙的并发错误的困扰。当然，这是以更多的内存使用和稍低的速度为代价的。

总之，**为了启用分叉，我们只需使用`forkCount `属性，并将其设置为任意正值:**

```java
<forkCount>3</forkCount>
```

在这里，surefire 将从 JVM 创建最多三个分支，并在其中运行测试。`forkCount `的默认值是 1，这意味着`maven-surefire-plugin`创建一个新的 JVM 进程来执行一个 Maven 模块中的所有测试。

`forkCount `属性支持与`-T`相同的语法。也就是说，如果我们将`C `附加到这个值上，这个值将会乘以我们系统中可用的 CPU 内核的数量。例如:

```java
<forkCount>2.5C</forkCount>
```

那么在双核机器中，Surefire 最多可以创建 5 个分叉来进行并行测试执行。

默认情况下， **Surefire 将为其他测试重用创建的分叉**。然而，如果我们将`reuseForks `属性设置为`false`，它将在运行一个测试类后销毁每个分支。

此外，为了禁用分叉，我们可以将`forkCount `设置为零。

## 6.结论

总而言之，我们从启用多线程行为和使用`parallel`参数定义并行度开始。随后，我们对 Surefire 应该创建的线程数量进行了限制。稍后，我们设置超时参数来控制测试执行时间。

最后，我们研究了如何减少构建执行时间，从而减少多模块 Maven 项目中的测试执行时间。

和往常一样，这里展示的代码是 GitHub 上的[。](https://web.archive.org/web/20221118191751/https://github.com/eugenp/tutorials/tree/master/testing-modules/parallel-tests-junit)