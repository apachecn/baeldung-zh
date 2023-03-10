# Java 基于时间的版本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-time-based-releases>

## 1.介绍

在本文中，我们将讨论新的基于时间的 Java 版本以及对所有类型开发人员的影响。

发布时间表的变化包括更新 Java 版本的特性交付和支持级别。总的来说，这些变化与 Oracle 自 2010 年以来支持的 Java 截然不同。

## 2.为什么六个月发布？

对于我们这些习惯了 Java 历史上缓慢的发布节奏的人来说，这是一个非常重大的改变。为什么会有如此巨大的变化？

最初，Java 围绕大型特性的引入定义了它的主要版本。这有造成延迟的趋势，就像我们在 Java 8 和 9 中经历的那样。这也减缓了语言创新，而其他具有更紧密反馈周期的语言也在进化。

简单地说，更短的发布周期导致更小的、更易管理的前进步伐。并且更小的特征更容易被采用。

这种模式在当前条件下很好地匹配，并允许 JDK 开发在类似于它所支持的社区的敏捷方法中工作。此外，它使 Java 与 NodeJS 和 Python 等运行时相比更具竞争力。

当然，较慢的速度也有其好处，因此六个月的发布周期也在更大的长期支持框架中发挥作用，我们将在第 4 节中对此进行介绍。

## 3.版本号更改

这种变化的一个机械方面是一个新的版本号方案。

### 3.1.JEP 223 版本-字符串方案

我们都熟悉旧的一个，编纂在 [JEP 223](https://web.archive.org/web/20221206194521/https://openjdk.java.net/jeps/223) 中。这个方案使版本号递增，并传递额外的信息。

```java
 Actual                    Hypothetical
Release Type           long               short
------------           ------------------------ 
Security 2013/06       1.7.0_25-b15       7u25
Minor    2013/09       1.7.0_40-b43       7u40
Security 2013/10       1.7.0_45-b18       7u45
Security 2014/01       1.7.0_51-b13       7u51
Minor    2014/05       1.7.0_60-b19       7u60
```

**如果我们在版本 8 或更高版本的 JVM 上运行`java -version` ，我们会看到类似于**

```java
>java -version
java version "1.6.0_27"
Java(TM) 2 Runtime Environment, Standard Edition (build 1.6.0_27-b07)
Java HotSpot(TM) Client VM (build 1.6.0_27-b13, mixed mode, sharing)
```

在这种情况下，我们可能会猜测这是针对 Java 6 的，这是正确的，而第 27 次更新是错误的。编号方案并不像看起来那么直观。

次要版本是 10 的倍数，安全版本填充了其他所有内容。通常，我们会看到短字符串附加到我们的本地安装上，例如`JDK 1.8u174.`下一个版本可能是`JDK` 1.8u180，这将是一个带有新修复的小版本。

### 3.2.新版本-字符串方案

根据 JEP 的马克·莱因霍尔德的说法，新版本的字符串方案将“`recast version numbers to encode not compatibility and significance but, rather, the passage of time, in terms of release cycles,`”[。](https://web.archive.org/web/20221206194521/https://openjdk.java.net/jeps/322)

让我们来看一些:

```java
9.0.4
11.0.2
10.0.1
```

乍一看，这似乎是[语义版本控制](/web/20221206194521/https://www.baeldung.com/cs/semantic-versioning)；**然而，事实并非如此。**

对于语义版本控制，典型的结构是`$MAJOR.$MINOR.$PATCH`，但是 Java 的新版本结构是:

```java
$FEATURE.$INTERIM.$UPDATE.$PATCH
```

***$FEATURE* 是我们可能认为的主要版本**，但是不管兼容性保证如何，它将每六个月增加一次。`$PATCH` 用于维护发布。但是，这就是相似之处。

首先，`$INTERIM`是一个占位符，由 Oracle 保留以备将来之需。**暂时来说，永远是零。**

第二， **`$UPDATE`是基于时间的，就像`$FEATURE, `在最新功能发布后每月**更新。

最后，尾随零被截断。

这意味着`11`是 2018 年 9 月发布的 Java 11 的发布号，`11.0.1 `是其 10 月份的第一个月度更新版本，`11.0.1.3`将是 10 月份版本的第三个补丁发布。

## 4.多版本发行版

接下来，我们来看看如何挑选合适的版本。

### 4.1.稳定性

简单来说，Java 现在有一个快速通道，每六个月一次，还有一个慢速通道，每三年一次。 **每三年一次的发布被称为一次 LTS 发布。**

在快速通道上，语言在孵化中释放特性。这些语言特性在 LTS 版本中保持稳定。

因此，对于可以接受波动性以换取使用新功能的公司来说，他们可以使用快速通道。对于重视稳定性并愿意等待升级的企业，他们可以在每次 LTS 发布时进行升级。

对 JDK 版本的实验使开发人员能够找到最合适的版本。

### 4.2.支持

当然，还有支持的问题。现在 Java 8 支持已经日落，我们该怎么办？

如前所述，答案来自 LTS 版本， **Java 11 是最新的 LTS 版本，17 是下一个**。Oracle 和 Azul 等供应商将提供并支持更新。

如果我们可以信任社区的支持，那么 Redhat、IBM 和其他公司已经声明支持为 OpenJDK 应用错误修复。另外， [AdoptOpenJDK](https://web.archive.org/web/20221206194521/https://adoptopenjdk.net/) 项目为 OpenJDK 提供了预构建的二进制文件。

### 4.3.批准

一些人感到困惑的一个方面是 OpenJDK 和 Oracle JDK 之间的区别。

据 Brian Goetz 称，[实际上，它们几乎是相同的，不同的只是修补了错误和安全补丁。](https://web.archive.org/web/20221206194521/https://www.infoq.com/podcasts/java-language-architect-brian-goetz)

OpenJDK 充当大多数派生 JDK 的来源，并且保持免费。从 Java 11 开始，甲骨文将对甲骨文 JDK 收取商业许可费，包括额外的支持和服务。

### 4.4.分裂

随着更频繁的发布，碎片可能会成为一个问题。假设，每个人都可以运行不同版本的 Java，拥有比现在更多的不同特性。

当然，集装箱化可以帮助解决这个问题。从 Docker 和 CoreOS 到 Red Hat 的 OpenShift，容器化提供了所需的隔离，不再强迫 Java 在一个安装位置跨服务器使用。

## 5.结论

总之，随着 Java 每六个月一次的定期发布，我们可以对 Oracle 的 Java 团队有更多的期待。作为一名 Java 开发人员，每六个月就有新的语言特性的前景令人兴奋。

让我们记住一些影响，当我们决定我们的升级渠道是什么，如果我们需要支持和许可，以及如何处理碎片。