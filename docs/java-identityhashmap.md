# Java IdentityHashMap 类及其用例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-identityhashmap>

## 1.概观

在本教程中，我们将学习如何在 Java 中使用`IdentityHashMap`类。我们还将研究它与一般的`HashMap`类有何不同。虽然这个类实现了`Map`、**接口，但是它违反了`Map`接口**的约定。

更详细的文档，我们可以参考 [`IdenityHashMap`](https://web.archive.org/web/20221106214437/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/IdentityHashMap.html) java doc 页面。关于通用`HashMap`类的更多细节，我们可以阅读[Java HashMap 指南](/web/20221106214437/https://www.baeldung.com/java-hashmap)。

## 2.关于`IdentityHashMap`类

这个类实现了`Map`接口。`Map`接口要求在键比较中使用 `equals()`方法。然而，`IdentityHashMap`类违反了那个契约。相反，它**在关键字搜索操作**中使用引用等式(==)。

在搜索操作中，`HashMap`使用 `hashCode()`方法进行散列，而`IdentityHashMap`使用`System.identityHashCode()`方法。它还使用散列表的线性探测技术进行搜索操作。

参考等式、`System.identityHashCode(),`和线性探测技术的使用赋予了`IdentityHashMap` 类更好的性能。

## 3.使用`IdentityHashMap`类

对象构造和方法签名与`HashMap,` 相同，但是由于引用相等，行为有所不同。

### 3.1.创建`IdentityHashMap`对象

我们可以使用默认的构造函数创建它:

```java
IdentityHashMap<String, String> identityHashMap = new IdentityHashMap<>();
```

也可以使用初始预期容量来创建:

```java
IdentityHashMap<Book, String> identityHashMap = new IdentityHashMap<>(10);
```

如果我们不像上面那样指定初始的`expectedCapcity`参数，它将使用 21 作为默认容量。

我们也可以使用另一个地图对象来创建它:

```java
IdentityHashMap<String, String> identityHashMap = new IdentityHashMap<>(otherMap);
```

在这种情况下，它用`otherMap`的条目初始化创建的`identityHashMap`。

### 3.2.添加、检索、更新和删除条目

`put()`方法用于添加一个条目:

```java
identityHashMap.put("title", "Harry Potter and the Goblet of Fire");
identityHashMap.put("author", "J. K. Rowling");
identityHashMap.put("language", "English");
identityHashMap.put("genre", "Fantasy");
```

我们还可以使用`putAll()`方法添加其他地图中的所有条目:

```java
identityHashMap.putAll(otherMap);
```

为了检索值，我们使用了`get()`方法:

```java
String value = identityHashMap.get(key);
```

为了更新一个键的值，我们使用`put()`方法:

```java
String oldTitle = identityHashMap.put("title", "Harry Potter and the Deathly Hallows");
assertEquals("Harry Potter and the Goblet of Fire", oldTitle);
```

在上面的代码片段中，`the put()`方法返回更新后的旧值。第二条语句确保`oldTitle`匹配前面的“title”值。

我们可以使用 `remove()`方法删除一个元素:

```java
identityHashMap.remove("title");
```

### 3.3.遍历所有条目

我们可以使用`entitySet()`方法遍历所有条目:

```java
Set<Map.Entry<String, String>> entries = identityHashMap.entrySet();
for (Map.Entry<String, String> entry: entries) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

我们还可以使用`the keySet()`方法遍历所有条目:

```java
for (String key: identityHashMap.keySet()) {
    System.out.println(key + ": " + identityHashMap.get(key));
}
```

这些迭代器使用一种[快速失效](/web/20221106214437/https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator)机制。如果在迭代过程中修改了地图，就会抛出一个`ConcurrentModificationException`。

### 3.4.其他方法

我们还有不同的方法可以使用，它们的工作方式与其他`Map`对象类似:

*   `clear()`:删除所有条目
*   `containsKey()`:查找地图中是否存在关键字。只有引用是等同的
*   `containsValue()`:查找值是否存在于地图中。只有引用是等同的
*   `keySet()`:返回基于身份的密钥集
*   `size()`:返回条目数
*   `values()`:返回值的集合

### 3.5.支持`Null`键和`Null`值

`IdentityHashMap`允许`null`用于键和值:

```java
IdentityHashMap<String, String> identityHashMap = new IdentityHashMap<>();
identityHashMap.put(null, "Null Key Accepted");
identityHashMap.put("Null Value Accepted", null);
assertEquals("Null Key Accepted", identityHashMap.get(null));
assertEquals(null, identityHashMap.get("Null Value Accepted"));
```

上面的代码片段确保了`null`既是键又是值。

### 3.6.与`IdentityHashMap`的并发

**`IdentityHashMap`不是线程安全**，同`HashMap`。因此，如果我们有多个线程并行访问/修改`IdentityHashMap`条目，我们应该将它们转换成同步图。

我们可以使用`Collections`类获得一个同步的地图:

```java
Map<String, String> synchronizedMap = Collections.synchronizedMap(new IdentityHashMap<String, String>());
```

## 4.引用等式的用法示例

`IdentityHashMap`在`equals()`方法上使用引用等式(==)来搜索/存储/访问键对象。

用四个属性创建的一个`IdentityHashMap`:

```java
IdentityHashMap<String, String> identityHashMap = new IdentityHashMap<>();
identityHashMap.put("title", "Harry Potter and the Goblet of Fire");
identityHashMap.put("author", "J. K. Rowling");
identityHashMap.put("language", "English");
identityHashMap.put("genre", "Fantasy"); 
```

用相同属性创建的另一个`HashMap`:

```java
HashMap<String, String> hashMap = new HashMap<>(identityHashMap);
hashMap.put(new String("genre"), "Drama");
assertEquals(4, hashMap.size()); 
```

当使用新的字符串对象`“` genre”作为关键字时，`HashMap`将其等同于现有的关键字并更新其值。因此，哈希映射的大小仍然是 4。

以下代码片段显示了`IdentityHashMap`的不同表现:

```java
identityHashMap.put(new String("genre"), "Drama");
assertEquals(5, identityHashMap.size()); 
```

`IdentityHashMap`将新的“流派”字符串对象视为新的关键字。因此，它断言 size 为 5。“流派”的两个不同对象作为两个键，以`“`剧情`“`和`“`奇幻`“`作为值。

## 5.可变键

**`IdentityHashMap`允许可变键**。这是这个类的另一个有用的特性。

这里我们将一个简单的`Book`类作为可变对象:

```java
class Book {
    String title;
    int year;

    // other methods including equals, hashCode and toString
}
```

首先，创建了两个`Book`类的可变对象:

```java
Book book1 = new Book("A Passage to India", 1924);
Book book2 = new Book("Invisible Man", 1953); 
```

以下代码显示了使用`HashMap`的可变键的用法:

```java
HashMap<Book, String> hashMap = new HashMap<>(10);
hashMap.put(book1, "A great work of fiction");
hashMap.put(book2, "won the US National Book Award");
book2.year = 1952;
assertEquals(null, hashMap.get(book2));
```

尽管`book2`条目存在于`HashMap`中，但它无法检索其值。因为它已经被修改了，而`equals()`法现在并不等同于被修改的对象。这就是为什么一般的`Map`对象要求不可变对象作为键。

下面的代码片段使用了与`IdentityHashMap`相同的可变键:

```java
IdentityHashMap<Book, String> identityHashMap = new IdentityHashMap<>(10);
identityHashMap.put(book1, "A great work of fiction");
identityHashMap.put(book2, "won the US National Book Award");
book2.year = 1951;
assertEquals("won the US National Book Award", identityHashMap.get(book2));
```

有趣的是，`IdentityHashMap`能够在键对象被修改时检索值。在上面的代码中，`assertEquals`确保再次检索相同的文本。由于引用相等，这是可能的。

## 6.一些使用案例

由于其特性，`IdentiyHashMap`与其他`Map`物体区别开来。然而，它并不用于一般目的，因此我们在使用这个类时需要小心。

它有助于构建特定的框架，包括:

*   维护一组可变对象的代理对象
*   基于对象引用构建快速缓存
*   在内存中保存带有引用的对象图

## 7.结论

在本文中，我们学习了如何使用`IdentityHashMap`，它与一般的`HashMap`有何不同，以及一些用例。

完整的代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221106214437/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5)