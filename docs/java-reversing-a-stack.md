# 在 Java 中反转堆栈

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reversing-a-stack>

## 1.概观

在本文中，我们将研究使用 Java 反转堆栈的不同方法。一个[栈](/web/20221212031550/https://www.baeldung.com/java-stack)是一个 LIFO(后进先出)数据结构，支持从同侧插入(推入)和移除(弹出)元素。

我们可以把一堆想象成桌子上的一堆盘子；从顶部接近金属板是最安全的。

## 2.问题:反转堆栈

让我们深入探讨一下问题陈述。给我们一个`Stack`对象作为输入，我们需要以相反的顺序返回堆栈中的元素。这里有一个例子。

输入:[1，2，3，4，5，6，7，8，9]

输出:[9，8，7，6，5，4，3，2，1]

输入是前九个自然数的堆栈，我们代码的输出应该是逆序的相同自然数。**我们可以将这个问题扩展到任何类型的堆栈，例如，`String` 元素的堆栈，`,`自定义对象的堆栈，如`Node,`等。**

例如:

输入:["红色"、"蓝色"、"绿色"、"黄色"]

输出:["黄色"、"绿色"、"蓝色"、"红色"]

## 3.使用`Queue`反转堆栈

在本节中，让我们看看如何通过使用一个 [`Queue`](/web/20221212031550/https://www.baeldung.com/java-queue) 数据结构来解决这个问题。`Queue`是一种 FIFO(先进先出)数据结构，支持从后侧添加元素和从前侧移除元素。

给定一个元素栈作为输入，**我们可以从栈顶挑选元素，一次一个，并将它们插入到我们的队列**中。从自然数的第一个例子开始，我们将从指向 9 的栈顶开始。我们在每一步都将堆栈的顶部元素插入队列的后端，最终，我们将清空堆栈。此时，我们已经按以下顺序用元素填充了队列:

(后)[1，2，3，4，5，6，7，8，9](前)。

完成后，**我们可以从队列中移除元素，这发生在前端，并将其推回我们的堆栈**。在这个活动完成之后，我们将得到我们想要的输出堆栈[9，8，7，6，5，4，3，2，1]。

这是代码看起来的样子，考虑到堆栈是类型`Integer`:

```java
public Stack reverseIntegerStack(Stack<Integer> inputStack) {
    Queue<Integer> queue = new LinkedList<>();
    while (!inputStack.isEmpty()) {
        queue.add(inputStack.pop());
    }
    while (!queue.isEmpty()) {
        inputStack.add(queue.remove());
    }
    return inputStack;
} 
```

## 4.使用递归反转堆栈

让我们讨论一种不使用任何额外的数据结构来解决这个问题的方法。 **[递归](/web/20221212031550/https://www.baeldung.com/java-recursion)是计算机科学中丰富的一个核心概念，处理方法只要满足一个前提条件就反复调用自身的思想**。任何递归函数都应该有两个组成部分:

*   递归调用:在我们的例子中，对方法的递归调用将从给定的堆栈中删除元素
*   停止条件:当给定的堆栈为空时，递归将结束

对递归方法的每次调用都会添加到 JVM 内存中的调用堆栈中。我们可以利用这个事实来反转给定的堆栈。递归调用将从堆栈顶部删除一个元素，并将其添加到内存调用堆栈中。

**当输入栈为空时，内存调用栈包含逆序的栈元素。**调用堆栈的顶部包含数字 1，而最底部的调用堆栈包含数字 10。此时，我们从调用堆栈**中取出项目，并将元素插入堆栈底部，颠倒元素的原始顺序。**

让我们看看代码中的两步递归算法:

```java
private void reverseStack(Stack<Integer> stack) {
    if (stack.isEmpty()) {
        return;
    }
    int top = stack.pop();
    reverseStack(stack);
    insertBottom(stack, top);
}
```

`reverseStack() `方法递归地从堆栈中弹出顶部元素。一旦输入堆栈为空，我们将当前在调用堆栈中的元素插入到堆栈的底部:

```java
private void insertBottom(Stack<Integer> stack, int value) {
    if (stack.isEmpty()) {
        stack.add(value);
    } else {
        int top = stack.pop();
        insertBottom(stack, value);
        stack.add(top);
    }
}
```

## 5.比较反转堆栈的方法

我们讨论了反转给定堆栈问题的两种方法。这些算法适用于任何类型的`Stack`。**使用额外的`Queue`数据结构的第一种解决方案以 O(n)时间复杂度**反转堆栈。然而，由于我们使用了`Queue`形式的额外空间，空间复杂度也是 O(n)。

另一方面，由于递归调用**，递归解决方案具有 O(n ) 的时间复杂度[，但是没有额外的空间复杂度](/web/20221212031550/https://www.baeldung.com/cs/master-theorem-asymptotic-analysis)**，因为我们利用程序调用堆栈来存储堆栈的元素。

## 6.结论

在本文中，我们讨论了在 Java 中反转 a `Stack`的两种方法，并比较了算法的运行时间和空间复杂度。像往常一样，所有代码样本都可以在 GitHub 上找到[。](https://web.archive.org/web/20221212031550/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)