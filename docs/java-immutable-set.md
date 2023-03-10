# Java 中的不可变集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-immutable-set>

## 1。简介

在本教程中，我们将看看在 Java 中构造不可变集合的不同方法。

但是首先，让我们了解一下不可变集合，看看我们为什么需要它。

## 2。什么是不可变集合？

一般来说，**一个[不可变对象](/web/20220926153229/https://www.baeldung.com/java-immutable-object)一旦被我们创建就不会改变它的内部状态。**这使得它在默认情况下是线程安全的。同样的逻辑也适用于不可变集合。

假设我们有一个带有一些值的 [`HashSet`](/web/20220926153229/https://www.baeldung.com/java-hashset) 实例。使它成为不可变的将会创建我们的 set 的一个“只读”版本。由此，**任何修改其状态的企图都会抛出`UnsupportedOperationException`** 。

那么，我们为什么需要它呢？

当然，不可变集合最常见的用例是多线程环境。因此，我们可以跨线程共享不可变的数据，而不用担心同步问题。

同时，有一点需要记住:**不变性只与集合有关，而与其元素无关**。此外，我们可以毫无问题地修改集合元素的实例引用。

## 3。在核心 Java 中创建不可变集合

**有了核心 Java 类，我们就可以使用`Collections`。`unmodifiableSet()`法裹原`Set`。**

首先，让我们创建一个简单的 [`HashSet`](/web/20220926153229/https://www.baeldung.com/java-hashset) 实例，并用`String`值初始化它:

```java
Set<String> set = new HashSet<>();
set.add("Canada");
set.add("USA");
```

接下来，我们用`Collections`来总结一下。`unmodifiableSet():`

```java
Set<String> unmodifiableSet = Collections.unmodifiableSet(set);
```

最后，为了确保我们的`unmodifiableSet`实例是不可变的，让我们创建一个简单的测试用例:

```java
@Test(expected = UnsupportedOperationException.class)
public void testUnmodifiableSet() {
    // create and initialize the set instance

    Set<String> unmodifiableSet = Collections.unmodifiableSet(set);
    unmodifiableSet.add("Costa Rica");
}
```

正如我们所料，测试将成功运行。此外，`the unmodifiableSet`实例上禁止`add()`操作，会抛出`UnsupportedOperationException`。

现在，让我们通过添加相同的值来改变初始的`set`实例:

```java
set.add("Costa Rica");
```

这样，我们间接地修改了不可修改的集合。因此，当我们打印`unmodifiableSet`实例时:

```java
[Canada, USA, Costa Rica]
```

正如我们所见，`“Costa Rica”`项也出现在`unmodifiableSet.`中

## 4。在 Java 9 中创建不可变集合

从 Java 9 开始，`Set.of(elements)`静态工厂方法可用于创建不可变集合:

```java
Set<String> immutable = Set.of("Canada", "USA");
```

## 5。在番石榴中创建不可变集合

**另一种构造不可变集合的方法是使用 Guava 的`ImmutableSet`类**。它将现有数据复制到一个新的不可变实例中。因此，当我们改变原来的`Set`时，`ImmutableSet`中的数据不会改变。

与核心 Java 实现一样，任何修改已创建的不可变实例的尝试都会抛出`UnsupportedOperationException`。

现在，让我们探索创建不可变实例的不同方法。

### 5.1。使用`ImmutableSet.` `**copyOf()**`

简单来说就是`ImmutableSet`。`copyOf()`方法返回集合中所有元素的副本:

```java
Set<String> immutable = ImmutableSet.copyOf(set);
```

因此，在更改初始设置后，不可变实例将保持不变:

```java
[Canada, USA]
```

### 5.2。使用 `**ImmutableSet****.of()**`

类似地，使用`ImmutableSet.of()`方法，我们可以立即用给定的值创建一个不可变的集合:

```java
Set<String> immutable = ImmutableSet.of("Canada", "USA");
```

当我们不指定任何元素时，`ImmutableSet.of()`将返回一个空的不可变集合。

这可以比作 Java 9 的`Set` `.of().`

## 6。结论

在这篇简短的文章中，我们讨论了 Java 语言中的不可变`Sets`。**此外，我们展示了如何使用来自核心 Java、Java 9 和 Guava 库的集合 API 创建不可变的`Sets`。**

最后，像往常一样，本文的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220926153229/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set)