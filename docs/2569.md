# 在构造时初始化 HashSet

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-initialize-hashset>

## 1.概观

在这个快速教程中，我们将介绍在构造`HashSet`时用值初始化**的各种方法。**

相反，要探索`HashSet`的特性，请参考[这里的](/web/20220926153229/https://www.baeldung.com/java-hashset)这篇核心文章。

我们将深入探讨从 Java 5 开始到之前的 Java **内置方法** **，以及从 Java 8 开始引入的新的**机制。****

我们还将看到一个**自定义实用程序方法**，最后探索由**第三方收藏库**提供的特性，特别是 Google Guava。

如果我们已经迁移到 JDK9+，我们可以简单地使用[集合工厂方法。](/web/20220926153229/https://www.baeldung.com/java-9-collections-factory-methods)

## 2.Java 内置方法

让我们从检查自 Java 5 或更早版本以来可用的三种内置**机制开始。**

### 2.1.使用另一个集合实例

我们可以传递另一个集合的现有**实例来初始化`Set`。**

这里我们使用了内联创建的`List`:

```
Set<String> set = new HashSet<>(Arrays.asList("a", "b", "c"));
```

### 2.2.使用匿名类

在另一种方法中，我们可以使用匿名类向`HashSet`添加一个元素。

注意双花括号的使用。这种方法在技术上非常昂贵，因为它在每次被调用时都会创建一个匿名类。

因此，根据我们需要初始化`Set`的频率，我们可以**尽量避免使用这种方法**:

```
Set<String> set = new HashSet<String>(){{
    add("a");
    add("b");
    add("c");
}};
```

### 2.3.从 Java 5 开始使用集合实用程序方法

Java 的 **`Collections`实用程序**类提供了名为`singleton`的方法来创建具有单个元素`.`的`Set`。用`singleton`方法创建的`Set`实例是**不可变的**，这意味着我们不能向它添加更多的值。

有些情况下，特别是在单元测试中，我们需要创建一个只有一个值的`Set`:

```
Set<String> set = Collections.singleton("a");
```

## 3.定义自定义实用程序方法

我们可以如下定义一个`static final`方法。方法**接受变量参数。**

使用接受集合对象和一组值的`Collections.addAll`，是**中最好的**，因为复制值的开销很低。

**方法使用泛型**,所以我们可以传递任何类型的值:

```
public static final <T> Set<T> newHashSet(T... objs) {
    Set<T> set = new HashSet<T>();
    Collections.addAll(set, objs);
    return set;
}
```

下面是我们如何在代码中使用 utility 方法:

```
Set<String> set = newHashSet("a","b","c");
```

## 4.从 Java 8 开始使用`Stream`

随着 Java 8 中`Stream` API 的引入，我们有了额外的选项，如 **`Stream`与`Collectors`** :

```
Set<String> set = Stream.of("a", "b", "c")
  .collect(Collectors.toCollection(HashSet::new));
```

## 5.使用第三方收藏库

有多个第三方收藏库，包括 Google Guava、Apache Commons Collections 和 Eclipse Collections 等等。

这些库提供了方便的实用方法来初始化集合，如 Set。由于 [**谷歌番石榴**](https://web.archive.org/web/20220926153229/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22) 是最常用的一种，我们已经收录了其中的一个例子。

Guava 为可变和不可变的`Set`对象提供了方便的方法:

```
Set<String> set = Sets.newHashSet("a", "b", "c");
```

类似地，Guava 有一个用于创建**不可变`Set`实例**的实用程序类:

```
Set<String> set = ImmutableSet.of("a", "b", "c");
```

## 6.结论

在本文中，我们看到了在构造时初始化`HashSet`的多种方式。

这些方法不一定涵盖所有可能的方式。本文只是试图展示最常见的方法。

例如，这里没有介绍的一种方法是使用对象构建器来构建一个`Set`。

与往常一样，GitHub 上的[提供了工作代码示例。](https://web.archive.org/web/20220926153229/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set)