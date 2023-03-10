# 用 Java 实现多键映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-multiple-keys-map>

## 1。简介

我们经常在程序中使用映射，作为一种将键和值关联起来的方法。通常在我们的 Java 程序中，特别是自从引入了[泛型](/web/20220915150108/https://www.baeldung.com/java-generics)之后，我们将所有的键都是相同的类型，所有的值也是相同的类型。例如，id 到数据存储中的值的映射。

在某些情况下，我们可能希望使用键不总是相同类型的映射。**例如，如果我们将 ID 类型从`Long`更改为`String, `，那么我们的数据存储将需要支持两种关键类型——旧条目的`Long`和新条目的`String` 。**

不幸的是，Java `Map`接口不允许多种键类型，所以我们需要找到另一种解决方案。在本文中，我们将探索实现这一目标的几种方法。

## 2。使用通用超类型

实现这一点最简单的方法是建立一个映射，其中键类型是与所有键最接近的超类型。在某些情况下，这可能很容易——例如，如果我们的键是`Long`和`Double`,那么最接近的超类型是`Number`:

```java
Map<Number, User> users = new HashMap<>();

users.get(longId);
users.get(doubleId);
```

但是，在其他情况下，最接近的超类型是`Object`。这有一个缺点，它完全从我们的映射中移除了类型安全:

```java
Map<Object, User> users = new HashMap<>();

users.get(longId); /// Works.
users.get(stringId); // Works.
users.get(Instant.now()); // Also works.
```

在这种情况下，编译器不会阻止我们传入错误的类型，实际上从我们的映射中删除了所有类型安全。在某些情况下，这可能是好的。例如，如果另一个类封装了映射以加强类型安全本身，这可能没问题。

然而，它仍然在如何使用地图方面带来了风险。

## 3。多张地图

如果类型安全很重要，并且我们将把我们的映射封装在另一个类中，另一个简单的选择是拥有多个映射。在这种情况下，我们对每个支持的键都有不同的映射:

```java
Map<Long, User> usersByLong = new HashMap<>();
Map<String, User> usersByString = new HashMap<>();
```

这样做可以确保编译器为我们保持类型安全。如果我们试图在这里使用一个 `Instant`，那么编译器不会允许，所以我们在这里是安全的。

不幸的是，这增加了复杂性，因为我们需要知道使用哪张地图。这意味着我们要么用不同的方法处理不同的地图，要么到处做类型检查。

这也不能很好地扩展。如果我们需要添加一个新的键类型，我们将需要添加一个新的地图和新的检查。对于两三种关键类型，这是可以管理的，但很快就会变得太多。

## 4。关键包装类型

如果我们需要类型安全，并且我们不想要许多映射的可维护性负担，那么我们需要找到一种方法来拥有一个可以在键中有不同值的单个映射。这意味着我们需要找到某种方法来拥有一个实际上是不同类型的单一类型。我们可以通过两种不同的方式实现这一点——使用单个包装器或者使用接口和子类。

### 4.1。单一包装类

我们的一个选择是编写一个类，它可以包装任何可能的键类型。这将有一个用于实际键值的单独字段，正确的`equals`和`hashCode`方法，以及一个用于每个可能类型的构造函数:

```java
class MultiKeyWrapper {
    private final Object key;

    MultiKeyWrapper(Long key) {
        this.key = key;
    }

    MultiKeyWrapper(String key) {
        this.key = key;
    }

    @Override
    public bool equals(Object other) { ... }

    @Override
    public int hashCode() { ... }
}
```

这保证是类型安全的，因为它只能用`Long`或`String`来构造。我们可以在地图中将它作为一个单独的类型，因为它本身就是一个单独的类:

```java
Map<MultiKeyWrapper, User> users = new HashMap<>();
users.get(new MultiKeyWrapper(longId)); // Works
users.get(new MultiKeyWrapper(stringId)); // Works
users.get(new MultiKeyWrapper(Instant.now())); // Compilation error
```

我们只需要在新的`MultiKeyWrapper`中包装我们的`Long` 或`String`来访问地图。

这相对简单，但是会使扩展稍微困难一些。无论何时我们想要支持任何额外的类型，我们都需要改变我们的`MultiKeyWrapper`类来支持它。

### 4.2。接口和子类

另一种方法是编写一个接口来表示我们的密钥包装器，然后为我们想要支持的每种类型编写该接口的实现:

```java
interface MultiKeyWrapper {}

record LongMultiKeyWrapper(Long value) implements MultiKeyWrapper {}
record StringMultiKeyWrapper(String value) implements MultiKeyWrapper {}
```

正如我们所看到的，这些实现可以使用 Java 14 中引入的[记录功能](/web/20220915150108/https://www.baeldung.com/java-record-keyword)，这将使实现变得更加容易。

像以前一样，我们可以使用我们的`MultiKeyWrapper`作为地图的单键类型。然后，我们对想要使用的键类型使用适当的实现:

```java
Map<MultiKeyWrapper, User> users = new HashMap<>();
users.get(new LongMultiKeyWrapper(longId)); // Works
users.get(new StringMultiKeyWrapper(stringId)); // Works 
```

在这种情况下，我们没有类型可用于其他任何事情，所以我们甚至不能在第一时间编写无效代码。

有了这个解决方案，我们支持额外的键类型，不是通过改变现有的类，而是通过编写一个新的类。这更容易支持，但也意味着我们对支持什么键类型的控制更少。

然而，这可以通过正确使用[可见性修饰符](/web/20220915150108/https://www.baeldung.com/java-access-modifiers)来管理。只有当类可以访问我们的接口时，它们才能实现我们的接口，所以如果我们使它成为包私有的，那么只有同一个包中的类才能实现它。

## 5。结论

这里我们看到了一些表示键到值的映射的方法，但是键并不总是相同的类型。这些策略的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220915150108/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5)