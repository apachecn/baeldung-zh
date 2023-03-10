# 在 Java 中定义字符堆栈

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-char-stack>

## 1.概观

在本教程中，我们将讨论如何在 Java 中创建一个`char`栈。我们将首先看看如何通过使用 Java API 来实现这一点，然后我们将看看一些定制的实现。

[栈](/web/20221208143854/https://www.baeldung.com/java-stack)是一种遵循后进先出(LIFO)原则的数据结构。一些常见的方法有:

*   `push(E item)`–将项目推到堆栈的顶部
*   `pop()`–移除并返回堆栈顶部的对象
*   `peek()`–返回堆栈顶部的对象，但不移除它

## 2.`Char`使用 Java API 堆栈

Java 有一个名为`java.util.Stack`的内置 API。**由于`char`是一个原始数据类型**，不能在泛型中使用，**我们必须使用`java.lang.Character`** 的包装类来创建一个`Stack`:

```java
Stack<Character> charStack = new Stack<>();
```

现在，我们可以对我们的`Stack`使用`push`、`pop`和`peek`方法。

另一方面，我们可能被要求构建一个栈的定制实现。因此，我们将研究几种不同的方法。

## 3.使用`LinkedList`自定义实现

让我们使用一个`LinkedList`作为后端数据结构来实现一个`char`栈:

```java
public class CharStack {

    private LinkedList<Character> items;

    public CharStack() {
        this.items = new LinkedList<Character>();
    }
}
```

我们创建了一个在构造函数中初始化的`items`变量。

现在，我们必须提供`push`、`peek`和`pop`方法的实现:

```java
public void push(Character item) {
    items.push(item);
}

public Character peek() {
    return items.getFirst();
}

public Character pop() {
    Iterator<Character> iter = items.iterator();
    Character item = iter.next();
    if (item != null) {
        iter.remove();
        return item;
    }
    return null;
}
```

`push`和`peek`方法使用了`LinkedList`的内置方法。对于`pop`，我们首先使用一个`Iterator`来检查顶部是否有一个项目。如果它在那里，我们通过调用`remove`方法`.`从列表中删除该项

## 4.使用数组的自定义实现

我们也可以使用数组作为我们的数据结构:

```java
public class CharStackWithArray {

    private char[] elements;
    private int size;

    public CharStackWithArray() {
        size = 0;
        elements = new char[4];
    }

}
```

上面，我们创建了一个`char`数组，在构造函数中初始化，初始容量为 4。此外，我们有一个`size`变量来跟踪堆栈中有多少记录。

现在，让我们实现`push`方法:

```java
public void push(char item) {
    ensureCapacity(size + 1);
    elements[size] = item;
    size++;
}

private void ensureCapacity(int newSize) {
    char newBiggerArray[];
    if (elements.length < newSize) {
        newBiggerArray = new char[elements.length * 2];
        System.arraycopy(elements, 0, newBiggerArray, 0, size);
        elements = newBiggerArray;
    }
}
```

在将一个条目压入堆栈时，我们首先需要检查我们的数组是否有存储它的容量。如果没有，我们创建一个新的数组，并将其大小加倍。然后，我们将旧的元素复制到新创建的数组中，并将其赋给我们的`elements`变量。

注意:要解释我们为什么要将数组的大小增加一倍，而不是简单地增加一倍，请参考这篇 [StackOverflow 帖子](https://web.archive.org/web/20221208143854/https://stackoverflow.com/questions/10419250/why-double-stack-capacity-instead-of-just-increasing-it-by-fixed-amount)。

最后，让我们实现`peek`和`pop`方法:

```java
public char peek() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    return elements[size - 1];
}

public char pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    return elements[--size];
}
```

对于这两种方法，在验证堆栈不为空之后，我们返回位置`size –` 1 的元素。对于`pop`，除了返回元素，我们还将`size`减 1。

## 5.结论

在本文中，我们学习了如何使用 Java API 创建一个`char`堆栈，并且我们看到了几个定制的实现。

本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections)