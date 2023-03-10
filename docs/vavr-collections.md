# Vavr 中的集合 API 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-collections>

## 1。概述

Vavr 库，以前称为 Javaslang，是 Java 的函数库。在本文中，我们将探索其强大的集合 API。

要获得关于这个库的更多信息，请阅读本文。

## 2。持续收集

持久性集合在被修改时会生成集合的新版本，同时保留当前版本。

维护同一集合的多个版本可能会导致低效的 CPU 和内存使用。然而，Vavr 集合库通过在集合的不同版本之间共享数据结构克服了这一点。

这与 Java 的`unmodifiableCollection()`和`Collections`实用程序类有着本质的不同，后者仅仅提供了一个底层集合的包装器。

试图修改这样的集合会导致`UnsupportedOperationException`而不是创建一个新版本。此外，底层集合通过其直接引用仍然是可变的。

## 3。`Traversable`

是所有 Vavr 集合的基本类型——这个接口定义了所有数据结构共享的方法。

它提供了一些有用的默认方法，如 `size()`、`get()`、`filter()`、`isEmpty()`等，这些方法被子接口继承。

让我们进一步探索收藏库。

## 4。`Seq`

我们从序列开始。

`Seq`接口代表顺序数据结构。它是`List`、`Stream`、`Queue`、`Array`、`Vector`和`CharSeq`的父接口。所有这些数据结构都有自己独特的属性，我们将在下面探讨。

### 4.1。`List`

一个 `List`是扩展了`LinearSeq`接口的元素的急切求值序列。

持久`Lists`由头部和尾部递归形成:

*   头部——第一要素
*   tail–包含剩余元素的列表(该列表也由头部和尾部组成)

在`List` API 中有静态工厂方法可以用来创建`List`。我们可以使用静态的`of()`方法从一个或多个对象中创建一个`List`的实例。

我们也可以使用静态的`empty()`来创建一个空的`List` 和`ofAll()`来从一个`Iterable`类型创建一个`List`:

```java
List<String> list = List.of(
  "Java", "PHP", "Jquery", "JavaScript", "JShell", "JAVA");
```

让我们看一些如何操作列表的例子。

我们可以使用`drop()`及其变体来删除第一个`N`元素:

```java
List list1 = list.drop(2);                                      
assertFalse(list1.contains("Java") && list1.contains("PHP"));   

List list2 = list.dropRight(2);                                 
assertFalse(list2.contains("JAVA") && list2.contains("JShell"));

List list3 = list.dropUntil(s -> s.contains("Shell"));          
assertEquals(list3.size(), 2);                                  

List list4 = list.dropWhile(s -> s.length() > 0);               
assertTrue(list4.isEmpty());
```

`drop(int n)`从列表的第一个元素开始删除`n`个元素，而`dropRight()` 从列表的最后一个元素开始执行相同的操作。

`dropUntil()`继续从列表中删除元素，直到谓词评估为真，而当谓词为真时`dropWhile()`继续删除元素。

还有`dropRightWhile()`和 `dropRightUntil()`开始从右边移除元素。

接下来，`take(int n)`用于从列表中抓取元素。它从列表中取出`n`个元素，然后停止。还有一个`takeRight(int n)`，它从列表末尾开始获取元素:

```java
List list5 = list.take(1);                       
assertEquals(list5.single(), "Java");            

List list6 = list.takeRight(1);                  
assertEquals(list6.single(), "JAVA");            

List list7 = list.takeUntil(s -> s.length() > 6);
assertEquals(list7.size(), 3);
```

最后，`takeUntil()`继续从列表中获取元素，直到谓词为真。还有一个`takeWhile()`变量也接受谓词参数。

此外，API 中还有其他有用的方法，例如，实际上返回非重复元素列表的`distinct()`以及接受`Comparator`来确定相等的`distinctBy()`。

非常有趣的是，还有在列表的每个元素之间插入一个元素的 `intersperse()`。对于`String`操作来说非常方便:

```java
List list8 = list
  .distinctBy((s1, s2) -> s1.startsWith(s2.charAt(0) + "") ? 0 : 1);
assertEquals(list8.size(), 2);

String words = List.of("Boys", "Girls")
  .intersperse("and")
  .reduce((s1, s2) -> s1.concat( " " + s2 ))
  .trim();  
assertEquals(words, "Boys and Girls");
```

想把一个列表分成几类？这也有一个 API:

```java
Iterator<List<String>> iterator = list.grouped(2);
assertEquals(iterator.head().size(), 2);

Map<Boolean, List<String>> map = list.groupBy(e -> e.startsWith("J"));
assertEquals(map.size(), 2);
assertEquals(map.get(false).get().size(), 1);
assertEquals(map.get(true).get().size(), 5);
```

`group(int n)`将一个`List`分成每组`n`个元素的组。`groupdBy()`接受一个包含划分列表逻辑的`Function`，并返回一个包含两个条目的`Map`—`true`和`false`。

`true`键映射到满足`Function;`中指定条件的 `List`元素,`false`键映射到不满足条件的`List`元素。

正如所料，当突变一个`List`时，原来的`List` 实际上并没有被修改。相反，新版本的`List`总是被返回。

我们还可以使用堆栈语义——元素的后进先出(LIFO)检索来与`List`交互。在这个意义上，有一些 API 方法用于操作堆栈，如`peek()`、`pop()`和`push()`:

```java
List<Integer> intList = List.empty();

List<Integer> intList1 = intList.pushAll(List.rangeClosed(5,10));

assertEquals(intList1.peek(), Integer.valueOf(10));

List intList2 = intList1.pop();
assertEquals(intList2.size(), (intList1.size() - 1) );
```

`pushAll()`函数用于将一系列整数插入堆栈，而`peek()`用于获取堆栈的头部。还有可以将结果包装在一个`Option`对象中的`peekOption()`。

在`List`接口中还有其他有趣且非常有用的方法，这些方法在 [Java 文档](https://web.archive.org/web/20221004032439/https://static.javadoc.io/io.vavr/vavr/0.9.0/io/vavr/collection/List.html)中有清晰的记录。

### 4.2。`Queue`

不可变的`Queue`存储允许先进先出(FIFO)检索的元素。

一个`Queue`内部由两个链表组成，前`List`和后`List`。前面的`List`包含出列的元素，后面的`List`包含入队的元素。

这允许在 O(1)中执行`enqueue`和`dequeue`操作。当前`List`元件用完时，前后`List's`互换，后`List`反转。

让我们创建一个队列:

```java
Queue<Integer> queue = Queue.of(1, 2);
Queue<Integer> secondQueue = queue.enqueueAll(List.of(4,5));

assertEquals(3, queue.size());
assertEquals(5, secondQueue.size());

Tuple2<Integer, Queue<Integer>> result = secondQueue.dequeue();
assertEquals(Integer.valueOf(1), result._1);

Queue<Integer> tailQueue = result._2;
assertFalse(tailQueue.contains(secondQueue.get(0)));
```

`dequeue`函数从`Queue`中移除头部元素并返回一个`Tuple2<T, Q>`。元组包含作为第一个条目被移除的头元素和作为第二个条目的`Queue` 的剩余元素。

我们可以使用`combination(n)`来获得`Queue`中元素的所有可能的`N`组合:

```java
Queue<Queue<Integer>> queue1 = queue.combinations(2);
assertEquals(queue1.get(2).toCharSeq(), CharSeq.of("23"));
```

同样，我们可以看到原始的`Queue`在元素入队/出队时没有被修改。

### 4.3。`Stream`

一个`Stream`是一个惰性链表的实现，与`java.util.stream`有很大的不同。与 `java.util.stream`不同，Vavr `Stream`存储数据，并延迟评估下一个元素。

假设我们有一个`Stream`整数:

```java
Stream<Integer> s = Stream.of(2, 1, 3, 4);
```

将`s.toString()`的结果打印到控制台只会显示`Stream(2, ?)`。这意味着只有`Stream`的头部被评估，而尾部没有被评估。

调用`s.get(3)`并随后显示`s.tail()`的结果返回`Stream(1, 3, 4, ?)`。相反，如果不调用第一个 `–`的`s.get(3)`，导致`Stream`对最后一个元素求值，那么`s.tail()`的结果只会是`Stream(1, ?)`。这意味着只计算了尾部的第一个元素。

这种行为可以提高性能，并使使用`Stream`来表示(理论上)无限长的序列成为可能。

Vavr `Stream`是不可变的，可能是`Empty`或`Cons`。一个`Cons`由一个头部元素和一个懒惰计算的尾部`Stream`组成。与`List`不同，对于`Stream`，只有头部元素被保存在内存中。尾部元素是按需计算的。

让我们创建一个由 10 个正整数组成的`Stream`,并计算偶数的总和:

```java
Stream<Integer> intStream = Stream.iterate(0, i -> i + 1)
  .take(10);

assertEquals(10, intStream.size());

long evenSum = intStream.filter(i -> i % 2 == 0)
  .sum()
  .longValue();

assertEquals(20, evenSum); 
```

与 Java 8 `Stream` API 相反，Vavr 的`Stream`是用于存储元素序列的数据结构。

因此，它有像`get()`、`append(),`、`insert()`等方法来操作它的元素。前面考虑的`drop()`、`distinct()`等一些方法也是可以的。

最后，让我们快速演示一下一个`Stream`中的`tabulate()`。该方法返回一个长度为`n`的`Stream`，其中包含应用函数的结果元素:

```java
Stream<Integer> s1 = Stream.tabulate(5, (i)-> i + 1);
assertEquals(s1.get(2).intValue(), 3);
```

我们还可以使用`zip()`生成一个`Tuple2<Integer, Integer>`的`Stream`，其中包含由两个`Streams`组合而成的元素:

```java
Stream<Integer> s = Stream.of(2,1,3,4);

Stream<Tuple2<Integer, Integer>> s2 = s.zip(List.of(7,8,9));
Tuple2<Integer, Integer> t1 = s2.get(0);

assertEquals(t1._1().intValue(), 2);
assertEquals(t1._2().intValue(), 7);
```

### 4.4。`Array`

`Array`是一个不可变的索引序列，允许高效的随机访问。它由对象的 Java `array`支持。本质上，它是类型为`T`的对象数组的`Traversable`包装器。

我们可以通过使用静态方法`of()`来实例化一个`Array`。我们还可以通过使用静态的`range()`和`rangeBy()`方法来生成一个范围元素。`rangeBy()`有第三个参数让我们定义步骤。

`range()`和`rangeBy()`方法将只生成从开始值到结束值减一的元素。如果我们需要包含最终值，我们可以使用`rangeClosed()`或`rangeClosedBy()`:

```java
Array<Integer> rArray = Array.range(1, 5);
assertFalse(rArray.contains(5));

Array<Integer> rArray2 = Array.rangeClosed(1, 5);
assertTrue(rArray2.contains(5));

Array<Integer> rArray3 = Array.rangeClosedBy(1,6,2);
assertEquals(rArray3.size(), 3);
```

让我们通过索引来操作元素:

```java
Array<Integer> intArray = Array.of(1, 2, 3);
Array<Integer> newArray = intArray.removeAt(1);

assertEquals(3, intArray.size());
assertEquals(2, newArray.size());
assertEquals(3, newArray.get(1).intValue());

Array<Integer> array2 = intArray.replace(1, 5);
assertEquals(array2.get(0).intValue(), 5);
```

### 4.5。`Vector`

一个`Vector`是一种介于`Array`和`List`之间的元素，它提供了另一个索引元素序列，允许在固定时间内进行随机访问和修改:

```java
Vector<Integer> intVector = Vector.range(1, 5);
Vector<Integer> newVector = intVector.replace(2, 6);

assertEquals(4, intVector.size());
assertEquals(4, newVector.size());

assertEquals(2, intVector.get(1).intValue());
assertEquals(6, newVector.get(1).intValue());
```

### 4.6。`CharSeq`

`CharSeq`是一个集合对象，用来表示一个原始字符序列。它本质上是一个添加了集合操作的`String`包装器。

创建一个`CharSeq`:

```java
CharSeq chars = CharSeq.of("vavr");
CharSeq newChars = chars.replace('v', 'V');

assertEquals(4, chars.size());
assertEquals(4, newChars.size());

assertEquals('v', chars.charAt(0));
assertEquals('V', newChars.charAt(0));
assertEquals("Vavr", newChars.mkString());
```

## 5。`Set`

在这一节中，我们将详细介绍集合库中的各种`Set`实现。`Set`数据结构的独特之处在于它不允许重复值。

然而，`Set`有不同的实现方式，其中 `HashSet`是基本的实现方式。`TreeSet`不允许重复元素，可以排序。`LinkedHashSet`保持其元素的插入顺序。

让我们一个一个地仔细看看这些实现。

### 5.1。`HashSet`

`HashSet`有创建新实例的静态工厂方法——其中一些我们在本文前面已经探讨过——像`of()`、`ofAll()`和`range()`方法的变体。

我们可以通过使用`diff()`方法得到两个集合之间的差异。同样，`union()`和`intersect()`方法返回两个集合的并集和交集:

```java
HashSet<Integer> set0 = HashSet.rangeClosed(1,5);
HashSet<Integer> set1 = HashSet.rangeClosed(3, 6);

assertEquals(set0.union(set1), HashSet.rangeClosed(1,6));
assertEquals(set0.diff(set1), HashSet.rangeClosed(1,2));
assertEquals(set0.intersect(set1), HashSet.rangeClosed(3,5));
```

我们还可以执行基本操作，例如添加和删除元素:

```java
HashSet<String> set = HashSet.of("Red", "Green", "Blue");
HashSet<String> newSet = set.add("Yellow");

assertEquals(3, set.size());
assertEquals(4, newSet.size());
assertTrue(newSet.contains("Yellow"));
```

`HashSet`实现由一个[散列数组映射的 trie (HAMT)](https://web.archive.org/web/20221004032439/https://en.wikipedia.org/wiki/Hash_array_mapped_trie) 支持，与普通的`HashTable` 相比，它拥有更好的性能，并且它的结构使它适合支持持久集合。

### 5.2。`TreeSet`

不可变的`TreeSet`是`SortedSet`接口的实现。它存储排序元素的`Set`，并使用二分搜索法树实现。它的所有操作都在 O(log n)时间内运行。

默认情况下，`TreeSet`的元素按自然顺序排序。

让我们使用自然排序顺序创建一个`SortedSet`:

```java
SortedSet<String> set = TreeSet.of("Red", "Green", "Blue");
assertEquals("Blue", set.head());

SortedSet<Integer> intSet = TreeSet.of(1,2,3);
assertEquals(2, intSet.average().get().intValue());
```

为了以定制的方式排序元素，在创建一个`TreeSet.` 的同时传递一个`Comparator`实例，我们也可以从集合元素中生成一个字符串:

```java
SortedSet<String> reversedSet
  = TreeSet.of(Comparator.reverseOrder(), "Green", "Red", "Blue");
assertEquals("Red", reversedSet.head());

String str = reversedSet.mkString(" and ");
assertEquals("Red and Green and Blue", str);
```

### 5.3。`BitSet`

Vavr 集合还包含一个不可变的 `BitSet`实现。`BitSet`接口扩展了`SortedSet`接口。`BitSet`可以使用`BitSet.Builder`中的静态方法进行实例化。

像其他数据结构的实现一样，`BitSet`不允许重复的条目被添加到集合中。

它从`Traversable`接口继承了操作方法。注意，它与标准 Java 库中的`java.util.BitSet`不同。`BitSet`数据不能包含`String`值。

让我们看看如何使用工厂方法`of()`创建一个`BitSet`实例:

```java
BitSet<Integer> bitSet = BitSet.of(1,2,3,4,5,6,7,8);
BitSet<Integer> bitSet1 = bitSet.takeUntil(i -> i > 4);
assertEquals(bitSet1.size(), 4);
```

我们使用`takeUntil()`来选择`BitSet.`的前四个元素，该操作返回一个新的实例。注意，`takeUntil()`是在`Traversable`接口中定义的，它是`BitSet.`的父接口

上面演示的在`Traversable`接口中定义的其他方法和操作也适用于`BitSet`。

## 6。`Map`

映射是一种键值数据结构。Vavr 的`Map`是不可变的，并且有`HashMap`、`TreeMap`和`LinkedHashMap`的实现。

通常，映射契约不允许重复的键，尽管可能有重复的值映射到不同的键。

### 6.1。`HashMap`

一个`HashMap`是一个不可变的`Map`接口的实现。它使用键的哈希代码存储键值对。

Vavr 的`Map`使用`Tuple2`来表示键值对，而不是传统的`Entry`类型:

```java
Map<Integer, List<Integer>> map = List.rangeClosed(0, 10)
  .groupBy(i -> i % 2);

assertEquals(2, map.size());
assertEquals(6, map.get(0).get().size());
assertEquals(5, map.get(1).get().size());
```

与`HashSet`类似，`HashMap`实现由散列数组映射的 trie (HAMT)支持，导致几乎所有操作的时间恒定。

我们可以使用`filterKeys()`方法按键过滤映射条目，或者使用`filterValues()`方法按值过滤。两种方法都接受一个`Predicate`作为参数:

```java
Map<String, String> map1
  = HashMap.of("key1", "val1", "key2", "val2", "key3", "val3");

Map<String, String> fMap
  = map1.filterKeys(k -> k.contains("1") || k.contains("2"));
assertFalse(fMap.containsKey("key3"));

Map<String, String> fMap2
  = map1.filterValues(v -> v.contains("3"));
assertEquals(fMap2.size(), 1);
assertTrue(fMap2.containsValue("val3"));
```

我们还可以通过使用`map()`方法来转换地图条目。例如，让我们将`map1` 转换为`Map<String, Integer>`:

```java
Map<String, Integer> map2 = map1.map(
  (k, v) -> Tuple.of(k, Integer.valueOf(v.charAt(v.length() - 1) + "")));
assertEquals(map2.get("key1").get().intValue(), 1);
```

### 6.2。`TreeMap`

不可变的`TreeMap`是`SortedMap`接口的实现。类似于`TreeSet`，一个`Comparator`实例被用来定制一个`TreeMap`的排序元素。

让我们演示一下`SortedMap`的创建:

```java
SortedMap<Integer, String> map
  = TreeMap.of(3, "Three", 2, "Two", 4, "Four", 1, "One");

assertEquals(1, map.keySet().toJavaArray()[0]);
assertEquals("Four", map.get(4).get());
```

默认情况下，`TreeMap`的条目按关键字的自然顺序排序。然而，我们可以指定一个用于排序的`Comparator`:

```java
TreeMap<Integer, String> treeMap2 =
  TreeMap.of(Comparator.reverseOrder(), 3,"three", 6, "six", 1, "one");
assertEquals(treeMap2.keySet().mkString(), "631");
```

和`TreeSet`一样，`TreeMap`的实现也是用树来建模的，因此它的操作时间是 O(log n)。`map.get(key)`返回一个`Option`，它在地图中的指定键处包装一个值。

## 7。与 Java 的互操作性

集合 API 与 Java 的集合框架完全互操作。让我们看看这在实践中是如何做到的。

### 7.1。Java 到 Vavr 的转换

Vavr 中的每个集合实现都有一个静态工厂方法`ofAll()`，该方法使用一个`java.util.Iterable`。这允许我们从 Java 集合中创建一个 Vavr 集合。同样，另一个工厂方法`ofAll()`直接采用 Java `Stream`。

要将 Java `List`转换成不可变的`List`:

```java
java.util.List<Integer> javaList = java.util.Arrays.asList(1, 2, 3, 4);
List<Integer> vavrList = List.ofAll(javaList);

java.util.stream.Stream<Integer> javaStream = javaList.stream();
Set<Integer> vavrSet = HashSet.ofAll(javaStream);
```

另一个有用的函数是`collector()`，它可以与`Stream.collect()`结合使用，以获得 Vavr 集合:

```java
List<Integer> vavrList = IntStream.range(1, 10)
  .boxed()
  .filter(i -> i % 2 == 0)
  .collect(List.collector());

assertEquals(4, vavrList.size());
assertEquals(2, vavrList.head().intValue());
```

### 7.2。Vavr 到 Java 的转换

接口有很多方法将 Vavr 类型转换成 Java 类型。这些方法的格式是`toJavaXXX()`。

让我们举几个例子:

```java
Integer[] array = List.of(1, 2, 3)
  .toJavaArray(Integer.class);
assertEquals(3, array.length);

java.util.Map<String, Integer> map = List.of("1", "2", "3")
  .toJavaMap(i -> Tuple.of(i, Integer.valueOf(i)));
assertEquals(2, map.get("2").intValue());
```

我们还可以使用 Java 8 `Collectors`从 Vavr 集合中收集元素:

```java
java.util.Set<Integer> javaSet = List.of(1, 2, 3)
  .collect(Collectors.toSet());

assertEquals(3, javaSet.size());
assertEquals(1, javaSet.toArray()[0]);
```

### 7.3。Java 集合视图

或者，该库提供了所谓的集合视图，在转换为 Java 集合时性能会更好。上一节中的转换方法遍历所有元素来构建一个 Java 集合。

另一方面，视图实现标准的 Java 接口，并将方法调用委托给底层的 Vavr 集合。

在撰写本文时，仅支持`List`视图。每个顺序集合有两个方法，一个用于创建不可变视图，另一个用于创建可变视图。

在不可变视图上调用 mutator 方法会导致一个`UnsupportedOperationException`。

让我们看一个例子:

```java
@Test(expected = UnsupportedOperationException.class)
public void givenVavrList_whenViewConverted_thenException() {
    java.util.List<Integer> javaList = List.of(1, 2, 3)
      .asJava();

    assertEquals(3, javaList.get(2).intValue());
    javaList.add(4);
}
```

要创建不可变视图:

```java
java.util.List<Integer> javaList = List.of(1, 2, 3)
  .asJavaMutable();
javaList.add(4);

assertEquals(4, javaList.get(3).intValue());
```

## 8。结论

在本教程中，我们学习了 Vavr 的集合 API 提供的各种功能数据结构。在 Vavr 的集合 [JavaDoc](https://web.archive.org/web/20221004032439/https://static.javadoc.io/io.vavr/vavr/0.9.0/io/vavr/collection/package-frame.html) 和[用户指南](https://web.archive.org/web/20221004032439/http://www.vavr.io/vavr-docs/)中可以找到更多有用和高效的 API 方法。

最后，需要注意的是，这个库还定义了`Try`、`Option`、`Either`和`Future`，它们扩展了`Value`接口，因此实现了 Java 的`Iterable`接口。这意味着它们在某些情况下可以表现为一个集合。

本文中所有示例的完整源代码可以在 Github 的[中找到。](https://web.archive.org/web/20221004032439/https://github.com/eugenp/tutorials/tree/master/vavr-modules/vavr)