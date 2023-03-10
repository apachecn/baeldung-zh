# 在 Java 中连接和拆分数组和集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-join-and-split>

## 1。概述

在这个快速教程中，我们将学习如何在 Java 中连接和拆分`Arrays`和`Collections`，充分利用新的流支持`.`

## 2。加入两个`Arrays`

让我们从使用`Stream.concat` `:`将两个`Arrays` 连接在一起开始

```java
@Test
public void whenJoiningTwoArrays_thenJoined() {
    String[] animals1 = new String[] { "Dog", "Cat" };
    String[] animals2 = new String[] { "Bird", "Cow" };

    String[] result = Stream.concat(
      Arrays.stream(animals1), Arrays.stream(animals2)).toArray(String[]::new);

    assertArrayEquals(result, new String[] { "Dog", "Cat", "Bird", "Cow" });
}
```

## 3。加入两个`Collections`

让我们用两个`Collections:`做同样的连接

```java
@Test
public void whenJoiningTwoCollections_thenJoined() {
    Collection<String> collection1 = Arrays.asList("Dog", "Cat");
    Collection<String> collection2 = Arrays.asList("Bird", "Cow", "Moose");

    Collection<String> result = Stream.concat(
      collection1.stream(), collection2.stream())
      .collect(Collectors.toList());

    assertTrue(result.equals(Arrays.asList("Dog", "Cat", "Bird", "Cow", "Moose")));
}
```

## 4。用过滤器连接两个`Collections W`

现在，让我们将两个`Collections` 数字连接起来，过滤任何大于 10 的数字:

```java
@Test
public void whenJoiningTwoCollectionsWithFilter_thenJoined() {
    Collection<String> collection1 = Arrays.asList("Dog", "Cat");
    Collection<String> collection2 = Arrays.asList("Bird", "Cow", "Moose");

    Collection<String> result = Stream.concat(
      collection1.stream(), collection2.stream())
      .filter(e -> e.length() == 3)
      .collect(Collectors.toList());

    assertTrue(result.equals(Arrays.asList("Dog", "Cat", "Cow")));
}
```

## 5。将一个`Array`连接成一个`String`

接下来，让我们使用一个`Collector:`将一个数组加入到一个`String` 中

```java
@Test
public void whenConvertArrayToString_thenConverted() {
    String[] animals = new String[] { "Dog", "Cat", "Bird", "Cow" };
    String result = Arrays.stream(animals).collect(Collectors.joining(", "));

    assertEquals(result, "Dog, Cat, Bird, Cow");
}
```

## 6。将一个`Collection`连接成一个`String`

让我们做同样的事情，但是用一个`Collection` `:`

```java
@Test
public void whenConvertCollectionToString_thenConverted() {
    Collection<String> animals = Arrays.asList("Dog", "Cat", "Bird", "Cow");
    String result = animals.stream().collect(Collectors.joining(", "));

    assertEquals(result, "Dog, Cat, Bird, Cow");
}
```

## 7。将一个`Map` 连接成一个`String`

接下来，让我们从一个`Map`创建一个`String`。

这个过程与前面的例子非常相似，但是这里我们有一个额外的步骤来首先连接每个`Map` `Entry`:

```java
@Test
public void whenConvertMapToString_thenConverted() {
    Map<Integer, String> animals = new HashMap<>();
    animals.put(1, "Dog");
    animals.put(2, "Cat");
    animals.put(3, "Cow");

    String result = animals.entrySet().stream()
      .map(entry -> entry.getKey() + " = " + entry.getValue())
      .collect(Collectors.joining(", "));

    assertEquals(result, "1 = Dog, 2 = Cat, 3 = Cow");
}
```

## 8。将嵌套的`Collections`连接成一个`String`

让我们做一些更复杂的事情。让我们把一些嵌套的`Collections`连接成一个`String`。

在下面的示例中，我们首先在每个嵌套集合内连接，然后连接每个集合的结果:

```java
@Test
public void whenConvertNestedCollectionToString_thenConverted() {
    Collection<List<String>> nested = new ArrayList<>();
    nested.add(Arrays.asList("Dog", "Cat"));
    nested.add(Arrays.asList("Cow", "Pig"));

    String result = nested.stream().map(
      nextList -> nextList.stream()
        .collect(Collectors.joining("-")))
      .collect(Collectors.joining("; "));

    assertEquals(result, "Dog-Cat; Cow-Pig");
}
```

## 9。加入时处理`Null`值

让我们看看如何使用一个`Filter` 来跳过任何`null`值:

```java
@Test
public void whenConvertCollectionToStringAndSkipNull_thenConverted() {
    Collection<String> animals = Arrays.asList("Dog", "Cat", null, "Moose");
    String result = animals.stream()
      .filter(Objects::nonNull)
      .collect(Collectors.joining(", "));

    assertEquals(result, "Dog, Cat, Moose");
}
```

## 10。将一个`Collection` 一分为二

让我们把一个`Collection`的数字分成中间的两个`Collections` :

```java
@Test
public void whenSplitCollectionHalf_thenConverted() {
    Collection<String> animals = Arrays.asList(
        "Dog", "Cat", "Cow", "Bird", "Moose", "Pig");
    Collection<String> result1 = new ArrayList<>();
    Collection<String> result2 = new ArrayList<>();
    AtomicInteger count = new AtomicInteger();
    int midpoint = Math.round(animals.size() / 2);

    animals.forEach(next -> {
        int index = count.getAndIncrement();
        if (index < midpoint) {
            result1.add(next);
        } else {
            result2.add(next);
        }
    });

    assertTrue(result1.equals(Arrays.asList("Dog", "Cat", "Cow")));
    assertTrue(result2.equals(Arrays.asList("Bird", "Moose", "Pig")));
}
```

## 11。按字长分割一个`Array`

接下来，让我们按照单词的长度来分割数组:

```java
@Test
public void whenSplitArrayByWordLength_thenConverted() {
    String[] animals = new String[] { "Dog", "Cat", "Bird", "Cow", "Pig", "Moose"};
    Map<Integer, List<String>> result = Arrays.stream(animals)
      .collect(Collectors.groupingBy(String::length));

    assertTrue(result.get(3).equals(Arrays.asList("Dog", "Cat", "Cow", "Pig")));
    assertTrue(result.get(4).equals(Arrays.asList("Bird")));
    assertTrue(result.get(5).equals(Arrays.asList("Moose")));
}
```

## 12。将一个`String` 拆分成一个`Array`和一个

让我们现在做相反的事情，让我们把一个`String` 分裂成一个`Array:`

```java
@Test
public void whenConvertStringToArray_thenConverted() {
    String animals = "Dog, Cat, Bird, Cow";
    String[] result = animals.split(", ");

    assertArrayEquals(result, new String[] { "Dog", "Cat", "Bird", "Cow" });
}
```

## 13。将`String` 拆分成`Collection`和

这个例子与上一个类似，只是多了一个从`Array` 转换为`Collection`的步骤:

```java
@Test
public void whenConvertStringToCollection_thenConverted() {
    String animals = "Dog, Cat, Bird, Cow";
    Collection<String> result = Arrays.asList(animals.split(", "));

    assertTrue(result.equals(Arrays.asList("Dog", "Cat", "Bird", "Cow")));
}
```

## 14。将一个`String`拆分成一个`Map`和一个

现在，让我们从一个`String`创建一个`Map` 。我们需要将字符串拆分两次，一次用于每个条目，最后一次用于键和值:

```java
@Test
public void whenConvertStringToMap_thenConverted() {
    String animals = "1 = Dog, 2 = Cat, 3 = Bird";

    Map<Integer, String> result = Arrays.stream(
      animals.split(", ")).map(next -> next.split(" = "))
      .collect(Collectors.toMap(entry -> Integer.parseInt(entry[0]), entry -> entry[1]));

    assertEquals(result.get(1), "Dog");
    assertEquals(result.get(2), "Cat");
    assertEquals(result.get(3), "Bird");
}
```

## 15。用多个分隔符分割`String W`

最后，让我们使用正则表达式拆分一个有多个分隔符的`String` ,我们还将删除任何空结果:

```java
@Test
public void whenConvertCollectionToStringMultipleSeparators_thenConverted() {
    String animals = "Dog. , Cat, Bird. Cow";

    Collection<String> result = Arrays.stream(animals.split("[,|.]"))
      .map(String::trim)
      .filter(next -> !next.isEmpty())
      .collect(Collectors.toList());

    assertTrue(result.equals(Arrays.asList("Dog", "Cat", "Bird", "Cow")));
}
```

## 16。结论

在本教程中，利用简单的`String.split`函数和强大的 Java 8 `Stream,` ，我们演示了如何连接和拆分`Arrays`和`Collections.`

你可以在 GitHub 上找到这篇文章[的代码。](https://web.archive.org/web/20220626205009/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2)