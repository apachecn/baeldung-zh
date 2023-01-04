# 在 Java 中查找链表的中间元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-linked-list-middle-element>

## 1。概述

在本教程中，我们将解释如何在 Java 中找到链表的中间元素。

我们将在接下来的章节中介绍主要问题，并展示解决这些问题的不同方法。

## 2。记录尺寸

当我们向列表添加新元素时，这个问题可以通过**跟踪大小来轻松解决。如果我们知道大小，我们也知道中间元素在哪里，所以解是平凡的。**

让我们看一个使用 Java 实现`LinkedList`的例子:

```
public static Optional<String> findMiddleElementLinkedList(
  LinkedList<String> linkedList) {
    if (linkedList == null || linkedList.isEmpty()) {
        return Optional.empty();
    }

    return Optional.of(linkedList.get(
      (linkedList.size() - 1) / 2));
}
```

如果我们检查`LinkedList`类的内部代码，我们可以看到，在这个例子中，我们只是遍历列表，直到到达中间元素:

```
Node<E> node(int index) {
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++) {
            x = x.next;
        }
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--) {
            x = x.prev;
        }
        return x;
    }
}
```

## 3.在不知道尺寸的情况下找到中间

我们经常会遇到这样的问题:我们只有一个链表的头节点，我们需要找到中间的元素。在这种情况下，我们不知道列表的大小，这使得这个问题更难解决。

我们将在接下来的部分展示解决这个问题的几种方法，但是首先，我们需要创建一个类来表示列表的一个节点。

让我们创建一个`Node`类，它存储`String`值:

```
public static class Node {

    private Node next;
    private String data;

    // constructors/getters/setters

    public boolean hasNext() {
        return next != null;
    }

    public void setNext(Node next) {
        this.next = next;
    }

    public String toString() {
        return this.data;
    }
}
```

此外，我们将在测试用例中使用这个助手方法来创建一个只使用我们的节点的单链表:

```
private static Node createNodesList(int n) {
    Node head = new Node("1");
    Node current = head;

    for (int i = 2; i <= n; i++) {
        Node newNode = new Node(String.valueOf(i));
        current.setNext(newNode);
        current = newNode;
    }

    return head;
}
```

### 3.1。先找到尺寸

解决这个问题的最简单的方法是首先找到列表的大小，然后遵循我们之前使用的相同方法——迭代直到中间的元素。

让我们来看看这个解决方案的实际应用:

```
public static Optional<String> findMiddleElementFromHead(Node head) {
    if (head == null) {
        return Optional.empty();
    }

    // calculate the size of the list
    Node current = head;
    int size = 1;
    while (current.hasNext()) {
        current = current.next();
        size++;
    }

    // iterate till the middle element
    current = head;
    for (int i = 0; i < (size - 1) / 2; i++) {
        current = current.next();
    }

    return Optional.of(current.data());
}
```

正如我们所见，**这段代码遍历了列表两次。因此，这种解决方案性能较差，不推荐使用**。

### 3.2。在一次迭代中找到中间元素

我们现在要改进前面的解决方案，找到中间的元素，只对列表进行一次迭代。

为了迭代地实现这一点，我们需要两个指针同时遍历列表。**一个指针每次迭代前进 2 个节点，另一个指针每次迭代只前进一个节点**。

当较快的指针到达列表末尾时，较慢的指针将位于中间:

```
public static Optional<String> findMiddleElementFromHead1PassIteratively(Node head) {
    if (head == null) {
        return Optional.empty();
    }

    Node slowPointer = head;
    Node fastPointer = head;

    while (fastPointer.hasNext() && fastPointer.next().hasNext()) {
        fastPointer = fastPointer.next().next();
        slowPointer = slowPointer.next();
    }

    return Optional.ofNullable(slowPointer.data());
}
```

我们可以用一个简单的单元测试来测试这个解决方案，使用奇数和偶数元素的列表:

```
@Test
public void whenFindingMiddleFromHead1PassIteratively_thenMiddleFound() {

    assertEquals("3", MiddleElementLookup
      .findMiddleElementFromHead1PassIteratively(
        createNodesList(5)).get());
    assertEquals("2", MiddleElementLookup
      .findMiddleElementFromHead1PassIteratively(
        reateNodesList(4)).get());
}
```

### 3.3。递归查找一遍中的中间元素

另一种一次性解决这个问题的方法是使用递归。我们可以迭代到列表的末尾来知道大小，在回调中，我们只计数到一半的大小。

为了在 Java 中做到这一点，我们将创建一个辅助类，在所有递归调用的执行过程中保存列表大小和中间元素的引用:

```
private static class MiddleAuxRecursion {
    Node middle;
    int length = 0;
}
```

现在，让我们实现递归方法:

```
private static void findMiddleRecursively(
  Node node, MiddleAuxRecursion middleAux) {
    if (node == null) {
        // reached the end
        middleAux.length = middleAux.length / 2;
        return;
    }
    middleAux.length++;
    findMiddleRecursively(node.next(), middleAux);

    if (middleAux.length == 0) {
        // found the middle
        middleAux.middle = node;
    }

    middleAux.length--;
}
```

最后，让我们创建一个调用递归方法的方法:

```
public static Optional<String> findMiddleElementFromHead1PassRecursively(Node head) {

    if (head == null) {
        return Optional.empty();
    }

    MiddleAuxRecursion middleAux = new MiddleAuxRecursion();
    findMiddleRecursively(head, middleAux);
    return Optional.of(middleAux.middle.data());
}
```

同样，我们可以像以前一样测试它:

```
@Test
public void whenFindingMiddleFromHead1PassRecursively_thenMiddleFound() {
    assertEquals("3", MiddleElementLookup
      .findMiddleElementFromHead1PassRecursively(
        createNodesList(5)).get());
    assertEquals("2", MiddleElementLookup
      .findMiddleElementFromHead1PassRecursively(
        createNodesList(4)).get());
}
```

## 4。结论

在本文中，我们介绍了在 Java 中查找链表中间元素的问题，并展示了解决这个问题的不同方法。

我们从跟踪大小的最简单的方法开始，之后，我们继续使用解决方案从列表的头节点找到中间的元素。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220626110128/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-4)