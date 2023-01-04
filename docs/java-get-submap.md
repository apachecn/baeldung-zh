# 从 Java 的 HashMap 中获取一个 Submap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-submap>

## 1.概观

在我们之前的教程中，[Java 散列表](/web/20220626081700/https://www.baeldung.com/java-hashmap)指南，我们展示了如何在 Java 中使用`HashMap`。

在这个简短的教程中，我们将学习**如何基于一个键列表从** `**HashMap**`中获取一个 submap。

## 2.使用 Java 8 流

例如，假设我们有一个`HashMap`和一个键列表:

```
Map<Integer, String> map = new HashMap<>();
map.put(1, "A");
map.put(2, "B");
map.put(3, "C");
map.put(4, "D");
map.put(5, "E");

List<Integer> keyList = Arrays.asList(1, 2, 3);
```

我们可以使用 Java 8 流来获得基于`keyList`的 submap:

```
Map<Integer, String> subMap = map.entrySet().stream()
  .filter(x -> keyList.contains(x.getKey()))
  .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));

System.out.println(subMap);
```

输出将如下所示:

```
{1=A, 2=B, 3=C}
```

## 3.使用`retainAll()`方法

我们可以获取地图的`keySet`并使用`retainAll()`方法删除所有键不在`keyList`中的条目:

```
map.keySet().retainAll(keyList);
```

**注意这个方法会编辑原来的地图**。如果我们不想影响原始地图，我们可以首先使用`HashMap`的复制构造函数创建一个新地图:

```
Map<Integer, String> newMap = new HashMap<>(map);
newMap.keySet().retainAll(keyList);

System.out.println(newMap);
```

输出和上面一样。

## 4.结论

总之，我们已经学习了两种方法来从基于键列表的 `**HashMap**`中获取 submap。