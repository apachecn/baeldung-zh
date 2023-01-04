# 从 Java 散列表中删除一个条目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap-remove-entry>

## 1.概观

在本文中，我们将讨论从 Java `HashMap`中删除条目的不同方法。

## 2.介绍

`HashMap`用唯一键将条目存储在`(Key, Value)`对中。因此，一个想法是**使用这个键作为标识符从地图上删除一个相关的条目**。

我们可以使用`java.util.Map`接口提供的方法，通过使用键作为输入来删除条目。

### 2.1.使用方法`remove(Object key)`

让我们用一个简单的例子来尝试一下。我们有一张把食物和食物类型联系起来的地图:

```
HashMap<String, String> foodItemTypeMap = new HashMap<>();
foodItemTypeMap.put("Apple", "Fruit");
foodItemTypeMap.put("Grape", "Fruit");
foodItemTypeMap.put("Mango", "Fruit");
foodItemTypeMap.put("Carrot", "Vegetable");
foodItemTypeMap.put("Potato", "Vegetable");
foodItemTypeMap.put("Spinach", "Vegetable");
```

让我们删除带有关键字“Apple”的条目:

```
foodItemTypeMap.remove("Apple");
// Current Map Status: {Potato=Vegetable, Carrot=Vegetable, Grape=Fruit, Mango=Fruit, Spinach=Vegetable}
```

### 2.2.使用方法`remove(Object key, Object value)`

这是第一种方法的变体，接受键和值作为输入。我们使用这种方法是为了防止只有当一个键被映射到一个特定的值时**才删除一个条目。**

在`foodItemTypeMap`中，关键字“葡萄”没有映射到“蔬菜”值。

因此，以下操作不会导致任何更新:

```
foodItemTypeMap.remove("Grape", "Vegetable");
// Current Map Status: {Potato=Vegetable, Carrot=Vegetable, Grape=Fruit, Mango=Fruit, Spinach=Vegetable}
```

现在，让我们探索在`HashMap`中删除条目的其他场景。

## 3.迭代时删除条目

**`HashMap`级不同步**。如果我们试图同时添加或删除一个条目，可能会导致`ConcurrentModificationException`。因此，我们需要**对外同步`remove`操作**。

### 3.1.在外部对象上同步

一种方法是在封装了`HashMap` 的对象上**同步。例如，我们可以使用`java.util.Map `接口的`entrySet()`方法来获取`HashMap`中的`Set`条目。返回的`Set`由关联的`Map.`支持**

**因此，`Set`的任何结构修改都会导致`Map`的更新。**

让我们使用这种方法从`foodItemTypeMap`中删除一个条目:

```
Iterator<Entry<String, String>> iterator = foodItemTypeMap.entrySet().iterator();
while (iterator.hasNext()) {
    if (iterator.next().getKey().equals("Carrot"))
        iterator.remove();
}
```

除非我们使用迭代器自己的方法进行更新，否则可能不支持 map 上的结构修改。正如我们在上面的代码片段中看到的，我们在迭代器对象上调用`remove()`方法，而不是映射。这提供了线程安全的移除操作。

**我们可以在 Java 8 或更高版本中使用`removeIf`操作**获得相同的结果:

```
foodItemTypeMap.entrySet()
  .removeIf(entry -> entry.getKey().equals("Grape"));
```

### 3.2.使用`ConcurrentHashMap<K, V>`

`java.util.concurrent.ConcurrentHashMap`类**提供线程安全操作**。`ConcurrentHashMap` 的迭代器一次只使用一个线程。因此，它们支持并发操作的确定性行为。

我们可以使用`ConcurrencyLevel` `.`指定允许的并发线程操作的数量

让我们使用基本的`remove `方法来删除`ConcurrentHashMap`中的条目:

```
ConcurrentHashMap<String, String> foodItemTypeConcMap = new ConcurrentHashMap<>();
foodItemTypeConcMap.put("Apple", "Fruit");
foodItemTypeConcMap.put("Carrot", "Vegetable");
foodItemTypeConcMap.put("Potato", "Vegetable");

for (Entry<String, String> item : foodItemTypeConcMap.entrySet()) {
    if (item.getKey() != null && item.getKey().equals("Potato")) {
        foodItemTypeConcMap.remove(item.getKey());
    }
} 
```

## 4.结论

我们已经探索了 Java `HashMap`中条目移除的不同场景。如果不迭代，我们可以安全地使用`java.util.Map `接口提供的标准条目移除方法。

如果我们在迭代过程中更新`Map`，那么在封装对象上使用`remove`方法是必要的。此外，我们分析了另一个类`ConcurrentHashMap`，它支持在`Map`上进行线程安全的更新操作。