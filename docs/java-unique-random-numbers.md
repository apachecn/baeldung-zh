# 在 Java 中创建无重复的随机数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-unique-random-numbers>

## 1.介绍

在这个快速教程中，我们将学习如何使用核心 Java 类生成无重复的随机数。首先，我们将从头实现几个解决方案，然后利用 Java 8+的特性实现一个更具可扩展性的方法。

## 2.小范围内的随机数

如果我们需要的数字范围很小，我们可以继续向列表中添加连续的数字，直到达到大小`n`。**那么，我们称之为 [`Collections.shuffle()`](/web/20221113160541/https://www.baeldung.com/java-shuffle-collection) ，它具有线性的[时间复杂度](/web/20221113160541/https://www.baeldung.com/java-algorithm-complexity)。之后，我们将得到一个唯一数字的随机列表。**让我们创建一个实用程序类来生成和使用这些数字:

```java
public class UniqueRng implements Iterator<Integer> {
    private List<Integer> numbers = new ArrayList<>();

    public UniqueRng(int n) {
        for (int i = 1; i <= n; i++) {
            numbers.add(i);
        }

        Collections.shuffle(numbers);
    }
}
```

在构建我们的对象之后，我们将拥有从 1 到`size`的随机顺序的数字。**注意我们正在实现 [`Iterator`](/web/20221113160541/https://www.baeldung.com/java-iterator-vs-iterable) ，所以我们每次调用`next()`都会得到一个随机数。**此外，我们可以检查是否还有剩余的`hasNext()`号码。所以，让我们用[取代](/web/20221113160541/https://www.baeldung.com/java-override)吧:

```java
@Override
public Integer next() {
    if (!hasNext()) {
        throw new NoSuchElementException();
    }
    return numbers.remove(0);
}

@Override
public boolean hasNext() {
    return !numbers.isEmpty();
}
```

因此，`remove()`从列表中返回第一个删除的项目。类似地，如果我们没有打乱我们的集合，我们可以传递给它一个随机索引。但是，在构建时洗牌的好处是让我们提前知道整个序列。

### 2.1.将它投入使用

要使用它，我们只需选择想要多少个号码并消费它们:

```java
UniqueRng rng = new UniqueRng(5);
while (rng.hasNext()) {
    System.out.print(rng.next() + " ");
}
```

这可能会产生如下输出:

```java
4 1 2 5 3
```

## 3.大范围的随机数

如果我们想要更广泛的数字范围，只使用其中的几个，我们需要一个不同的策略。首先，我们不能依赖于给一个`ArrayList`添加随机数，因为这可能会产生重复。**因此，我们将使用`[Set](/web/20221113160541/https://www.baeldung.com/java-set-operations)`，因为它保证了唯一的项目。**然后，我们将使用`LinkedHashSet`实现，因为它维护插入顺序。

**这一次，我们将循环添加元素到集合中，直到达到*大小*。同样，我们将使用 [`Random`](/web/20221113160541/https://www.baeldung.com/java-generating-random-numbers-in-range) 来生成从零到`max` :** 的随机整数

```java
public class BigUniqueRng implements Iterator<Integer> {
    private Random random = new Random();
    private Set<Integer> generated = new LinkedHashSet<>();

    public BigUniqueRng(int size, int max) {
        while (generated.size() < size) {
            Integer next = random.nextInt(max);
            generated.add(next);
        }
    }
}
```

请注意，我们不需要检查一个数字是否已经存在于我们的集合中，因为`add()`会这样做。**现在，由于我们不能通过索引删除项目，我们需要一个`Iterator`的帮助来实现`next()` :**

```java
public Integer next() {
    Iterator<Integer> iterator = generated.iterator();
    Integer next = iterator.next();
    iterator.remove();
    return next;
}
```

## 4.利用 Java 8+的特性

虽然定制实现更具可重用性，但我们可以仅使用 [`Stream` s](/web/20221113160541/https://www.baeldung.com/java-streams) 来创建解决方案。**从 Java 8 开始，`Random`有一个返回 [`IntStream`](/web/20221113160541/https://www.baeldung.com/java-intstream-convert) 的`ints()`方法。我们可以对其进行流式处理，并强加与之前相同的必要条件，如范围和限制。**让我们将这些特征和`[collect](/web/20221113160541/https://www.baeldung.com/java-stream-immutable-collection)`结果组合成一个`Set`:

```java
Set<Integer> set = new Random().ints(-5, 15)
  .distinct()
  .limit(5)
  .boxed()
  .collect(Collectors.toSet());
```

遍历的集合可能会产生如下输出:

```java
-5 13 9 -4 14
```

有了`ints(),`,范围从负整数开始就更简单了。**但是，我们必须小心不要以无限流结束，例如，如果我们不调用`limit()`，就会发生无限流。**

## 5.结论

在本文中，我们编写了两个解决方案，在两种情况下生成无重复的随机数。首先，我们把这些类做成可迭代的，这样我们就可以很容易地使用它们。然后，我们使用 streams 创建了一个更加有机的解决方案。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221113160541/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-5)