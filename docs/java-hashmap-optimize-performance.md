# 优化 HashMap 的性能

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap-optimize-performance>

## 1.介绍

`HashMap`是一种功能强大的数据结构，具有广泛的应用，尤其是在需要快速查找的时候。然而，如果我们不注意细节，它可能会变得次优。

在本教程中，我们将看看如何使 [`HashMap`](/web/20221128113736/https://www.baeldung.com/java-hashmap) 尽可能快。

## 2.`HashMap`的瓶颈

**`HashMap`的元素检索的乐观常数时间(`O(1)`)来自哈希的威力。**对于每个元素，`HashMap`计算散列码并将元素放入与该散列码相关的桶中。因为不相等的对象可以有相同的哈希代码(这种现象称为哈希代码冲突)，所以存储桶的大小可以增长。

桶实际上是一个简单的链表。在链表中查找元素不是很快(`O(n)`)，但是如果链表很小，这不是问题。当我们有很多哈希代码冲突时，问题就开始了，所以我们没有大量的小桶，而是有少量的大桶。

**在最坏的情况下，我们把所有东西都放在一个桶里，我们的`HashMap`被降级为一个链表。**因此，我们得到的不是`O(1)`查找时间，而是一个非常不令人满意的`O(n)`。

## 3.树而不是`LinkedList`

从 Java 8 开始，[在`HashMap` : **中内置了一个优化**](https://web.archive.org/web/20221128113736/https://hg.openjdk.java.net/jdk8u/jdk8u/jdk/file/a006fa0a9e8f/src/share/classes/java/util/HashMap.java#l143)，当桶变得太大时，它们被转换成树，而不是链表。这样就把`O(n)` 的悲观时间带到了`O(log(n))`，这样就好多了。**为此，`HashMap`的按键需要实现 [`Comparable`](/web/20221128113736/https://www.baeldung.com/java-comparator-comparable) 接口。**

这是一个不错的自动解决方案，但并不完美。仍然比期望的常数时间差，并且变换和存储树需要额外的能量和内存。

## 4.最佳`hashCode`实施

在选择哈希函数时，我们需要考虑两个因素:生成的哈希代码的质量和速度。

### 4.1.测量`hashCode`质量

哈希代码存储在`int`变量中，所以可能的哈希数受限于`int`类型的容量。这是必然的，因为散列被用来计算带有桶的数组的索引。这意味着在没有哈希冲突的情况下，我们可以存储在`HashMap`中的键的数量也是有限的。

为了尽可能避免冲突，我们希望尽可能均匀地分布散列。换句话说，我们希望实现均匀分布。这意味着每个哈希码值出现的机会与其他值相同。

类似地，一个不好的`hashCode`方法会有一个非常不平衡的分布。在最坏的情况下，它总是返回相同的数字。

### 4.2.默认`Object`的`hashCode`

一般来说，我们不应该使用默认的`Object's` `hashCode`方法，因为我们[不想在`equals`方法中使用对象标识](/web/20221128113736/https://www.baeldung.com/java-map-key-byte-array#4-meaningful-equality)。然而，在那种非常不可能的场景中，我们真的想在一个`HashMap`中为键使用对象标识，默认的`hashCode`函数会工作得很好。否则，我们将需要一个自定义实现。

### 4.3.自定义`hashCode`

通常，我们希望[覆盖`equals`方法，然后我们还需要覆盖`hashCode`](/web/20221128113736/https://www.baeldung.com/java-equals-hashcode-contracts) 。有时候，我们可以利用类的特定身份，轻松做出一个非常快速的 [`hashCode`方法](/web/20221128113736/https://www.baeldung.com/java-hashcode)。

假设我们的对象的身份纯粹基于它的整数`id`。然后，我们可以使用这个`id`作为散列函数:

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    MemberWithId that = (MemberWithId) o;

    return id.equals(that.id);
}

@Override
public int hashCode() {
    return id;
}
```

它会非常快，不会产生任何碰撞。**我们的`HashMap`将表现得像它有一个整数键，而不是一个复杂的对象。**

如果我们需要考虑更多的领域，情况会变得更加复杂。假设我们希望将平等建立在`id`和`name`的基础上:

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    MemberWithIdAndName that = (MemberWithIdAndName) o;

    if (!id.equals(that.id)) return false;
    return name != null ? name.equals(that.name) : that.name == null;
}
```

现在，我们需要以某种方式组合`id`和`name`的散列。

首先，我们将像以前一样得到`id`的散列。然后，我们将它乘以一些精心选择的数字，并加上`name`的散列:

```java
@Override
public int hashCode() {
    int result = id.hashCode();
    result = PRIME * result + (name != null ? name.hashCode() : 0);
    return result;
}
```

如何选择这个数字并不是一个容易回答的问题。从历史上看，最流行的数字是`31. `它是质数，它产生良好的分布，它很小，乘以它可以使用移位操作进行优化:

```java
31 * i == (i << 5) - i
```

但是，现在不需要每个 CPU 周期都去争了，可以用一些更大的素数。例如，`524287 `也可以被优化:

```java
524287 * i == i << 19 - i
```

并且，它可以提供更好质量的散列，导致更小的冲突机会。请注意，这些**比特移位优化是由 JVM** 自动完成的，所以我们不需要用它们来混淆我们的代码。

### 4.4.`Objects`实用程序类

我们刚刚实现的算法已经很好地建立起来了，我们通常不需要每次都手工重新创建它。相反，我们可以使用由`Objects`类提供的 helper 方法:

```java
@Override
public int hashCode() {
    return Objects.hash(id, name);
}
```

在引擎盖下，它使用前面描述的算法，用数字`31`作为乘数。

### 4.5.其他哈希函数

有许多散列函数提供了比前面描述的更小的冲突机会。问题是它们的计算量更大，因此无法提供我们所寻求的速度增益。

如果出于某种原因，我们真的需要质量而不太在乎速度，我们可以看看来自[番石榴](/web/20221128113736/https://www.baeldung.com/guava-guide)库的`Hashing`类:

```java
@Override
public int hashCode() {
    HashFunction hashFunction = Hashing.murmur3_32();
    return hashFunction.newHasher()
      .putInt(id)
      .putString(name, Charsets.UTF_8)
      .hash().hashCode();
}
```

选择 32 位函数很重要，因为我们无论如何都不能存储更长的散列。

## 5.结论

现代 Java 的`HashMap `是一种强大且优化良好的数据结构。然而，设计不良的`hashCode`方法会降低其性能。在本教程中，我们研究了使散列快速有效的可能方法。

和往常一样，本文的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20221128113736/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5)