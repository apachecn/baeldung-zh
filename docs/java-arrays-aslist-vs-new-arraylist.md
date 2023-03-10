# arrays . aslist vs . new ArrayList(arrays . aslist())

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arrays-aslist-vs-new-arraylist>

## 1.概观

在这个简短的教程中，我们将看看`Arrays.asList(array)` 和`ArrayList(Arrays.asList(array)).`之间的区别

## 2.`Arrays.asList`

先说 [`Arrays.asList`](https://web.archive.org/web/20220625230045/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#asList(T...)) 法。

使用这个方法，我们可以从一个数组转换成一个固定大小的`List` 对象`.` **`List `只是一个包装器，使数组作为一个列表可用。没有数据被复制或创建**。

此外，我们不能修改它的长度，因为**不允许添加或删除元素**。

但是，我们可以修改数组中的单个项目。请注意，我们对`List`的单项所做的所有**修改都将反映在我们的原始数组**中:

```java
String[] stringArray = new String[] { "A", "B", "C", "D" };
List stringList = Arrays.asList(stringArray); 
```

现在，让我们看看修改`stringList`的第一个元素会发生什么:

```java
stringList.set(0, "E");

assertThat(stringList).containsExactly("E", "B", "C", "D");
assertThat(stringArray).containsExactly("E", "B", "C", "D");
```

如我们所见，我们的原始数组也被修改了。列表和数组现在都以相同的顺序包含完全相同的元素。

现在让我们尝试向`stringList`插入一个新元素:

```java
stringList.add("F");
```

```java
java.lang.UnsupportedOperationException
	at java.base/java.util.AbstractList.add(AbstractList.java:153)
	at java.base/java.util.AbstractList.add(AbstractList.java:111)
```

正如我们所看到的，**向/从`List`添加/删除元素会抛出** `**java.lang.UnsupportedOperationException**.`

## 3.`ArrayList(Arrays.asList(array))`

类似于`Arrays.asList`方法，当我们需要从数组中创建一个`List`时，我们可以使用`ArrayList<>(Arrays.asList(array))` 。

但是，与我们之前的例子不同，这是数组的独立副本，这意味着**修改新列表不会影响原始数组**。此外，我们拥有常规`ArrayList,` 的所有功能，比如添加和删除元素:

```java
String[] stringArray = new String[] { "A", "B", "C", "D" }; 
List stringList = new ArrayList<>(Arrays.asList(stringArray)); 
```

现在让我们修改`stringList`的第一个元素:

```java
stringList.set(0, "E");

assertThat(stringList).containsExactly("E", "B", "C", "D");
```

现在，让我们看看原始阵列发生了什么:

```java
assertThat(stringArray).containsExactly("A", "B", "C", "D");
```

正如我们所见，**我们的原始数组保持不变**。

在结束之前，如果我们看一下 [JDK 源代码](https://web.archive.org/web/20220625230045/http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/Arrays.java#l3791)，我们可以看到`Arrays.asList`方法返回一种不同于 [`java.util.ArrayList`](https://web.archive.org/web/20220625230045/http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/ArrayList.java#l106) 的`[ArrayList](https://web.archive.org/web/20220625230045/http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/Arrays.java#l3800)`。主要区别在于返回的`ArrayList`只包装了一个现有的数组——它没有实现`add`和`remove`方法。

## 4.结论

在这篇短文中，我们看了一下将数组转换成`ArrayList`的两种方法之间的区别。我们看到了这两个选项的行为方式，以及它们实现内部数组的方式之间的差异。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220625230045/https://github.com/eugenp/tutorials/tree/master/core-java-modules/java-collections-conversions-2)