# Apache Commons 多值地图指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-multi-valued-map>

## 1。概述

在这个快速教程中，我们将看看 Apache Commons Collections 库`. `中提供的 [`MultiValuedMap`](https://web.archive.org/web/20221208143854/https://commons.apache.org/proper/commons-collections/apidocs/index.html?org/apache/commons/collections4/MultiValuedMap.html) 接口

**`MultiValuedMap`提供了一个简单的 API，用于将 Java 中的每个键映射到一个值集合。**它是`[org.apache.commons.collections4.MultiMap](https://web.archive.org/web/20221208143854/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/MultiMap.html), `的继任者，后者在 Commons Collection 4.1 中已被弃用。

## 2。Maven 依赖关系

对于 Maven 项目，我们需要添加 [`commons-collections4`](https://web.archive.org/web/20221208143854/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22) 依赖项:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.2</version>
</dependency>
```

## 3。将元素添加到`MultiValuedMap`

我们可以使用`put`和`putAll`方法添加元素。

让我们首先创建一个`MultiValuedMap`的实例:

```
MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
```

接下来，让我们看看如何使用`put`方法一次添加一个元素:

```
map.put("fruits", "apple");
map.put("fruits", "orange");
```

此外，让我们使用`putAll`方法添加一些元素，该方法在一次调用中将一个键映射到多个元素:

```
map.putAll("vehicles", Arrays.asList("car", "bike"));
assertThat((Collection<String>) map.get("vehicles"))
  .containsExactly("car", "bike");
```

## 4。 **从`MultiValuedMap`** 中检索元素

`MultiValuedMap`提供检索键、值和键值映射的方法。让我们来看看其中的每一项。

### 4.1.获取一个键的所有值

要获得与一个键相关的所有值，我们可以使用 `get`方法，该方法返回一个 [`Collection`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html) :

```
assertThat((Collection<String>) map.get("fruits"))
  .containsExactly("apple", "orange");
```

### 4.2.获取所有键值映射

或者，我们可以使用`entries`方法得到一个 [`Collection`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html) 的映射中包含的所有键值映射:

```
Collection<Map.Entry<String, String>> entries = map.entries();
```

### 4.3.获取所有密钥

有两种方法可以检索包含在`MultiValuedMap.`中的所有键

让我们用`keys`方法得到一个 [`MultiSet`](https://web.archive.org/web/20221208143854/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/MultiSet.html) 的按键视图:

```
MultiSet<String> keys = map.keys();
assertThat(keys).contains("fruits", "vehicles");
```

或者，我们可以使用`keySet `方法获得键的 [`Set`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html) 视图:

```
Set<String> keys = map.keySet();
assertThat(keys).contains("fruits", "vehicles");
```

### 4.4.获取地图的所有值

最后，如果我们想获得一个 [`Collection`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html) 视图中包含的所有值的映射，我们可以使用 `values`方法:

```
Collection<String> values = map.values();
assertThat(values).contains("apple", "orange", "car", "bike");
```

## 5。**`MultiValuedMap`**去除元素

现在，让我们看看删除元素和键值映射的所有方法。

### 5.1.删除映射到键的所有元素

首先，让我们看看如何使用` remove `方法删除与指定键相关的所有值:

```
Collection<String> removedValues = map.remove("fruits");
assertThat(map.containsKey("fruits")).isFalse();
assertThat(removedValues).contains("apple", "orange");
```

这个方法返回一个 [`Collection`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html) 视图中被删除的值。

### 5.2.移除单个键值映射

现在，假设我们有一个映射到多个值的键，但是我们只想删除其中一个映射值，留下其他的。我们可以使用`removeMapping `方法轻松做到这一点:

```
boolean isRemoved = map.removeMapping("fruits","apple");
assertThat(map.containsMapping("fruits","apple")).isFalse();
```

### 5.3.移除所有键值映射

最后，我们可以使用`clear `方法从映射中删除所有映射:

```
map.clear();
assertThat(map.isEmpty()).isTrue();
```

## 6。正在检查 **元素来自`MultiValuedMap`**

接下来，让我们看看检查指定的键或值是否存在于我们的映射中的各种方法。

### 6.1.检查密钥是否存在

为了找出我们的映射是否包含指定键的映射，我们可以使用`containsKey`方法:

```
assertThat(map.containsKey("vehicles")).isTrue();
```

### 6.2.检查值是否存在

接下来，假设我们想要检查映射中是否至少有一个键包含特定值的映射。我们可以使用`containsValue`方法来做到这一点:

```
assertThat(map.containsValue("orange")).isTrue();
```

### 6.3.检查键值映射是否存在

类似地，如果我们想检查一个映射是否包含特定键和值对的映射，我们可以使用 `containsMapping`方法:

```
assertThat(map.containsMapping("fruits","orange")).isTrue();
```

### 6.4.检查地图是否为空

要检查一个映射是否根本不包含任何键值映射，我们可以使用`isEmpty`方法:

```
assertThat(map.isEmpty()).isFalse;
```

### 6.5.检查地图的大小

最后，我们可以使用`size`方法来获得地图的总大小。当一个映射包含多个值的键时，映射的总大小是所有键中所有值的计数:

```
assertEquals(4, map.size());
```

## 7 .实施

Apache Commons Collections 库也提供了该接口的多种实现。让我们来看看它们。

### 7.1.`ArrayListValuedHashMap`

一个 [`ArrayListValuedHashMap`](https://web.archive.org/web/20221208143854/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/multimap/ArrayListValuedHashMap.html) 在内部使用一个`ArrayList`来存储与每个键相关的值，因此**允许重复的键-值对**:

```
MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
map.put("fruits", "apple");
map.put("fruits", "orange");
map.put("fruits", "orange");
assertThat((Collection<String>) map.get("fruits"))
  .containsExactly("apple", "orange", "orange");
```

现在，值得注意的是这个**类不是线程安全的**。因此，如果我们想在多个线程中使用这个映射，我们必须确保使用正确的同步。

### 7.2.`HashSetValuedHashMap`

一个 [`HashSetValuedHashMap`](https://web.archive.org/web/20221208143854/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/multimap/HashSetValuedHashMap.html) 使用一个`HashSet`来存储每个给定键的值。因此，**不允许重复的键值对**。

让我们看一个简单的例子，我们添加了两次相同的键值映射:

```
MultiValuedMap<String, String> map = new HashSetValuedHashMap<>();
map.put("fruits", "apple");
map.put("fruits", "apple");
assertThat((Collection<String>) map.get("fruits"))
  .containsExactly("apple");
```

注意，与我们之前使用`ArrayListValuedHashMap,`的例子不同，`HashSetValuedHashMap`实现忽略了重复的映射。

`HashSetValuedHashMap` **类也不是线程安全的**。

### 7.3.`UnmodifiableMultiValuedMap`

`UnmodifiableMultiValuedMap`是一个装饰类，当我们需要一个 [`MultiValuedMap`](https://web.archive.org/web/20221208143854/https://commons.apache.org/proper/commons-collections/apidocs/index.html?org/apache/commons/collections4/MultiValuedMap.html) 的不可变实例时很有用——也就是说，它不允许进一步的修改:

```
@Test(expected = UnsupportedOperationException.class)
public void givenUnmodifiableMultiValuedMap_whenInserting_thenThrowingException() {
    MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
    map.put("fruits", "apple");
    map.put("fruits", "orange");
    MultiValuedMap<String, String> immutableMap =
      MultiMapUtils.unmodifiableMultiValuedMap(map);
    immutableMap.put("fruits", "banana"); // throws exception
}
```

同样，值得注意的是，修改最后一个`put`的**将导致一个`UnsupportedOperationException`。**

## 8。结论

我们已经看到了 Apache Commons Collections 库中的各种接口方法。此外，我们还探索了一些流行的实现。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps)