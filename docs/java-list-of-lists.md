# 在 Java 中使用列表列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-of-lists>

## 1.概观

`[List](/web/20221208143830/https://www.baeldung.com/tag/java-list/)`是 Java 中非常常用的数据结构。有时候，对于某些需求，我们可能需要一个嵌套的`List`结构，比如`List<List<T>>`。

在本教程中，我们将仔细研究这个“列表列表”数据结构，并探索一些日常操作。

## 2.列表数组与列表列表

我们可以把“列表的列表”数据结构看作一个二维矩阵。因此，如果我们想要将多个`List<T>`对象分组，我们有两个选择:

*   基于数组:`List<T>[]`
*   基于列表:`List<List<T>>`

接下来，我们来看看什么时候选择哪一个。

**[`Array`](/web/20221208143830/https://www.baeldung.com/java-arrays-guide) 是快速进行`get`、`set`操作，在`O(1)`时间**运行。然而，**因为数组的长度是固定的，所以为了插入或删除元素而调整数组的大小是很费钱的。**

另一方面， **`List`对[插入和删除操作](/web/20221208143830/https://www.baeldung.com/java-add-element-to-array-vs-list)更加灵活，在`O(1)`时间**运行。一般来说，`List`在“获取/设置”操作上比`Array`慢。但是一些`List`的实现，比如`[ArrayList](/web/20221208143830/https://www.baeldung.com/java-arraylist)`，在内部是基于数组的。因此，通常情况下，`Array`和`ArrayList`在“获取/设置”操作上的性能差异并不明显。

因此，**我们在大多数情况下会选择`List<List<T>>`数据结构以获得更好的灵活性**。

当然，如果我们正在开发一个性能关键的应用程序，并且我们不改变第一维的大小——例如，我们从不添加或删除内部的`Lists`——我们可以考虑使用`List<T>[]`结构。

我们将在本教程中主要讨论`List<List<T>>`。

## 3.列表列表的常见操作

现在，我们来探讨一下`List<List<T>>`上的一些日常操作。

为了简单起见，我们将操作`List<List<String>>`对象并在单元测试方法中验证结果。

此外，为了直观地看到变化，让我们也创建一个方法来打印`List` s 的`List`的内容:

```java
private void printListOfLists(List<List<String>> listOfLists) {
    System.out.println("\n           List of Lists          ");
    System.out.println("-------------------------------------");
    listOfLists.forEach(innerList -> {
        String line = String.join(", ", innerList);
        System.out.println(line);
    });
} 
```

接下来，我们先初始化一个列表的列表。

### 3.1.初始化列表列表

我们将把数据从 CSV 文件导入到一个`List<List<T>>`对象中。让我们先来看看 CSV 文件的内容:

```java
Linux, Microsoft Windows, Mac OS, Delete Me
Kotlin, Delete Me, Java, Python
Delete Me, Mercurial, Git, Subversion
```

假设我们将文件命名为`example.csv`，并将其放在`resources/listoflists`目录下。

接下来，让我们创建一个方法来让[读取文件](/web/20221208143830/https://www.baeldung.com/reading-file-in-java#1-reading-a-small-file)并将数据存储在一个`List<List<T>>`对象中:

```java
private List<List<String>> getListOfListsFromCsv() throws URISyntaxException, IOException {
    List<String> lines = Files.readAllLines(Paths.get(getClass().getResource("/listoflists/example.csv")
        .toURI()));

    List<List<String>> listOfLists = new ArrayList<>();
    lines.forEach(line -> {
        List<String> innerList = new ArrayList<>(Arrays.asList(line.split(", ")));
        listOfLists.add(innerList);
    });
    return listOfLists;
} 
```

在`getListOfListsFromCsv`方法中，我们首先将 CSV 文件中的所有行读入一个`List<String>`对象。然后，我们遍历`lines List`，将每一行(`String`)转换成`List<String>`。

最后，我们将每个转换后的`List<String>`对象添加到`listOfLists`。因此，我们已经初始化了一个列表列表。

好奇的眼睛可能已经察觉到我们用新的`ArrayList<>()`包装`Arrays.asList(..)`。这是因为 **[`Arrays.asList`](/web/20221208143830/https://www.baeldung.com/java-arrays-aslist-vs-new-arraylist) 方法会创建一个不可变的** `**List**`。但是，我们稍后会对内部列表进行一些更改。因此，我们将它包装在一个新的`ArrayList`对象中。

如果列表对象的列表创建正确，我们应该在外部列表中有三个元素，即 CSV 文件中的行数。

此外，每个元素都是一个内部列表，每个列表都应该包含四个元素。接下来，让我们编写一个单元测试方法来验证这一点。此外，我们将打印列表的初始化列表:

```java
List<List<String>> listOfLists = getListOfListsFromCsv();

assertThat(listOfLists).hasSize(3);
assertThat(listOfLists.stream()
  .map(List::size)
  .collect(Collectors.toSet())).hasSize(1)
  .containsExactly(4);

printListOfLists(listOfLists); 
```

如果我们执行该方法，测试通过并产生输出:

```java
 List of Lists           
-------------------------------------
Linux, Microsoft Windows, Mac OS, Delete Me
Kotlin, Delete Me, Java, Python
Delete Me, Mercurial, Git, Subversion
```

接下来，让我们对`listOfLists`对象做一些修改。但是，首先，让我们看看如何将更改应用到外部列表。

### 3.2.将更改应用到外部列表

如果我们专注于外部列表，我们可以忽略内部列表。换句话说，**我们可以把`List<List<String>>`看成常规的`List<T>.`**

因此，改变一个常规的`List`对象并不困难。我们可以调用`List`的方法，比如`add`和`remove`，来操作数据。

接下来，让我们向外部列表添加一个新元素:

```java
List<List<String>> listOfLists = getListOfListsFromCsv();
List<String> newList = new ArrayList<>(Arrays.asList("Slack", "Zoom", "Microsoft Teams", "Telegram"));
listOfLists.add(2, newList);

assertThat(listOfLists).hasSize(4);
assertThat(listOfLists.get(2)).containsExactly("Slack", "Zoom", "Microsoft Teams", "Telegram");

printListOfLists(listOfLists); 
```

外部列表的一个元素实际上是一个`List<String>`对象。正如上面的方法所示，我们创建了一个流行的在线通信工具列表。然后，我们将新列表添加到`listOfLists`中带有`index=2`的位置。

同样，在断言之后，我们打印出`listOfLists`的内容:

```java
 List of Lists           
-------------------------------------
Linux, Microsoft Windows, Mac OS, Delete Me
Kotlin, Delete Me, Java, Python
Slack, Zoom, Microsoft Teams, Telegram
Delete Me, Mercurial, Git, Subversion 
```

### 3.3.将更改应用到内部列表

最后，让我们看看如何操作内部列表。

由于`listOfList`是一个嵌套的`List`结构，我们需要首先导航到我们想要改变的内部列表对象。如果我们确切地知道指数，我们可以简单地使用`get`方法:

```java
List<String> innerList = listOfLists.get(x);
// innerList.add(), remove() ....
```

然而，如果我们想对所有内部列表进行修改，我们可以通过一个循环或[流 API](/web/20221208143830/https://www.baeldung.com/java-8-streams) 来传递列表对象列表。

接下来，让我们看一个从`listOfLists`对象中删除所有“`Delete Me`”条目的例子:

```java
List<List<String>> listOfLists = getListOfListsFromCsv();

listOfLists.forEach(innerList -> innerList.remove("Delete Me"));

assertThat(listOfLists.stream()
    .map(List::size)
    .collect(Collectors.toSet())).hasSize(1)
    .containsExactly(3);

printListOfLists(listOfLists); 
```

正如我们在上面的方法中看到的，我们通过`listOfLists.forEach{ … }`迭代每个内部列表，并使用 lambda 表达式从`innerList`中删除`Delete Me`条目。

如果我们执行测试，它会通过并产生以下输出:

```java
 List of Lists           
-------------------------------------
Linux, Microsoft Windows, Mac OS
Kotlin, Java, Python
Mercurial, Git, Subversion 
```

## 4.结论

在本文中，我们讨论了列表的列表数据结构。

此外，我们已经通过示例解决了列表列表的常见操作。

像往常一样，这篇文章的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-4)