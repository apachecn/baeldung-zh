# 在 Java 中复制集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-copy-sets>

## 1.概观

简单地说，`Set`是一个不包含重复元素的集合。在 Java 中，`Set`是一个扩展了 [`Collection`](/web/20221206224036/https://www.baeldung.com/java-collections) 接口的接口。

在这个快速教程中，我们将通过不同的方式在 Java 中复制集合。

## 2.复制构造函数

复制`Set`的一种方式是使用`Set` 实现的[复制构造函数](/web/20221206224036/https://www.baeldung.com/java-constructors):

```java
Set<T> copy = new HashSet<>(original);
```

**复制构造函数是一种特殊类型的构造函数，用于通过[复制现有对象](/web/20221206224036/https://www.baeldung.com/java-deep-copy)来创建新对象。**

在这里，我们并没有真正克隆给定集合的元素。我们只是将对象引用复制到新的集合中。因此，在一个元素中所做的每个更改都会影响到两个集合。

## 3.`Set.addAll`

`Set`接口有一个`[addAll](/web/20221206224036/https://www.baeldung.com/java-set-operations) `方法`. `，它将集合中的元素添加到目标集合中。因此，我们可以使用`addAll`方法将现有集合的元素复制到空集:

```java
Set<T> copy = new HashSet<>();
copy.addAll(original);
```

## 4.`Set.clone`

让我们记住，`Set`是一个扩展了`Collection`接口的接口，因此**我们需要引用一个实现了`Set`接口的对象来创建`Set`的另一个实例。** `HashSet`、`TreeSet`、`LinkedHashSet,`、`EnumSet `都是`Set`在 Java 中实现的例子。

**所有这些`Set`实现都有一个克隆方法，因为它们都实现了 [`Cloneable`](/web/20221206224036/https://www.baeldung.com/java-deep-copy) 接口。**

因此，作为复制集合的另一种方法，我们可以调用集合的`clone`方法:

```java
Set<T> copy = (Set<T>) original.clone();
```

**让我们也注意一下，克隆最初来自于`Object.clone`。** Set 实现覆盖了`Object`类的`clone`方法。克隆的性质取决于实际的实现。例如，`HashSet`只做浅层复制，尽管我们可以通过编码让[做深层复制](/web/20221206224036/https://www.baeldung.com/java-deep-copy)。

正如我们所看到的，我们被迫将克隆的对象类型转换为`Set<T>`，因为**`clone`方法实际上返回了一个`Object`。**

## 5.JSON

复制集合的另一种方法是将其序列化到一个`JSON String`中，并从生成的`JSON String`中创建一个新的集合。还值得注意的是 **对于这种方法，集合中的所有元素和引用的元素都必须是可序列化的**并且**我们将执行所有对象的深度复制**。

在这个例子中，我们将通过使用 Google 的`Gson` 库的[序列化](/web/20221206224036/https://www.baeldung.com/gson-serialization-guide)和[反序列化](/web/20221206224036/https://www.baeldung.com/gson-deserialization-guide)方法来复制集合:

```java
Gson gson = new Gson();
String jsonStr = gson.toJson(original);
Set<T> copy = gson.fromJson(jsonStr, Set.class);
```

## 6.Apache Commons Lang

[Apache Commons Lang](/web/20221206224036/https://www.baeldung.com/java-commons-lang-3) 有一个类`SerializationUtils`，它提供了一个特殊的方法`clone`，可以用来克隆一个给定的对象。我们可以利用这种方法复制一个集合:

```java
for (T item : original) {
    copy.add(SerializationUtils.clone(item));
}
```

让我们注意一下， **`SerializationUtils.clone`期望它的参数扩展`Serializable` 类**。

## 7.`Collectors.toSet`

或者，我们可以用 [Java 8 的`Stream` API](/web/20221206224036/https://www.baeldung.com/java-8-streams) 搭配 [`Collectors`](/web/20221206224036/https://www.baeldung.com/java-8-collectors) 克隆一套:

```java
Set<T> copy = original.stream()
    .collect(Collectors.toSet());
```

`Stream API`的一个优点是它允许我们使用[跳过](/web/20221206224036/https://www.baeldung.com/java-8-streams)、[过滤器](/web/20221206224036/https://www.baeldung.com/java-stream-filter-lambda)等等，从而提供了更多的便利。

## 8.使用 Java 10

Java 10 给`Set`接口带来了一个新特性，即**允许我们从给定集合**的元素中创建一个不可变集合:

```java
Set<T> copy = Set.copyOf(original);
```

**注意，`Set.copyOf`需要一个非`null`参数。**

## 9.结论

在本文中，我们探索了在 Java 中复制集合的不同方法。

像往常一样，看看我们的例子的源代码[，包括 Java 10](https://web.archive.org/web/20221206224036/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set) 的源代码[。](https://web.archive.org/web/20221206224036/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-10)