# JUnit 5 @测试注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-test-annotation>

## 1。概述

在本文中，我们将快速回顾一下 JUnit 的`@Test`注释。这个注释为执行单元和回归测试提供了一个强大的工具。

## 2。Maven 配置

要使用 JUnit 5 的最新版本[，我们需要添加以下 Maven 依赖项:](https://web.archive.org/web/20220628055528/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.junit.jupiter%22%20AND%20a%3A%22junit-jupiter-engine%22)

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

我们使用`test`范围是因为我们不希望 Maven 在我们的最终构建中包含这种依赖性。

由于 surefire 插件本身并不完全支持 JUnit 5，**我们还需要[添加一个提供者](https://web.archive.org/web/20220628055528/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.junit.platform%22%20AND%20a%3A%22junit-platform-surefire-provider%22)** ，它告诉 Maven 在哪里可以找到我们的测试:

```java
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.19.1</version>
    <dependencies>
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-surefire-provider</artifactId>
            <version>1.0.2</version>
        </dependency>
    </dependencies>
</plugin>
```

在我们的配置中，我们将使用 surefire 2.19.1，因为在撰写本文时，**2.20 . x 版本与 `junit-platform-surefire-provider`** 不兼容。

## 3。被测方法

首先，让我们构建一个简单的方法，我们将在测试场景中使用它来展示`@Test`注释的功能:

```java
public boolean isNumberEven(Integer number) {
    return number % 2 == 0;
}
```

如果传递的参数是偶数，这个方法应该返回`true` ，否则返回`false`。现在，让我们检查一下它是否按照预期的方式工作。

## 4。测试方法

在我们的例子中，我们想要特别检查两个场景:

*   当给定一个偶数时，该方法应该返回`true`
*   当给定一个奇数时，该方法应该返回`false`

这意味着实现代码将使用不同的参数调用我们的`isNumberEven`方法，并检查结果是否是我们所期望的。

**为了让测试被识别出来，我们将添加`@Test` 注释。**我们可以在一个类中包含任意数量的这些内容，但最好只将相关的内容放在一起。还要注意**测试不能是`private,`也不能返回值**——否则它会被忽略。

考虑到这些因素，让我们编写我们的测试方法:

```java
@Test
void givenEvenNumber_whenCheckingIsNumberEven_thenTrue() {
    boolean result = bean.isNumberEven(8);

    Assertions.assertTrue(result);
}

@Test
void givenOddNumber_whenCheckingIsNumberEven_thenFalse() {
    boolean result = bean.isNumberEven(3);

    Assertions.assertFalse(result);
}
```

如果我们现在运行一个 Maven 构建，**surefire 插件将遍历放置在`src/test/java` 下的类中所有带注释的方法并执行它们**，如果任何测试失败发生`.`，导致构建失败

如果您来自 JUnit 4，**请注意，在这个版本中，注释不接受任何参数。**为了检查超时或抛出的异常，我们将使用断言:

```java
@Test
void givenLowerThanTenNumber_whenCheckingIsNumberEven_thenResultUnderTenMillis() {
    Assertions.assertTimeout(Duration.ofMillis(10), () -> bean.isNumberEven(3));
}

@Test
void givenNull_whenCheckingIsNumberEven_thenNullPointerException() {
    Assertions.assertThrows(NullPointerException.class, () -> bean.isNumberEven(null));
}
```

## 5。结论

在这个快速教程中，我们展示了如何用`@Test`注释实现和运行一个简单的 JUnit 测试。

关于 JUnit 框架的更多信息可以在[这篇文章](/web/20220628055528/https://www.baeldung.com/junit-5)中找到，这篇文章提供了一个总体介绍。

示例中使用的所有代码都可以在 [GitHub 项目](https://web.archive.org/web/20220628055528/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-basics)中获得。