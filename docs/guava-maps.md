# 番石榴-地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-maps>

## 1。概述

在本教程中，我们将举例说明如何最有效地利用 Guava 来处理 Java Maps。

让我们从非常简单的开始，**使用番石榴创建一个没有`new`操作符的`HashMap`** :

```
Map<String, String> aNewMap = Maps.newHashMap();
```

## 2。`ImmutableMap`

接下来，让我们看看如何用**番石榴**制作`ImmutableMap`:

```
@Test
public void whenCreatingImmutableMap_thenCorrect() {
    Map<String, Integer> salary = ImmutableMap.<String, Integer> builder()
      .put("John", 1000)
      .put("Jane", 1500)
      .put("Adam", 2000)
      .put("Tom", 2000)
      .build();

    assertEquals(1000, salary.get("John").intValue());
    assertEquals(2000, salary.get("Tom").intValue());
}
```

## 3。`SortedMap`

现在，让我们来看看如何创建和使用`SortedMap`。

在下面的例子中，我们使用相应的番石榴构建器创建一个排序的地图:

```
@Test
public void whenUsingSortedMap_thenKeysAreSorted() {
    ImmutableSortedMap<String, Integer> salary = new ImmutableSortedMap
      .Builder<String, Integer>(Ordering.natural())
      .put("John", 1000)
      .put("Jane", 1500)
      .put("Adam", 2000)
      .put("Tom", 2000)
      .build();

    assertEquals("Adam", salary.firstKey());
    assertEquals(2000, salary.lastEntry().getValue().intValue());
}
```

## 4。`BiMap`

接下来，我们来讨论如何使用`BiMap`。我们可以使用`BiMap`将键映射回值，因为它确保值是惟一的。

在下面的例子中，我们创建了一个`BiMap` ，并得到了它的 `inverse()`:

```
@Test
public void whenCreateBiMap_thenCreated() {
    BiMap<String, Integer> words = HashBiMap.create();
    words.put("First", 1);
    words.put("Second", 2);
    words.put("Third", 3);

    assertEquals(2, words.get("Second").intValue());
    assertEquals("Third", words.inverse().get(3));
}
```

## 5。`Multimap`

现在，让我们来看看`Multimap`。

我们可以使用`Multimap`到**将每个键与多个值**相关联，如下例所示:

```
@Test
public void whenCreateMultimap_thenCreated() {
    Multimap<String, String> multimap = ArrayListMultimap.create();
    multimap.put("fruit", "apple");
    multimap.put("fruit", "banana");
    multimap.put("pet", "cat");
    multimap.put("pet", "dog");

    assertThat(multimap.get("fruit"), containsInAnyOrder("apple", "banana"));
    assertThat(multimap.get("pet"), containsInAnyOrder("cat", "dog"));
}
```

## 5。`Table`

现在让我们来看看番石榴`Table`；如果我们需要**多个键来索引一个值**，我们就使用`Table`。

在以下示例中，我们将使用一个表格来存储城市之间的距离:

```
@Test
public void whenCreatingTable_thenCorrect() {
    Table<String,String,Integer> distance = HashBasedTable.create();
    distance.put("London", "Paris", 340);
    distance.put("New York", "Los Angeles", 3940);
    distance.put("London", "New York", 5576);

    assertEquals(3940, distance.get("New York", "Los Angeles").intValue());
    assertThat(distance.columnKeySet(), 
      containsInAnyOrder("Paris", "New York", "Los Angeles"));
    assertThat(distance.rowKeySet(), containsInAnyOrder("London", "New York"));
}
```

我们也可以使用`Tables.transpose()`来翻转行和列键，如下例所示:

```
@Test
public void whenTransposingTable_thenCorrect() {
    Table<String,String,Integer> distance = HashBasedTable.create();
    distance.put("London", "Paris", 340);
    distance.put("New York", "Los Angeles", 3940);
    distance.put("London", "New York", 5576);

    Table<String, String, Integer> transposed = Tables.transpose(distance);

    assertThat(transposed.rowKeySet(), 
      containsInAnyOrder("Paris", "New York", "Los Angeles"));
    assertThat(transposed.columnKeySet(), containsInAnyOrder("London", "New York"));
}
```

## 6。`ClassToInstanceMap`

接下来——我们来看看`ClassToInstanceMap`。如果我们想让对象的类成为键，我们可以使用`ClassToInstanceMap`,如下例所示:

```
@Test
public void whenCreatingClassToInstanceMap_thenCorrect() {
    ClassToInstanceMap<Number> numbers = MutableClassToInstanceMap.create();
    numbers.putInstance(Integer.class, 1);
    numbers.putInstance(Double.class, 1.5);

    assertEquals(1, numbers.get(Integer.class));
    assertEquals(1.5, numbers.get(Double.class));
}
```

## 7。使用`Multimap` 分组`List`

接下来，让我们看看如何使用`Multimap`将 a `List`分组。在下面的例子中，我们使用`Multimaps.index()`将一个`List`的名字按长度分组:

```
@Test
public void whenGroupingListsUsingMultimap_thenGrouped() {
    List<String> names = Lists.newArrayList("John", "Adam", "Tom");
    Function<String,Integer> func = new Function<String,Integer>(){
        public Integer apply(String input) {
            return input.length();
        }
    };
    Multimap<Integer, String> groups = Multimaps.index(names, func);

    assertThat(groups.get(3), containsInAnyOrder("Tom"));
    assertThat(groups.get(4), containsInAnyOrder("John", "Adam"));
}
```

## 8。结论

在这个快速教程中，我们讨论了使用番石榴库处理地图的**的最常见和最有用的用例。**

所有这些示例和代码片段的实现可以在[我的番石榴 GitHub 项目](https://web.archive.org/web/20220525130934/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-map "The Github Project with the impl of all examples using Guava Collections") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。