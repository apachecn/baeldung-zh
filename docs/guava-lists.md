# 番石榴–列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-lists>

## 1。概述

在本教程中，我们将举例说明使用番石榴库处理列表的最常见和最有用的方法。

让我们从简单的开始——看看**使用 Guava 语法创建一个新的`ArrayList`**——没有`new`:

```java
List<String> names = Lists.newArrayList("John", "Adam", "Jane");
```

## 2。`List` 反转一个

首先，让我们**使用`Lists.reverse()`反转一个`List`** ，如下例所示:

```java
@Test
public void whenReverseList_thenReversed() {
    List<String> names = Lists.newArrayList("John", "Adam", "Jane");

    List<String> reversed = Lists.reverse(names);
    assertThat(reversed, contains("Jane", "Adam", "John"));
}
```

## 3。从`String` 生成`Character` `List`

现在，让我们看看如何将一个字符串分解成一个列表`Characters`。

在下面的例子中——我们使用`Lists.CharactersOf()` API 从`String` `“John”`创建一个`Character` `List`:

```java
@Test
public void whenCreateCharacterListFromString_thenCreated() {
    List<Character> chars = Lists.charactersOf("John");

    assertEquals(4, chars.size());
    assertThat(chars, contains('J', 'o', 'h', 'n'));
}
```

## 4。`List`分区一

接下来——让我们看看如何将**分区为** `List`。

在下面的例子中——我们使用`Lists.partition()`来获得连续的大小为 2 的子列表:

```java
@Test
public void whenPartitionList_thenPartitioned(){
    List<String> names = Lists.newArrayList("John","Jane","Adam","Tom","Viki","Tyler");

    List<List<String>> result = Lists.partition(names, 2);

    assertEquals(3, result.size());
    assertThat(result.get(0), contains("John", "Jane"));
    assertThat(result.get(1), contains("Adam", "Tom"));
    assertThat(result.get(2), contains("Viki", "Tyler"));
}
```

## 5。从`List` 中删除重复项

现在，让我们用一个简单的技巧来删除`List`中的重复项。

在下面的例子中，我们将元素复制到一个`Set`中，然后从剩余的元素中创建一个`List`:

```java
@Test
public void whenRemoveDuplicatesFromList_thenRemoved() {
    List<Character> chars = Lists.newArrayList('h','e','l','l','o');
    assertEquals(5, chars.size());

    List<Character> result = ImmutableSet.copyOf(chars).asList();
    assertThat(result, contains('h', 'e', 'l', 'o'));
}
```

## 6。从`List` 中删除空值

接下来——让我们看看如何通过**从`List`** 中移除`null`值。

在下面的例子中，我们使用非常有用的`Iterables.removeIf()` API 和库本身提供的谓词删除了所有的`null`值:

```java
@Test
public void whenRemoveNullFromList_thenRemoved() {
    List<String> names = Lists.newArrayList("John", null, "Adam", null, "Jane");
    Iterables.removeIf(names, Predicates.isNull());

    assertEquals(3, names.size());
    assertThat(names, contains("John", "Adam", "Jane"));
}
```

## 7。将一个`List`转换为一个`ImmutableList`

最后，让我们看看如何使用`ImmutableList.copyOf()` API 创建一个`List`—`ImmutableList`的不可变副本:

```java
@Test
public void whenCreateImmutableList_thenCreated() {
    List<String> names = Lists.newArrayList("John", "Adam", "Jane");

    names.add("Tom");
    assertEquals(4, names.size());

    ImmutableList<String> immutable = ImmutableList.copyOf(names);
    assertThat(immutable, contains("John", "Adam", "Jane", "Tom"));
}
```

## 8。结论

这就是我们——一个快速教程，回顾了你可以用番石榴做的大多数有用的事情。

要更深入地研究列表，请查看[谓词和函数列表指南](/web/20220812064513/https://www.baeldung.com/guava-filter-and-transform-a-collection "Filtering and Transforming Collections in Guava")以及深入的[在 Guava 中连接和拆分列表指南](/web/20220812064513/https://www.baeldung.com/guava-joiner-and-splitter-tutorial "Guava – Join and Split Collections")。

所有这些示例和代码片段的实现可以在[我的番石榴 github 项目](https://web.archive.org/web/20220812064513/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-list "The Github Project with the impl of all examples using Guava Collections") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。