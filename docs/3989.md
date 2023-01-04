# mockito-core 和 mockito-all 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-core-vs-mockito-all>

## 1.概观

Mockito 是一个流行的 Java 模仿框架。但是，在我们开始之前，我们有一些不同的工件可供选择。

在这个快速教程中，我们将探索`mockito-core`和`mockito-all`之间的区别。之后，我们将能够选择正确的。

## 2.`mockito-core`

**[`mockito-core`神器](https://web.archive.org/web/20220627081848/https://search.maven.org/artifact/org.mockito/mockito-core)是莫克托的主要神器。**具体来说，它包含了 API 和实现库。

我们可以通过将依赖项添加到我们的`pom.xml`来获得工件:

```
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.3.3</version>
</dependency>
```

至此，我们已经可以开始使用 [Mockito](/web/20220627081848/https://www.baeldung.com/mockito-series) 了。

## 3.`mockito-all`

当然，`mockito-core`有一些像`hamcrest `和`objenesis`这样的依赖项，Maven 可以单独下载，但是`mockito-all` 是**的一个过时的依赖项，它捆绑了** **的 Mockito 及其所需的依赖项**。

为了验证这一点，让我们看看`mockito-all.jar`的内部，看看它包含的包:

```
mockito-all.jar
|-- org
|   |-- hamcrest
|   |-- mockito
|   |-- objenesis
```

[`mockito-all`](https://web.archive.org/web/20220627081848/https://search.maven.org/artifact/org.mockito/mockito-all) 最新 GA 版本是 2014 年发布的 1.x 版本。**新版的 Mockito 不再发布`mockito-all`**。

维护者发布这种依赖是为了简化。如果开发人员没有带有依赖管理的构建工具，他们应该使用这种方法。

## 4.结论

正如我们上面所探讨的，`mockito-core`是 Mockito 的主要工件。更新的版本不再发布`mockito-all`。从今以后，我们应该只用`mockito-core`。