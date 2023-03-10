# Java 集合面试问题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collections-interview-questions>

[This article is part of a series:](javascript:void(0);)• Java Collections Interview Questions (current article)[• Java Type System Interview Questions](/web/20221208143854/https://www.baeldung.com/java-type-system-interview-questions)
[• Java Concurrency Interview Questions (+ Answers)](/web/20221208143854/https://www.baeldung.com/java-concurrency-interview-questions)
[• Java Class Structure and Initialization Interview Questions](/web/20221208143854/https://www.baeldung.com/java-classes-initialization-questions)
[• Java 8 Interview Questions(+ Answers)](/web/20221208143854/https://www.baeldung.com/java-8-interview-questions)
[• Memory Management in Java Interview Questions (+Answers)](/web/20221208143854/https://www.baeldung.com/java-memory-management-interview-questions)
[• Java Generics Interview Questions (+Answers)](/web/20221208143854/https://www.baeldung.com/java-generics-interview-questions)
[• Java Flow Control Interview Questions (+ Answers)](/web/20221208143854/https://www.baeldung.com/java-flow-control-interview-questions)
[• Java Exceptions Interview Questions (+ Answers)](/web/20221208143854/https://www.baeldung.com/java-exceptions-interview-questions)
[• Java Annotations Interview Questions (+ Answers)](/web/20221208143854/https://www.baeldung.com/java-annotations-interview-questions)
[• Top Spring Framework Interview Questions](/web/20221208143854/https://www.baeldung.com/spring-interview-questions)

## 1.介绍

Java 集合是 Java 开发人员技术访谈中经常提到的一个话题。本文回顾了一些最常被问到的重要问题，这些问题可能很难回答正确。

## 2.问题

### Q1。描述集合类型层次结构。主要有哪些接口，它们之间有什么区别？

**`Iterable`** 接口表示可以使用`for-each`循环迭代的任何集合。 **`Collection`** 接口继承了`Iterable`并增加了检查元素是否在集合中、从集合中添加和移除元素、确定元素大小等通用方法。

**`List`** 、 **`Set`** 、 **`Queue`** 接口继承自`Collection`接口。

**`List`** 是一个有序的集合，其元素可以通过它们在列表中的索引来访问。

**`Set`** 是具有不同元素的无序集合，类似于集合的数学概念。

**`Queue`** 是一个附加方法的集合，用于添加、删除和检查元素，在处理之前保存元素。

**`Map`** 接口也是集合框架的一部分，然而它并没有扩展`Collection`。这是为了强调集合和映射之间的区别，集合和映射很难在一个共同的抽象下聚集。`Map`接口表示一个键-值数据结构，具有唯一的键，每个键不超过一个值。

### Q2。描述地图接口的各种实现及其用例差异。

`Map`接口最常用的实现之一是 **`HashMap`** 。它是一种典型的哈希映射数据结构，允许在常数时间内访问元素，即 O(1)，但**不保持顺序，也不是线程安全的**。

为了保持元素的插入顺序，您可以使用 **`LinkedHashMap`** 类，该类扩展了`HashMap`并将元素绑定到一个链表中，具有可预见的开销。

**`TreeMap`** 类将其元素存储在红黑树结构中，这允许以对数时间或 O(log(n))访问元素。在大多数情况下，它比`HashMap`慢，但是它允许根据一些`Comparator`来保持元素的顺序。

**ConcurrentHashMap** 是哈希映射的线程安全实现。它提供了检索的完全并发性(因为`get`操作不需要锁定)和更新的高预期并发性。

从 1.0 版本开始， **`Hashtable`** 类就出现在 Java 中了。它没有被弃用，但通常被认为是过时的。它是一个线程安全的哈希映射，但与`ConcurrentHashMap`不同的是，它的所有方法都是简单的`synchronized`，这意味着在这个映射上的所有操作都是阻塞的，甚至是对独立值的检索。

### Q3。解释 Linkedlist 和 Arraylist 的区别。

**`ArrayList`** 是基于数组的`List`接口的实现。`ArrayList`当添加或删除元素时，在内部处理数组的大小调整。你可以通过数组中的索引在常量时间内访问它的元素。然而，插入或移除元素意味着移动所有后续元素，如果数组很大并且插入或移除的元素靠近列表的开头，这可能会很慢。

**`LinkedList`** 是一个双向链表:单个元素被放入引用前一个和下一个`Node`的`Node`对象中。如果在列表的不同部分有很多插入或删除，这个实现可能比`ArrayList`更有效，尤其是当列表很大的时候。

然而，在大多数情况下，`ArrayList`的表现优于`LinkedList`。在`ArrayList`中，即使元素移位是 O(n)操作，也是作为一个非常快速的`System.arraycopy()`调用来实现的。它甚至比`LinkedList`的 O(1)插入更快，后者需要实例化一个`Node`对象并更新多个引用。由于创建了多个小的`Node`对象，所以`LinkedList`也会有很大的内存开销。

### Q4。Hashset 和 Treeset 有什么区别？

**`HashSet`** 和 **`TreeSet`** 类都实现了`Set`接口，并表示不同元素的集合。另外，`TreeSet`实现了`NavigableSet`接口。该接口定义了利用元素排序的方法。

`HashSet`在内部基于一个`HashMap`，`TreeSet`由一个`TreeMap`实例支持，该实例定义了它们的属性:`HashSet`不按任何特定的顺序保存元素。对一个`HashSet`中的元素进行迭代会产生一个无序的顺序。另一方面，`TreeSet`根据一些预定义的`Comparator`按顺序产生元素。

### Q5。Hashmap 在 Java 中是如何实现的？它的实现如何使用对象的 Hashcode 和 Equals 方法？从这样的结构中放置和获取元素的时间复杂度是多少？

`HashMap`类代表了一种典型的散列映射数据结构，具有某些设计选择。

`HashMap`由一个大小为 2 的幂的可调整数组支持。当元素被添加到一个`HashMap`时，首先计算它的`hashCode`(一个`int`值)。则该值的一定数量的低位比特被用作数组索引。这个索引直接指向数组中应该放置这个键值对的单元格(称为 bucket)。通过数组中的索引访问元素是一个非常快速的 O(1)操作，这是哈希映射结构的主要特性。

然而，`hashCode`并不是唯一的，即使对于不同的`hashCodes`，我们也可能接收到相同的数组位置。这叫做碰撞。在散列映射数据结构中，解决冲突的方法不止一种。在 Java 的`HashMap`中，每个桶实际上指的不是单个对象，而是一个包含所有落在这个桶中的对象的红黑树(在 Java 8 之前，这是一个链表)。

所以当`HashMap`已经确定了一个键的桶时，它必须遍历这棵树，将键-值对放到它的位置上。如果桶中已经存在具有这种密钥的密钥对，则用新的密钥替换它。

为了通过关键字检索对象，`HashMap`必须再次计算关键字的`hashCode`,找到相应的桶，遍历树，调用树中关键字的`equals`,找到匹配的那个。

具有 O(1)复杂度，或常数时间复杂度的放置和获取元素。当然，在最坏的情况下，当所有元素都在一个桶中时，大量的冲突会将性能降低到 O(log(n))时间复杂度。这通常通过提供具有均匀分布的良好散列函数来解决。

当`HashMap`内部数组被填满时(下一个问题会详细介绍)，它会自动调整大小为原来的两倍。这个操作意味着重新散列(内部数据结构的重建)，这是很昂贵的，所以您应该预先计划好您的`HashMap`的大小。

### Q6。Hashmap 的初始容量和负载系数参数的目的是什么？它们的默认值是什么？

`HashMap`构造函数的`initialCapacity`参数影响了`HashMap`内部数据结构的大小，但是推断地图的实际大小有点棘手。`HashMap`的内部数据结构是一个大小为 2 的幂的数组。所以`initialCapacity`参数值增加到 2 的下一个幂(例如，如果你把它设置为 10，内部数组的实际大小将是 16)。

`HashMap`的加载因子是元素计数除以桶计数的比值(即内部数组大小)。例如，如果一个 16 桶`HashMap`包含 12 个元素，那么它的负载系数是 12/16 = 0.75。高负载系数意味着大量的碰撞，这反过来意味着地图的大小应该调整到 2 的次方。所以`loadFactor`参数是地图装载系数的最大值。当 map 达到这个加载因子时，它会将其内部数组的大小调整为下一个 2 的幂值。

默认情况下,`initialCapacity`是 16，默认情况下 loadFactor 是 0.75，所以您可以将 12 个元素放在用默认构造函数实例化的`HashMap`中，并且它不会调整大小。对于`HashSet`也是一样，它由内部的`HashMap`实例支持。

因此，想出满足你需求的`initialCapacity`并不容易。这就是为什么 Guava 库有`Maps.newHashMapWithExpectedSize()`和`Sets.newHashSetWithExpectedSize()`方法，允许您构建一个`HashMap`或`HashSet`，可以保存预期数量的元素而无需调整大小。

### Q7。描述枚举的特殊集合。与常规收集相比，它们的实现有什么好处？

**`EnumSet`****`EnumMap`**是`Set``Map`接口对应的特殊实现。当你处理枚举时，你应该总是使用这些实现，因为它们非常有效。

一个`EnumSet`只是一个位向量，在对应于集合中存在的枚举的序数值的位置上具有“1”。要检查枚举值是否在集合中，实现只需检查向量中相应的位是否为“1”，这是一个非常简单的操作。类似地，`EnumMap`是一个数组，使用 enum 的序数值作为索引进行访问。在`EnumMap`的情况下，不需要计算散列码或解决冲突。

### Q8。快速失效迭代器和安全失效迭代器的区别是什么？

不同集合的迭代器要么是故障快速的，要么是故障安全的，这取决于它们对并发修改的反应。并发修改不仅是来自另一个线程的集合的修改，也是来自同一线程但使用另一个迭代器或直接修改集合的修改。

**快速失效**迭代器(由`HashMap`、`ArrayList`和其他非线程安全集合返回的迭代器)遍历集合的内部数据结构，一旦检测到并发修改，它们就会抛出`ConcurrentModificationException`。

**故障安全**迭代器(由线程安全集合如`ConcurrentHashMap`、`CopyOnWriteArrayList`返回)创建它们迭代的结构的副本。它们保证了并发修改的安全性。它们的缺点包括过多的内存消耗和在集合被修改的情况下对可能过期的数据进行迭代。

### Q9。如何使用 Comparable 和 Comparator 接口对集合进行排序？

`Comparable`接口是可以按照某种顺序进行比较的对象的接口。它的单个方法是`compareTo`，操作两个值:对象本身和同类型的 argument 对象。例如，`Integer`、`Long`等数字类型实现了这个接口。`String`也实现了这个接口，它的`compareTo`方法按照字典顺序比较字符串。

`Comparable`接口允许使用`Collections.sort()`方法对相应对象的列表进行排序，并支持实现`SortedSet`和`SortedMap`的集合中的迭代顺序。如果你的对象可以用某种逻辑排序，它们应该实现`Comparable`接口。

`Comparable`接口通常使用元素的自然排序来实现。例如，所有的`Integer`数字从较小值到较大值排序。但有时您可能希望实现另一种排序，例如，按降序对数字进行排序。这里的`Comparator`界面可以提供帮助。

要排序的对象的类不需要实现这个接口。您只需创建一个实现类并定义接收两个对象并决定如何对它们排序的`compare`方法。然后你可以使用这个类的实例来覆盖`Collections.sort()`方法或者`SortedSet`和`SortedMap`实例的自然排序。

由于`Comparator`接口是一个函数接口，您可以用 lambda 表达式替换它，如下例所示。它展示了使用自然排序(`Integer`的`Comparable`接口)和使用定制迭代器(`Comparator<Integer>`接口)对列表进行排序。

```java
List<Integer> list1 = Arrays.asList(5, 2, 3, 4, 1);
Collections.sort(list1);
assertEquals(new Integer(1), list1.get(0));

List<Integer> list1 = Arrays.asList(5, 2, 3, 4, 1);
Collections.sort(list1, (a, b) -> b - a);
assertEquals(new Integer(5), list1.get(0));
```

Next **»**[Java Type System Interview Questions](/web/20221208143854/https://www.baeldung.com/java-type-system-interview-questions)