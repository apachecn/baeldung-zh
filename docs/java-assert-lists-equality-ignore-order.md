# 断言两个列表相等，忽略 Java 中的顺序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-assert-lists-equality-ignore-order>

## 1.概观

有时当编写单元测试时，我们需要对列表进行顺序不可知的比较。在这个简短的教程中，我们将看看如何编写这样的单元测试的不同例子。

## 2.设置

根据`[List#equals](https://web.archive.org/web/20221011102112/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#equals(java.lang.Object))` Java 文档，如果两个列表以相同的顺序包含相同的元素，那么它们就是相等的。因此，我们不能仅仅使用`equals`方法，因为我们想要进行顺序不可知的比较。

在本教程中，我们将使用这三个列表作为测试的示例输入:

```java
List first = Arrays.asList(1, 3, 4, 6, 8);
List second = Arrays.asList(8, 1, 6, 3, 4);
List third = Arrays.asList(1, 3, 3, 6, 6);
```

进行顺序不可知比较有不同的方法。让我们一个一个来看看。

## 3.使用 JUnit

JUnit 是一个众所周知的用于 Java 生态系统中单元测试的框架。

我们可以使用下面的逻辑，使用 [`assertTrue`和`assertFalse`](/web/20221011102112/https://www.baeldung.com/junit-assertions#junit5-condition) 方法来比较两个列表的相等性。

这里我们检查两个列表的大小，并检查第一个列表是否包含第二个列表的所有元素，反之亦然。虽然这个解决方案可行，但可读性不是很好。现在让我们来看一些替代方案:

```java
@Test
public void whenTestingForOrderAgnosticEquality_ShouldBeTrue() {
    assertTrue(first.size() == second.size() && first.containsAll(second) && second.containsAll(first));
}
```

在第一个测试中，在我们检查两个列表中的元素是否相同之前，比较两个列表的大小。当这两个条件都出现时，我们的测试就通过了。

现在让我们来看看一个失败的测试:

```java
@Test
public void whenTestingForOrderAgnosticEquality_ShouldBeFalse() {
    assertFalse(first.size() == third.size() && first.containsAll(third) && third.containsAll(first));
}
```

相比之下，在这个版本的测试中，虽然两个列表的大小相同，但所有元素都不匹配。

## 4.使用 AssertJ

AssertJ 是一个开源社区驱动的库，用于在 Java 测试中编写流畅而丰富的断言。

为了在我们的 maven 项目中使用它，让我们在`pom.xml`文件中添加 [`assertj-core`](https://web.archive.org/web/20221011102112/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22assertj-core%22) 依赖项:

```java
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.16.1</version>
</dependency>
```

让我们编写一个测试来比较相同元素和相同大小的两个列表实例的相等性:

```java
@Test
void whenTestingForOrderAgnosticEqualityBothList_ShouldBeEqual() {
    assertThat(first).hasSameElementsAs(second);
}
```

在这个例子中，我们验证了`first`包含了给定 iterable 的所有元素，除此之外，没有任何顺序。这种方法的主要限制是`hasSameElementsAs`方法会忽略重复项。

让我们在实践中看看这是什么意思:

```java
@Test
void whenTestingForOrderAgnosticEqualityBothList_ShouldNotBeEqual() {
    List a = Arrays.asList("a", "a", "b", "c");
    List b = Arrays.asList("a", "b", "c");
    assertThat(a).hasSameElementsAs(b);
}
```

在这个测试中，虽然我们有相同的元素，但是两个列表的大小不相等，但是断言仍然为真，因为它忽略了重复的元素。为了使它工作，我们需要为两个列表添加一个大小检查:

```java
assertThat(a).hasSize(b.size()).hasSameElementsAs(b);
```

在方法`hasSameElementsAs`之后添加对两个列表大小的检查确实会像预期的那样失败。

## 5.使用 Hamcrest

如果我们已经在使用 Hamcrest 或者想用它来编写单元测试，下面是我们如何使用 [`Matchers#containsInAnyOrder`](/web/20221011102112/https://www.baeldung.com/hamcrest-collections-arrays) 方法进行顺序不可知的比较。

为了在我们的 maven 项目中使用 Hamcrest，让我们在`pom.xml`文件中添加 [`hamcrest-all`](https://web.archive.org/web/20221011102112/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22hamcrest-all%22) 依赖项:

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-all</artifactId>
    <version>1.3</version>
</dependency>
```

让我们来看看测试:

```java
@Test
public void whenTestingForOrderAgnosticEquality_ShouldBeEqual() {
    assertThat(first, Matchers.containsInAnyOrder(second.toArray()));
}
```

这里，方法`containsInAnyOrder`为`Iterables`创建了一个顺序不可知的匹配器，它与被检查的`Iterable` 元素进行匹配。该测试匹配两个列表的元素，忽略列表中元素的顺序。

幸运的是，这个解决方案没有遇到上一节中解释的相同问题，所以我们不需要明确地比较大小。

## 6.使用 Apache Commons

除了 JUnit、Hamcrest 或 AssertJ 之外，我们可以使用的另一个库或框架是 [`Apache` `CollectionUtils`](/web/20221011102112/https://www.baeldung.com/apache-commons-collection-utils) 。它为覆盖广泛用例的常见操作提供了实用方法，并帮助我们避免编写样板代码。

为了在我们的 maven 项目中使用它，让我们在`pom.xml`文件中添加 [`commons-collections4`](https://web.archive.org/web/20221011102112/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-collections4%22) 依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

下面是一个使用`CollectionUtils`的测试:

```java
@Test
public void whenTestingForOrderAgnosticEquality_ShouldBeTrueIfEqualOtherwiseFalse() {
    assertTrue(CollectionUtils.isEqualCollection(first, second));
    assertFalse(CollectionUtils.isEqualCollection(first, third));
}
```

如果给定的集合包含具有相同基数[和](https://web.archive.org/web/20221011102112/https://simple.wikipedia.org/wiki/Cardinality#:~:text=In%20mathematics%2C%20the%20cardinality%20of,the%20same%20number%20of%20elements.)的完全相同的元素，则`isEqualCollection` 方法返回`true`。否则，它返回`false`。

## 7.结论

在本文中，我们探讨了如何检查两个`List `实例的相等性，其中两个列表的元素排序不同。

所有这些例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221011102112/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-assertions)