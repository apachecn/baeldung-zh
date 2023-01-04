# 从 HashMap 中获取第一个键和值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap-get-first-entry>

## 1.概观

在本教程中，我们将讨论如何在不知道密钥的情况下从`[HashMap](/web/20221208143854/https://www.baeldung.com/java-hashmap)`中获得第一个密钥-值对。

首先，我们将使用一个迭代器，然后使用一个流来获取第一个条目。最后，我们将讨论当我们想要获得第一个条目时,`HashMap`提出的一个问题以及如何解决它。

## 2.使用迭代器

让我们考虑下面的`HashMap<Integer, String>`:

```
Map<Integer, String> hashMap = new HashMap<>();
hashMap.put(5, "A");
hashMap.put(1, "B");
hashMap.put(2, "C");
```

在这个例子中，我们将使用一个`iterator`来获得第一个键值对。因此，让我们在`HashMap`的`entry set`上创建一个`iterator`，并调用`next()`方法来检索第一个条目:

```
Iterator<Map.Entry<Integer, String>> iterator = hashMap.entrySet().iterator();

Map.Entry<Integer, String> actualValue = iterator.next();
Map.Entry<Integer, String> expectedValue = new AbstractMap.SimpleEntry<Integer, String>(1, "B");

assertEquals(expectedValue, actualValue);
```

## 3.使用 Java 流

另一种方法是使用 Java 流 API。让我们在`entry set`上创建一个流，并调用`findFirst()` 方法来获取它的第一个条目:

```
Map.Entry<Integer, String> actualValue = hashMap.entrySet()
  .stream()
  .findFirst()
  .get(); 
```

```
Map.Entry<Integer, String> expectedValue = new AbstractMap.SimpleEntry<Integer, String>(1, "B");

assertEquals(expectedValue, actualValue);
```

## 4.插入顺序有问题

为了呈现这个问题，让我们记住我们是如何创建`hashMap`的，一对`5=A` 作为第一个条目被插入，然后是`1=B`，最后是`2=C`。让我们通过打印我们的`HashMap`的内容来检查一下:

```
System.out.println(hashMap);
```

```
{1=B, 2=C, 5=A}
```

我们可以看到，排序是不一样的。**`HashMap class` 实现不保证插入顺序**。

现在让我们给`hashMap`再添加一个元素:

```
hashMap.put(0, "D");

Iterator<Map.Entry<Integer, String>> iterator = hashMap.entrySet().iterator();

Map.Entry<Integer, String> actualValue = iterator.next();
Map.Entry<Integer, String> expectedValue = new AbstractMap.SimpleEntry<Integer, String>(0, "D");

assertEquals(expectedValue, actualValue);
```

正如我们所看到的，第一个条目又发生了变化(在本例中为`0=D` )。这也证明了`HashMap`并不能保证一个插入顺序。

所以，**如果我们想保留顺序，我们应该用 [`LinkedHashMap`](/web/20221208143854/https://www.baeldung.com/java-linked-hashmap) 代替**:

```
Map<Integer, String> linkedHashMap = new LinkedHashMap<>();
linkedHashMap.put(5, "A");
linkedHashMap.put(1, "B");
linkedHashMap.put(2, "C");
linkedHashMap.put(0, "D");

Iterator<Map.Entry<Integer, String>> iterator = linkedHashMap.entrySet().iterator();
Map.Entry<Integer, String> actualValue = iterator.next();
Map.Entry<Integer, String> expectedValue = new AbstractMap.SimpleEntry<Integer, String>(5, "A");

assertEquals(expectedValue, actualValue);
```

## 5.结论

在这篇短文中，我们讨论了从`HashMap`获取第一个条目的不同方法。

需要注意的最重要的一点是，`HashMap`实现并不保证任何插入顺序。因此，如果我们对保持插入顺序感兴趣，我们应该使用一个`LinkedHashMap`。

GitHub 上的[提供了代码示例。](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)