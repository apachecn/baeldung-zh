# 集合的 Java 便利工厂方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-collections-factory-methods>

## 1。概述

Java 9 带来了期待已久的语法糖，可以使用简洁的一行代码创建小型不可修改的`Collection`实例。根据 [JEP 269](https://web.archive.org/web/20220926194623/https://openjdk.java.net/jeps/269) ，新的便利工厂方法将包含在 JDK 9 中。

在本文中，我们将介绍它的用法以及实现细节。

## 2。历史和动机

使用传统的方式在 Java 中创建一个小的不可变的`Collection`是非常冗长的。

让我们举一个`Set`的例子:

```java
Set<String> set = new HashSet<>();
set.add("foo");
set.add("bar");
set.add("baz");
set = Collections.unmodifiableSet(set);
```

对于一个简单的任务来说，代码太多了，应该可以用一个表达式来完成。

以上对于 a `Map.`也是成立的

然而，对于`List`，有一个工厂方法:

```java
List<String> list = Arrays.asList("foo", "bar", "baz");
```

尽管这种`List`创建比构造函数初始化要好，但这种[不如](https://web.archive.org/web/20220926194623/https://en.wikipedia.org/wiki/Principle_of_least_astonishment)明显，因为一般直觉不会查看`Arrays`类中的方法来创建`List`:

还有其他减少冗长的方法，如**双括号初始化**技术:

```java
Set<String> set = Collections.unmodifiableSet(new HashSet<String>() {{
    add("foo"); add("bar"); add("baz");
}});
```

或者通过使用 Java 8 `Streams`:

```java
Stream.of("foo", "bar", "baz")
  .collect(collectingAndThen(toSet(), Collections::unmodifiableSet));
```

双括号技术只是稍微不那么冗长，但是大大降低了可读性(并且被认为是一种反模式)。

然而，Java 8 版本是一个单行表达式，它也有一些问题。第一，不明显，不直观。第二，还是啰嗦。第三，它涉及到创建不必要的对象。第四，这个方法不能用于创建一个`Map`。

总结一下缺点，上面的方法都没有处理创建一个小的不可修改的`Collection` 一级类问题的特定用例。

## 3。描述和用途

已经为`List`、`Set`和`Map`接口提供了静态方法，这些接口将元素作为参数，并分别返回`List`、`Set`和`Map`的实例。

这三个接口都将这个方法命名为`of(…)`。

### 3.1。`List`和`Set`

`List`和`Set`工厂方法的签名和特征是相同的:

```java
static <E> List<E> of(E e1, E e2, E e3)
static <E> Set<E>  of(E e1, E e2, E e3)
```

方法的使用:

```java
List<String> list = List.of("foo", "bar", "baz");
Set<String> set = Set.of("foo", "bar", "baz");
```

正如我们所看到的，它非常简单、简短、简洁。

在这个例子中，我们使用的方法正好取三个元素作为参数，并返回一个大小为 3 的`List` / `Set`。

但是，这个方法有 12 个重载版本——11 个有 0 到 10 个参数，1 个有 var-args:

```java
static <E> List<E> of()
static <E> List<E> of(E e1)
static <E> List<E> of(E e1, E e2)
// ....and so on

static <E> List<E> of(E... elems)
```

对于大多数实际用途，10 个元素就足够了，但是如果需要更多，可以使用 var-args 版本。

现在，我们可能会问，如果有一个 var-args 版本可以为任意数量的元素工作，那么拥有 11 个额外的方法有什么意义呢？

答案是性能。**每个 var-args 方法调用都会隐式创建一个数组。让重载方法避免不必要的对象创建及其垃圾收集开销。**相反，`Arrays.asList`总是创建隐式数组，因此，当元素数量较低时效率较低。

在使用工厂方法创建`Set`的过程中，如果重复的元素作为参数传递，那么`IllegalArgumentException`在运行时被抛出:

```java
@Test(expected = IllegalArgumentException.class)
public void onDuplicateElem_IfIllegalArgExp_thenSuccess() {
    Set.of("foo", "bar", "baz", "foo");
}
```

这里需要注意的重要一点是，由于工厂方法使用泛型，所以基本类型被自动装箱。

如果传递了原始类型的数组，则返回该原始类型的`array`的`List`。

例如:

```java
int[] arr = { 1, 2, 3, 4, 5 };
List<int[]> list = List.of(arr);
```

在这种情况下，返回大小为 1 的`List<int[]>`,索引为 0 的元素包含数组。

### 3.2。`Map`

`Map`工厂方法的签名是:

```java
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3)
```

以及用法:

```java
Map<String, String> map = Map.of("foo", "a", "bar", "b", "baz", "c");
```

与`List`和`Set`类似，`of(…)`方法被重载为具有 0 到 10 个键值对。

在`Map`的情况下，有一个不同的方法用于 10 个以上的键值对:

```java
static <K,V> Map<K,V> ofEntries(Map.Entry<? extends K,? extends V>... entries)
```

它的用法是:

```java
Map<String, String> map = Map.ofEntries(
  new AbstractMap.SimpleEntry<>("foo", "a"),
  new AbstractMap.SimpleEntry<>("bar", "b"),
  new AbstractMap.SimpleEntry<>("baz", "c"));
```

为 Key 传入重复的值会抛出一个`IllegalArgumentException`:

```java
@Test(expected = IllegalArgumentException.class)
public void givenDuplicateKeys_ifIllegalArgExp_thenSuccess() {
    Map.of("foo", "a", "foo", "b");
}
```

同样，在`Map`的情况下，基本类型也是自动装箱的。

## 4。实施说明

使用工厂方法创建的集合不是常用的实现。

例如，`List`不是`ArrayList`，`Map`也不是`HashMap`。这些是 Java 9 中引入的不同实现。这些实现是内部的，它们的构造函数具有受限的访问权限。

在这一节中，我们将看到所有三种类型的集合共有的一些重要的实现差异。

### 4.1。不可变

使用工厂方法创建的集合是不可变的，改变一个元素、添加新元素或者删除一个元素都会抛出`UnsupportedOperationException`:

```java
@Test(expected = UnsupportedOperationException.class)
public void onElemAdd_ifUnSupportedOpExpnThrown_thenSuccess() {
    Set<String> set = Set.of("foo", "bar");
    set.add("baz");
}
```

```java
@Test(expected = UnsupportedOperationException.class)
public void onElemModify_ifUnSupportedOpExpnThrown_thenSuccess() {
    List<String> list = List.of("foo", "bar");
    list.set(0, "baz");
}
```

```java
@Test(expected = UnsupportedOperationException.class)
public void onElemRemove_ifUnSupportedOpExpnThrown_thenSuccess() {
    Map<String, String> map = Map.of("foo", "a", "bar", "b");
    map.remove("foo");
}
```

### 4.2。不允许有`null`元素

在`List`和`Set`的情况下，没有元素可以是`null`。在`Map`的情况下，键和值都不能是`null`。传递`null`参数抛出一个`NullPointerException`:

```java
@Test(expected = NullPointerException.class)
public void onNullElem_ifNullPtrExpnThrown_thenSuccess() {
    List.of("foo", "bar", null);
}
```

与`List.of`相反，`Arrays.asList`方法接受`null`值。

### 4.3。基于值的实例

由工厂方法创建的实例是基于值的。这意味着工厂可以自由地创建新的实例或返回现有的实例。

因此，如果我们创建具有相同值的列表，它们可能引用也可能不引用堆上的相同对象:

```java
List<String> list1 = List.of("foo", "bar");
List<String> list2 = List.of("foo", "bar");
```

在这种情况下，`list1 == list2`可能会也可能不会评估为`true`，这取决于 JVM。

### 4.4。序列化

如果集合的元素是`Serializable.`，则从工厂方法创建的集合是`Serializable`

## 5。结论

在本文中，我们介绍了 Java 9 中引入的集合的新工厂方法。

通过回顾过去创建不可修改集合的一些方法，我们总结了为什么这个特性是一个受欢迎的变化。我们介绍了它的用法，并强调了使用它们时需要考虑的要点。

最后，我们澄清了这些集合不同于常用的实现，并指出了关键的区别。

GitHub 上的[提供了本文的完整源代码和单元测试。](https://web.archive.org/web/20220926194623/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-improvements)