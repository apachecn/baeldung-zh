# Eclipse 集合简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/eclipse-collections>

## 1。概述

Eclipse Collections 是另一个改进的 Java 集合框架。

简单地说，它提供了优化的实现以及一些核心 Java 中没有的额外的数据结构和特性。

该库提供了所有数据结构的可变和不可变实现。

## 2。Maven 依赖关系

让我们从向我们的`pom.xml`添加以下 Maven 依赖项开始:

```java
<dependency
    <groupId>org.eclipse.collections</groupId>
    <artifactId>eclipse-collections</artifactId>
    <version>8.2.0</version>
</dependency>
```

我们可以在 [Maven 中央存储库](https://web.archive.org/web/20220707143819/https://mvnrepository.com/artifact/org.eclipse.collections/eclipse-collections)中找到该库的最新版本。

## 3。大局

### 3.1。基本收藏类型

Eclipse 集合中的基本集合类型有:

*   **`ListIterable`**–保持插入顺序并允许重复元素的有序集合。子接口包括:`MutableList`、`FixedSizeList`和`ImmutableList`。最常见的`ListIterable implementation is FastList, which is a subclass of MutableList`
*   **`SetIterable`**–不允许重复元素的集合。可以排序，也可以不排序。子接口包括:*已排序可更改*和*未排序可更改。*最常见的未排序*集合可迭代*实现是*统一集合*
*   **`MapIterable`**–键/值对的集合。子接口包括`MutableMap`、`FixedSizeMap`和`ImmutableMap`。两种常见的实现是`UnifiedMap`和`MutableSortedMap`。虽然`UnifiedMap`不保持任何顺序，但是`MutableSortedMap`保持元素的自然顺序
*   **`BiMap`**–可以双向迭代的键/值对的集合。`BiMap`扩展了`MapIterable`接口
*   **`Bag`**–允许重复的无序集合。子接口包括`MutableBag and` `FixedSizeBag`。最常见的实现是`HashBag`
*   **`StackIterable`**–保持“后进先出”顺序的集合，以反向插入顺序遍历元素。子接口包括`MutableStack` 和`ImmutableStack`
*   **`MultiMap`**–允许每个键有多个值的键/值对的集合

### 3.2。原始集合

**框架还提供了一个庞大的原语集合集合**；它们的实现是根据它们持有的类型命名的。每种类型都有可变的、不可变的、同步的和不可修改的形式:

*   原语`Lists`
*   原语`Sets`
*   原语`Stacks`
*   原语`Bags`
*   原语`Maps`
*   `IntInterval`

有大量的图元映射表，涵盖了图元或对象键以及图元或对象值的所有可能组合。

快速注意——`IntInterval`是一个整数范围，可以使用步长值进行迭代。

## 4。实例化集合

为了向`ArrayList`或`HashSet`添加元素，我们通过调用无参数构造函数实例化一个集合，然后逐个添加每个元素。

虽然我们仍然可以在 Eclipse 集合中这样做，但是我们也可以实例化一个集合，并且在一行中同时提供所有的初始元素。

让我们看看如何实例化一个`FastList`:

```java
MutableList<String> list = FastList.newListWith(
  "Porsche", "Volkswagen", "Toyota", "Mercedes", "Toyota");
```

类似地，我们可以实例化一个`UnifiedSet`，并通过将元素传递给`newSetWith()`静态方法向其添加元素:

```java
Set<String> comparison = UnifiedSet.newSetWith(
  "Porsche", "Volkswagen", "Toyota", "Mercedes");
```

下面是我们如何实例化一个`HashBag`:

```java
MutableBag<String> bag = HashBag.newBagWith(
  "Porsche", "Volkswagen", "Toyota", "Porsche", "Mercedes");
```

实例化映射并向其添加键和值对是类似的。唯一的区别是我们将键和值对作为`Pair`接口的实现传递给`newMapWith()`方法。

我们以`UnifiedMap`为例:

```java
Pair<Integer, String> pair1 = Tuples.pair(1, "One");
Pair<Integer, String> pair2 = Tuples.pair(2, "Two");
Pair<Integer, String> pair3 = Tuples.pair(3, "Three");

UnifiedMap<Integer, String> map = new UnifiedMap<>(pair1, pair2, pair3);
```

我们仍然可以使用 Java 集合 API 方法:

```java
UnifiedMap<Integer, String> map = new UnifiedMap<>();

map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
```

由于**不可变集合不能被修改，它们没有修改集合**的方法的实现，比如`add()`和`remove()`。

**然而，不可修改的集合允许我们调用这些方法，但是如果我们这样做了，就会抛出一个`UnsupportedOperationException`。**

## 5。从集合中检索元素

就像使用标准的`Lists`一样，Eclipse 集合`Lists`的元素可以通过它们的索引来检索:

```java
list.get(0);
```

和 Eclipse 集合映射的值可以使用它们的键来检索:

```java
map.get(0);
```

`getFirst()`和`getLast()`方法可以分别用于检索列表的第一个和最后一个元素。对于其他集合，它们返回迭代器返回的第一个和最后一个元素。

```java
map.getFirst();
map.getLast();
```

方法`max()`和`min()`可用于根据自然排序获得集合的最大值和最小值。

```java
map.max();
map.min();
```

## 6。迭代一个集合

Eclipse 集合提供了许多迭代集合的方法。让我们看看它们是什么，以及它们在实践中是如何工作的。

### 6.1。集合过滤

select 模式返回一个新集合，其中包含满足逻辑条件的集合元素。它本质上是一个过滤操作。

这里有一个例子:

```java
@Test
public void givenListwhenSelect_thenCorrect() {
    MutableList<Integer> greaterThanThirty = list
      .select(Predicates.greaterThan(30))
      .sortThis();

    Assertions.assertThat(greaterThanThirty)
      .containsExactly(31, 38, 41);
}
```

同样的事情可以用一个简单的 lambda 表达式来完成:

```java
return list.select(i -> i > 30)
  .sortThis();
```

剔除模式则相反。它返回不满足逻辑条件的所有元素的集合。

让我们看一个例子:

```java
@Test
public void whenReject_thenCorrect() {
    MutableList<Integer> notGreaterThanThirty = list
      .reject(Predicates.greaterThan(30))
      .sortThis();

    Assertions.assertThat(notGreaterThanThirty)
      .containsExactlyElementsOf(this.expectedList);
}
```

这里，我们拒绝所有大于 30 的元素。

### 6.2。`collect()`法

`collect`方法返回一个新的集合，其元素是所提供的 lambda 表达式返回的结果——本质上它是来自 Stream API 的`map()`和`collect()`的组合。

让我们来看看它的实际应用:

```java
@Test
public void whenCollect_thenCorrect() {
    Student student1 = new Student("John", "Hopkins");
    Student student2 = new Student("George", "Adams");

    MutableList<Student> students = FastList
      .newListWith(student1, student2);

    MutableList<String> lastNames = students
      .collect(Student::getLastName);

    Assertions.assertThat(lastNames)
      .containsExactly("Hopkins", "Adams");
}
```

创建的集合`lastNames`包含从`students`列表中收集的姓氏。

但是，**如果返回的集合是集合的集合，而我们不想维护嵌套结构，该怎么办？**

例如，如果每个学生有多个地址，我们需要一个包含地址的集合作为`Strings`而不是集合的集合，我们可以使用`flatCollect()`方法。

这里有一个例子:

```java
@Test
public void whenFlatCollect_thenCorrect() {
    MutableList<String> addresses = students
      .flatCollect(Student::getAddresses);

    Assertions.assertThat(addresses)
      .containsExactlyElementsOf(this.expectedAddresses);
}
```

### 6.3。元素检测

**`detect`方法查找并返回满足逻辑条件的第一个元素。**

让我们看一个简单的例子:

```java
@Test
public void whenDetect_thenCorrect() {
    Integer result = list.detect(Predicates.greaterThan(30));

    Assertions.assertThat(result)
      .isEqualTo(41);
}
```

**`anySatisfy`方法确定集合中的任何元素是否满足逻辑条件。**

这里有一个例子:

```java
@Test
public void whenAnySatisfiesCondition_thenCorrect() {
    boolean result = list.anySatisfy(Predicates.greaterThan(30));

    assertTrue(result);
}
```

**类似地，`allSatisfy`方法确定集合中的所有元素是否满足逻辑条件。**

让我们看一个简单的例子:

```java
@Test
public void whenAnySatisfiesCondition_thenCorrect() {
    boolean result = list.allSatisfy(Predicates.greaterThan(0));

    assertTrue(result);
}
```

### 6.4。`partition()`法

`partition`方法根据元素是否满足逻辑条件，将集合中的每个元素分配到两个集合中的一个。

让我们看一个例子:

```java
@Test
public void whenAnySatisfiesCondition_thenCorrect() {
    MutableList<Integer> numbers = list;
    PartitionMutableList<Integer> partitionedFolks = numbers
      .partition(i -> i > 30);

    MutableList<Integer> greaterThanThirty = partitionedFolks
      .getSelected()
      .sortThis();
    MutableList<Integer> smallerThanThirty = partitionedFolks
      .getRejected()
      .sortThis();

    Assertions.assertThat(smallerThanThirty)
      .containsExactly(1, 5, 8, 17, 23);
    Assertions.assertThat(greaterThanThirty)
      .containsExactly(31, 38, 41);
}
```

### 6.5。懒惰迭代

惰性迭代是一种优化模式，在这种模式下，迭代方法被调用，但它的实际执行被推迟，直到另一个后续方法需要它的操作或返回值。

```java
@Test
public void whenLazyIteration_thenCorrect() {
    Student student1 = new Student("John", "Hopkins");
    Student student2 = new Student("George", "Adams");
    Student student3 = new Student("Jennifer", "Rodriguez");

    MutableList<Student> students = Lists.mutable
      .with(student1, student2, student3);
    LazyIterable<Student> lazyStudents = students.asLazy();
    LazyIterable<String> lastNames = lazyStudents
      .collect(Student::getLastName);

    Assertions.assertThat(lastNames)
      .containsAll(Lists.mutable.with("Hopkins", "Adams", "Rodriguez"));
}
```

这里，`lazyStudents`对象并不检索`students`列表的元素，直到`collect()`方法被调用。

## 7。配对收集元素

方法`zip()`通过将两个集合的元素组合成对来返回一个新的集合。如果两个集合中的任何一个更长，剩余的元素将被截断。

让我们看看如何使用它:

```java
@Test
public void whenZip_thenCorrect() {
    MutableList<String> numbers = Lists.mutable
      .with("1", "2", "3", "Ignored");
    MutableList<String> cars = Lists.mutable
      .with("Porsche", "Volvo", "Toyota");
    MutableList<Pair<String, String>> pairs = numbers.zip(cars);

    Assertions.assertThat(pairs)
      .containsExactlyElementsOf(this.expectedPairs);
}
```

我们还可以使用`zipWithIndex()`方法将集合的元素与其索引配对:

```java
@Test
public void whenZip_thenCorrect() {
    MutableList<String> cars = FastList
      .newListWith("Porsche", "Volvo", "Toyota");
    MutableList<Pair<String, Integer>> pairs = cars.zipWithIndex();

    Assertions.assertThat(pairs)
      .containsExactlyElementsOf(this.expectedPairs);
}
```

## 8。转换收藏

Eclipse 集合提供了将一种容器类型转换成另一种容器类型的简单方法。这些方法是`toList()`、`toSet()`、`toBag()`和`toMap().`

让我们看看如何使用它们:

```java
public static List convertToList() {
    UnifiedSet<String> cars = new UnifiedSet<>();

    cars.add("Toyota");
    cars.add("Mercedes");
    cars.add("Volkswagen");

    return cars.toList();
}
```

让我们运行我们的测试:

```java
@Test
public void whenConvertContainerToAnother_thenCorrect() {
    MutableList<String> cars = (MutableList) ConvertContainerToAnother 
      .convertToList();

    Assertions.assertThat(cars)
      .containsExactlyElementsOf(
      FastList.newListWith("Volkswagen", "Toyota", "Mercedes"));
}
```

## 9。结论

在本教程中，我们已经快速了解了 Eclipse 集合及其提供的特性。

本教程的完整实现可在 GitHub 上的[处获得。](https://web.archive.org/web/20220707143819/https://github.com/eugenp/tutorials/tree/master/libraries-4)