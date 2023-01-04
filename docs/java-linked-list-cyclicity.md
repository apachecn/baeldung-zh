# 测试链表的循环性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-linked-list-cyclicity>

## 1。简介

单链表是一系列以引用`null`结束的连接节点。然而，在某些情况下，最后一个节点可能指向前一个节点，这实际上形成了一个循环。

在大多数情况下，我们希望能够检测和意识到这些周期；本文将集中讨论这一点——检测并潜在地消除循环。

## 2。检测周期

现在让我们来探索几个用于[检测链表](/web/20221128051921/https://www.baeldung.com/cs/find-cycle-in-list)中的循环的算法。

### 2.1。蛮力-o(n^2)时间复杂度

在这个算法中，我们使用两个嵌套循环来遍历列表。在外部循环中，我们逐个遍历。在内部循环中，我们从头开始，遍历与外部循环相同数量的节点。

**如果外部循环访问的节点被内部循环访问两次，则检测到循环。**相反，如果外部循环到达列表的末尾，这意味着没有循环:

```
public static <T> boolean detectCycle(Node<T> head) {
    if (head == null) {
        return false;
    }

    Node<T> it1 = head;
    int nodesTraversedByOuter = 0;
    while (it1 != null && it1.next != null) {
        it1 = it1.next;
        nodesTraversedByOuter++;

        int x = nodesTraversedByOuter;
        Node<T> it2 = head;
        int noOfTimesCurrentNodeVisited = 0;

        while (x > 0) {
            it2 = it2.next;

            if (it2 == it1) {
                noOfTimesCurrentNodeVisited++;
            }

            if (noOfTimesCurrentNodeVisited == 2) {
                return true;
            }

            x--;
        }
    }

    return false;
}
```

这种方法的优点是它需要恒定的内存量。缺点是当提供大列表作为输入时，性能非常慢。

### 2.2。哈希——O(n)空间复杂度

通过这个算法，我们维护了一组已经访问过的节点。对于每个节点，我们检查它是否存在于集合中。如果没有，那么我们把它添加到集合中。集合中节点的存在意味着我们已经访问了该节点，并在列表中提出了循环的存在。

当我们遇到一个已经存在于集合中的节点时，我们就发现了循环的开始。发现这一点后，我们可以通过将前一个节点的`next`字段设置为`null`来轻松打破循环，如下所示:

```
public static <T> boolean detectCycle(Node<T> head) {
    if (head == null) {
        return false;
    }

    Set<Node<T>> set = new HashSet<>();
    Node<T> node = head;

    while (node != null) {
        if (set.contains(node)) {
            return true;
        }
        set.add(node);
        node = node.next;
    }

    return false;
}
```

在这个解决方案中，我们访问并存储每个节点一次。这相当于 O(n)的时间复杂度和 O(n)的空间复杂度，平均来说，这对于大型列表来说不是最佳的。

### 2.3。快速和慢速指针

下面的寻找循环的算法可以用一个比喻来最好地解释**。**

考虑两个人正在比赛的赛道。假设第二个人的速度是第一个人的两倍，那么第二个人绕跑道的速度将是第一个人的两倍，并将在一圈开始时再次与第一个人相遇。

这里我们使用类似的方法，用一个慢速迭代器和一个快速迭代器(2 倍速度)同时遍历列表。一旦两个迭代器都进入了一个循环，它们最终会在一个点相遇。

因此，如果两个迭代器在任何一点相遇，那么我们可以得出结论，我们遇到了一个循环:

```
public static <T> CycleDetectionResult<T> detectCycle(Node<T> head) {
    if (head == null) {
        return new CycleDetectionResult<>(false, null);
    }

    Node<T> slow = head;
    Node<T> fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow == fast) {
            return new CycleDetectionResult<>(true, fast);
        }
    }

    return new CycleDetectionResult<>(false, null);
}
```

其中`CycleDetectionResult` 是保存结果的便利类:一个`boolean`变量，它表示循环是否存在，如果存在，那么它还包含对循环内的会合点的引用:

```
public class CycleDetectionResult<T> {
    boolean cycleExists;
    Node<T> node;
}
```

这种方法也被称为“龟兔算法”或“Flyods 循环查找算法”。

## 3。从列表中删除循环

让我们来看看几种去除周期的方法。**所有这些方法都假设“Flyods 循环查找算法”用于循环检测，并在此基础上构建。**

### 3.1。蛮力

一旦快速迭代器和慢速迭代器在循环中的某一点相遇，我们就多取一个迭代器(比如说`ptr`)并将其指向列表的头部。我们开始用 ptr 迭代链表。在每一步，我们检查是否可以从会合点到达`ptr`。

当`ptr`到达循环的开始时终止，因为这是它进入循环并从会合点可到达的第一个点。

一旦循环的开始(`bg`)被发现，那么找到循环的结束(其下一个字段指向`bg`的节点)是很容易的。然后，该结束节点的下一个指针被设置为`null`以移除循环:

```
public class CycleRemovalBruteForce {
    private static <T> void removeCycle(
      Node<T> loopNodeParam, Node<T> head) {
        Node<T> it = head;

        while (it != null) {
            if (isNodeReachableFromLoopNode(it, loopNodeParam)) {
                Node<T> loopStart = it;
                findEndNodeAndBreakCycle(loopStart);
                break;
            }
            it = it.next;
        }
    }

    private static <T> boolean isNodeReachableFromLoopNode(
      Node<T> it, Node<T> loopNodeParam) {
        Node<T> loopNode = loopNodeParam;

        do {
            if (it == loopNode) {
                return true;
            }
            loopNode = loopNode.next;
        } while (loopNode.next != loopNodeParam);

        return false;
    }

    private static <T> void findEndNodeAndBreakCycle(
      Node<T> loopStartParam) {
        Node<T> loopStart = loopStartParam;

        while (loopStart.next != loopStartParam) {
            loopStart = loopStart.next;
        }

        loopStart.next = null;
    }
}
```

不幸的是，这种算法在大列表和大循环的情况下表现也很差，因为我们必须多次遍历循环。

### 3.2。优化解决方案–计算循环节点数

让我们先定义几个变量:

*   `n` =列表的大小
*   `k` =从列表头到循环开始的距离
*   `l` =周期的大小

我们在这些变量之间有如下关系:
**`k + l = n`**

我们在这种方法中利用了这种关系。更具体地说，当一个迭代器从列表的开头开始，已经遍历了 *l* 个节点，那么它必须再遍历`k`个节点才能到达列表的结尾。

这是算法的概要:

1.  一旦快速迭代器和慢速迭代器相遇，求循环的长度。这可以通过保持其中一个迭代器不动，同时继续另一个迭代器(以正常速度一个接一个地迭代)直到到达第一个指针，保持被访问节点的计数来实现。这计为 *l*
2.  在列表的开头使用两个迭代器(`ptr1`和`ptr2`)。移动迭代器中的一个`ptr2`*l*步骤
3.  现在迭代两个迭代器，直到它们在循环的开始相遇，随后，找到循环的结尾并指向`null`

这是因为`ptr1`离循环有`k`步，而`l`步前进的`ptr2,` 也需要`k`步才能到达循环的终点(`n – l = k`)。

这里有一个简单的、潜在的实现:

```
public class CycleRemovalByCountingLoopNodes {
    private static <T> void removeCycle(
      Node<T> loopNodeParam, Node<T> head) {
        int cycleLength = calculateCycleLength(loopNodeParam);
        Node<T> cycleLengthAdvancedIterator = head;
        Node<T> it = head;

        for (int i = 0; i < cycleLength; i++) {
            cycleLengthAdvancedIterator 
              = cycleLengthAdvancedIterator.next;
        }

        while (it.next != cycleLengthAdvancedIterator.next) {
            it = it.next;
            cycleLengthAdvancedIterator 
              = cycleLengthAdvancedIterator.next;
        }

        cycleLengthAdvancedIterator.next = null;
    }

    private static <T> int calculateCycleLength(
      Node<T> loopNodeParam) {
        Node<T> loopNode = loopNodeParam;
        int length = 1;

        while (loopNode.next != loopNodeParam) {
            length++;
            loopNode = loopNode.next;
        }

        return length;
    }
}
```

接下来，让我们关注一种方法，在这种方法中，我们甚至可以消除计算循环长度的步骤。

### 3.3。优化解决方案–不计算循环节点

让我们用数学方法比较一下快指针和慢指针走过的距离。

为此，我们需要更多的变量:

*   `y` =两个迭代器相遇点的距离，从循环的开始处看
*   `z` =从循环结束时看，两个迭代器交汇点的距离(也等于`l – y`)
*   `m` =在慢速迭代器进入循环之前，快速迭代器完成循环的次数

保持其他变量与上一节中定义的相同，距离方程将定义为:

*   慢速指针移动的距离= `k`(周期距头部的距离)+ `y`(周期内的会合点)
*   快速指针移动的距离= `k`(周期从头开始的距离)+ `m`(慢速指针进入前快速指针完成周期的次数)* `l`(周期长度)+ `y`(周期内的会合点)

我们知道快指针移动的距离是慢指针的两倍，因此:

`k + m * l + y = 2 * (k + y)`

其计算结果为:

`y = m * l – k`

从`l`中减去两边得到:

`l – y = l – m * l + k`

或者相当于:

`k = (m – 1) * l + z (where, l – y is z as defined above)`

这导致:

**T2`k = (m – 1) Full loop runs + An extra distance z`**

换句话说，如果我们在列表的头部保留一个迭代器，在交汇点保留一个迭代器，并以相同的速度移动它们，那么，第二个迭代器将围绕循环完成`m – 1`个循环，并在循环开始时遇到第一个指针。利用这种洞察力，我们可以制定算法:

1.  使用“Flyods 循环查找算法”来检测循环。如果存在循环，该算法将在循环内的某个点结束(称之为会合点)
2.  取两个迭代器，一个在列表的头部(`it1`)，一个在会合点(`it2`)
3.  以相同的速度遍历两个迭代器
4.  由于循环到 head 的距离是 k(如上定义)，从 head 开始的迭代器将在`k`步后到达循环
5.  在`k`步骤中，迭代器`it2`将遍历循环的`m – 1`周期和额外的距离`z.`，因为这个指针已经在距离周期开始`z`的位置，遍历这个额外的距离`z`，也将把它带到周期的开始
6.  两个迭代器在循环的开始相遇，随后，我们可以找到循环的结尾并指向`null`

这可以通过以下方式实现:

```
public class CycleRemovalWithoutCountingLoopNodes {
    private static <T> void removeCycle(
      Node<T> meetingPointParam, Node<T> head) {
        Node<T> loopNode = meetingPointParam;
        Node<T> it = head;

        while (loopNode.next != it.next) {
            it = it.next;
            loopNode = loopNode.next;
        }

        loopNode.next = null;
    }
}
```

这是从链表中检测和移除循环的最佳方法。

## 4。结论

在本文中，我们描述了检测列表中循环的各种算法。我们研究了具有不同计算时间和存储空间需求的算法。

最后，我们还展示了三种删除循环的方法，一旦使用“Flyods 循环查找算法”检测到循环。

Github 上的[提供了完整的代码示例。](https://web.archive.org/web/20221128051921/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-2)