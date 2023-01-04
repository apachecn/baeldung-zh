# 循环链表的 Java 实现

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-circular-linked-list>

## 1.介绍

在本教程中，我们将看看循环链表在 Java 中的实现。

## 2.循环链表

**循环链表是[链表](/web/20221208143940/https://www.baeldung.com/java-linkedlist)的变体，其中最后一个节点指向第一个节点，完成一整圈节点**。换句话说，这个链表的变体在末尾没有一个`null`元素。

通过这一简单的改变，我们获得了一些好处:

*   **循环链表中的任何节点都可以是起点**
*   因此，可以从任何节点开始遍历整个列表
*   因为循环链表的最后一个节点有指向第一个节点的指针，所以很容易执行入队和出队操作

总之，**这在队列数据结构的实现中非常有用。**

在性能方面，除了一点之外，它与其他链表实现相同:**从最后一个节点遍历到头节点可以在常数时间内完成**。对于传统的链表，这是一个线性操作。

## 3.用 Java 实现

让我们首先创建一个辅助的`Node`类，它将存储`int`值和一个指向下一个节点`:`的指针

```
class Node {

    int value;
    Node nextNode;

    public Node(int value) {
        this.value = value;
    }
}
```

现在让我们创建循环链表中的第一个和最后一个节点，通常称为`head`和`tail:`

```
public class CircularLinkedList {
    private Node head = null;
    private Node tail = null;

    // ....
}
```

在接下来的小节中，我们将看看我们可以在循环链表上执行的最常见的操作。

### 3.1.插入元素

我们要介绍的第一个操作是插入新节点。在插入新元素时，我们需要处理两种情况:

*   **`head`节点为空**，即没有已经添加的元素。在这种情况下，我们将添加的新节点作为列表的`head`和`tail`，因为只有一个节点
*   **`head`节点不为空**，也就是说，有一个或多个元素已经添加到列表中。在这种情况下，现有的`tail`应该指向新节点，新添加的节点将成为`tail`

在上述两种情况下，`tail`的`nextNode`将指向`head`

让我们创建一个`addNode`方法，它将待插入的`value`作为参数:

```
public void addNode(int value) {
    Node newNode = new Node(value);

    if (head == null) {
        head = newNode;
    } else {
        tail.nextNode = newNode;
    }

    tail = newNode;
    tail.nextNode = head;
}
```

现在，我们可以向循环链表添加一些数字:

```
private CircularLinkedList createCircularLinkedList() {
    CircularLinkedList cll = new CircularLinkedList();

    cll.addNode(13);
    cll.addNode(7);
    cll.addNode(24);
    cll.addNode(1);
    cll.addNode(8);
    cll.addNode(37);
    cll.addNode(46);

    return cll;
}
```

### 3.2.寻找元素

我们要看的下一个操作是搜索以确定一个元素是否出现在列表中。

为此，**我们将把列表中的一个节点(通常是`head`)固定为`currentNode `，并使用这个节点**的`nextNode`遍历整个列表，直到找到所需的元素。

让我们添加一个新方法`containsNode`，它将`searchValue`作为参数:

```
public boolean containsNode(int searchValue) {
    Node currentNode = head;

    if (head == null) {
        return false;
    } else {
        do {
            if (currentNode.value == searchValue) {
                return true;
            }
            currentNode = currentNode.nextNode;
        } while (currentNode != head);
        return false;
    }
}
```

现在，让我们添加几个测试来验证上面创建的列表包含我们添加的元素，并且没有新的元素:

```
@Test
 public void givenACircularLinkedList_WhenAddingElements_ThenListContainsThoseElements() {
    CircularLinkedList cll = createCircularLinkedList();

    assertTrue(cll.containsNode(8));
    assertTrue(cll.containsNode(37));
}

@Test
public void givenACircularLinkedList_WhenLookingForNonExistingElement_ThenReturnsFalse() {
    CircularLinkedList cll = createCircularLinkedList();

    assertFalse(cll.containsNode(11));
}
```

### 3.3.删除元素

接下来，我们来看看删除操作。

**一般来说，我们删除一个元素后，需要更新前一个节点的`nextNode`引用，指向被删除节点的`nextNode`引用。**

然而，我们需要考虑一些特殊情况:

*   **循环链表只有一个元素，我们想要移除元素**——在这种情况下，我们只需要将`head`节点和`tail`节点设置为`null`
*   **要删除的元素是`head`节点**——我们必须使`head.nextNode`成为新的`head`
*   **要删除的元素是`tail`节点**——我们需要将我们要删除的节点的前一个节点作为新的`tail`

让我们来看看删除一个元素的实现:

```
public void deleteNode(int valueToDelete) {
    Node currentNode = head;
    if (head == null) { // the list is empty
        return;
    }
    do {
        Node nextNode = currentNode.nextNode;
        if (nextNode.value == valueToDelete) {
            if (tail == head) { // the list has only one single element
                head = null;
                tail = null;
            } else {
                currentNode.nextNode = nextNode.nextNode;
                if (head == nextNode) { //we're deleting the head
                    head = head.nextNode;
                }
                if (tail == nextNode) { //we're deleting the tail
                    tail = currentNode;
                }
            }
            break;
        }
        currentNode = nextNode;
    } while (currentNode != head);
}
```

现在，让我们创建一些测试来验证删除在所有情况下都能正常工作:

```
@Test
public void givenACircularLinkedList_WhenDeletingInOrderHeadMiddleTail_ThenListDoesNotContainThoseElements() {
    CircularLinkedList cll = createCircularLinkedList();

    assertTrue(cll.containsNode(13));
    cll.deleteNode(13);
    assertFalse(cll.containsNode(13));

    assertTrue(cll.containsNode(1));
    cll.deleteNode(1);
    assertFalse(cll.containsNode(1));

    assertTrue(cll.containsNode(46));
    cll.deleteNode(46);
    assertFalse(cll.containsNode(46));
}

@Test
public void givenACircularLinkedList_WhenDeletingInOrderTailMiddleHead_ThenListDoesNotContainThoseElements() {
    CircularLinkedList cll = createCircularLinkedList();

    assertTrue(cll.containsNode(46));
    cll.deleteNode(46);
    assertFalse(cll.containsNode(46));

    assertTrue(cll.containsNode(1));
    cll.deleteNode(1);
    assertFalse(cll.containsNode(1));

    assertTrue(cll.containsNode(13));
    cll.deleteNode(13);
    assertFalse(cll.containsNode(13));
}

@Test
public void givenACircularLinkedListWithOneNode_WhenDeletingElement_ThenListDoesNotContainTheElement() {
    CircularLinkedList cll = new CircularLinkedList();
    cll.addNode(1);
    cll.deleteNode(1);
    assertFalse(cll.containsNode(1));
}
```

### 3.4.遍历列表

**在最后一节**中，我们将看看循环链表的遍历。类似于搜索和删除操作，对于遍历，我们将`currentNode`固定为`head`，并使用该节点的`nextNode`遍历整个列表。

让我们添加一个新方法`traverseList`,它打印添加到列表中的元素:

```
public void traverseList() {
    Node currentNode = head;

    if (head != null) {
        do {
            logger.info(currentNode.value + " ");
            currentNode = currentNode.nextNode;
        } while (currentNode != head);
    }
} 
```

正如我们所看到的，在上面的例子中，在遍历过程中，我们只是打印每个节点的值，直到我们回到头节点。

## 4.结论

在本教程中，我们看到了如何在 Java 中实现循环链表，并探讨了一些最常见的操作。

首先，我们学习了循环链表到底是什么，包括一些最常见的特性和与传统链表的区别。然后，我们看到了如何在循环链表实现中插入、搜索、删除和遍历条目。

像往常一样，本文中使用的所有例子都可以在 GitHub 上找到。