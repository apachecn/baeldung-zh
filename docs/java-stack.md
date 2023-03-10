# Java 堆栈快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stack>

## 1。概述

在这篇简短的文章中，我们将介绍`java.util.Stack`类，并开始研究如何利用它。

一个[栈](/web/20220630131039/https://www.baeldung.com/cs/stack-data-structure)是一个通用数据结构，它代表一个 LIFO(后进先出)对象集合，允许在固定时间内推入/弹出元素。

对于新的实现，**我们更喜欢** **`Deque`** **接口及其实现**。 `Deque` 定义了一套更加完整和一致的后进先出操作。然而，我们可能仍然需要处理 `Stack` 类，特别是在遗留代码中，所以很好地理解它很重要。

## 2。创建一个堆栈

让我们从创建一个空的`Stack`实例开始，使用默认的无参数构造函数:

```java
@Test
public void whenStackIsCreated_thenItHasSizeZero() {
    Stack<Integer> intStack = new Stack<>();

    assertEquals(0, intStack.size());
}
```

这将**创建一个默认容量为 10 的`Stack`。**如果添加的元素数量超过总*栈*大小，将自动加倍。但是，在移除元素后，它的大小永远不会缩小。

## 3。堆栈同步

`Stack`是`Vector`的直接子类；这意味着**类似于它的超类，它是** `synchronized` **的实现。**

然而，并不总是需要同步，在这种情况下，建议使用`ArrayDeque`。

## 4。添加到堆栈中

让我们从使用`push()`方法向`Stack`的顶部添加一个元素开始——该方法也返回添加的元素:

```java
@Test
public void whenElementIsPushed_thenStackSizeIsIncreased() {
    Stack<Integer> intStack = new Stack<>();
    intStack.push(1);

    assertEquals(1, intStack.size());
}
```

使用`push()`方法与使用`addElement(). **T**` **效果相同，唯一的区别是`addElement()`返回操作的结果，而不是被添加的元素。**

我们还可以一次添加多个元素:

```java
@Test
public void whenMultipleElementsArePushed_thenStackSizeIsIncreased() {
    Stack<Integer> intStack = new Stack<>();
    List<Integer> intList = Arrays.asList(1, 2, 3, 4, 5, 6, 7);

    boolean result = intStack.addAll(intList);

    assertTrue(result);
    assertEquals(7, intList.size());
}
```

## 5。从堆栈中检索

接下来，让我们看看如何获取和移除一个`Stack`中的最后一个元素:

```java
@Test
public void whenElementIsPoppedFromStack_thenElementIsRemovedAndSizeChanges() {
    Stack<Integer> intStack = new Stack<>();
    intStack.push(5);

    Integer element = intStack.pop();

    assertEquals(Integer.valueOf(5), element);
    assertTrue(intStack.isEmpty());
}
```

我们也可以不移除 S `tack`的最后一个元素:

```java
@Test
public void whenElementIsPeeked_thenElementIsNotRemovedAndSizeDoesNotChange() {
    Stack<Integer> intStack = new Stack<>();
    intStack.push(5);

    Integer element = intStack.peek();

    assertEquals(Integer.valueOf(5), element);
    assertEquals(1, intStack.search(5));
    assertEquals(1, intStack.size());
}
```

## 6。在堆栈中搜索元素

### 6.1。搜索

*Stack* 允许我们搜索一个元素并得到它与顶部的距离:

```java
@Test
public void whenElementIsOnStack_thenSearchReturnsItsDistanceFromTheTop() {
    Stack<Integer> intStack = new Stack<>();
    intStack.push(5);
    intStack.push(8);

    assertEquals(2, intStack.search(5));
}
```

结果是给定对象的索引。如果存在不止一个元素，则返回最接近顶部的一个的索引。栈顶的项目被认为是在位置 1。

如果没有找到对象，`search()` 将返回-1。

### 6.2。获取元素的索引

为了获得 S `tack,` 上元素的索引，我们也可以使用 `indexOf()` 和`lastIndexOf()`方法:

```java
@Test
public void whenElementIsOnStack_thenIndexOfReturnsItsIndex() {
    Stack<Integer> intStack = new Stack<>();
    intStack.push(5);

    int indexOf = intStack.indexOf(5);

    assertEquals(0, indexOf);
}
```

**`lastIndexOf()`将总是找到最靠近栈顶的元素的索引**。这与`search()`非常相似——重要的区别在于它返回的是索引，而不是与顶部的距离:

```java
@Test
public void whenMultipleElementsAreOnStack_thenIndexOfReturnsLastElementIndex() {
    Stack<Integer> intStack = new Stack<>();
    intStack.push(5);
    intStack.push(5);
    intStack.push(5);

    int lastIndexOf = intStack.lastIndexOf(5);

    assertEquals(2, lastIndexOf);
}
```

## 7 .**。从堆栈中移除元素**

除了用于移除和检索元素的`pop()`操作之外，我们还可以使用从`Vector`类继承的多个操作来移除元素。

### 7.1。移除指定元素

我们可以使用`removeElement()`方法删除第一次出现的给定元素:

```java
@Test
public void whenRemoveElementIsInvoked_thenElementIsRemoved() {
    Stack<Integer> intStack = new Stack<>();
    intStack.push(5);
    intStack.push(5);

    intStack.removeElement(5);

    assertEquals(1, intStack.size());
}
```

我们也可以使用`removeElementAt()`来删除`Stack:`中指定索引下的元素

```java
 @Test
    public void whenRemoveElementAtIsInvoked_thenElementIsRemoved() {
        Stack<Integer> intStack = new Stack<>();
        intStack.push(5);
        intStack.push(7);

        intStack.removeElementAt(1);

        assertEquals(-1, intStack.search(7));
    }
```

### 7.2。移除多个元素

让我们快速看一下如何使用`removeAll()` API 从`Stack` 中移除多个元素——它将把*集合*作为参数，并从`Stack`中移除所有匹配的元素:

```java
@Test
public void givenElementsOnStack_whenRemoveAllIsInvoked_thenAllElementsFromCollectionAreRemoved() {
    Stack<Integer> intStack = new Stack<>();
    List<Integer> intList = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
    intStack.addAll(intList);
    intStack.add(500);

    intStack.removeAll(intList);

    assertEquals(1, intStack.size());
    assertEquals(1, intStack.search(500));
}
```

也可以使用`clear()` 或`removeAllElements()`方法从`Stack`中移除所有元素**；这两种方法的工作原理相同:**

```java
@Test
public void whenRemoveAllElementsIsInvoked_thenAllElementsAreRemoved() {
    Stack<Integer> intStack = new Stack<>();
    intStack.push(5);
    intStack.push(7);

    intStack.removeAllElements();

    assertTrue(intStack.isEmpty());
}
```

### 7.3。使用过滤器移除元素

我们还可以使用一个条件来从`Stack.`中删除元素，让我们看看如何使用`removeIf` `()`来实现这一点，使用一个过滤表达式作为参数:

```java
@Test
public void whenRemoveIfIsInvoked_thenAllElementsSatysfyingConditionAreRemoved() {
    Stack<Integer> intStack = new Stack<>();
    List<Integer> intList = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
    intStack.addAll(intList);

    intStack.removeIf(element -> element < 6);

    assertEquals(2, intStack.size());
}
```

## 8。迭代堆栈

`Stack`允许我们使用一个`Iterator` 和一个`ListIterator.` ，主要区别是第一个允许我们在一个方向上移动`Stack`，第二个允许我们在两个方向上移动:

```java
@Test
public void whenAnotherStackCreatedWhileTraversingStack_thenStacksAreEqual() {
    Stack<Integer> intStack = new Stack<>();
    List<Integer> intList = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
    intStack.addAll(intList);

    ListIterator<Integer> it = intStack.listIterator();

    Stack<Integer> result = new Stack<>();
    while(it.hasNext()) {
        result.push(it.next());
    }

    assertThat(result, equalTo(intStack));
}
```

由`Stack` 返回的所有*迭代器*都是快速失效的。

## 9。Java 栈的流 API

`Stack`是一个集合，这意味着我们可以用 Java 8 `Streams` API 来使用它。**将`Stream`与 `Stack`一起使用类似于将它与任何其他`Collection:`** 一起使用

```java
@Test
public void whenStackIsFiltered_allElementsNotSatisfyingFilterConditionAreDiscarded() {
    Stack<Integer> intStack = new Stack<>();
    List<Integer> inputIntList = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 9, 10);
    intStack.addAll(inputIntList);

    List<Integer> filtered = intStack
      .stream()
      .filter(element -> element <= 3)
      .collect(Collectors.toList());

    assertEquals(3, filtered.size());
}
```

## 10。总结

本教程是理解 Java 核心类`Stack`的快速实用指南。

当然，您可以在 [Javadoc](https://web.archive.org/web/20220630131039/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Stack.html) 中探索完整的 API。

和往常一样，所有代码样本都可以在 GitHub 上找到[。](https://web.archive.org/web/20220630131039/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)