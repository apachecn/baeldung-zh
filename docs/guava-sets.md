# 番石榴–套装

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-sets>

## 1。概述

在本教程中，我们将举例说明如何最有效地利用 Guava 来处理 Java 集合。

让我们从非常简单的开始，**创建一个没有新操作符的`HashSet`** ，使用番石榴:

```java
Set<String> aNewSet = Sets.newHashSet();
```

## 2。集合的并集

首先，让我们看看如何使用简单的 API 对集合进行联合运算:

```java
@Test
public void whenCalculatingUnionOfSets_thenCorrect() {
    Set<Character> first = ImmutableSet.of('a', 'b', 'c');
    Set<Character> second = ImmutableSet.of('b', 'c', 'd');

    Set<Character> union = Sets.union(first, second);
    assertThat(union, containsInAnyOrder('a', 'b', 'c', 'd'));
}
```

## 3。集合的笛卡尔积

我们还可以使用`Sets.cartesianProduct()`得到两个集合的乘积**，如下例所示:**

```java
@Test
public void whenCalculatingCartesianProductOfSets_thenCorrect() {
    Set<Character> first = ImmutableSet.of('a', 'b');
    Set<Character> second = ImmutableSet.of('c', 'd');
    Set<List<Character>> result =
      Sets.cartesianProduct(ImmutableList.of(first, second));

    Function<List<Character>, String> func =
      new Function<List<Character>, String>() {
        public String apply(List<Character> input) {
            return Joiner.on(" ").join(input);
        }
    };
    Iterable<String> joined = Iterables.transform(result, func);
    assertThat(joined, containsInAnyOrder("a c", "a d", "b c", "b d"));
}
```

请注意–为了能够方便地测试结果，我们使用`Function`和`Joiner`将复杂的`Set<List<Character>>`结构转换为更易于管理的`Iterable<String>`。

## 4。`Sets`十字路口

接下来，让我们看看如何使用`Sets.intersection()` API 得到两个集合的交集**:**

```java
@Test
public void whenCalculatingSetIntersection_thenCorrect() {
    Set<Character> first = ImmutableSet.of('a', 'b', 'c');
    Set<Character> second = ImmutableSet.of('b', 'c', 'd');

    Set<Character> intersection = Sets.intersection(first, second);
    assertThat(intersection, containsInAnyOrder('b', 'c'));
}
```

## 5.集合的对称差

现在，让我们来看看两个集合的对称差异——包含在集合 1 或集合 2 中但不同时包含在两个集合中的所有元素:

```java
@Test
public void whenCalculatingSetSymmetricDifference_thenCorrect() {
    Set<Character> first = ImmutableSet.of('a', 'b', 'c');
    Set<Character> second = ImmutableSet.of('b', 'c', 'd');

    Set<Character> intersection = Sets.symmetricDifference(first, second);
    assertThat(intersection, containsInAnyOrder('a', 'd'));
}
```

## 6。电源设置

现在，让我们看看如何计算幂集，即该集合的所有可能子集的集合。

在下面的例子中，我们使用`Sets.powerSet()`来计算给定字符集的幂集:

```java
@Test
public void whenCalculatingPowerSet_thenCorrect() {
    Set<Character> chars = ImmutableSet.of('a', 'b');

    Set<Set<Character>> result = Sets.powerSet(chars);

    Set<Character> empty =  ImmutableSet.<Character> builder().build();
    Set<Character> a = ImmutableSet.of('a');
    Set<Character> b = ImmutableSet.of('b');
    Set<Character> aB = ImmutableSet.of('a', 'b');

    assertThat(result, contains(empty, a, b, aB));
}
```

## 7。`ContiguousSet`

接下来——让我们看看一组排序的连续值——`ContiguousSet`。

在下面的例子中，我们将一组整数[10，11，…，30]放入一个`ContiguousSet`:

```java
@Test
public void whenCreatingRangeOfIntegersSet_thenCreated() {
    int start = 10;
    int end = 30;
    ContiguousSet<Integer> set = ContiguousSet.create(
      Range.closed(start, end), DiscreteDomain.integers());

    assertEquals(21, set.size());
    assertEquals(10, set.first().intValue());
    assertEquals(30, set.last().intValue());
}
```

这种类型的数据结构当然是你可以在普通 Java 中用一个`TreeSet`-但是**如果你需要用这种方式表示你的数据，这种特殊类型的集合的语义更好用**。

## 8。`RangeSet`

现在，让我们来看看`RangeSet`。我们可以使用`RangeSet`来保存断开的和非空的范围。

在以下示例中，当从两个不相连的范围开始，然后我们将它们连接成一个大范围时:

```java
@Test
public void whenUsingRangeSet_thenCorrect() {
    RangeSet<Integer> rangeSet = TreeRangeSet.create();
    rangeSet.add(Range.closed(1, 10));
    rangeSet.add(Range.closed(12, 15));

    assertEquals(2, rangeSet.asRanges().size());

    rangeSet.add(Range.closed(10, 12));
    assertTrue(rangeSet.encloses(Range.closed(1, 15)));
    assertEquals(1, rangeSet.asRanges().size());
}
```

让我们来详细回顾一下这个例子:

*   首先，我们插入两个不相连的范围:`[1, 10]`和`[12, 15]`
*   接下来，我们添加第三个范围来连接现有的 2: `[10, 12]`
*   最后，我们验证了`RangeSet`是否足够聪明，能够看出 3 个范围现在是一个大范围，并将它们合并成:`[1, 15]`

## 9。`MultiSet`

接下来，我们来讨论如何使用`Multiset`。与普通集合相反， **a `Multiset`支持添加重复元素——它将重复元素计为出现次数**。

在下面的例子中，我们通过一些简单的多集逻辑:

```java
@Test
public void whenInsertDuplicatesInMultiSet_thenInserted() {
    Multiset<String> names = HashMultiset.create();
    names.add("John");
    names.add("Adam", 3);
    names.add("John");

    assertEquals(2, names.count("John"));
    names.remove("John");
    assertEquals(1, names.count("John"));

    assertEquals(3, names.count("Adam"));
    names.remove("Adam", 2);
    assertEquals(1, names.count("Adam"));
}
```

## 10。获取前 N 个元素中的一个`MultiSet`

现在，让我们来看一个更复杂、更有用的使用`MultiSet`的例子。我们将得到集合中出现次数最多的 N 个元素——基本上是最常见的元素。

在下面的例子中，我们使用`Multisets.copyHighCountFirst()`对`Multiset`中的元素进行排序:

```java
@Test
public void whenGetTopOcurringElementsWithMultiSet_thenCorrect() {
    Multiset<String> names = HashMultiset.create();
    names.add("John");
    names.add("Adam", 5);
    names.add("Jane");
    names.add("Tom", 2);

    Set<String> sorted = Multisets.copyHighestCountFirst(names).elementSet();
    List<String> sortedAsList = Lists.newArrayList(sorted);
    assertEquals("Adam", sortedAsList.get(0));
    assertEquals("Tom", sortedAsList.get(1));
}
```

## 11。结论

在这个快速教程中，我们讨论了使用番石榴库处理集合的**的最常见和最有用的用例。**

所有这些示例和代码片段的实现可以在[我的番石榴 github 项目](https://web.archive.org/web/20221113112047/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-set "The Github Project with the impl of all examples using Guava Collections") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。