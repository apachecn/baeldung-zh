# 番石榴收藏食谱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-collections>

## 1。简介

这篇烹饪书文章被组织成**小而集中的食谱和代码片段**用于使用番石榴风格的集合。

这种格式是**的一个不断增长的代码示例列表**，不需要额外的解释——这意味着在开发过程中保持 API 的常见用法易于访问。

## 2。菜谱

**向下列表<父>向列表<子>**

–**注意**:这是 Java中非协变泛型集合的变通方法

```java
class CastFunction<F, T extends F> implements Function<F, T> {
    @Override
    public final T apply(final F from) {
        return (T) from;
    }
}
List<TypeParent> originalList = Lists.newArrayList();
List<TypeChild> theList = Lists.transform(originalList, 
    new CastFunction<TypeParent, TypeChild>());
```

**不含番石榴的更简单替代方案——包括两次铸造操作**

```java
List<Number> originalList = Lists.newArrayList();
List<Integer> theList = (List<Integer>) (List<? extends Number>) originalList;
```

**向集合添加可迭代对象**

```java
Iterable<String> iter = Lists.newArrayList();
Collection<String> collector = Lists.newArrayList();
Iterables.addAll(collector, iter);
```

**根据自定义匹配规则检查集合是否包含元素** 

```java
Iterable<String> theCollection = Lists.newArrayList("a", "bc", "def");
    boolean contains = Iterables.any(theCollection, new Predicate<String>() {
    @Override
    public boolean apply(final String input) {
        return input.length() == 1;
    }
});
assertTrue(contains);
```

**使用搜索的替代解决方案**

```java
Iterable<String> theCollection = Sets.newHashSet("a", "bc", "def");
boolean contains = Iterables.find(theCollection, new Predicate<String>() {
    @Override
    public boolean apply(final String input) {
       return input.length() == 1;
    }
}) != null;
assertTrue(contains);
```

**替代解决方案仅适用于器械包**

```java
Set<String> theCollection = Sets.newHashSet("a", "bc", "def");
boolean contains = !Sets.filter(theCollection, new Predicate<String>() {
    @Override
    public boolean apply(final String input) {
        return input.length() == 1;
    }
}).isEmpty();
assertTrue(contains);
```

**`NoSuchElementException`上`Iterables.find`下**

```java
Iterable<String> theCollection = Sets.newHashSet("abcd", "efgh", "ijkl");
Predicate<String> inputOfLengthOne = new Predicate<String>() {
    @Override
    public boolean apply(final String input) {
        return input.length() == 1;
    }
};
String found = Iterables.find(theCollection, inputOfLengthOne);
```

–这将抛出**异常`NoSuchElementException`**:

```java
java.util.NoSuchElementException
	at com.google.common.collect.AbstractIterator.next(AbstractIterator.java:154)
	at com.google.common.collect.Iterators.find(Iterators.java:712)
	at com.google.common.collect.Iterables.find(Iterables.java:643)
```

–**解决方案**:有一个**重载的`find` 方法**，它将默认返回值作为一个参数，可以用`null`调用它以获得想要的行为:

```java
String found = Iterables.find(theCollection, inputOfLengthOne, null);
```

**从集合中删除所有空值**

```java
List<String> values = Lists.newArrayList("a", null, "b", "c");
Iterable<String> withoutNulls = Iterables.filter(values, Predicates.notNull());
```

**直接创建不可变列表/集合/映射**

```java
ImmutableList<String> immutableList = ImmutableList.of("a", "b", "c");
ImmutableSet<String> immutableSet = ImmutableSet.of("a", "b", "c");
ImmutableMap<String, String> imuttableMap = 
    ImmutableMap.of("k1", "v1", "k2", "v2", "k3", "v3");
```

**从标准集合中创建不可变列表/集合/映射**

```java
List<String> muttableList = Lists.newArrayList();
ImmutableList<String> immutableList = ImmutableList.copyOf(muttableList);

Set<String> muttableSet = Sets.newHashSet();
ImmutableSet<String> immutableSet = ImmutableSet.copyOf(muttableSet);

Map<String, String> muttableMap = Maps.newHashMap();
ImmutableMap<String, String> imuttableMap = ImmutableMap.copyOf(muttableMap);
```

**使用建筑商的替代解决方案**

```java
List<String> muttableList = Lists.newArrayList();
ImmutableList<String> immutableList = 
    ImmutableList.<String> builder().addAll(muttableList).build();

Set<String> muttableSet = Sets.newHashSet();
ImmutableSet<String> immutableSet = 
    ImmutableSet.<String> builder().addAll(muttableSet).build();

Map<String, String> muttableMap = Maps.newHashMap();
ImmutableMap<String, String> imuttableMap = 
    ImmutableMap.<String, String> builder().putAll(muttableMap).build();
```

## 3。更多番石榴食谱

番石榴是一个全面且非常有用的库——以下是烹饪书形式的更多 API:

*   #### [Guava ordering recipe](/web/20221126225347/https://www.baeldung.com/guava-order "The Ordering Cookbook")

*   #### [Guava functional recipe](/web/20221126225347/https://www.baeldung.com/guava-functions-predicates "Guava Functional Cookbook")

享受吧。

## 4。向前发展

正如我在开始时提到的，我正在尝试这种**不同的格式——烹饪书**——试图在一个地方收集使用番石榴的简单的共同任务。这种格式的重点是简单和快速，所以大多数菜谱除了代码示例本身之外没有其他解释**。**

最后——我把这看作是一个活的文档——我会不断添加食谱和例子。欢迎在评论中提供更多信息，我会把它们整合到食谱中。

所有这些例子和代码片段的实现**可以在 GitHub** 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20221126225347/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections)