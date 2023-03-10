# EnumMap 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enum-map>

## 1。概述

`EnumMap`是一个`[Map](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) `实现，它只把`Enum`作为它的键。

在本教程中，我们将讨论它的属性、常见用例以及何时应该使用它。

## 2。项目设置

假设有一个简单的需求，我们需要将一周中的每一天与当天我们参加的运动对应起来:

```java
Monday     Soccer                         
Tuesday    Basketball                     
Wednesday  Hiking                         
Thursday   Karate 
```

为此，我们可以使用枚举:

```java
public enum DayOfWeek {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

我们很快就会看到这将是我们地图的关键。

## 3。创作

要开始探索`EnumMap`，首先我们需要实例化一个:

```java
EnumMap<DayOfWeek, String> activityMap = new EnumMap<>(DayOfWeek.class);
activityMap.put(DayOfWeek.MONDAY, "Soccer"); 
```

这是我们与更常见的东西的第一个区别，比如`HashMap`。注意，有了`HashMap`，类型参数化就足够了，这意味着我们可以不用`new HashMap<>(). ` **，然而`EnumMap `需要构造函数**中的键类型。

### 3.1。`EnumMap `复制构造函数

还附带了两个复制构造函数。第一部拍另一部`EnumMap`:

```java
EnumMap<DayOfWeek, String> activityMap = new EnumMap<>(DayOfWeek.class);
activityMap.put(DayOfWeek.MONDAY, "Soccer");
activityMap.put(DayOfWeek.TUESDAY, "Basketball");

EnumMap<DayOfWeek, String> activityMapCopy = new EnumMap<>(dayMap);
assertThat(activityMapCopy.size()).isEqualTo(2);
assertThat(activityMapCopy.get(DayOfWeek.MONDAY)).isEqualTo("Soccer");
assertThat(activityMapCopy.get(DayOfWeek.TUESDAY)).isEqualTo("Basketball");
```

### 3.2.`Map `复制构造函数

或者，**如果我们有一个非空的`Map`，它的键是一个枚举，那么我们也可以这样做:**

```java
Map<DayOfWeek, String> ordinaryMap = new HashMap();
ordinaryMap.put(DayOfWeek.MONDAY, "Soccer");

EnumMap enumMap = new EnumMap(ordinaryMap);
assertThat(enumMap.size()).isEqualTo(1);
assertThat(enumMap.get(DayOfWeek.MONDAY)).isEqualTo("Soccer");
```

**注意，映射必须非空，以便`EnumMap `可以从现有条目中确定键类型。**

如果指定的映射包含多个枚举类型，构造函数将抛出`ClassCastException`。

## 4。添加和检索元素

在实例化一个`EnumMap`之后，我们可以使用 [`put()`](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumMap.html#put(K,V)) 方法添加我们的运动:

```java
activityMap.put(DayOfWeek.MONDAY, "Soccer");
```

而要检索它，我们可以用 [`get()`](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#get(java.lang.Object)) :

```java
assertThat(clubMap.get(DayOfWeek.MONDAY)).isEqualTo("Soccer");
```

## 5。检查元素

为了检查我们是否为某一天定义了映射，我们使用 [`containsKey()`](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumMap.html#containsKey(java.lang.Object)) :

```java
activityMap.put(DayOfWeek.WEDNESDAY, "Hiking");
assertThat(activityMap.containsKey(DayOfWeek.WEDNESDAY)).isTrue();
```

并且，为了检查一个特定的运动是否映射到任何键，我们使用 [`containsValue()`](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumMap.html#containsValue(java.lang.Object)) :

```java
assertThat(activityMap.containsValue("Hiking")).isTrue(); 
```

### 5.1.`null`作为值

**现在，`null`是`EnumMap`的语义有效值。**

让我们把`null` 和“无所事事”联系起来，映射到周六:

```java
assertThat(activityMap.containsKey(DayOfWeek.SATURDAY)).isFalse();
assertThat(activityMap.containsValue(null)).isFalse();
activityMap.put(DayOfWeek.SATURDAY, null);
assertThat(activityMap.containsKey(DayOfWeek.SATURDAY)).isTrue();
assertThat(activityMap.containsValue(null)).isTrue();
```

## 6。移除元件

为了揭示特定的一天，我们简单地把它:

```java
activityMap.put(DayOfWeek.MONDAY, "Soccer");
assertThat(activityMap.remove(DayOfWeek.MONDAY)).isEqualTo("Soccer");
assertThat(activityMap.containsKey(DayOfWeek.MONDAY)).isFalse(); 
```

**正如我们所观察到的， [`remove(key)`](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumMap.html#remove(java.lang.Object)) 返回与键相关联的前一个值，如果键没有映射，则返回`null `。**

我们还可以选择**取消特定日期的映射`only if`该日期被映射到特定活动:**

```java
activityMap.put(DayOfWeek.Monday, "Soccer");
assertThat(activityMap.remove(DayOfWeek.Monday, "Hiking")).isEqualTo(false);
assertThat(activityMap.remove(DayOfWeek.Monday, "Soccer")).isEqualTo(true); 
```

[`remove(key, value)`](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#remove(java.lang.Object,java.lang.Object)) 仅当指定键当前映射到指定值时，删除该键的条目。

## 7。收藏视图

就像普通地图一样，使用任何`EnumMap`，我们可以有 3 个不同的视图或子集合。

首先，让我们创建一个新的活动地图:

```java
EnumMap<DayOfWeek, String> activityMap = new EnumMap(DayOfWeek.class);
activityMap.put(DayOfWeek.THURSDAY, "Karate");
activityMap.put(DayOfWeek.WEDNESDAY, "Hiking");
activityMap.put(DayOfWeek.MONDAY, "Soccer");
```

### 7.1.`values`

我们的活动地图的第一个视图是`[values()](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumMap.html#values()) `，顾名思义，它返回地图中的所有值:

```java
Collection values = dayMap.values();
assertThat(values)
  .containsExactly("Soccer", "Hiking", "Karate"); 
```

**注意这里的`EnumMap `是一个有序的地图。它使用`DayOfWeek `枚举的顺序来确定条目的顺序。**

### 7.2.`keySet`

类似地，`[keySet()](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumMap.html#keySet())`返回键的集合，同样按照枚举顺序:

```java
Set keys = dayMap.keySet();
assertThat(keys)
        .containsExactly(DayOfWeek.MONDAY, DayOfWeek.WEDNESDAY, DayOfWeek.SATURDAY); 
```

### 7.3.`entrySet`

最后，`[entrySet()](https://web.archive.org/web/20220712135027/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumMap.html#entrySet())`返回成对的键和值的映射:

```java
assertThat(dayMap.entrySet())
    .containsExactly(
        new SimpleEntry(DayOfWeek.MONDAY, "Soccer"),
        new SimpleEntry(DayOfWeek.WEDNESDAY, "Hiking"),
        new SimpleEntry(DayOfWeek.THURSDAY, "Karate")
    ); 
```

在地图中排序肯定会很方便，我们在教程中会更深入地将 TreeMap 和 HashMap 进行比较。

### 7.4.易变性

**现在，请记住，我们在原始活动地图中所做的任何更改都将反映在它的任何视图中:**

```java
activityMap.put(DayOfWeek.TUESDAY, "Basketball");
assertThat(values)
    .containsExactly("Soccer", "Basketball", "Hiking", "Karate"); 
```

反之亦然；我们对子视图所做的任何更改都将反映在原始活动图中:

```java
values.remove("Hiking");
assertThat(activityMap.containsKey(DayOfWeek.WEDNESDAY)).isFalse();
assertThat(activityMap.size()).isEqualTo(3); 
```

根据`EnumMap`与`Map `接口的合同，子视图由原始地图支持。

## 8。何时使用`EnumMap`

### 8.1.表演

使用`Enum`作为键可以进行一些额外的性能优化，**比如更快的哈希计算，因为所有可能的键都是预先知道的。**

将`enum`作为键的简单性意味着`EnumMap`只需要由一个普通的老式 Java `Array` 支持，具有非常简单的存储和检索逻辑。另一方面，泛型`Map `实现需要考虑到将一个泛型对象作为键的相关问题。例如， [`HashMap `](/web/20220712135027/https://www.baeldung.com/java-hashmap) [需要一个复杂的数据结构和一个复杂得多的存储和检索逻辑](/web/20220712135027/https://www.baeldung.com/java-hashmap)来应对哈希冲突的可能性。

### 8.2.功能

此外，正如我们所见，`EnumMap `是一个有序映射，因为它的视图将以枚举顺序迭代。为了在更复杂的场景中获得类似的行为，我们可以看看`TreeMap`或`LinkedHashMap`。

## 9.结论

在本文中，我们探索了`Map`接口的`EnumMap`实现。当以`Enum `为键工作时，`EnumMap `可以派上用场。

本教程中使用的所有示例的完整源代码可以在 GitHub 项目中找到。