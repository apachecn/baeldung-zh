# 在 Java HashMap 中使用自定义类作为键

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-custom-class-map-key>

## 1.概观

在本文中，我们将学习`HashMap`如何在内部管理键值对，以及如何编写定制的键实现。

## 2.密钥管理

### 2.1.内部结构

映射用于存储分配给键的值。该键用于识别`Map`中的值并检测重复值。

当`TreeMap`使用`Comparable#compareTo(Object)`方法对键进行排序(并识别相等性)时，`HashMap`使用基于散列的结构，这可以用一个简单的草图更容易地解释:

[![hashtable](img/a74f3165540292ea46ee296d7a01fc0a.png)](/web/20221129090458/https://www.baeldung.com/wp-content/uploads/2021/10/hashtable.svg)

一个`Map`不允许重复的键，所以使用`Object#equals(Object)`方法来比较这些键。因为这个方法的性能很差，所以应该尽可能避免调用。这是通过`Object#hashCode()`方法实现的。这个方法允许按照对象的哈希值对对象进行排序，然后只有当对象共享相同的哈希值时，才需要调用`Object#equals`方法。

这种密钥管理也适用于`[HashSet](/web/20221129090458/https://www.baeldung.com/java-hashset)`类，它的实现在内部使用了一个`HashMap`。

### 2.2.插入和查找键值对

让我们创建一个简单商店的`HashMap`示例，该商店通过商品 id ( `String`)管理库存商品的数量(`Integer`)。在这里，我们输入一个样本值:

```java
Map<String, Integer> items = new HashMap<>();
// insert
items.put("158-865-A", 56);
// find
Integer count = items.get("158-865-A");
```

插入键值对的算法:

1.  调用`“158-865-A”.hashCode()`来获取哈希值
2.  查找共享相同哈希值的现有键的列表
3.  将列表中的任意键与`“158-865-A”.equals(key)`进行比较
    1.  第一个等式被识别为已经存在，并且新的等式替换分配的值。
    2.  如果不相等，则键-值对作为新条目插入。

要查找一个值，算法是相同的，只是没有替换或插入任何值。

## 3.自定义键类

我们可以得出结论，要使用一个键的自定义类，**需要正确实现 [`hashCode()`和`equals()`](/web/20221129090458/https://www.baeldung.com/java-equals-hashcode-contracts)**。简单地说，我们必须确保`hashCode()`方法返回:

*   只要状态不变，对象的值不变(`Internal Consistency`)
*   相等对象的相同值(`Equals Consistency`)
*   不相等的对象有尽可能多的不同值。

我们通常会说`hashCode()`和`equals()`应该在它们的计算中考虑相同的字段，我们必须覆盖这两个字段或者都不覆盖。我们可以通过使用[龙目岛](/web/20221129090458/https://www.baeldung.com/intro-to-project-lombok)或我们 IDE 的生成器轻松实现这一点。

另一个要点是:**当一个对象正被用作键时，不要改变该对象的散列码。**一个简单的解决方案是将键类设计成[不可变的](/web/20221129090458/https://www.baeldung.com/java-immutable-object)，但是只要我们能够确保操作不会发生在键上，这就不是必需的。

不变性在这里有一个优势:哈希值可以在对象实例化时计算一次，这可以提高性能，特别是对于复杂的对象。

### 3.1.好榜样

例如，我们将设计一个由`x`和`y`值组成的`Coordinate`类，并将其用作`HashMap`中的一个键:

```java
Map<Coordinate, Color> pixels = new HashMap<>();
Coordinate coord = new Coordinate(1, 2);
pixels.put(coord, Color.CYAN);
// read the color
Color color = pixels.get(new Coordinate(1, 2));
```

让我们实现我们的`Coordinate`类:

```java
public class Coordinate {
    private final int x;
    private final int y;
    private int hashCode;

    public Coordinate(int x, int y) {
        this.x = x;
        this.y = y;
        this.hashCode = Objects.hash(x, y);
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        Coordinate that = (Coordinate) o;
        return x == that.x && y == that.y;
    }

    @Override
    public int hashCode() {
        return this.hashCode;
    }
} 
```

作为替代，我们可以使用 Lombok 使我们的类更短:

```java
@RequiredArgsConstructor
@Getter
// no calculation in the constructor, but
// since Lombok 1.18.16, we can cache the hash code
@EqualsAndHashCode(cacheStrategy = CacheStrategy.LAZY)
public class Coordinate {
    private final int x;
    private final int y;
} 
```

最佳的内部结构是:

[![hashtable optimal](img/703d4d491d2dbb23041fa98d10ce5e39.png)](/web/20221129090458/https://www.baeldung.com/wp-content/uploads/2021/10/hashtable_optimal.svg)

### 3.2.错误示例:静态哈希值

如果我们通过对所有实例使用静态哈希值来实现`Coordinate`类，那么`HashMap`将会正确工作，但是性能会显著下降:

```java
public class Coordinate {

    ...

    @Override
    public int hashCode() {
        return 1; // return same hash value for all instances
    }
}
```

哈希结构如下所示:

[![hashtable worst](img/c17c53b7b666cebd6c8566e6188adc9f.png)](/web/20221129090458/https://www.baeldung.com/wp-content/uploads/2021/10/hashtable_worst.svg)

这完全否定了哈希值的优势。

### 3.3.错误示例:可修改的哈希值

如果我们使 key 类可变，我们应该确保实例的状态在用作键时不会改变:

```java
Map<Coordinate, Color> pixels = new HashMap<>();
Coordinate coord = new Coordinate(1, 2); // x=1, y=2
pixels.put(coord, Color.CYAN);
coord.setX(3); // x=3, y=2
```

因为`Coordinate`存储在旧的哈希值下，所以在新的哈希值下找不到它。因此，下面这条线将导致一个`null`值:

```java
Color color = pixels.get(coord);
```

下面一行将导致对象在`Map`中存储两次:

```java
pixels.put(coord, Color.CYAN);
```

## 4.结论

在本文中，我们已经阐明了为一个`HashMap`实现一个自定义键类就是正确实现`equals()`和`hashCode()`的问题。我们已经看到了哈希值是如何在内部使用的，以及它是如何受到好的和坏的影响的。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221129090458/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-4)