# 幕后的 Java 散列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap-advanced>

## 1。概述

在本文中，我们将更详细地探索 Java Collections 框架中最流行的接口`Map`的实现，继续我们的 [intro](/web/20221225231549/https://www.baeldung.com/java-hashmap) 文章的未完部分。

在我们开始实现之前，重要的是要指出主`List`和`Set`集合接口扩展了`Collection`，但是`Map`没有。

简单地说，`HashMap`通过键存储值，并提供 API 以各种方式添加、检索和操作存储的数据。实现是基于散列表原理的**，这听起来有点复杂，但实际上非常容易理解。**

键值对存储在所谓的桶中，桶一起构成了所谓的表，表实际上是一个内部数组。

一旦我们知道了一个对象被存储或将要被存储的关键字，在一个良好维度的散列映射中，**存储和检索操作以常数时间**，`O(1)` 发生。

为了理解散列映射是如何工作的，我们需要理解`HashMap.` 所使用的存储和检索机制，我们将重点讨论这些。

最后， **`HashMap`相关问题在**面试中很常见，所以这是一个准备面试或为面试做准备的可靠方法。

## 2。`put()` API

为了在散列图中存储一个值，我们调用带两个参数的`put`API；一个键和相应的值:

```
V put(K key, V value);
```

当一个值被添加到键下的映射中时，键对象的`hashCode()` API 被调用来检索所谓的初始哈希值。

为了看到这一点，让我们创建一个充当键的对象。我们将只创建一个属性作为哈希代码来模拟哈希的第一阶段:

```
public class MyKey {
    private int id;

    @Override
    public int hashCode() {
        System.out.println("Calling hashCode()");
        return id;
    }

    // constructor, setters and getters 
}
```

我们现在可以使用这个对象来映射哈希表中的一个值:

```
@Test
public void whenHashCodeIsCalledOnPut_thenCorrect() {
    MyKey key = new MyKey(1);
    Map<MyKey, String> map = new HashMap<>();
    map.put(key, "val");
}
```

上面的代码没有什么变化，但是请注意控制台的输出。实际上，调用了`hashCode`方法:

```
Calling hashCode()
```

接下来，内部调用哈希表的`hash()` API，使用初始哈希值计算最终哈希值。

这个最终的哈希值最终归结为内部数组中的一个索引，或者我们称之为桶位置。

`HashMap`的`hash`函数是这样的:

```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这里我们应该注意的是，只使用来自 key 对象的散列码来计算最终的散列值。

在`put`函数内部，最终的哈希值是这样使用的:

```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

注意，调用了一个内部的`putVal`函数，并把最终的哈希值作为第一个参数。

有人可能会问，既然我们已经用它来计算哈希值，为什么还要在这个函数中使用这个键。

原因是**哈希映射将关键字和值作为`Map.Entry`对象**存储在桶位置。

如前所述，所有 Java 集合框架接口都扩展了`Collection`接口，但`Map`没有。对比一下我们之前看到的地图接口的声明和`Set`接口的声明:

```
public interface Set<E> extends Collection<E>
```

原因是**map 不像其他集合那样精确地存储单个元素，而是存储键值对的集合。**

所以`Collection`接口的通用方法如`add`、`toArray`对`Map`没有意义。

我们在前三段中介绍的概念是最受欢迎的 Java 集合框架面试问题之一。所以，值得理解。

散列映射的一个特殊属性是它接受`null`值和空键:

```
@Test
public void givenNullKeyAndVal_whenAccepts_thenCorrect(){
    Map<String, String> map = new HashMap<>();
    map.put(null, null);
}
```

**当在`put`操作中遇到一个空键时，它会被自动分配一个最终哈希值`0`** ，这意味着它成为底层数组的第一个元素。

这也意味着当键为空时，没有散列操作，因此键的`hashCode` API 不会被调用，最终避免了空指针异常。

在一个`put`操作中，当我们使用一个先前已经使用过的键来存储一个值时，它返回与该键相关的先前值:

```
@Test
public void givenExistingKey_whenPutReturnsPrevValue_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("key1", "val1");

    String rtnVal = map.put("key1", "val2");

    assertEquals("val1", rtnVal);
}
```

否则，它返回`null:`

```
@Test
public void givenNewKey_whenPutReturnsNull_thenCorrect() {
    Map<String, String> map = new HashMap<>();

    String rtnVal = map.put("key1", "val1");

    assertNull(rtnVal);
}
```

当`put`返回 null 时，也可能意味着与键相关联的前一个值为 null，而不一定是新的键-值映射:

```
@Test
public void givenNullVal_whenPutReturnsNull_thenCorrect() {
    Map<String, String> map = new HashMap<>();

    String rtnVal = map.put("key1", null);

    assertNull(rtnVal);
}
```

正如我们将在下一小节中看到的，API 可以用来区分这些场景。

## 3。`get` API

要检索已经存储在哈希表中的对象，我们必须知道它存储在哪个键下。我们调用`get` API 并传递给它一个关键对象:

```
@Test
public void whenGetWorks_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("key", "val");

    String val = map.get("key");

    assertEquals("val", val);
}
```

在内部，使用相同的散列原理。`The hashCode()`调用 key 对象的 API 获取初始哈希值:

```
@Test
public void whenHashCodeIsCalledOnGet_thenCorrect() {
    MyKey key = new MyKey(1);
    Map<MyKey, String> map = new HashMap<>();
    map.put(key, "val");
    map.get(key);
}
```

这次调用了两次`MyKey`的`hashCode`API；一次用于`put`，一次用于`get`:

```
Calling hashCode()
Calling hashCode()
```

然后通过调用内部的`hash()` API 来获得最终的哈希值，从而对这个值进行重新哈希。

正如我们在上一节中看到的，这个最终的哈希值最终归结为一个桶位置或内部数组的索引。

然后，存储在该位置的值对象被检索并返回给调用函数。

当返回值为 null 时，可能意味着键对象与哈希映射中的任何值都没有关联:

```
@Test
public void givenUnmappedKey_whenGetReturnsNull_thenCorrect() {
    Map<String, String> map = new HashMap<>();

    String rtnVal = map.get("key1");

    assertNull(rtnVal);
}
```

或者它可能仅仅意味着该键被显式映射到一个空实例:

```
@Test
public void givenNullVal_whenRetrieves_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("key", null);

    String val=map.get("key");

    assertNull(val);
}
```

为了区分这两种情况，我们可以使用`containsKey` API，我们将键传递给它，当且仅当在哈希映射中为指定的键创建了映射时，它才返回 true:

```
@Test
public void whenContainsDistinguishesNullValues_thenCorrect() {
    Map<String, String> map = new HashMap<>();

    String val1 = map.get("key");
    boolean valPresent = map.containsKey("key");

    assertNull(val1);
    assertFalse(valPresent);

    map.put("key", null);
    String val = map.get("key");
    valPresent = map.containsKey("key");

    assertNull(val);
    assertTrue(valPresent);
}
```

对于上面测试中的两种情况，`get` API 调用的返回值都是 null，但是我们能够区分哪一个是哪一个。

## 4。`HashMap`收藏观点

`HashMap`提供了三个视图，使我们能够将其键和值视为另一个集合。我们可以得到地图的所有**键的集合:**

```
@Test
public void givenHashMap_whenRetrievesKeyset_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "baeldung");
    map.put("type", "blog");

    Set<String> keys = map.keySet();

    assertEquals(2, keys.size());
    assertTrue(keys.contains("name"));
    assertTrue(keys.contains("type"));
}
```

布景由地图本身支撑。因此**对集合所做的任何更改都会反映在映射图**中:

```
@Test
public void givenKeySet_whenChangeReflectsInMap_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "baeldung");
    map.put("type", "blog");

    assertEquals(2, map.size());

    Set<String> keys = map.keySet();
    keys.remove("name");

    assertEquals(1, map.size());
}
```

我们还可以获得值的**集合视图:**

```
@Test
public void givenHashMap_whenRetrievesValues_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "baeldung");
    map.put("type", "blog");

    Collection<String> values = map.values();

    assertEquals(2, values.size());
    assertTrue(values.contains("baeldung"));
    assertTrue(values.contains("blog"));
}
```

就像键集一样，在这个集合中所做的任何**更改都将反映在底层映射**中。

最后，我们可以获得地图中所有条目的**集合视图:**

```
@Test
public void givenHashMap_whenRetrievesEntries_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "baeldung");
    map.put("type", "blog");

    Set<Entry<String, String>> entries = map.entrySet();

    assertEquals(2, entries.size());
    for (Entry<String, String> e : entries) {
        String key = e.getKey();
        String val = e.getValue();
        assertTrue(key.equals("name") || key.equals("type"));
        assertTrue(val.equals("baeldung") || val.equals("blog"));
    }
}
```

请记住，哈希映射特别包含无序元素，因此在测试`for each`循环中条目的键和值时，我们假设任何顺序。

很多时候，您会像上一个例子一样在一个循环中使用集合视图，更具体地说是使用它们的迭代器。

请记住，以上所有视图的**迭代器都是`fail-fast`** 。

如果在迭代器创建后对映射进行了任何结构修改，将会抛出并发修改异常:

```
@Test(expected = ConcurrentModificationException.class)
public void givenIterator_whenFailsFastOnModification_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "baeldung");
    map.put("type", "blog");

    Set<String> keys = map.keySet();
    Iterator<String> it = keys.iterator();
    map.remove("type");
    while (it.hasNext()) {
        String key = it.next();
    }
}
```

唯一**允许的结构修改是通过迭代器本身执行的`remove`** 操作:

```
public void givenIterator_whenRemoveWorks_thenCorrect() {
    Map<String, String> map = new HashMap<>();
    map.put("name", "baeldung");
    map.put("type", "blog");

    Set<String> keys = map.keySet();
    Iterator<String> it = keys.iterator();

    while (it.hasNext()) {
        it.next();
        it.remove();
    }

    assertEquals(0, map.size());
}
```

关于这些集合视图，最后要记住的是迭代的性能。这就是散列映射与它的对应链接散列映射和树映射相比表现很差的地方。

在最坏的情况`O(n)`下，散列映射上的迭代发生，其中 n 是其容量和条目数量的总和。

## `**5\. HashMap Performance**`

哈希映射的性能受两个参数的影响:`Initial Capacity`和`Load Factor`。容量是存储桶的数量或底层数组的长度，初始容量就是创建时的容量。

简而言之，加载因子或 LF 是在调整大小之前添加一些值之后哈希映射应该有多满的度量。

默认初始容量为`16`，默认负载系数为`0.75`。我们可以使用初始容量和 LF 的自定义值创建一个哈希映射:

```
Map<String,String> hashMapWithCapacity=new HashMap<>(32);
Map<String,String> hashMapWithCapacityAndLF=new HashMap<>(32, 0.5f);
```

Java 团队设置的默认值在大多数情况下都得到了很好的优化。然而，如果你需要使用你自己的价值观，这是很好的，你需要理解性能的含义，以便你知道你在做什么。

当散列映射条目的数量超过 LF 和容量的乘积时，则发生**重新散列**，即**创建另一个内部数组，其大小是初始数组的两倍，并且所有条目被移动到新数组中的新桶位置**。

低初始容量降低了空间成本，但是 T2 增加了重新散列的频率。显然，重新散列是一个非常昂贵的过程。因此，作为一个规则，如果您预期有许多条目，您应该设置一个相当高的初始容量。

另一方面，如果您将初始容量设置得太高，您将在迭代时间中付出代价。正如我们在上一节中看到的。

因此**高初始容量对于大量条目以及很少甚至没有迭代是有益的**。

低初始容量对于具有大量迭代的少数条目是有益的。

## 6。`HashMap`碰撞中的

冲突，或者更具体地说，`HashMap`中的散列码冲突，是这样一种情况，其中**两个或更多的关键字对象产生相同的最终散列值**，并因此指向相同的桶位置或数组索引。

之所以会出现这种情况，是因为根据`equals`和`hashCode`契约，**Java 中两个不相等的对象可以有相同的哈希码**。

也可能是因为基础数组的大小有限，也就是说，在调整大小之前。这个阵列越小，碰撞的几率就越高。

也就是说，值得一提的是，Java 实现了哈希代码冲突解决技术，我们将通过一个例子来了解这一点。

请记住，是键的哈希值决定了存储对象的桶。因此，如果任意两个键的散列码发生冲突，它们的条目仍将存储在同一个桶中。

默认情况下，该实现使用一个链表作为桶实现。

在发生碰撞的情况下，最初恒定的时间`O(1)` `put`和`get`操作将在线性时间`O(n)`内发生。这是因为在找到具有最终哈希值的桶位置后，该位置的每个键将使用`equals` API 与提供的键对象进行比较。

为了模拟这种冲突解决技术，让我们稍微修改一下前面的关键对象:

```
public class MyKey {
    private String name;
    private int id;

    public MyKey(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // standard getters and setters

    @Override
    public int hashCode() {
        System.out.println("Calling hashCode()");
        return id;
    } 

    // toString override for pretty logging

    @Override
    public boolean equals(Object obj) {
        System.out.println("Calling equals() for key: " + obj);
        // generated implementation
    }

}
```

注意我们是如何简单地将`id`属性作为散列码返回的——从而强制发生冲突。

另外，请注意，我们已经在我们的`equals`和`hashCode` 实现中添加了日志语句——这样我们就可以准确地知道何时调用逻辑。

现在让我们继续存储和检索一些在某个点碰撞的对象:

```
@Test
public void whenCallsEqualsOnCollision_thenCorrect() {
    HashMap<MyKey, String> map = new HashMap<>();
    MyKey k1 = new MyKey(1, "firstKey");
    MyKey k2 = new MyKey(2, "secondKey");
    MyKey k3 = new MyKey(2, "thirdKey");

    System.out.println("storing value for k1");
    map.put(k1, "firstValue");
    System.out.println("storing value for k2");
    map.put(k2, "secondValue");
    System.out.println("storing value for k3");
    map.put(k3, "thirdValue");

    System.out.println("retrieving value for k1");
    String v1 = map.get(k1);
    System.out.println("retrieving value for k2");
    String v2 = map.get(k2);
    System.out.println("retrieving value for k3");
    String v3 = map.get(k3);

    assertEquals("firstValue", v1);
    assertEquals("secondValue", v2);
    assertEquals("thirdValue", v3);
}
```

在上面的测试中，我们创建了三个不同的键——一个有唯一的`id`,另外两个有相同的`id`。由于我们使用`id`作为初始哈希值，**在使用这些键存储和检索数据的过程中，肯定会出现**冲突。

除此之外，由于我们前面看到的冲突解决技术，我们期望我们存储的每个值都被正确地检索，因此最后三行中的断言。

当我们运行测试时，它应该通过，表明冲突已经解决，我们将使用生成的日志来确认冲突确实发生了:

```
storing value for k1
Calling hashCode()
storing value for k2
Calling hashCode()
storing value for k3
Calling hashCode()
Calling equals() for key: MyKey [name=secondKey, id=2]
retrieving value for k1
Calling hashCode()
retrieving value for k2
Calling hashCode()
retrieving value for k3
Calling hashCode()
Calling equals() for key: MyKey [name=secondKey, id=2]
```

请注意，在存储操作期间，`k1`和`k2`仅使用哈希代码成功映射到它们的值。

然而，`k3`的存储并不简单，系统检测到它的存储桶位置已经包含了`k2`的映射。因此，使用`equals`比较来区分它们，并创建一个链表来包含这两种映射。

如果`equals`比较对于所有现有节点返回假，其关键字散列到相同桶位置的任何其他后续映射将遵循相同的路线，并且最终替换链表中的一个节点，或者被添加到列表的头部。

同样，在检索过程中，`k3`和`k2`被`equals`比较，以识别应该检索其值的正确键。

最后一点，从 Java 8 开始，在给定桶位置的冲突数量超过某个阈值后，在冲突解决中，链表被动态地替换为平衡二分搜索法树。

这一改变提供了性能提升，因为在冲突的情况下，存储和检索发生在`O(log n).`

这一部分在技术面试中很常见，尤其是在基本的存储和检索问题之后。

## 7。结论

在本文中，我们探索了 Java `Map`接口的`HashMap`实现。

本文中使用的所有示例的完整源代码可以在 GitHub 项目中找到。