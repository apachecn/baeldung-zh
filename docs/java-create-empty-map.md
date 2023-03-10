# 用 Java 创建一个空地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-create-empty-map>

## 1.概观

在本文中，我们将探索在 Java 中初始化空`Map`的不同可能方式。

我们将使用 Java 8 和 Java 9 来检查不同的方法。

## 2.使用 Java 集合

我们可以使用 Java Collections 模块提供的`emptyMap()`方法创建一个空的`Map`。**这将形成一个空的`Map`，本质上是可序列化的。该方法是在 Java 1.5 的集合库下引入的。**这将创建一个不可变的`Map`:

```java
Map<String, String> emptyMap = Collections.emptyMap();
```

注意:由于创建的`Map`本质上是不可变的，它不允许用户添加任何条目或者对`Map`进行任何类型的修改。**这将在试图添加或修改`Map`中的任何键-值对时抛出一个`java.lang.UnsupportedOperationException`。**

我们还有两个方法支持空的`Map`的创建和初始化。**`emptySortedMap()`返回一个不可变类型的空`SortedMap`。**`Sorted``Map`是在其键上提供进一步总排序的键。由该方法创建的`Map`本质上是可序列化的:

```java
SortedMap<String, String> sortedMap = Collections.emptySortedMap();
```

**Java 集合提供的另一个方法是`emptyNavigableMap()`，它返回一个空的`NavigableMap`。**与空的已排序的`Map`具有相同的属性。唯一的区别是这个方法返回一个可导航的`Map`。一个`Navigable` `Map`是传统排序`Map`实现的扩展，它返回给定搜索目标的最接近匹配。

```java
NavigableMap<String, String> navigableMap = Collections.emptyNavigableMap();
```

以上所有方法都返回本质上不可变的`Maps`，我们不能向这些`Maps`添加任何新的条目。这使得`UnsupportedOperationException` 不得不试图添加、删除或修改任何键-值对。

## 3.使用构造函数初始化映射

我们可以使用不同的`Map`实现的构造函数来初始化`Maps`，例如`HashMap, LinkedHashMap, TreeMap`。所有这些初始化创建了一个空的`Map`,如果需要，我们可以在后面添加条目:

```java
Map hashMap = new HashMap();
Map linkedHashMap = new LinkedHashMap();
Map treeMap = new TreeMap();
```

**上面的`Maps`是可变的，可以接受新的条目，这是使用这种方法的优点之一。**此类初始化过程中创建的`Maps`为空。我们可以在代码的`static`块中定义[空`Maps`。](https://web.archive.org/web/20220817021739/https://drafts.baeldung.com/java-initialize-hashmap#the-static-initializer-for-a-static-hashmap)

## 4.Java 9 的方式与`Map.of()`

Java 9 附带了许多新特性，比如`Interface Private Methods, Anonymous classes, Platform Module System,`等等。**`Map.of()`是 Java 9 版本中引入的工厂方法。**这个方法返回一个不可变的`Map`，它创建零个映射。该方法提供的接口属于 [Java 集合框架](/web/20220817021739/https://www.baeldung.com/java-collections)。`Map.of(key1, value1, key2, value2, …..)`最多只能有 10 个键值对。

为了初始化一个空的`Map`，我们不会在这个方法中传递任何键值对:

```java
Map<String, String> emptyMapUsingJava9 = Map.of();
```

这个工厂方法产生一个不可变的`Map`，因此我们不能添加、删除或修改任何键值对。初始化后，试图在`Map`中进行任何突变时会抛出一个`UnsupportedOperationException `。的。也不支持添加或删除键值对，这将导致抛出上述异常。

注意:Java 9 [中的`Map.of()`方法简化了](/web/20220817021739/https://www.baeldung.com/java-initialize-hashmap#the-java-9-way)不可变`Maps`的初始化，使其具有所需的键值对。

## 5.用番石榴

到目前为止，我们已经研究了使用核心 Java 初始化空`Map`的不同方法。现在让我们继续，检查如何使用 Guava 库初始化一个`Map`:

```java
Map<String, String> articles = ImmutableMap.of();
```

**上面的方法将使用 Guava 库创建一个不可变的空的`Map`。**

在某些情况下，我们不需要不可变的`Map`。我们可以使用`Maps `类初始化一个可变的`Map`:

```java
Map<String, String> emptyMap = Maps.newHashMap();
```

**这种类型的初始化创建了一个可变的`Map`，即我们可以向这个`Map`添加条目。**但是这个`Map`的基本初始化是空的，不包含任何条目。

我们还可以用特定的键和值类型初始化`Map`。这将创建一个具有预定义元素类型的`Map`,如果没有遵循，将抛出一个异常:

```java
Map genericEmptyMap = Maps.<String, Integer>newHashMap();
```

简而言之，这会创建一个空的`Map`, key 为字符串，value 为整数。**用于初始化的一对尖括号称为`Diamond Syntax`。**这将创建一个带有已定义类型参数的`Map`,调用`Maps`类的构造函数。

我们也可以使用下面的语法在 guava 中创建一个可变的`Map`:

```java
Map<String, String> emptyMapUsingGuava = Maps.newHashMap(ImmutableMap.of());
```

总之，上面的方法在 Java 中创建了一个空的`Map`。我们可以向这个`Map`添加条目，因为它本质上是可变的。

`ImmutableMap.of()`还重载了创建带有条目的`Maps`的方法版本。因为我们正在创建一个空的`Map`，所以我们不需要在方法括号内传递任何参数来使用重载方法。

## 7.结论

在本文中，我们探索了初始化`Empty` `Map`的不同方式。我们可以看到，自 Java 9 以来，这个领域已经有了巨大的进步。我们有新的工厂方法来创建和[初始化`Maps`](/web/20220817021739/https://www.baeldung.com/java-initialize-hashmap) 。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220817021739/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)