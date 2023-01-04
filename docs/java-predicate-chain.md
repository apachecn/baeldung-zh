# Java 8 谓词链

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-predicate-chain>

## 1。概述

在这个快速教程中，**我们将讨论在 Java 8 中链接 [`Predicates`](/web/20221028012857/https://www.baeldung.com/cs/predicates) 的不同方式。**

## 2。基本示例

首先，**让我们看看如何使用一个简单的`Predicate`** 来过滤一个`List`的名字:

```java
@Test
public void whenFilterList_thenSuccess(){
   List<String> names = Arrays.asList("Adam", "Alexander", "John", "Tom");
   List<String> result = names.stream()
     .filter(name -> name.startsWith("A"))
     .collect(Collectors.toList());

   assertEquals(2, result.size());
   assertThat(result, contains("Adam","Alexander"));
}
```

在本例中，我们使用`Predicate`过滤了姓名的`List`，只留下以“A”开头的姓名:

```java
name -> name.startsWith("A")
```

但是如果我们想要应用多个`Predicates`呢？

## 3。多个过滤器

如果我们想要应用多个`Predicates`，**，一个选择是简单地链接多个过滤器:**

```java
@Test
public void whenFilterListWithMultipleFilters_thenSuccess(){
    List<String> result = names.stream()
      .filter(name -> name.startsWith("A"))
      .filter(name -> name.length() < 5)
      .collect(Collectors.toList());

    assertEquals(1, result.size());
    assertThat(result, contains("Adam"));
}
```

现在，我们已经更新了示例，通过提取以“A”开头且长度小于 5 的名称来过滤列表。

我们使用了两个过滤器——一个用于一个`Predicate`。

## 4。复杂`Predicate`

现在，我们可以用一个复杂的`Predicate` : 滤波器来代替使用多个滤波器

```java
@Test
public void whenFilterListWithComplexPredicate_thenSuccess(){
    List<String> result = names.stream()
      .filter(name -> name.startsWith("A") && name.length() < 5)
      .collect(Collectors.toList());

    assertEquals(1, result.size());
    assertThat(result, contains("Adam"));
}
```

这个选项比第一个更灵活，因为**我们可以使用位运算来构建我们想要的复杂的`Predicate`** 。

## 5。结合`Predicates`

接下来，如果我们不想使用位运算构建复杂的`Predicate`，Java 8 `Predicate`有一些有用的方法，我们可以用它们来组合`Predicates`。

**我们将使用`Predicate.and()`、`Predicate.or()`和`Predicate.negate().`、**的方法组合`Predicates`

### 5.1。`Predicate.and()`

在这个例子中，我们将显式定义我们的`Predicates`，然后我们将使用`Predicate.and():`组合它们

```java
@Test
public void whenFilterListWithCombinedPredicatesUsingAnd_thenSuccess(){
    Predicate<String> predicate1 =  str -> str.startsWith("A");
    Predicate<String> predicate2 =  str -> str.length() < 5;

    List<String> result = names.stream()
      .filter(predicate1.and(predicate2))
      .collect(Collectors.toList());

    assertEquals(1, result.size());
    assertThat(result, contains("Adam"));
}
```

正如我们所看到的，语法相当直观，方法名暗示了操作的类型。使用`and()`，我们通过只提取满足两个条件的名字来过滤`List`。

### 5.2。`Predicate.or()`

我们也可以用`Predicate.or()`来组合`Predicates.`

让我们提取以“J”开头的名称，以及长度小于 4 的名称:

```java
@Test
public void whenFilterListWithCombinedPredicatesUsingOr_thenSuccess(){
    Predicate<String> predicate1 =  str -> str.startsWith("J");
    Predicate<String> predicate2 =  str -> str.length() < 4;

    List<String> result = names.stream()
      .filter(predicate1.or(predicate2))
      .collect(Collectors.toList());

    assertEquals(2, result.size());
    assertThat(result, contains("John","Tom"));
}
```

### 5.3。`Predicate.negate()`

我们也可以在组合`Predicates`时使用`Predicate.negate()`:

```java
@Test
public void whenFilterListWithCombinedPredicatesUsingOrAndNegate_thenSuccess(){
    Predicate<String> predicate1 =  str -> str.startsWith("J");
    Predicate<String> predicate2 =  str -> str.length() < 4;

    List<String> result = names.stream()
      .filter(predicate1.or(predicate2.negate()))
      .collect(Collectors.toList());

    assertEquals(3, result.size());
    assertThat(result, contains("Adam","Alexander","John"));
}
```

这里，我们使用了`or()`和`negate()`的组合，根据以“J”开头或长度不小于 4 的名字来过滤`List`。

### 5.4。联合收割机`Predicates`内联

我们不需要明确定义我们的`Predicates`来使用`and(),` `or()`，和`negate().`

我们也可以通过将`Predicate`转换成:

```java
@Test
public void whenFilterListWithCombinedPredicatesInline_thenSuccess(){
    List<String> result = names.stream()
      .filter(((Predicate<String>)name -> name.startsWith("A"))
      .and(name -> name.length()<5))
      .collect(Collectors.toList());

    assertEquals(1, result.size());
    assertThat(result, contains("Adam"));
}
```

## 6。`Predicates`组合收藏

最后，**让我们看看如何通过减少它们来链接一个`Predicates`的集合。**

在下面的例子中，我们使用`Predicate.and()`组合了`Predicates`中的`List`:

```java
@Test
public void whenFilterListWithCollectionOfPredicatesUsingAnd_thenSuccess(){
    List<Predicate<String>> allPredicates = new ArrayList<Predicate<String>>();
    allPredicates.add(str -> str.startsWith("A"));
    allPredicates.add(str -> str.contains("d"));        
    allPredicates.add(str -> str.length() > 4);

    List<String> result = names.stream()
      .filter(allPredicates.stream().reduce(x->true, Predicate::and))
      .collect(Collectors.toList());

    assertEquals(1, result.size());
    assertThat(result, contains("Alexander"));
}
```

请注意，我们使用基本标识作为:

```java
x->true
```

但是如果我们想用`Predicate.or()`将它们结合起来，那就不一样了:

```java
@Test
public void whenFilterListWithCollectionOfPredicatesUsingOr_thenSuccess(){
    List<String> result = names.stream()
      .filter(allPredicates.stream().reduce(x->false, Predicate::or))
      .collect(Collectors.toList());

    assertEquals(2, result.size());
    assertThat(result, contains("Adam","Alexander"));
}
```

## 7。结论

在本文中，我们探索了在 Java 8 中链接谓词的不同方式，通过使用`filter(),`构建复合体`Predicates`，并结合`Predicates.`

完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221028012857/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-function)