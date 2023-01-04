# Java.util.Hashtable 类简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hash-table>

## 1。概述

**[`Hashtable`](https://web.archive.org/web/20221208143839/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Hashtable.html) 是 Java 中最古老的哈希表数据结构的实现。**`HashMap`是第二个实现，是在 JDK 1.2 中引入的。

这两个类提供了相似的功能，但是也有一些小的不同，我们将在本教程中探讨。

## 2。何时使用`Hashtable`

假设我们有一本字典，每个单词都有它的定义。此外，我们需要快速地从字典中获取、插入和删除单词。

因此，`Hashtable`(或`HashMap`)是有意义的。词语将是`Hashtable`中的钥匙，因为它们被认为是独一无二的。另一方面，定义将是值。

## 3。使用示例

让我们继续字典的例子。我们将`Word`建模为一个键:

```java
public class Word {
    private String name;

    public Word(String name) {
        this.name = name;
    }

    // ...
}
```

假设值为`Strings`。现在我们可以创建一个`Hashtable`:

```java
Hashtable<Word, String> table = new Hashtable<>();
```

首先，让我们添加一个条目:

```java
Word word = new Word("cat");
table.put(word, "an animal");
```

此外，要获取条目:

```java
String definition = table.get(word);
```

最后，让我们删除一个条目:

```java
definition = table.remove(word);
```

这个类中还有很多方法，我们将在后面描述其中的一些。

不过先说说对重点对象的一些要求。

## 4。`hashCode()` 【T2 的重要性】

**作为`Hashtable`中的密钥，对象不能违反 [`hashCode()`契约。](/web/20221208143839/https://www.baeldung.com/java-hashcode)** 简而言之，相等的对象必须返回相同的代码。为了理解为什么，让我们看看哈希表是如何组织的。

`Hashtable`使用一个数组。数组中的每个位置都是一个“桶”,可以为空，也可以包含一个或多个键值对。计算每对的指数。

但是为什么不按顺序存储元素，在数组末尾添加新元素呢？

关键是通过索引查找元素比通过顺序比较遍历元素要快得多。因此，我们需要一个将键映射到索引的函数。

### 4.1。直接地址表

这种映射最简单的例子是直接地址表。这里，键被用作索引:

```java
index(k)=k,
where k is a key
```

键是唯一的，即每个桶包含一个键值对。当整数键的可能范围相当小时，这种技术非常适用。

但是我们有两个问题:

*   首先，我们的键不是整数，而是`Word`对象
*   第二，如果它们是整数，没人能保证它们是小的。假设密钥是 1，2 和 1000000。我们将有一个只有三个元素的 1000000 大小的大数组，其余的都是浪费的空间

`hashCode()`方法解决了第一个问题。

`Hashtable`中的数据操作逻辑解决了第二个问题。

我们来深入讨论一下这个问题。

### 4.2。`hashCode()`法

任何 Java 对象都会继承返回一个`int`值的`hashCode()`方法。这个值是从对象的内部内存地址计算出来的。默认情况下，`hashCode()`为不同的对象返回不同的整数。

因此**任何关键对象都可以用`hashCode()`** 转换成整数。但是这个整数可能很大。

### 4.3。缩小范围

`get()`、`put()`和`remove()`方法包含解决第二个问题的代码——减少可能的整数范围。

该公式计算键的索引:

```java
int index = (hash & 0x7FFFFFFF) % tab.length;
```

其中`tab.length`是数组大小， `hash`是键的 `hashCode()`方法返回的数字。

正如我们所见,**索引是对数组大小**除以`hash`的提醒。注意，相等的散列码产生相同的索引。

### 4.4。碰撞

此外，即使**不同的哈希码也能产生相同的索引**。我们称之为碰撞。为了解决冲突，`Hashtable`存储了一个键-值对的`LinkedList`。

这种数据结构被称为带链接的哈希表。

### 4.5。负载系数

很容易猜测冲突会降低元素操作的速度。要获得一个条目，仅仅知道它的索引是不够的，我们还需要遍历列表并对每个条目进行比较。

因此，减少碰撞的数量是很重要的。数组越大，冲突的几率就越小。**负载系数决定了阵列大小和性能之间的平衡。**默认情况下，它是 0.75，这意味着当 75%的存储桶不为空时，数组大小会翻倍。该操作由`rehash()`方法执行。

但是让我们回到钥匙上。

### 4.6。覆盖 equals()和 hashCode()

当我们将一个条目放入`Hashtable`并从中取出时，我们期望不仅可以用相同的键实例获得值，还可以用相同的键获得值:

```java
Word word = new Word("cat");
table.put(word, "an animal");
String extracted = table.get(new Word("cat"));
```

**为了设置相等的规则，我们覆盖了键的`equals()`方法:**

```java
public boolean equals(Object o) {
    if (o == this)
        return true;
    if (!(o instanceof Word))
        return false;

    Word word = (Word) o;
    return word.getName().equals(this.name);
}
```

但是如果我们在覆盖`equals()`时不覆盖`hashCode()`，那么两个相等的键可能会在不同的桶中结束，因为`Hashtable`使用它的散列码计算键的索引。

让我们仔细看看上面的例子。如果我们不覆盖`hashCode()`会发生什么？

*   这里涉及到两个`Word`实例——第一个是放入条目，第二个是获取条目。尽管这些实例是相同的，但是它们的`hashCode()`方法返回不同的数字
*   每个键的指数由第 4.3 节中的公式计算。根据这个公式，不同的散列码可能产生不同的索引
*   这意味着我们将条目放入一个桶中，然后尝试从另一个桶中取出它。这样的逻辑中断`Hashtable`

**相等的键必须返回相等的散列码，这就是为什么我们覆盖了`hashCode()`方法:**

```java
public int hashCode() {
    return name.hashCode();
}
```

注意**还建议让不相等的键返回不同的散列码**，否则它们会在同一个桶中结束。这将影响性能，从而失去一些`Hashtable`的优势。

另外，请注意，我们并不关心`String`、`Integer`、`Long`或其他包装器类型的键。`equal()`和`hashCode()`方法已经在包装类中被覆盖。

## 5。迭代`Hashtables`

在这一节中有几种方法可以重复`Hashtables. `我们将讨论它们并解释一些含义。

### 5.1。`Iteration`失败快:

快速失败迭代意味着如果在创建了`Iterator `之后修改了`Hashtable`，那么`ConcurrentModificationException`将被抛出。让我们来演示一下。

首先，我们将创建一个`Hashtable`并向其中添加条目:

```java
Hashtable<Word, String> table = new Hashtable<Word, String>();
table.put(new Word("cat"), "an animal");
table.put(new Word("dog"), "another animal");
```

其次，我们将创建一个`Iterator`:

```java
Iterator<Word> it = table.keySet().iterator();
```

第三，我们将修改表格:

```java
table.remove(new Word("dog"));
```

现在，如果我们尝试遍历这个表，我们将得到一个`ConcurrentModificationException`:

```java
while (it.hasNext()) {
    Word key = it.next();
}
```

```java
java.util.ConcurrentModificationException
	at java.util.Hashtable$Enumerator.next(Hashtable.java:1378)
```

`ConcurrentModificationException`有助于发现错误，从而避免不可预测的行为，例如，当一个线程正在遍历表，而另一个线程同时试图修改它时。

### 5.2。`Enumeration`不倒快:

`Hashtable`中的`Enumeration`不是防故障的。让我们看一个例子。

首先，让我们创建一个`Hashtable`并向其中添加条目:

```java
Hashtable<Word, String> table = new Hashtable<Word, String>();
table.put(new Word("1"), "one");
table.put(new Word("2"), "two");
```

其次，让我们创建一个`Enumeration`:

```java
Enumeration<Word> enumKey = table.keys();
```

第三，让我们修改表格:

```java
table.remove(new Word("1"));
```

现在，如果我们遍历该表，它不会抛出异常:

```java
while (enumKey.hasMoreElements()) {
    Word key = enumKey.nextElement();
}
```

### 5.3。不可预测的迭代顺序

另外，请注意,`Hashtable`中的迭代顺序是不可预测的，并且与条目添加的顺序不匹配。

这是可以理解的，因为它使用键的哈希代码计算每个索引。此外，重新散列会不时发生，重新安排数据结构的顺序。

因此，让我们添加一些条目并检查输出:

```java
Hashtable<Word, String> table = new Hashtable<Word, String>();
    table.put(new Word("1"), "one");
    table.put(new Word("2"), "two");
    // ...
    table.put(new Word("8"), "eight");

    Iterator<Map.Entry<Word, String>> it = table.entrySet().iterator();
    while (it.hasNext()) {
        Map.Entry<Word, String> entry = it.next();
        // ...
    }
}
```

```java
five
four
three
two
one
eight
seven
```

## 6。`Hashtable`对`HashMap`对

`Hashtable`和`HashMap`提供非常相似的功能。

它们都提供:

*   快速失效迭代
*   不可预测的迭代顺序

但是也有一些不同之处:

*   `HashMap`不提供任何`Enumeration, while ` `Hashtable`提供不防故障`Enumeration`
*   `Hashtable`不允许`null`键和`null`值，而`HashMap`允许一个`null`键和任意数量的`null`值
*   `Hashtable`的方法是同步的，而`HashMaps`的方法不是

## 7。`Hashtable`Java 8 中的 API

Java 8 引入了新的方法，帮助我们的代码更加简洁。特别是我们可以去掉一些`if`块。让我们来演示一下。

### 7.1。`getOrDefault()`

假设我们需要得到单词“dog `” `的定义，如果它在表上，就把它赋给变量。否则，将“未找到”赋给变量。

Java 8 之前:

```java
Word key = new Word("dog");
String definition;

if (table.containsKey(key)) {
     definition = table.get(key);
} else {
     definition = "not found";
}
```

Java 8 之后:

```java
definition = table.getOrDefault(key, "not found");
```

### 7.2。`putIfAbsent()`

假设我们只需要把一个单词“cat `“`放入字典中。

Java 8 之前:

```java
if (!table.containsKey(new Word("cat"))) {
    table.put(new Word("cat"), definition);
}
```

Java 8 之后:

```java
table.putIfAbsent(new Word("cat"), definition);
```

### 7.3。`boolean remove()`

假设我们需要删除“猫”这个词，但前提是它的定义是“一种动物”。

Java 8 之前:

```java
if (table.get(new Word("cat")).equals("an animal")) {
    table.remove(new Word("cat"));
}
```

Java 8 之后:

```java
boolean result = table.remove(new Word("cat"), "an animal");
```

最后，当旧的`remove()`方法返回值时，新的方法返回*布尔值*。

### 7.4。`replace()`

假设我们需要替换“猫”的一个定义，但前提是它的旧定义是“一种小型家养食肉哺乳动物”。

Java 8 之前:

```java
if (table.containsKey(new Word("cat")) 
    && table.get(new Word("cat")).equals("a small domesticated carnivorous mammal")) {
    table.put(new Word("cat"), definition);
}
```

Java 8 之后:

```java
table.replace(new Word("cat"), "a small domesticated carnivorous mammal", definition);
```

### 7.5。`computeIfAbsent()`

这个方法类似于 `putIfabsent()`。但是`putIfabsent()`直接取值，`computeIfAbsent()`取映射函数。它只在检查完密钥后才计算值，这样效率更高，尤其是在很难获得值的情况下。

```java
table.computeIfAbsent(new Word("cat"), key -> "an animal");
```

因此，上面的行相当于:

```java
if (!table.containsKey(cat)) {
    String definition = "an animal"; // note that calculations take place inside if block
    table.put(new Word("cat"), definition);
}
```

### 7.6。`computeIfPresent()`

这种方法类似于`replace()`方法。但是，`replace()`直接取值，`computeIfPresent()`取映射函数。它计算`if`块内部的值，这就是它更高效的原因。

假设我们需要改变定义:

```java
table.computeIfPresent(cat, (key, value) -> key.getName() + " - " + value);
```

因此，上面的行相当于:

```java
if (table.containsKey(cat)) {
    String concatination=cat.getName() + " - " + table.get(cat);
    table.put(cat, concatination);
}
```

### 7.7。`compute()`

现在我们将解决另一个任务。假设我们有一个数组`String`，其中的元素不是唯一的。同样，让我们计算一个字符串在数组中出现多少次。这是数组:

```java
String[] animals = { "cat", "dog", "dog", "cat", "bird", "mouse", "mouse" };
```

此外，我们想创建一个`Hashtable`，它包含一个动物作为键，它出现的次数作为值。

这里有一个解决方案:

```java
Hashtable<String, Integer> table = new Hashtable<String, Integer>();

for (String animal : animals) {
    table.compute(animal, 
        (key, value) -> (value == null ? 1 : value + 1));
}
```

最后，让我们确定一下，这个表包含两只猫、两只狗、一只鸟和两只老鼠:

```java
assertThat(table.values(), hasItems(2, 2, 2, 1));
```

### 7.8。`merge()`

还有另一种方法来解决上述任务:

```java
for (String animal : animals) {
    table.merge(animal, 1, (oldValue, value) -> (oldValue + value));
}
```

第二个参数`1`是映射到该键的值，如果该键还不在表上的话。如果键已经在表中，那么我们计算它为`oldValue+1`。

### 7.9。`foreach()`

这是一种遍历条目的新方法。让我们打印所有条目:

```java
table.forEach((k, v) -> System.out.println(k.getName() + " - " + v)
```

### 7.10。`replaceAll()`

此外，我们可以替换所有值，而无需迭代:

```java
table.replaceAll((k, v) -> k.getName() + " - " + v);
```

## 8。结论

在本文中，我们已经描述了哈希表结构的用途，并展示了如何通过复杂的直接地址表结构来获得它。

此外，我们已经讨论了什么是碰撞，什么是`Hashtable.`中的负载因子，我们还学习了为什么要为关键对象覆盖`equals()`和`hashCode()`。

最后，我们讨论了`Hashtable`的属性和 Java 8 特定的 API。

和往常一样，完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections)