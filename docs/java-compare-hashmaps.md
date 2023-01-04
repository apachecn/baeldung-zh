# 比较 Java 中的两个 HashMaps

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compare-hashmaps>

## 1。概述

在本教程中，**我们将探索在 Java** 中比较两个`HashMaps` 的不同方法。

我们将讨论检查两个`HashMaps`是否相似的多种方法。我们还将使用 Java 8 Stream API 和 Guava 来获得不同`HashMaps`之间的详细差异。

## 2。使用`Map.equals()`

首先，我们将使用`Map.equals()`来检查两个`HashMaps`是否有相同的条目:

```
@Test
public void whenCompareTwoHashMapsUsingEquals_thenSuccess() {
    Map<String, String> asiaCapital1 = new HashMap<String, String>();
    asiaCapital1.put("Japan", "Tokyo");
    asiaCapital1.put("South Korea", "Seoul");

    Map<String, String> asiaCapital2 = new HashMap<String, String>();
    asiaCapital2.put("South Korea", "Seoul");
    asiaCapital2.put("Japan", "Tokyo");

    Map<String, String> asiaCapital3 = new HashMap<String, String>();
    asiaCapital3.put("Japan", "Tokyo");
    asiaCapital3.put("China", "Beijing");

    assertTrue(asiaCapital1.equals(asiaCapital2));
    assertFalse(asiaCapital1.equals(asiaCapital3));
}
```

这里，我们创建三个`HashMap`对象并添加条目。然后我们使用`Map.equals()`来检查两个`HashMaps`是否有相同的条目。

**`Map.equals()`的工作方式是通过使用** `**Object.equals()**` **方法** `.`来比较键和值，这意味着只有当键和值对象都正确实现了`equals()`时，它才会工作。

例如，当值类型为 array 时，`Map.equals()`不起作用，因为数组的`equals()`方法比较的是标识，而不是数组的内容:

```
@Test
public void whenCompareTwoHashMapsWithArrayValuesUsingEquals_thenFail() {
    Map<String, String[]> asiaCity1 = new HashMap<String, String[]>();
    asiaCity1.put("Japan", new String[] { "Tokyo", "Osaka" });
    asiaCity1.put("South Korea", new String[] { "Seoul", "Busan" });

    Map<String, String[]> asiaCity2 = new HashMap<String, String[]>();
    asiaCity2.put("South Korea", new String[] { "Seoul", "Busan" });
    asiaCity2.put("Japan", new String[] { "Tokyo", "Osaka" });

    assertFalse(asiaCity1.equals(asiaCity2));
}
```

## 3。使用 Java `Stream` API

**我们也可以使用 Java 8 `Stream` API:** 实现我们自己的方法来比较`HashMaps`

```
private boolean areEqual(Map<String, String> first, Map<String, String> second) {
    if (first.size() != second.size()) {
        return false;
    }

    return first.entrySet().stream()
      .allMatch(e -> e.getValue().equals(second.get(e.getKey())));
}
```

为了简单起见，我们实现了现在可以用来比较`HashMap<String, String>`对象的`areEqual()`方法:

```
@Test
public void whenCompareTwoHashMapsUsingStreamAPI_thenSuccess() {
    assertTrue(areEqual(asiaCapital1, asiaCapital2));
    assertFalse(areEqual(asiaCapital1, asiaCapital3));
}
```

但是我们也可以定制我们自己的方法`areEqualWithArrayValue()`来处理数组值，方法是使用`Arrays.equals()`来比较两个数组:

```
private boolean areEqualWithArrayValue(Map<String, String[]> first, Map<String, String[]> second) {
    if (first.size() != second.size()) {
        return false;
    }

    return first.entrySet().stream()
      .allMatch(e -> Arrays.equals(e.getValue(), second.get(e.getKey())));
}
```

与`Map.equals()`不同，我们自己的方法将成功地将`HashMaps`与数组值进行比较:

```
@Test
public void whenCompareTwoHashMapsWithArrayValuesUsingStreamAPI_thenSuccess() {
    assertTrue(areEqualWithArrayValue(asiaCity1, asiaCity2)); 
    assertFalse(areEqualWithArrayValue(asiaCity1, asiaCity3));
}
```

## 4。比较`HashMap`键和值

接下来，我们来看看如何比较两个`HashMap`键及其对应的值。

### 4.1。比较`HashMap`键

首先，我们可以通过比较两个`HashMaps`的`KeySet()`来检查它们是否有相同的键:

```
@Test
public void whenCompareTwoHashMapKeys_thenSuccess() {
    assertTrue(asiaCapital1.keySet().equals(asiaCapital2.keySet())); 
    assertFalse(asiaCapital1.keySet().equals(asiaCapital3.keySet()));
}
```

### 4.2。比较`HashMap`值

接下来，我们将看到如何逐个比较`HashMap`值。

我们将使用`Stream` API 实现一个简单的方法来检查哪些键在两个`HashMaps`中具有相同的值:

```
private Map<String, Boolean> areEqualKeyValues(Map<String, String> first, Map<String, String> second) {
    return first.entrySet().stream()
      .collect(Collectors.toMap(e -> e.getKey(), 
        e -> e.getValue().equals(second.get(e.getKey()))));
}
```

我们现在可以使用`areEqualKeyValues()`来比较两个不同的`HashMaps`以详细查看哪些键具有相同的值，哪些具有不同的值:

```
@Test
public void whenCompareTwoHashMapKeyValuesUsingStreamAPI_thenSuccess() {
    Map<String, String> asiaCapital3 = new HashMap<String, String>();
    asiaCapital3.put("Japan", "Tokyo");
    asiaCapital3.put("South Korea", "Seoul");
    asiaCapital3.put("China", "Beijing");

    Map<String, String> asiaCapital4 = new HashMap<String, String>();
    asiaCapital4.put("South Korea", "Seoul");
    asiaCapital4.put("Japan", "Osaka");
    asiaCapital4.put("China", "Beijing");

    Map<String, Boolean> result = areEqualKeyValues(asiaCapital3, asiaCapital4);

    assertEquals(3, result.size());
    assertThat(result, hasEntry("Japan", false));
    assertThat(result, hasEntry("South Korea", true));
    assertThat(result, hasEntry("China", true));
}
```

## 5。使用番石榴绘制差异图

最后，我们将看到**如何使用番石榴`Maps.difference().`** 得到两个`HashMaps`的详细区别

这个方法返回一个 [`MapDifference`](https://web.archive.org/web/20221206030737/https://google.github.io/guava/releases/20.0/api/docs/com/google/common/collect/MapDifference.html) 对象，这个对象有许多有用的方法来分析`Maps.`之间的差异，让我们来看看其中的一些。

### 5.1。`MapDifference.entriesDiffering()`

首先，我们将使用`MapDifference.entriesDiffering()` 获得在每个`HashMap`中具有不同值的**公共键:**

```
@Test
public void givenDifferentMaps_whenGetDiffUsingGuava_thenSuccess() {
    Map<String, String> asia1 = new HashMap<String, String>();
    asia1.put("Japan", "Tokyo");
    asia1.put("South Korea", "Seoul");
    asia1.put("India", "New Delhi");

    Map<String, String> asia2 = new HashMap<String, String>();
    asia2.put("Japan", "Tokyo");
    asia2.put("China", "Beijing");
    asia2.put("India", "Delhi");

    MapDifference<String, String> diff = Maps.difference(asia1, asia2);
    Map<String, ValueDifference<String>> entriesDiffering = diff.entriesDiffering();

    assertFalse(diff.areEqual());
    assertEquals(1, entriesDiffering.size());
    assertThat(entriesDiffering, hasKey("India"));
    assertEquals("New Delhi", entriesDiffering.get("India").leftValue());
    assertEquals("Delhi", entriesDiffering.get("India").rightValue());
}
```

`entriesDiffering()`方法返回一个新的`Map`，它包含一组公共键和作为一组值的`ValueDifference`对象。

**每个`ValueDifference`对象都有一个`leftValue()`和`rightValue()`方法，分别返回两个`Maps`** 中的值。

### 5.2。`MapDifference.entriesOnlyOnRight()`和`MapDifference.entriesOnlyOnLeft()`

然后，我们可以使用`MapDifference.entriesOnlyOnRight()`和`MapDifference.entriesOnlyOnLeft():`获得只存在于一个`HashMap`中的条目

```
@Test
public void givenDifferentMaps_whenGetEntriesOnOneSideUsingGuava_thenSuccess() {
    MapDifference<String, String> diff = Maps.difference(asia1, asia2);
    Map<String, String> entriesOnlyOnRight = diff.entriesOnlyOnRight();
    Map<String, String> entriesOnlyOnLeft = diff.entriesOnlyOnLeft();

    assertEquals(1, entriesOnlyOnRight.size());
    assertEquals(1, entriesOnlyOnLeft.size());
    assertThat(entriesOnlyOnRight, hasEntry("China", "Beijing"));
    assertThat(entriesOnlyOnLeft, hasEntry("South Korea", "Seoul"));
}
```

### 5.3。`MapDifference.entriesInCommon()`

接下来，**我们将使用`MapDifference.entriesInCommon():`** 获得常见条目

```
@Test
public void givenDifferentMaps_whenGetCommonEntriesUsingGuava_thenSuccess() {
    MapDifference<String, String> diff = Maps.difference(asia1, asia2);
    Map<String, String> entriesInCommon = diff.entriesInCommon();

    assertEquals(1, entriesInCommon.size());
    assertThat(entriesInCommon, hasEntry("Japan", "Tokyo"));
}
```

### 5.4.定制`Maps.difference()`行为

因为默认情况下`Maps.difference()`使用`equals()`和`hashCode()`来比较条目，所以它对没有正确实现它们的对象不起作用:

```
@Test
public void givenSimilarMapsWithArrayValue_whenCompareUsingGuava_thenFail() {
    MapDifference<String, String[]> diff = Maps.difference(asiaCity1, asiaCity2);
    assertFalse(diff.areEqual());
}
```

但是，**我们可以使用`Equivalence`** 自定义比较中使用的方法。

例如，我们将为类型`String[]`定义`Equivalence `来比较`HashMaps`中的`String[]`值:

```
@Test
public void givenSimilarMapsWithArrayValue_whenCompareUsingGuavaEquivalence_thenSuccess() {
    Equivalence<String[]> eq = new Equivalence<String[]>() {
        @Override
        protected boolean doEquivalent(String[] a, String[] b) {
            return Arrays.equals(a, b);
        }

        @Override
        protected int doHash(String[] value) {
            return value.hashCode();
        }
    };

    MapDifference<String, String[]> diff = Maps.difference(asiaCity1, asiaCity2, eq);
    assertTrue(diff.areEqual());

    diff = Maps.difference(asiaCity1, asiaCity3, eq); 
    assertFalse(diff.areEqual());
}
```

## 6。结论

在本文中，我们讨论了在 Java 中比较`HashMaps`的不同方法。我们学习了多种方法来检查两个`HashMaps`是否相等，以及如何获得详细的差异。

完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221206030737/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-3)