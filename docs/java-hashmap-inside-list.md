# 如何在列表中存储 HashMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap-inside-list>

## 1.概观

在本教程中，我们将讨论如何在 Java 中把一个 [`HashMap`](/web/20221208143817/https://www.baeldung.com/java-hashmap) 存储在一个 [`List`](/web/20221208143817/https://www.baeldung.com/java-collections) 中。首先，我们将简单解释一下 Java 中的`HashMap`和`List`数据结构。然后，我们将编写一个简单的代码来解决这个问题。

## 2.Java 中的`HashMap`和`List`

Java 为我们提供了具有各种属性和特征的不同数据结构来存储对象。其中， **`HashMap`是一个键-值对的集合，它将一个惟一的键映射到一个值。**同样，**一个** **[`List`](/web/20221208143817/https://www.baeldung.com/java-arraylist) 持有一系列相同类型的对象。**

我们可以在这些数据结构中放置简单的值或复杂的对象。

## 3.将`HashMap<String, ArrayList<String>>`存储在`List`中

让我们举一个简单的例子，我们为每个图书类别创建一个`HashMaps.` **的`List`，有一个`HashMap`将图书的名称映射到其作者。**

首先，我们定义`javaB` `ookAuthorsMap,`，它将 Java 相关书籍的名称映射到其作者列表:

```java
HashMap<String, List<String>> javaBooksAuthorsMap = new HashMap<>(); 
```

此外，我们定义`phpBooksAuthorsMap`来保存 PHP 类别书籍的名称和作者:

```java
HashMap<String, List<String>> phpBooksAuthorsMap = new HashMap<>();
```

然后，我们定义`booksAuthorsMapsList`来保存不同类别的`HashMap`:

```java
List<HashMap<String, List<String>>> booksAuthorsMapsList = new ArrayList<>();
```

现在，我们有一个包含两个`HashMaps`的`List`。

为了测试它，我们可以将一些图书信息放在 *`javaB` `ookAuthorsMap `* 和`phpBooksAuthorsMap` 列表`.`中，然后将它们添加到`booksAuthorsMapsList.`中，最后，我们确保将`HashMaps`添加到`List.`中

让我们看看下面的单元测试:

```java
@Test
public void givenMaps_whenAddToList_thenListContainsMaps() {
    HashMap<String, List<String>> javaBooksAuthorsMap = new HashMap<>();
    HashMap<String, List<String>> phpBooksAuthorsMap = new HashMap<>();
    javaBooksAuthorsMap.put("Head First Java", Arrays.asList("Kathy Sierra", "Bert Bates"));
    javaBooksAuthorsMap.put("Effective Java", Arrays.asList("Joshua Bloch"));
    javaBooksAuthorsMap.put("OCA Java SE 8",
      Arrays.asList("Kathy Sierra", "Bert Bates", "Elisabeth Robson"));
    phpBooksAuthorsMap.put("The Joy of PHP", Arrays.asList("Alan Forbes"));
    phpBooksAuthorsMap.put("Head First PHP & MySQL",
    Arrays.asList("Lynn Beighley", "Michael Morrison"));

    booksAuthorsMapsList.add(javaBooksAuthorsMap);
    booksAuthorsMapsList.add(phpBooksAuthorsMap);

    assertTrue(booksAuthorsMapsList.get(0).keySet()
      .containsAll(javaBooksAuthorsMap.keySet().stream().collect(Collectors.toList())));
    assertTrue(booksAuthorsMapsList.get(1).keySet()
      .containsAll(phpBooksAuthorsMap.keySet().stream().collect(Collectors.toList())));
}
```

## 4.结论

在本文中，我们讨论了用 Java 在列表中存储 HashMaps。然后，我们编写了一个简单的例子，在这个例子中，我们为两个图书类别的一个`List<String>`添加了`HashMap<String, ArrayList<String>>`。

GitHub 上的[提供了示例。](https://web.archive.org/web/20221208143817/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-4)