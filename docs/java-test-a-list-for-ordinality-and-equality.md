# 在 Java 中检查两个列表是否相等

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-test-a-list-for-ordinality-and-equality>

## 1。简介

在这篇短文中，我们将关注测试两个`List`实例是否以完全相同的顺序包含相同的元素这一常见问题。

`[List](https://web.archive.org/web/20221022111202/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html)` 是一个**有序的**数据结构，所以元素的顺序是由设计决定的。

请看一下`[List#equals](https://web.archive.org/web/20221022111202/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#equals(java.lang.Object))` Java 文档的摘录:

> …如果两个列表以相同的顺序包含相同的元素，则它们被定义为相等。

这个定义确保 equals 方法在 List 接口的不同实现中都能正常工作。

我们可以在编写断言时使用这些知识。

在以下代码片段中，我们将使用以下列表作为示例输入:

```java
List<String> list1 = Arrays.asList("1", "2", "3", "4");
List<String> list2 = Arrays.asList("1", "2", "3", "4");
List<String> list3 = Arrays.asList("1", "2", "4", "3");
```

## 2。JUnit

在纯 JUnit 测试中，以下断言将是正确的:

```java
@Test
public void whenTestingForEquality_ShouldBeEqual() throws Exception {
    Assert.assertEquals(list1, list2);
    Assert.assertNotSame(list1, list2);
    Assert.assertNotEquals(list1, list3);
}
```

## 3。测试

当使用 TestNG 的断言时，它们看起来与 JUnit 的断言非常相似，但是需要注意的是 [`Assert`](https://web.archive.org/web/20221022111202/https://static.javadoc.io/org.testng/testng/6.9.5/org/testng/Assert.html) 类来自不同的包:

```java
@Test
public void whenTestingForEquality_ShouldBeEqual() throws Exception {
    Assert.assertEquals(list1, list2);
    Assert.assertNotSame(list1, list2);
    Assert.assertNotEquals(list1, list3);
}
```

## 4。评估

如果您喜欢使用 [AssertJ](https://web.archive.org/web/20221022111202/https://joel-costigliola.github.io/assertj/) ，它的断言将如下所示:

```java
@Test
public void whenTestingForEquality_ShouldBeEqual() throws Exception {
    assertThat(list1)
      .isEqualTo(list2)
      .isNotEqualTo(list3);

    assertThat(list1.equals(list2)).isTrue();
    assertThat(list1.equals(list3)).isFalse();
}
```

## 5。结论

在本文中，我们探讨了如何测试两个`List`实例是否以相同的顺序包含相同的元素。这个问题最重要的部分是正确理解`List`数据结构是如何工作的。

所有代码示例都可以在 [GitHub](https://web.archive.org/web/20221022111202/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2) 上找到。