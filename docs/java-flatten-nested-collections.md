# 在 Java 中展平嵌套集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-flatten-nested-collections>

## 1。概述

在这篇简短的文章中，我们将探索如何在 Java 中展平嵌套集合。

## 2。嵌套集合的示例

假设我们有一个类型为`String`的列表列表。

```
List<List<String>> nestedList = asList(
  asList("one:one"), 
  asList("two:one", "two:two", "two:three"), 
  asList("three:one", "three:two", "three:three", "three:four"));
```

## 3。用`forEach` 拉平`List`

为了将这个嵌套集合展平成一个字符串列表，我们可以使用`forEach`和一个 Java 8 方法引用:

```
public <T> List<T> flattenListOfListsImperatively(
    List<List<T>> nestedList) {
    List<T> ls = new ArrayList<>();
    nestedList.forEach(ls::addAll);
    return ls;
} 
```

在这里你可以看到这个方法在起作用:

```
@Test
public void givenNestedList_thenFlattenImperatively() {
    List<String> ls = flattenListOfListsImperatively(nestedList);

    assertNotNull(ls);
    assertTrue(ls.size() == 8);
    assertThat(ls, IsIterableContainingInOrder.contains(
      "one:one",
      "two:one", "two:two", "two:three", "three:one",
      "three:two", "three:three", "three:four"));
}
```

## 4。用`flatMap` 拉平`List`

我们还可以通过利用来自`Stream` API 的`flatMap`方法来展平嵌套列表。

这允许我们展平嵌套的`Stream`结构，并最终将所有元素收集到一个特定的集合中:

```
public <T> List<T> flattenListOfListsStream(List<List<T>> list) {
    return list.stream()
      .flatMap(Collection::stream)
      .collect(Collectors.toList());    
} 
```

这是实际的逻辑:

```
@Test
public void givenNestedList_thenFlattenFunctionally() {
    List<String> ls = flattenListOfListsStream(nestedList);

    assertNotNull(ls);
    assertTrue(ls.size() == 8);
}
```

## 5。结论

Java 8 中一个简单的`forEach or flatMap`方法，结合方法引用，可以用来展平嵌套集合。

你可以在 GitHub 上找到本文[中讨论的代码。](https://web.archive.org/web/20221208143940/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2)