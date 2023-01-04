# Java Map–key set()与 entrySet()与 values()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-entries-methods>

## 1.概观

在本教程中，我们将讨论 Java 中`Map`接口的三种方法 `keySet()`、`entrySet()`和`values()` 。这些方法分别用于检索一组键、一组键-值映射和值的集合。

## 2.地图初始化

虽然我们可以在任何实现了`Map`接口的类上使用这些方法，比如`[HashMap,](/web/20220525121012/https://www.baeldung.com/java-hashmap) [TreeMap,](/web/20220525121012/https://www.baeldung.com/java-treemap)` 和 `[LinkedHashMap](/web/20220525121012/https://www.baeldung.com/java-linked-hashmap),`，但是这里我们将使用`HashMap`。

让我们创建并初始化一个`HashMap` ，它的键的类型是`String` ，值的类型是`Integer`:

```
Map<String, Integer> map = new HashMap<>();
map.put("one", 1);
map.put("two", 2);
```

## 3.`keySet()`法

**`keySet()`方法返回`Map`中包含的键的`Set`。**

让我们将方法`keySet()`应用于 `Map` ，并将其存储在一个`Set` 变量`actualValues`中:

```
Set<String> actualValues = map.keySet(); 
```

现在，让我们确认返回的`Set` 的大小是 2:

```
assertEquals(2, actualValues.size());
```

此外，我们可以看到返回的`Set`包含了`Map`的键:

```
assertTrue(actualValues.contains("one"));
assertTrue(actualValues.contains("two"));
```

## 4.`entrySet()`法

**`entrySet() `方法返回一组键值映射。**该方法没有任何参数，并且有一个`Map.Entry. `的返回类型`Set`

让我们将方法`entrySet()`应用于 `Map:`

```
Set<Map.Entry<String, Integer>> actualValues = map.entrySet();
```

正如我们所看到的，`actualValues`是`Map.Entry` 对象`.` 的一个`Set`

`Map.Entry` 是一个静态接口，同时保存键和值。在内部，它有两个实现——`AbstractMap.SimpleEntry` 和`AbstractMap.SimpleImmutableEntry`。

和以前一样，让我们确认返回的`Set` 的大小是 2:

```
assertEquals(2, actualValues.size());
```

此外，我们可以看到返回的`Set`包含了`Map`的键值条目:

```
assertTrue(actualValues.contains(new SimpleEntry<>("one", 1)));
assertTrue(actualValues.contains(new SimpleEntry<>("two", 2)));
```

在这里，我们为我们的测试选择了接口`Map.Entry` 的`AbstractMap.SimpleEntry `实现。

## 5.`values()`法

**`values()`方法返回包含在 M `ap`中的值的`Collection`。**该方法没有任何参数，并且有一个返回类型`Collection. `

让我们将方法`values()`应用于 `Map` ，并将其存储在一个`Collection` 变量`actualValues:`中

```
Collection<Integer> actualValues = map.values();
```

现在，让我们验证返回的`Collection:`的大小

```
assertEquals(2, actualValues.size());
```

此外，我们可以看到返回的`Set`包含了`Map`的值:

```
assertTrue(actualValues.contains(1));
assertTrue(actualValues.contains(2));
```

## 6.结论

在本文中，我们已经讨论了`Map`接口的`keySet()`、`entrySet(),`和`values()`方法。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220525121012/https://github.com/eugenp/tutorials/tree/master/core-java-modules/java-collections-maps-3)