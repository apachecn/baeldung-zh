# 更新与 HashMap 中的键相关联的值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap-update-value-by-key>

## 1.概观

本教程将通过**不同的方法来更新与 [`HashMap`](/web/20220716164105/https://www.baeldung.com/java-hashmap)** 中给定键相关的值。首先，我们来看一些常见的解决方案，它们只使用 Java 8 之前的特性。然后，我们将看看 Java 8 和更高版本中可用的一些附加解决方案。

## 2.初始化我们的示例`HashMap`

为了展示如何更新`HashMap`中的值，我们必须首先创建并填充一个。因此，我们将创建一个以水果为关键字、以水果价格为值的地图:

```java
Map<String, Double> priceMap = new HashMap<>();
priceMap.put("apple", 2.45);
priceMap.put("grapes", 1.22);
```

我们将在整个例子中使用这个`HashMap`。现在，我们已经准备好熟悉更新与一个`HashMap`键相关的值的方法。

## 3.Java 8 之前

让我们从 Java 8 之前可用的方法开始。

### 3.1.`put`法

**`put` 方法要么更新值，要么添加一个新条目**。如果它与一个已经存在的键一起使用，那么`put`方法将更新相关的值。否则，它将添加一个新的`(key, value)`对。

让我们用两个简单的例子来测试这个方法的行为:

```java
@Test
public void givenFruitMap_whenPuttingAList_thenHashMapUpdatesAndInsertsValues() {
    Double newValue = 2.11;
    fruitMap.put("apple", newValue);
    fruitMap.put("orange", newValue);

    Assertions.assertEquals(newValue, fruitMap.get("apple"));
    Assertions.assertTrue(fruitMap.containsKey("orange"));
    Assertions.assertEquals(newValue, fruitMap.get("orange"));
} 
```

钥匙`apple`已经在地图上了。因此，第一个断言将通过。

因为`orange`不在地图中，所以`put`方法会添加它。因此，其他两个断言也将通过。

### 3.2.`containsKey` 和`put` 方法的组合

`containsKey` 和`put` 方法的组合是更新`HashMap`中键值的另一种方式。**该选项检查地图是否已经包含一个键。在这种情况下，我们可以使用`put`方法** `**.** `来更新值，否则，我们可以向地图添加一个条目，或者什么也不做。

在我们的例子中，我们将通过一个简单的测试来检验这种方法:

```java
@Test
public void givenFruitMap_whenKeyExists_thenValuesUpdated() {
    double newValue = 2.31;
    if (fruitMap.containsKey("apple")) {
        fruitMap.put("apple", newValue);
    }

    Assertions.assertEquals(Double.valueOf(newValue), fruitMap.get("apple"));
}
```

因为`apple`在地图上，所以`containsKey `方法将返回`true`。因此，将执行对`put `方法的调用，并且值将被更新。

## 4.Java 8 及以上版本

从 Java 8 开始，许多新的方法可以简化更新`HashMap.`中键值的过程，所以，让我们来了解一下它们。

### 4.1.`replace `方法

从版本 8 开始，`Map`接口中有两个重载的`replace`方法。让我们看看方法签名:

```java
public V replace(K key, V value);
public boolean replace(K key, V oldValue, V newValue);
```

**第一个`replace`方法只接受一个键和一个新值。它还返回旧值。**

让我们看看这个方法是如何工作的:

```java
@Test
public void givenFruitMap_whenReplacingOldValue_thenNewValueSet() {
    double newPrice = 3.22;
    Double applePrice = fruitMap.get("apple");

    Double oldValue = fruitMap.replace("apple", newPrice);

    Assertions.assertNotNull(oldValue);
    Assertions.assertEquals(oldValue, applePrice);
    Assertions.assertEquals(Double.valueOf(newPrice), fruitMap.get("apple"));
}
```

关键字`apple`的值将通过`replace`方法更新为新价格。因此，第二个和第三个断言将通过。

然而，**第一个断言很有趣**。如果我们的`HashMap`里没有键`apple`怎么办？**如果我们试图更新一个不存在的键的值，将返回`null`。考虑到这一点，另一个问题出现了:如果有一个值为`null`的键会怎么样？我们无法知道从`replace`方法返回的值是否确实是所提供的键的值，或者我们是否试图更新一个不存在的键的值。**

所以，为了避免误解，我们可以使用第二种`replace`方法。它需要三个参数:

*   一把钥匙
*   与键关联的当前值
*   与键关联的新值

它将在一个条件下将键值更新为一个新值:**如果第二个参数是当前值，键值将被更新为一个新值。对于成功的更新，该方法返回`true`。否则，返回`false`。**

因此，让我们实现一些测试来检查第二个`replace`方法:

```java
@Test
public void givenFruitMap_whenReplacingWithRealOldValue_thenNewValueSet() {
    double newPrice = 3.22;
    Double applePrice = fruitMap.get("apple");

    boolean isUpdated = fruitMap.replace("apple", applePrice, newPrice);

    Assertions.assertTrue(isUpdated);
}

@Test
public void givenFruitMap_whenReplacingWithWrongOldValue_thenNewValueNotSet() {
    double newPrice = 3.22;
    boolean isUpdated = fruitMap.replace("apple", Double.valueOf(0), newPrice);

    Assertions.assertFalse(isUpdated);
}
```

由于第一个测试用键的当前值调用了`replace` 方法，该值将被替换。

另一方面，第二个测试不是用当前值调用的。因此，`false`被返回。

### 4.2.`getOrDefault `和`put M`方法的组合

如果我们没有提供的键的条目，那么`getOrDefault`方法是一个完美的选择**。在这种情况下，我们为一个不存在的键设置默认值。然后，该条目被添加到地图中。**用这种方法，我们可以很容易地躲过** `**NullPointerException**.`**

让我们用一个原本不在地图上的键来试试这个组合:

```java
@Test
public void givenFruitMap_whenGetOrDefaultUsedWithPut_thenNewEntriesAdded() {
    fruitMap.put("plum", fruitMap.getOrDefault("plum", 2.41));

    Assertions.assertTrue(fruitMap.containsKey("plum"));
    Assertions.assertEquals(Double.valueOf(2.41), fruitMap.get("plum"));
}
```

由于没有这样的键，`getOrDefault`方法将返回默认值。然后，`put`方法将添加一个新的(键，值)对。因此，所有断言都会通过。

### 4.3.`putIfAbsent` 法

`putIfAbsent` 方法与之前组合的`getOrDefault` 和`put`方法`.`的作用相同

**如果`HashMap`中没有带有所提供密钥的配对，`putIfAbsent`方法将添加配对。然而，如果有这样一对,`putIfAbsent` 方法不会改变地图。**

但是，有一个例外:**如果现有对有一个`null`值，那么该对将被更新为一个新值。**

让我们实现对`putIfAbsent` 方法的测试。我们将通过两个例子来测试这种行为:

```java
@Test
public void givenFruitMap_whenPutIfAbsentUsed_thenNewEntriesAdded() {
    double newValue = 1.78;
    fruitMap.putIfAbsent("apple", newValue);
    fruitMap.putIfAbsent("pear", newValue);

    Assertions.assertTrue(fruitMap.containsKey("pear"));
    Assertions.assertNotEquals(Double.valueOf(newValue), fruitMap.get("apple"));
    Assertions.assertEquals(Double.valueOf(newValue), fruitMap.get("pear"));
}
```

一个键`apple `出现在地图`.`上`putIfAbsent`方法不会改变它的当前值。

与此同时，钥匙`pear`从地图上消失了。因此，将添加`.`

### 4.4.`compute` 法

`compute `方法**基于作为第二参数**提供的 [`BiFunction`](/web/20220716164105/https://www.baeldung.com/java-bifunction-interface) 来更新键的值。如果钥匙不在地图上，我们可以期待一个`NullPointerException`。

让我们通过一个简单的测试来检查这个方法的行为:

```java
@Test
public void givenFruitMap_whenComputeUsed_thenValueUpdated() {
    double oldPrice = fruitMap.get("apple");
    BiFunction<Double, Integer, Double> powFunction = (x1, x2) -> Math.pow(x1, x2);

    fruitMap.compute("apple", (k, v) -> powFunction.apply(v, 2));

    Assertions.assertEquals(
      Double.valueOf(Math.pow(oldPrice, 2)), fruitMap.get("apple"));

    Assertions.assertThrows(
      NullPointerException.class, () -> fruitMap.compute("blueberry", (k, v) -> powFunction.apply(v, 2)));
}
```

正如所料，由于键`apple`存在，它在映射中的值将被更新。另一方面，没有键`blueberry`，所以第二次调用最后一个断言中的`compute`方法将产生一个`NullPointerException`。

### 4.5.`computeIfAbsent `法

如果`HashMap`中没有特定键的配对，前面的方法会抛出异常。如果不存在，`computeIfAbsent` 方法**将通过添加一个`(key, value)`对来更新地图。**

让我们测试这个方法的行为:

```java
@Test
public void givenFruitMap_whenComputeIfAbsentUsed_thenNewEntriesAdded() {
    fruitMap.computeIfAbsent("lemon", k -> Double.valueOf(k.length()));

    Assertions.assertTrue(fruitMap.containsKey("lemon"));
    Assertions.assertEquals(Double.valueOf("lemon".length()), fruitMap.get("lemon"));
}
```

钥匙*柠檬*在地图上不存在。因此，`comp` `uteIfAbsent` 方法添加了一个条目。

### 4.6.`computeIfPresent` 法

`computeIfPresent `方法**更新一个键的值，如果它存在于`HashMap`T3 中的话。**

让我们看看如何使用这种方法:

```java
@Test
public void givenFruitMap_whenComputeIfPresentUsed_thenValuesUpdated() {
    Double oldAppleValue = fruitMap.get("apple");
    BiFunction<Double, Integer, Double> powFunction = (x1, x2) -> Math.pow(x1, x2);

    fruitMap.computeIfPresent("apple", (k, v) -> powFunction.apply(v, 2));

    Assertions.assertEquals(Double.valueOf(Math.pow(oldAppleValue, 2)), fruitMap.get("apple"));
}
```

断言将通过，因为键`apple`在映射中，并且`computeIfPresent`方法将根据`BiFunction`更新值。

### 4.7.`merge` 法

`merge`方法**使用 [`BiFunction`](/web/20220716164105/https://www.baeldung.com/java-bifunction-interface) 更新`HashMap`中的键值，如果有这样的键值的话。否则，它将添加一个新的`(key, value)`对，其值被设置为作为该方法的第二个参数提供的`value`。**

所以，让我们来检查这个方法的行为:

```java
@Test
public void givenFruitMap_whenMergeUsed_thenNewEntriesAdded() {
    double defaultValue = 1.25;
    BiFunction<Double, Integer, Double> powFunction = (x1, x2) -> Math.pow(x1, x2);

    fruitMap.merge("apple", defaultValue, (k, v) -> powFunction.apply(v, 2));
    fruitMap.merge("strawberry", defaultValue, (k, v) -> powFunction.apply(v, 2));

    Assertions.assertTrue(fruitMap.containsKey("strawberry"));
    Assertions.assertEquals(Double.valueOf(defaultValue), fruitMap.get("strawberry"));
    Assertions.assertEquals(Double.valueOf(Math.pow(defaultValue, 2)), fruitMap.get("apple"));
}
```

测试首先在键`apple`上执行`merge `方法。它已经在地图上了，所以它的值会改变。它将是我们传递给方法的`defaultValue`参数的平方。

键`strawberry`不在地图上。因此，`merge`方法会将它加上`defaultValue`作为值。

## 5.结论

在本文中，我们描述了几种更新与`HashMap`中的键相关的值的方法。

首先，我们从最常见的方法开始。然后，我们展示了从 Java 8 开始就可用的几种方法。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220716164105/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5)