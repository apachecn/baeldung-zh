# 找出两组之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-difference-between-sets>

## 1.概观

[`Set`](/web/20220926153220/https://www.baeldung.com/java-hashset) 是 Java 中常用的集合类型之一。今天，我们将讨论如何找出两个给定集合之间的差异。

## 2.问题简介

在我们仔细研究实现之前，我们需要首先理解问题。通常，一个例子可以帮助我们快速理解需求。

假设我们有两个`Set`对象，`set1`和`set2`:

```java
set1: {"Kotlin", "Java", "Rust", "Python", "C++"}
set2: {"Kotlin", "Java", "Rust", "Ruby", "C#"}
```

正如我们所看到的，这两个集合都包含一些编程语言名称。需求“`Finding the difference between two Sets`”可能有两种变体:

*   不对称差异——找出那些被`set1`包含但不被`set2`包含的元素；在这种情况下，预期的结果是`{“Python”, “C++”}`
*   [对称差](https://web.archive.org/web/20220926153220/https://en.wikipedia.org/wiki/Symmetric_difference)–在任一集合中寻找元素，但不在它们的交集中；如果我们看我们的例子，结果应该是`{“Python”, “C++”, “Ruby”, “C#”}`

在本教程中，我们将解决这两种情况。首先，我们将重点寻找不对称差异。之后，我们将探索寻找两个集合之间的对称差异。

接下来，让我们看看他们的行动。

## 3.不对称差异

### 3.1.使用标准`removeAll`方法

`Set`类提供了一个 [`removeAll`](https://web.archive.org/web/20220926153220/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html#removeAll(java.util.Collection)) 方法。这个方法从`Collection`接口实现了`removeAll`方法。

**`removeAll`方法接受一个`Collection`对象作为参数，并从给定的`Set`对象中移除参数中的所有元素。**所以，如果我们以这种方式传递`set2`对象作为参数，`set1.removeAll(set2)`，`set1`对象中的其余元素将是结果。

为了简单起见，我们把它作为一个单元测试来展示:

```java
Set<String> set1 = Stream.of("Kotlin", "Java", "Rust", "Python", "C++").collect(Collectors.toSet());
Set<String> set2 = Stream.of("Kotlin", "Java", "Rust", "Ruby", "C#").collect(Collectors.toSet());
Set<String> expectedOnlyInSet1 = Set.of("Python", "C++");

set1.removeAll(set2);

assertThat(set1).isEqualTo(expectedOnlyInSet1);
```

如上面的方法所示，首先，我们[使用`Stream`初始化两个`Set`](/web/20220926153220/https://www.baeldung.com/java-initialize-hashset) 对象。然后，在调用了`removeAll`方法之后，`set` 1 对象包含了预期的元素。

这种方法非常简单。然而，缺点也是显而易见的:去掉`set1`、**中的共同元素后，原来的`set1`被修改为**。

因此，**如果我们在调用`removeAll` 方法后仍然需要它，我们需要备份原来的`set1`对象，或者如果`set1`是一个[不可变的`Set`](/web/20220926153220/https://www.baeldung.com/java-immutable-set)** `.`，我们必须创建一个新的可变集合对象

接下来，让我们看看另一种在新的`Set`对象中返回不对称差异而不修改原始集合的方法。

### 3.2.使用`Stream.filter`方法

流 API 从 Java 8 开始就已经存在了。它允许我们使用 [`Stream.filter`](/web/20220926153220/https://www.baeldung.com/java-stream-filter-lambda) 方法从集合中过滤元素。

我们也可以使用`Stream.filter`来解决这个问题，而不需要修改原来的`set1`对象。让我们首先将这两个集合初始化为不可变集合:

```java
Set<String> immutableSet1 = Set.of("Kotlin", "Java", "Rust", "Python", "C++");
Set<String> immutableSet2 = Set.of("Kotlin", "Java", "Rust", "Ruby", "C#");
Set<String> expectedOnlyInSet1 = Set.of("Python", "C++");
```

从 Java 9 开始，`Set`接口引入了静态的 [`of`](https://web.archive.org/web/20220926153220/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html#of(E...)) 方法。它允许我们方便地初始化一个不可变的`Set`对象。也就是说，如果我们试图修改`immutableSet1,`，就会抛出一个`UnsupportedOperationException`。

接下来，让我们编写一个单元测试，使用`Stream.filter`来找出不同之处:

```java
Set<String> actualOnlyInSet1 = immutableSet1.stream().filter(e -> !immutableSet2.contains(e)).collect(Collectors.toSet());
assertThat(actualOnlyInSet1).isEqualTo(expectedOnlyInSet1); 
```

在上面的方法中我们可以看到，关键是“`filter(e -> !immutableSet2.contains(e))`”。这里，我们只取在`immutableSet1`中而不在 `immutableSet2`中的元素。

如果我们执行这个测试方法，它将毫无例外地通过。这意味着这种方法是可行的，并且原始集合没有被修改。

### 3.3.使用番石榴图书馆

Guava 是一个流行的 Java 库，附带了一些新的集合类型和方便的助手方法。番石榴提供了一种方法来寻找两组之间的不对称差异。因此，我们可以使用这种方法来轻松解决我们的问题。

但是首先，我们需要在我们的类路径中包含这个库。假设我们通过[专家](/web/20220926153220/https://www.baeldung.com/maven)来管理项目依赖。我们可能需要将[番石榴属地](https://web.archive.org/web/20220926153220/https://search.maven.org/search?q=g:com.google.guava%20AND%20a:guava)添加到`pom.xml`中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency> 
```

一旦我们的 Java 项目中有了番石榴，**我们就可以使用它的 [`Sets.difference`](https://web.archive.org/web/20220926153220/https://guava.dev/releases/31.0-jre/api/docs/com/google/common/collect/Sets.html#difference(java.util.Set,java.util.Set)) 方法来得到预期的结果**:

```java
Set<String> actualOnlyInSet1 = Sets.difference(immutableSet1, immutableSet2);
assertThat(actualOnlyInSet1).isEqualTo(expectedOnlyInSet1); 
```

**值得一提的是，`Sets.difference`方法返回包含结果的不可变的`Set`视图。**它的意思是:

*   我们不能修改返回的集合
*   如果原始集合是可变的，对原始集合的更改可能会反映在我们得到的集合视图中

### 3.4.使用 Apache Commons 库

Apache Commons 是另一个广泛使用的库。Apache Commons Collections4 库提供了许多与集合相关的好方法，作为标准集合 API 的补充。

在我们开始使用它之前，让我们将依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency> 
```

类似地，我们可以在 Maven 的中央存储库中找到最新版本的。

`commons-collections4`库有一个`CollectionUtils.removeAll`方法。这类似于标准的`Collection.removeAll`方法，但是**在新的`Collection `对象中返回结果，而不是修改第一个`Collection`对象**。

接下来，让我们用两个不可变的`Set`对象来测试它:

```java
Set<String> actualOnlyInSet1 = new HashSet<>(CollectionUtils.removeAll(immutableSet1, immutableSet2));
assertThat(actualOnlyInSet1).isEqualTo(expectedOnlyInSet1); 
```

如果我们执行它，测试就会通过。但是，我们应该注意到，**`CollectionUtils.removeAll`方法返回的是`Collection`类型**的结果。

如果需要一个具体的类型——例如，在我们的例子中是`Set` ——我们将需要手动转换它。在上面的测试方法中，我们已经使用返回的集合初始化了一个新的`HashSet`对象。

## 4.对称差

到目前为止，我们已经学会了如何得到两个集合之间的不对称差。现在，让我们仔细看看另一个场景:寻找两个集合之间的对称差。

我们将从两个不可变集合示例中提出两种方法来获得对称差异。

预期的结果是:

```java
Set<String> expectedDiff = Set.of("Python", "C++", "Ruby", "C#");
```

接下来，我们来看看如何解决问题。

### 4.1.使用`HashMap`

解决这个问题的一个想法是首先创建一个`Map<T, Integer>`对象。

然后，我们遍历两个给定的集合，并将每个元素作为键放入映射中。**如果键存在于映射中，这意味着这是两个集合中的公共元素。我们设置一个特殊的数字作为值——例如，`Integer.MAX_VALUE`** 。否则，我们将元素和值 1 作为新的条目放入 map 中。

最后，我们在映射中找出值为 1 的键，这些键是两个给定集合之间的对称差。

接下来，让我们用 Java 实现这个想法:

```java
public static <T> Set<T> findSymmetricDiff(Set<T> set1, Set<T> set2) {
    Map<T, Integer> map = new HashMap<>();
    set1.forEach(e -> putKey(map, e));
    set2.forEach(e -> putKey(map, e));
    return map.entrySet().stream()
      .filter(e -> e.getValue() == 1)
      .map(Map.Entry::getKey)
      .collect(Collectors.toSet());
}

private static <T> void putKey(Map<T, Integer> map, T key) {
    if (map.containsKey(key)) {
        map.replace(key, Integer.MAX_VALUE);
    } else {
        map.put(key, 1);
    }
} 
```

现在，让我们测试我们的解决方案，看看它是否能给出预期的结果:

```java
Set<String> actualDiff = SetDiff.findSymmetricDiff(immutableSet1, immutableSet2);
assertThat(actualDiff).isEqualTo(expectedDiff); 
```

如果我们运行它，测试就通过了。也就是说，我们的实现按预期工作。

### 4.2.使用 Apache Commons 库

当发现两个集合之间的不对称差异时，我们已经介绍了 Apache Commons 库。实际上，**`commons-collections4`库有一个简便的`SetUtils.disjunction`方法可以直接返回两个集合之间的对称差**:

```java
Set<String> actualDiff = SetUtils.disjunction(immutableSet1, immutableSet2);
assertThat(actualDiff).isEqualTo(expectedDiff); 
```

正如上面的方法所示，与`CollectionUtils.removeAll`方法不同，`SetUtils.disjunction`方法返回一个`Set`对象。我们不需要手动转换成`Set`。

## 5.结论

在本文中，我们已经通过例子探索了如何找到两个`Set`对象之间的差异。此外，我们已经讨论了这个问题的两种变体:寻找不对称差异和对称差异。

我们已经使用标准的 Java API 和广泛使用的外部库(如 Apache Commons-Collections 和 Guava)解决了这两种变体。

和往常一样，本教程中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220926153220/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set)