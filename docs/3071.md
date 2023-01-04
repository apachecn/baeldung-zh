# Java 队列与堆栈

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-deque-vs-stack>

## 1.概观

Java `[Stack](/web/20221208143859/https://www.baeldung.com/java-stack)`类实现了[栈数据结构](/web/20221208143859/https://www.baeldung.com/cs/stack-data-structure)。Java 1.6 引入了`[Deque](https://web.archive.org/web/20221208143859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Deque.html)`接口，用于实现一个“双端队列”，支持两端的元素插入和移除。

现在，我们也可以将`Deque`接口用作 LIFO(后进先出)堆栈。此外，如果我们看一下`Stack`类的 [Javadoc，我们会看到:](https://web.archive.org/web/20221208143859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Stack.html)

> 一组更完整和一致的 LIFO 堆栈操作由`Deque`接口及其实现提供，应该优先于该类使用。

在本教程中，我们将比较 Java `Stack`类和`Deque`接口。此外，我们将讨论**为什么对于 LIFO 堆栈**，我们应该使用`Deque`而不是`Stack`。

## 2.类与接口

Java 的`Stack `是一个`Class`:

```
public class Stack<E> extends Vector<E> { ... }
```

也就是说，如果我们想创建自己的`Stack`类型，就必须继承`java.util.Stack`类。

**由于 Java 不支持多重继承，如果我们的类已经是另一个类型**的子类，那么有时很难扩展`Stack`类:

```
public class UserActivityStack extends ActivityCollection { ... }
```

在上面的例子中，`UserActivityStack`类是`ActivityCollection` 类的子类。因此，它也不能扩展`java.util.Stack`类。为了使用 Java `Stack`类，我们可能需要重新设计我们的数据模型。

另一方面，Java 的`Deque`是一个接口:

```
public interface Deque<E> extends Queue<E> { ... }
```

我们知道在 Java 中一个类可以实现多个接口。因此，实现接口比扩展类进行继承更灵活。

例如，我们可以很容易地让我们的`UserActivityStack`实现`Deque`接口:

```
public class UserActivityStack extends ActivityCollection implements Deque<UserActivity> { ... }
```

因此，**从面向对象设计的角度来看，`Deque`接口比`Stack`类**带给我们更多的灵活性。

## 3.`synchronized`方法和性能

我们已经看到`Stack`类是`java.util.Vector`的子类。`Vector`类是同步的。它使用传统的方式来实现线程安全:使其方法“`synchronized.`”

作为其子类，**`Stack`类也是`synchronized`** 。

另一方面，**`Deque`接口不是线程安全的**。

所以，**如果线程安全不是一个需求，一个`Deque`可以比一个** `**Stack**.`带给我们更好的性能

## 4.迭代顺序

由于`Stack`和`Deque`都是`java.util.Collection`接口的子类型，所以它们也是`Iterable`。

然而，有趣的是，如果我们将相同的元素以相同的顺序推入一个`Stack`对象和一个`Deque`实例，它们的迭代顺序是不同的。

让我们通过例子来仔细看看它们。

首先，让我们将一些元素推入一个`Stack`对象，并检查它的迭代顺序:

```
@Test
void givenAStack_whenIterate_thenFromBottomToTop() {
    Stack<String> myStack = new Stack<>();
    myStack.push("I am at the bottom.");
    myStack.push("I am in the middle.");
    myStack.push("I am at the top.");

    Iterator<String> it = myStack.iterator();

    assertThat(it).toIterable().containsExactly(
      "I am at the bottom.",
      "I am in the middle.",
      "I am at the top.");
} 
```

如果我们执行上面的测试方法，它就会通过。这意味着，**当我们迭代一个`Stack`对象中的元素时，顺序是从栈底到栈顶**。

接下来，让我们在一个`Deque`实例上做同样的测试。我们将把 [`ArrayDeque`](/web/20221208143859/https://www.baeldung.com/java-array-deque) 类作为我们测试中的`Deque`实现。

此外，我们将使用`ArrayDeque`作为 LIFO 堆栈:

```
@Test
void givenADeque_whenIterate_thenFromTopToBottom() {
    Deque<String> myStack = new ArrayDeque<>();
    myStack.push("I am at the bottom.");
    myStack.push("I am in the middle.");
    myStack.push("I am at the top.");

    Iterator<String> it = myStack.iterator();

    assertThat(it).toIterable().containsExactly(
      "I am at the top.",
      "I am in the middle.",
      "I am at the bottom.");
} 
```

如果我们试一试，这次考试也会通过的。

因此，**`Deque`的迭代顺序是从上到下的**。

当我们谈论 LIFO 堆栈数据结构时，迭代堆栈中元素的正确顺序应该是从上到下。

也就是说，`Deque`的迭代器按照我们对堆栈的预期方式工作。

## 5.无效的 LIFO 堆栈操作

经典的 LIFO 堆栈数据结构只支持`push()`、`pop()`和`peek()`操作。`Stack`类和`Deque`接口都支持它们。到目前为止，一切顺利。

然而，**不允许通过 LIFO 栈**中的索引来访问或操作元素，因为这违反了 LIFO 规则。

在本节中，让我们看看是否可以用`Stack`和`Deque.`调用无效的堆栈操作

### 5.1.`java.util.Stack` 类

因为它的父类`Vector `是基于数组的数据结构，所以`Stack`类能够通过索引访问元素:

```
@Test
void givenAStack_whenAccessByIndex_thenElementCanBeRead() {
    Stack<String> myStack = new Stack<>();
    myStack.push("I am the 1st element."); //index 0
    myStack.push("I am the 2nd element."); //index 1
    myStack.push("I am the 3rd element."); //index 2

    assertThat(myStack.get(0)).isEqualTo("I am the 1st element.");
} 
```

如果我们运行它，测试将通过。

在测试中，我们将三个元素放入一个`Stack`对象中。之后，我们想访问位于堆栈底部的元素。

遵循 LIFO 规则，我们必须弹出上面的所有元素来访问底部的元素。

然而，在这里，**使用`Stack`对象，我们只能通过索引**访问一个元素。

此外，使用`Stack`对象，**我们甚至可以通过索引**来插入和移除元素。让我们创建一个测试方法来证明这一点:

```
@Test
void givenAStack_whenAddOrRemoveByIndex_thenElementCanBeAddedOrRemoved() {
    Stack<String> myStack = new Stack<>();
    myStack.push("I am the 1st element.");
    myStack.push("I am the 3rd element.");

    assertThat(myStack.size()).isEqualTo(2);

    myStack.add(1, "I am the 2nd element.");
    assertThat(myStack.size()).isEqualTo(3);
    assertThat(myStack.get(1)).isEqualTo("I am the 2nd element.");

    myStack.remove(1);
    assertThat(myStack.size()).isEqualTo(2);
} 
```

如果我们试一试，测试也会通过。

因此，使用`Stack`类，我们可以像处理数组一样操作其中的元素。这违反了后进先出合同。

### 5.2.`java.util.Deque`界面

**`Deque`不允许我们通过索引来访问、插入或删除一个元素。**听起来比`Stack`班好。

然而，由于`Deque`是一个“双端队列”，我们可以从两端插入或移除一个元素。

换句话说，**当我们使用`Deque`作为 LIFO 栈时，我们可以直接在栈底插入/移除一个元素**。

让我们构建一个测试方法来看看这是如何发生的。同样，我们将在测试中继续使用`ArrayDeque`类:

```
@Test
void givenADeque_whenAddOrRemoveLastElement_thenTheLastElementCanBeAddedOrRemoved() {
    Deque<String> myStack = new ArrayDeque<>();
    myStack.push("I am the 1st element.");
    myStack.push("I am the 2nd element.");
    myStack.push("I am the 3rd element.");

    assertThat(myStack.size()).isEqualTo(3);

    //insert element to the bottom of the stack
    myStack.addLast("I am the NEW element.");
    assertThat(myStack.size()).isEqualTo(4);
    assertThat(myStack.peek()).isEqualTo("I am the 3rd element.");

    //remove element from the bottom of the stack
    String removedStr = myStack.removeLast();
    assertThat(myStack.size()).isEqualTo(3);
    assertThat(removedStr).isEqualTo("I am the NEW element.");
} 
```

在测试中，首先，我们使用`addLast()`方法在堆栈底部插入一个新元素。如果插入成功，我们尝试用`removeLast()`方法移除它。

如果我们执行测试，它就通过了。

因此， **`Deque`也不遵守后进先出契约**。

### 5.3.基于`Deque`实现一个`LifoStack`

我们可以创建一个简单的`LifoStack`接口来遵守 LIFO 契约:

```
public interface LifoStack<E> extends Collection<E> {
    E peek();
    E pop();
    void push(E item);
} 
```

当我们创建我们的`LifoStack `接口的实现时，我们可以包装 Java 标准`Deque`实现。

让我们创建一个`ArrayLifoStack` 类作为例子来快速理解它:

```
public class ArrayLifoStack<E> implements LifoStack<E> {
    private final Deque<E> deque = new ArrayDeque<>();

    @Override
    public void push(E item) {
        deque.addFirst(item);
    }

    @Override
    public E pop() {
        return deque.removeFirst();
    }

    @Override
    public E peek() {
        return deque.peekFirst();
    }

    // forward methods in Collection interface
    // to the deque object

    @Override
    public int size() {
        return deque.size();
    }
...
}
```

正如`ArrayLifoStack` 类所示，它只支持在我们的`LifoStack`接口和`java.util.Collection`接口中定义的操作。

这样，就不会违反后进先出法则。

## 6.结论

从 Java 1.6 开始，`java.util.Stack`和`java.util.Deque`都可以作为 LIFO 栈使用。本文讨论了这两种类型之间的区别。

我们还分析了为什么我们应该在`Stack`类上使用`Deque`接口来处理 LIFO 堆栈。

此外，正如我们通过例子讨论的那样，`Stack`和`Deque`或多或少都违反了 LIFO 规则。

最后，我们展示了一种创建遵循 LIFO 契约的堆栈接口的方法。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143859/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)