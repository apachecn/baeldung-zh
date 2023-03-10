# 【新闻】AssertJ 3.6 . x——乔尔·科斯蒂廖拉访谈

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/assertj-3-6-interview>

## 1。简介

[AssertJ](https://web.archive.org/web/20221128114035/https://joel-costigliola.github.io/assertj/index.html) 是一个为 Java 提供流畅断言的库。你可以在这里[和](/web/20221128114035/https://www.baeldung.com/introduction-to-assertj)[了解更多信息。](/web/20221128114035/https://www.baeldung.com/assertJ-java-8-features)

最近，发布了 3.6.0 版本以及两个小的错误修复版本 3.6.1 和 3.6.2。

今天，本库的创建者 Joel Costigliola 和我们在一起，将告诉你更多关于发布和未来计划的信息。

> “我们正努力让 AssertJ 真正面向社区”

### 2。版本 2.6.0 和 3.6.0 几乎是同时发布的。两者有什么区别？

2.x versions target Java 7 while 3.x target Java 8\. Another way of seeing this is that 3.x = 2.x + Java 8 specific features.

### 3。3.6.0/2.6.0 中最显著的变化/增加是什么？

2.6.0 最终有了不同的小特性，但没有大的增加。如果让我选择的话，最有趣的应该是那些与被支持的异常相关的:
–`[hasSuppresedException()](https://web.archive.org/web/20221128114035/https://joel-costigliola.github.io/assertj/assertj-core-news.html#assertj-core-2.6.0-hasSuppressedException)`
–`[hasNoSuppresedExceptions()](https://web.archive.org/web/20221128114035/https://joel-costigliola.github.io/assertj/assertj-core-news.html#assertj-core-2.6.0-hasNoSuppressedExceptions)`

3.6.0 additionally got a way of checking multiples assertions on array/iterable/map entry elements:– `[allSatisfy()](https://web.archive.org/web/20221128114035/https://joel-costigliola.github.io/assertj/assertj-core-news.html#assertj-core-3.6.0-allSatisfy)`– `[hasEntrySatisfying()](https://web.archive.org/web/20221128114035/https://joel-costigliola.github.io/assertj/assertj-core-news.html#assertj-core-3.6.0-hasEntrySatisfying)`

### 4。自 3.6.0 发布以来，出现了两个 Bugfix 版本(3.6.1，3.6.2)。你能告诉我们更多一点那里发生了什么，需要修复什么吗？

在 3.6.1 中，`filteredOn(Predicate)`只和`List`一起工作，但和`Iterable,`不太好。

在 3.6.2 中，我们没有想到从 Java 8 默认的 getter 方法中提取属性，事实证明，经过一些内部重构后，它并没有开箱即用。

我问用户他们是否可以等待下一个版本，bug 报告员告诉我他可以等待，但是另一个用户想要，所以我发布了一个新版本。我们正努力使 [AssertJ](https://web.archive.org/web/20221128114035/https://joel-costigliola.github.io/assertj/) 真正面向社区，因为削减一个版本是很便宜的(除了文档部分)，我通常看不出发布有什么问题。

### 5。在开发最新版本时，您遇到过什么有趣的技术挑战吗？

我将指出我在下一个版本 3.7.0 中遇到的一个问题，该版本将在几周内发布。

Java 8 对“不明确的”方法签名很挑剔。我们添加了一个新的 assertThat 方法，它接受一个`ThrowingCallable`(一个简单的类，它是一个抛出异常的`Callable`)，结果是 Java 8 把它和另一个接受`Iterable!`的`assertThat`方法混淆了

这是最令我惊讶的，因为我看不出这两者之间有任何模糊之处。

### 6。你计划在近期发布新的主要版本吗？任何利用 Java 9 附加功能的东西？

在接下来的几周/一个月。我们通常试着每隔几个月发布一次，或者在有重大增加的时候发布。

加入 AssertJ 团队的 Pascal Schumacher 在 Java 9 上做了一些工作来检查兼容性，但有一些东西不起作用，主要是那些依赖于自省的东西，因为 Java 9 改变了访问规则。我们要做的是启动一个以 Java 9 为中心的 4.x 分支，遵循与 3.x 和 2.x 相同的策略，我们将拥有 4.x = 3.x + Java 9 的特性。

一旦 4.0 正式发布，我们可能会停止 2.x 的积极开发，但继续接受 PRs，因为我们没有能力保持 3 个版本同步，我的意思是，我们报告从 n.x 版本到 n+1.x 版本的任何变化，所以在 2.x 中添加一些东西需要在 3.x 和 4.x 中报告，这在目前是太多的工作。