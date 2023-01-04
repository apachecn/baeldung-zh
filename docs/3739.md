# Java 中原语的映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-primitives>

## 1。概述

在本教程中，我们将学习如何用原始键和值构建一个地图。

我们知道，核心 Java [`Map` s](/web/20220701020340/https://www.baeldung.com/java-hashmap) 不允许存储原始键或值。这就是为什么我们将引入一些提供原始地图实现的外部第三方库。

## 2。Eclipse 集合

**[Eclipse Collections](/web/20220701020340/https://www.baeldung.com/eclipse-collections) 是一个高性能的 Java 集合框架**。它提供了改进的实现以及一些额外的数据结构，包括**几个原始集合。**

### 2.1。 `Maps`可变与不可变

让我们创建一个空映射，其中键和值都是原语`int`，为此，我们将使用`IntIntMaps`工厂类:

```
MutableIntIntMap mutableIntIntMap = IntIntMaps.mutable.empty();
```

**`IntIntMaps`工厂类是创建原始地图**最便捷的方式。它允许我们创建所需类型地图的可变和不可变实例。在我们的例子中，我们创建了`IntIntMap`的可变实例。类似地，我们可以通过简单地用`IntIntMaps.immutable`替换`IntIntMaps.mutable`静态工厂调用来创建不可变实例:

```
ImmutableIntIntMap immutableIntIntMap = IntIntMaps.immutable.empty();
```

因此，让我们向可变映射添加一个键/值对:

```
mutableIntIntMap.addToValue(1, 1);
```

同样，**我们可以创建带有引用和原始类型键值对的混合映射。**让我们用`String`键和`double`值创建一个地图:

```
MutableObjectDoubleMap dObject = ObjectDoubleMaps.mutable.empty();
```

这里，我们使用`ObjectDoubleMaps`工厂类为`MutableObjectDoubleMap`创建一个可变实例。

现在让我们添加一些条目:

```
dObject.addToValue("price", 150.5);
dObject.addToValue("quality", 4.4);
dObject.addToValue("stability", 0.8);
```

### 2.2。一个原始的 API 树

在 Eclipse 集合中，有一个名为`PrimitiveIterable.`的基本接口，这是库的每个原始容器的基本接口。全部命名为`PrimitiveTypeIterable`，其中`PrimitiveType`可以是`Int, Long`、`Short`、`Byte`、`Char`、`Float`、`Double`或`Boolean`。

所有这些基本接口依次有它们的`XY` `Map`实现树，根据映射是可变的还是不可变的来划分**。例如，对于`IntIntMap`，我们有`MutableIntIntMap`和`ImmutableIntIntMap`。**

最后，正如我们在上面看到的，我们有**接口来覆盖所有类型的键和值的组合，包括原始值和对象值**。因此，举例来说，我们可以用 `IntObjectMap<K>`表示具有`Object`值的主键，或者用`ObjectIntMap<K> `表示相反的情况。

## 3。HPPC

HPPC 是一个面向高性能和内存效率的库。这意味着该库比其他库抽象性更低。然而，这样做的好处是将内部暴露给有用的低级操作。它提供了地图和集合。

### 3.1.简单的例子

让我们首先创建一个有一个`int`键和一个`long`值的映射。使用这个非常熟悉:

```
IntLongHashMap intLongHashMap = new IntLongHashMap();
intLongHashMap.put(25, 1L);
intLongHashMap.put(150, Long.MAX_VALUE);
intLongHashMap.put(1, 0L);

intLongHashMap.get(150);
```

HPPC 为键和值的所有组合提供映射:

*   原始键和原始值
*   主键值和对象类型值
*   对象类型键和原始值
*   对象类型键和值

对象类型映射支持泛型:

```
IntObjectOpenHashMap<BigDecimal>
ObjectIntOpenHashMap<LocalDate> 
```

第一个映射有一个原始的`int`键和一个`BigDecimal`值。第二个映射的键是`LocalDate `，值是`int`

### 3.2.散列图与散点图

由于密钥散列和分发函数的传统实现方式，我们在散列密钥时可能会发生冲突。根据键的分布方式，这可能会导致大型地图的性能问题。默认情况下，HPPC 实现了一个解决方案来避免这个问题。

然而，仍然有一个具有更简单分布函数的地图的位置。如果映射被用作查找表或用于计数，或者如果它们在加载后不需要大量的写操作，这是很有用的**。HHPC 提供`Scatter Maps`来进一步提升性能。**

所有散点图类都保持与地图相同的命名约定，但是使用单词`Scatter`:

*   `IntScatterSet`
*   `IntIntScatterMap`
*   `IntObjectScatterMap<BigDecimal>`

## 4。Fastutil

**[Fastutil](https://web.archive.org/web/20220701020340/https://search.maven.org/search?q=g:it.unimi.dsi%20a:fastutil) 是一个快速紧凑的框架**，它提供了特定于类型的集合，包括原始类型映射。

### 4.1。快速示例

类似于 Eclipse 集合和 HPPC。Fastutil 还提供了原语到原语和原语到对象类型的关联映射。

让我们创建一个`int`到`boolean`的映射:

```
Int2BooleanMap int2BooleanMap = new Int2BooleanOpenHashMap();
```

现在，让我们添加一些条目:

```
int2BooleanMap.put(1, true);
int2BooleanMap.put(7, false);
int2BooleanMap.put(4, true);
```

然后，我们可以从中检索值:

```
boolean value = int2BooleanMap.get(1);
```

### 4.2。就地迭代

实现`Iterable`接口的标准 JVM 集合通常在每个迭代步骤创建一个新的临时迭代器对象。对于大量收集，这可能会产生垃圾收集问题。

Fastutil 提供了一种替代方案，可以极大地缓解这一问题:

```
Int2FloatMap map = new Int2FloatMap();
//Add keys here
for(Int2FloatMap.Entry e : Fastutil.fastIterable(map)) {
    //e will be reused on each iteration, so it will be only one object
} 
```

Fastutil 还提供了`fastForeach`方法。这将采用一个`Consumer` [函数接口](/web/20220701020340/https://www.baeldung.com/java-8-functional-interfaces)并为每个循环执行一个λ表达式:

```
Int2FloatMap map = new Int2FloatMap();
//Add keys here
Int2FloatMaps.fastForEach(map , e ->  {
    // e is also reused across iterations
}); 
```

这非常类似于标准的 Java `foreach`构造:

```
Int2FloatMap map = new Int2FloatMap();
//Add keys here
map.forEach((key,value) -> {
    // use each key/value entry   
}); 
```

## 5。结论

在本文中，我们学习了如何使用 Eclipse 集合、HPPC 和 Fastutil 在 Java 中**创建原始地图。**

和往常一样，本文的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220701020340/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)