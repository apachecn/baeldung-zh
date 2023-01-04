# 在 Java 中设置 vs 列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-set-vs-list>

## 1.概观

在本教程中，我们将**借助一个简单的例子来讨论 Java** 中`[Set](/web/20221017061447/https://www.baeldung.com/java-set-operations)`和`[List](/web/20221017061447/https://www.baeldung.com/java-arraylist)`的区别。

## 2.概念差异

`List` 和`Set`都是 Java `Collections`的成员。但是，有几个重要的区别:

*   一个`List`可以包含重复，但是一个`Set`不能
*   一个`List`将保持插入的顺序，但是一个`Set`可能会也可能不会
*   由于在`Set`中可能没有维护插入顺序，所以它不允许像在`List`中那样基于索引的访问

请注意，`Set` 接口有一些维护顺序的实现，例如，`LinkedHashSet`。

## 3.代码示例

### 3.1.允许重复

`List`允许添加重复项目。然而，它不是为一个`Set`:

```
@Test
public void givenList_whenDuplicates_thenAllowed(){
    List<Integer> integerList = new ArrayList<>();
    integerList.add(2);
    integerList.add(3);
    integerList.add(4);
    integerList.add(4);
    assertEquals(integerList.size(), 4);
} 
```

```
@Test
public void givenSet_whenDuplicates_thenNotAllowed(){
    Set<Integer> integerSet = new HashSet<>();
    integerSet.add(2);
    integerSet.add(3);
    integerSet.add(4);
    integerSet.add(4);
    assertEquals(integerSet.size(), 3);
}
```

### 3.2.保持插入顺序

一个`Set`根据实现来维持秩序。例如，`HashSet`不能保证维持秩序，但是`LinkedHashSet`可以。让我们看一个用`LinkedHashSet`订购的例子:

```
@Test
public void givenSet_whenOrdering_thenMayBeAllowed(){
    Set<Integer> set1 = new LinkedHashSet<>();
    set1.add(2);
    set1.add(3);
    set1.add(4);
    Set<Integer> set2 = new LinkedHashSet<>();
    set2.add(2);
    set2.add(3);
    set2.add(4);
    Assert.assertArrayEquals(set1.toArray(), set2.toArray());
}
```

由于一个`Set`不能保证维持顺序，所以不能被索引。

## 4.结论

在本教程中，我们看到了 Java 中的`List`和`Set`的区别。

GitHub 上的[提供了源代码。](https://web.archive.org/web/20221017061447/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-4)