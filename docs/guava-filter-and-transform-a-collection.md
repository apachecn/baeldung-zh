# 在番石榴中过滤和转换集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-filter-and-transform-a-collection>

## 1。概述

在本教程中，我们将说明如何用 Guava 过滤和转换集合。

我们将使用[谓词](/web/20221109222622/https://www.baeldung.com/cs/predicates)进行过滤，使用库提供的函数进行转换，最后，我们将看到如何将过滤和转换结合起来。

## 延伸阅读:

## [新的流，比较器和收集器在番石榴 21](/web/20221109222622/https://www.baeldung.com/guava-21-new)

Quick and practical guide to tools in the common.collect package in Guava 21.[Read more](/web/20221109222622/https://www.baeldung.com/guava-21-new) →

## [番石榴多地图指南](/web/20221109222622/https://www.baeldung.com/guava-multimap)

A short guide to Guava Multimap in comparison with standard java.util.Map[Read more](/web/20221109222622/https://www.baeldung.com/guava-multimap) →

## [番石榴系列指南](/web/20221109222622/https://www.baeldung.com/guava-rangeset)

Learn how to use the Google Guava RangeSet and its implementations through practical examples.[Read more](/web/20221109222622/https://www.baeldung.com/guava-rangeset) →

## 2。过滤一个集合

让我们从一个简单的**过滤集合**的例子开始。我们将使用一个由库提供并通过`Predicates`实用程序类构建的现成谓词:

```java
@Test
public void whenFilterWithIterables_thenFiltered() {
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Iterable<String> result 
      = Iterables.filter(names, Predicates.containsPattern("a"));

    assertThat(result, containsInAnyOrder("Jane", "Adam"));
}
```

正如你所看到的，我们正在过滤名字的`List`，只得到包含字符“a”的名字——我们使用`Iterables.filter()`来做这件事。

或者，我们也可以很好地利用`Collections2.filter()` API:

```java
@Test
public void whenFilterWithCollections2_thenFiltered() {
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Collection<String> result 
      = Collections2.filter(names, Predicates.containsPattern("a"));

    assertEquals(2, result.size());
    assertThat(result, containsInAnyOrder("Jane", "Adam"));

    result.add("anna");
    assertEquals(5, names.size());
}
```

这里需要注意一些事情——首先，`Collections.filter()`的输出是**原始集合**的实时视图——对其中一个的更改将反映在另一个中。

理解这一点也很重要，现在，**结果受到谓词**的约束——如果我们添加一个不满足那个`Predicate`的元素，就会抛出一个`IllegalArgumentException`:

```java
@Test(expected = IllegalArgumentException.class)
public void givenFilteredCollection_whenAddingInvalidElement_thenException() {
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Collection<String> result 
      = Collections2.filter(names, Predicates.containsPattern("a"));

    result.add("elvis");
}
```

## 3。编写自定义滤镜`Predicate`

接下来——让我们写自己的`Predicate`,而不是使用库提供的。在下面的示例中，我们将定义一个谓词，该谓词仅获取以“A”或“J”开头的名称:

```java
@Test
public void whenFilterCollectionWithCustomPredicate_thenFiltered() {
    Predicate<String> predicate = new Predicate<String>() {
        @Override
        public boolean apply(String input) {
            return input.startsWith("A") || input.startsWith("J");
        }
    };

    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Collection<String> result = Collections2.filter(names, predicate);

    assertEquals(3, result.size());
    assertThat(result, containsInAnyOrder("John", "Jane", "Adam"));
}
```

## 4。组合多个谓词

我们可以使用`Predicates.or()`和`Predicates.and()`组合多个谓词。
在下面的例子中，我们过滤了一个`List`名称，以获得以“J”开头或不包含“a”的名称:

```java
@Test
public void whenFilterUsingMultiplePredicates_thenFiltered() {
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Collection<String> result = Collections2.filter(names, 
      Predicates.or(Predicates.containsPattern("J"), 
      Predicates.not(Predicates.containsPattern("a"))));

    assertEquals(3, result.size());
    assertThat(result, containsInAnyOrder("John", "Jane", "Tom"));
}
```

## 5。过滤集合时移除空值

我们可以通过使用`Predicates.notNull()`过滤来清除集合中的`null`值，如下例所示:

```java
@Test
public void whenRemoveNullFromCollection_thenRemoved() {
    List<String> names = 
      Lists.newArrayList("John", null, "Jane", null, "Adam", "Tom");
    Collection<String> result = 
      Collections2.filter(names, Predicates.notNull());

    assertEquals(4, result.size());
    assertThat(result, containsInAnyOrder("John", "Jane", "Adam", "Tom"));
}
```

## 6。检查集合中的所有元素是否匹配条件

接下来，让我们检查集合中的所有元素是否都符合某个条件。我们将使用`Iterables.all()`来检查是否所有的名字都包含“n”或“m”，然后我们将检查是否所有的元素都包含“a”:

```java
@Test
public void whenCheckingIfAllElementsMatchACondition_thenCorrect() {
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");

    boolean result = Iterables.all(names, Predicates.containsPattern("n|m"));
    assertTrue(result);

    result = Iterables.all(names, Predicates.containsPattern("a"));
    assertFalse(result);
}
```

## 7。变换一个集合

现在，让我们看看如何使用芭乐`Function` 来**改造一个系列。在下面的例子中，我们将一个名字的`List`转换成一个`Integers`(名字的长度)的`List`和`Iterables.transform()`:**

```java
@Test
public void whenTransformWithIterables_thenTransformed() {
    Function<String, Integer> function = new Function<String, Integer>() {
        @Override
        public Integer apply(String input) {
            return input.length();
        }
    };

    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Iterable<Integer> result = Iterables.transform(names, function);

    assertThat(result, contains(4, 4, 4, 3));
}
```

我们也可以像下面的例子一样使用 `Collections2.transform()` API:

```java
@Test
public void whenTransformWithCollections2_thenTransformed() {
    Function<String,Integer> func = new Function<String,Integer>(){
        @Override
        public Integer apply(String input) {
            return input.length();
        }
    };

    List<String> names = 
      Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Collection<Integer> result = Collections2.transform(names, func);

    assertEquals(4, result.size());
    assertThat(result, contains(4, 4, 4, 3));

    result.remove(3);
    assertEquals(3, names.size());
}
```

请注意，`Collections.transform()`的输出是**原始`Collection`** 的实时视图——对其中一个的更改会影响另一个。

和以前一样，如果我们试图向输出`Collection`添加一个元素，就会抛出一个`UnsupportedOperationException`。

## 8。从`Predicate`到创建`Function`

我们也可以使用`Functions.fromPredicate()`从`Predicate`创建`Function`。当然，这将是一个根据谓词的条件将输入转换为`Boolean`的函数。

在下面的例子中，我们将一个名字`List`转换成一个布尔列表，其中每个元素表示名字是否包含“m”:

```java
@Test
public void whenCreatingAFunctionFromAPredicate_thenCorrect() {
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Collection<Boolean> result =
      Collections2.transform(names,
      Functions.forPredicate(Predicates.containsPattern("m")));

    assertEquals(4, result.size());
    assertThat(result, contains(false, false, true, true));
}
```

## 9。两个功能的组合

接下来——让我们看看如何使用组合的`Function`来转换集合。

`Functions.compose()`将第二个`Function`应用于第一个`Function`的输出时，返回两个函数的组合。

在下面的例子中，第一个`Function`将名称转换成它的长度，然后第二个`Function`将长度转换成一个`boolean`值，该值表示名称的长度是否是偶数:

```java
@Test
public void whenTransformingUsingComposedFunction_thenTransformed() {
    Function<String,Integer> f1 = new Function<String,Integer>(){
        @Override
        public Integer apply(String input) {
            return input.length();
        }
    };

    Function<Integer,Boolean> f2 = new Function<Integer,Boolean>(){
        @Override
        public Boolean apply(Integer input) {
            return input % 2 == 0;
        }
    };

    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Collection<Boolean> result = 
      Collections2.transform(names, Functions.compose(f2, f1));

    assertEquals(4, result.size());
    assertThat(result, contains(true, true, true, false));
}
```

## 10。组合滤波和变换

现在，让我们看看 Guava 拥有的另一个很酷的 API 一个实际上允许我们一起链式过滤和转换的 API—`FluentIterable`。

在下面的例子中，我们过滤名字的`List`，然后使用`FluentIterable`对其进行转换:

```java
@Test
public void whenFilteringAndTransformingCollection_thenCorrect() {
    Predicate<String> predicate = new Predicate<String>() {
        @Override
        public boolean apply(String input) {
            return input.startsWith("A") || input.startsWith("T");
        }
    };

    Function<String, Integer> func = new Function<String,Integer>(){
        @Override
        public Integer apply(String input) {
            return input.length();
        }
    };

    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    Collection<Integer> result = FluentIterable.from(names)
                                               .filter(predicate)
                                               .transform(func)
                                               .toList();

    assertEquals(2, result.size());
    assertThat(result, containsInAnyOrder(4, 3));
}
```

值得一提的是，在某些情况下，命令式的可读性更强，应该比函数式的更好。

## 11。结论

最后，我们学习了如何使用番石榴过滤和转换集合。我们使用了`Collections2.filter()`和`Iterables.filter()`API 来过滤，以及`Collections2.transform()`和`Iterables.transform()`来转换集合。

最后，我们快速浏览了一下非常有趣的`FluentIterable` fluent API，它将过滤和转换结合在一起。

所有这些示例和代码片段**的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221109222622/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections "The Github Project with the impl of all examples using Guava Collections")** 中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。