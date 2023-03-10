# 在 Java 中复制散列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-copy-hashmap>

## 1.概观

在本教程中，我们将探索一个`[HashMap](/web/20221205221039/https://www.baeldung.com/java-hashmap)` 的浅层与[深层拷贝](/web/20221205221039/https://www.baeldung.com/java-deep-copy)的概念，以及在 Java 中拷贝一个`HashMap`的几种技术。

我们还将考虑一些在特定情况下可以帮助我们的外部库。

## 2.浅层拷贝与深层拷贝

首先让我们了解一下`HashMaps`中浅副本和深副本的概念。

### 2.1.浅拷贝

**一个`HashMap`的浅层副本是一个新的`HashMap`，它映射到与原始`HashMap`相同的键和值对象。**

例如，我们将创建一个`Employee`类，然后用`Employee`实例作为值创建一个映射:

```java
public class Employee {
    private String name;

    // constructor, getters and setters
} 
```

```java
HashMap<String, Employee> map = new HashMap<>();
Employee emp1 = new Employee("John");
Employee emp2 = new Employee("Norman");
map.put("emp1", emp1);
map.put("emp2", emp2); 
```

现在，我们将验证原始贴图及其浅层副本是不同的对象:

```java
HashMap<String, Employee> shallowCopy = // shallow copy implementation
assertThat(shallowCopy).isNotSameAs(map);
```

因为这是一个浅层拷贝，如果我们改变一个`Employee`实例的属性，它将影响原始地图和它的浅层拷贝:

```java
emp1.setFirstName("Johny");
assertThat(shallowCopy.get("emp1")).isEqualTo(map.get("emp1"));
```

### 2.2.深层拷贝

**一个`HashMap`的深层拷贝是一个新的`HashMap`，它深层拷贝了所有的映射。**因此，它为所有的键、值和映射创建新的对象。

这里，显式修改映射(键值)不会影响深层副本:

```java
HashMap<String, Employee> deepCopy = // deep copy implementation

emp1.setFirstName("Johny");

assertThat(deepCopy.get("emp1")).isNotEqualTo(map.get("emp1")); 
```

## 3\. `HashMap` API

### 3.1.使用`HashMap` **C** 构造器

`HashMap`的参数化构造器`HashMap(Map<? extends K,? extends V> m)` **提供了一种快速浅拷贝整张地图的方法:**

```java
HashMap<String, Employee> shallowCopy = new HashMap<String, Employee>(originalMap); 
```

### 3.2.使用`Map.clone()`

与构造函数类似，`HashMap` # `clone`方法也创建一个快速的浅层拷贝:

```java
HashMap<String, Employee> shallowCopy = originalMap.clone(); 
```

### 3.3.使用`Map.put()`

通过迭代每个条目并在另一个 map 上调用`put()`方法，可以很容易地对`HashMap`进行浅层复制:

```java
HashMap<String, Employee> shallowCopy = new HashMap<String, Employee>();
Set<Entry<String, Employee>> entries = originalMap.entrySet();
for (Map.Entry<String, Employee> mapEntry : entries) {
    shallowCopy.put(mapEntry.getKey(), mapEntry.getValue());
} 
```

### 3.4.使用`Map.putAll()`

我们可以使用`putAll()`方法，而不是遍历所有的条目，该方法在一个步骤中浅层复制所有的映射:

```java
HashMap<String, Employee> shallowCopy = new HashMap<>();
shallowCopy.putAll(originalMap); 
```

我们应该注意，如果有匹配的键，那么 **`put()` 和`putAll()`会替换这些值。**

有趣的是，如果我们看一下`HashMap`的构造函数、`clone()`和`putAll()`的实现，我们会发现它们都使用相同的内部方法来复制条目— `putMapEntries()`。

## 4.使用 Java 8 `Stream` API 复制`**HashMap**`

我们可以使用 [Java 8 `Stream` API](/web/20221205221039/https://www.baeldung.com/java-8-streams) 来创建一个`HashMap`的浅层副本:

```java
Set<Entry<String, Employee>> entries = originalMap.entrySet();
HashMap<String, Employee> shallowCopy = (HashMap<String, Employee>) entries.stream()
  .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue)); 
```

## 5.谷歌番石榴

使用[番石榴地图，](/web/20221205221039/https://www.baeldung.com/guava-maps)我们可以很容易地创建不可变的地图，以及排序和 bi 地图。要制作这些地图的不可变的浅层副本，我们可以使用`copyOf`方法:

```java
Map<String, Employee> map = ImmutableMap.<String, Employee>builder()
  .put("emp1",emp1)
  .put("emp2",emp2)
  .build();
Map<String, Employee> shallowCopy = ImmutableMap.copyOf(map);

assertThat(shallowCopy).isSameAs(map);
```

## 6.Apache Commons Lang

**现在，Java 没有任何内置的深度复制实现。**因此，为了进行深度复制，我们可以覆盖`clone()`方法，或者使用序列化-反序列化技术。

Apache Commons 已经用一个`clone()`方法创建了一个深层副本。为此，深度复制中包含的任何类都必须实现`Serializable`接口:

```java
public class Employee implements Serializable {
    // implementation details
}

HashMap<String, Employee> deepCopy = SerializationUtils.clone(originalMap);
```

## 7.结论

在这个快速教程中，我们看到了在 Java 中复制一个`HashMap`的各种技术，以及针对`HashMap` s 的浅层和深层复制的概念。

此外，我们探索了一些外部库，它们对于创建浅层和深层副本非常方便。

这些实现的完整源代码以及单元测试可以在 [GitHub](https://web.archive.org/web/20221205221039/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2) 项目中找到。