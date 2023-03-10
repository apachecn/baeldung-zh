# 如何在 Java 中 TDD 一个列表实现

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-test-driven-list>

## 1。概述

在本教程中，我们将使用测试驱动开发(TDD)过程来完成一个定制的`List`实现。

这并不是对 TDD 的介绍，所以我们假设你已经对它的含义有了一些基本的概念，并且对更好地使用它有持续的兴趣。

简单地说， **TDD 是一个设计工具，使我们能够在测试**的帮助下驱动我们的实现。

一个简短的声明——我们在这里并不关注创建有效的实现——只是用它作为展示 TDD 实践的借口。

## 2。入门

首先，让我们为我们的类定义框架:

```java
public class CustomList<E> implements List<E> {
    private Object[] internal = {};
    // empty implementation methods
} 
```

`CustomList`类实现了`List`接口，因此它必须包含该接口中声明的所有方法的实现。

首先，我们可以为这些方法提供空的主体。如果一个方法有一个返回类型，我们可以返回该类型的任意值，例如`Object`的`null`或`boolean`的 *false* 。

为了简洁起见，我们将省略可选方法，以及一些不常使用的强制方法。

## 3。TDD 周期

用 TDD 开发我们的实现意味着我们需要首先**创建测试用例**，从而为我们的实现定义需求。只有**然后我们将创建或修复实现代码**以使这些测试通过。

简单来说，每个周期中的三个主要步骤是:

1.  **编写测试—**以测试的形式定义需求
2.  **实现特性–**让测试通过，而不要过多关注代码的优雅
3.  **重构—**改进代码，使其更容易阅读和维护，同时仍然通过测试

我们将为`List`接口的一些方法经历这些 TDD 循环，从最简单的开始。

## 4。`isEmpty`法

`isEmpty`方法可能是在`List`接口中定义的最直接的方法。这是我们的初始实现:

```java
@Override
public boolean isEmpty() {
    return false;
}
```

这个初始方法定义足以编译。当越来越多的测试加入时，这种方法的主体将“被迫”改进。

### 4.1。第一周期

让我们编写第一个测试用例，确保当列表不包含任何元素时，`isEmpty`方法返回`true`:

```java
@Test
public void givenEmptyList_whenIsEmpty_thenTrueIsReturned() {
    List<Object> list = new CustomList<>();

    assertTrue(list.isEmpty());
}
```

给定的测试失败，因为`isEmpty`方法总是返回`false`。我们可以通过翻转返回值来使它通过:

```java
@Override
public boolean isEmpty() {
    return true;
}
```

### 4.2。第二循环

为了确认当列表不为空时`isEmpty`方法返回`false`，我们需要添加至少一个元素:

```java
@Test
public void givenNonEmptyList_whenIsEmpty_thenFalseIsReturned() {
    List<Object> list = new CustomList<>();
    list.add(null);

    assertFalse(list.isEmpty());
}
```

现在需要一个`add`方法的实现。下面是我们开始使用的`add`方法:

```java
@Override
public boolean add(E element) {
    return false;
}
```

这个方法实现不起作用，因为没有对列表的内部数据结构进行任何更改。让我们更新它来存储添加的元素:

```java
@Override
public boolean add(E element) {
    internal = new Object[] { element };
    return false;
}
```

我们的测试仍然失败，因为`isEmpty`方法没有得到增强。让我们这样做:

```java
@Override
public boolean isEmpty() {
    if (internal.length != 0) {
        return false;
    } else {
        return true;
    }
}
```

此时非空测试通过。

### 4.3。重构

到目前为止，我们看到的两个测试案例都通过了，但是`isEmpty`方法的代码可以更优雅。

让我们重构一下:

```java
@Override
public boolean isEmpty() {
    return internal.length == 0;
}
```

我们可以看到测试通过了，所以现在完成了`isEmpty`方法的实现。

## 5。`size`法

这是我们开始实现的`size`方法，使`CustomList`类能够编译:

```java
@Override
public int size() {
    return 0;
}
```

### 5.1。第一周期

使用现有的`add`方法，我们可以为`size`方法创建第一个测试，验证具有单个元素的列表的大小是`1`:

```java
@Test
public void givenListWithAnElement_whenSize_thenOneIsReturned() {
    List<Object> list = new CustomList<>();
    list.add(null);

    assertEquals(1, list.size());
}
```

测试失败，因为`size`方法正在返回`0`。让我们通过一个新的实现:

```java
@Override
public int size() {
    if (isEmpty()) {
        return 0;
    } else {
        return internal.length;
    }
}
```

### 5.2。重构

我们可以重构`size`方法，使其更加优雅:

```java
@Override
public int size() {
    return internal.length;
}
```

这个方法的实现现在已经完成。

## 6。`get`法

下面是`get`的开始实现:

```java
@Override
public E get(int index) {
    return null;
}
```

### 6.1。第一周期

让我们看一下这个方法的第一个测试，它验证了列表中单个元素的值:

```java
@Test
public void givenListWithAnElement_whenGet_thenThatElementIsReturned() {
    List<Object> list = new CustomList<>();
    list.add("baeldung");
    Object element = list.get(0);

    assertEquals("baeldung", element);
}
```

测试将通过`get`方法的实现:

```java
@Override
public E get(int index) {
    return (E) internal[0];
}
```

### 6.2。改进

通常，在对`get`方法进行额外改进之前，我们会添加更多的测试。这些测试需要其他的`List`接口方法来实现正确的断言。

然而，这些其他的方法还不够成熟，所以我们打破了 TDD 循环，创建了一个完整的`get`方法的实现，这实际上并不难。

很容易想象`get`必须使用`index`参数从`internal`数组的指定位置提取一个元素:

```java
@Override
public E get(int index) {
    return (E) internal[index];
}
```

## 7。`add`法

这是我们在第 4 节中创建的`add`方法:

```java
@Override
public boolean add(E element) {
    internal = new Object[] { element };
    return false;
}
```

### 7.1。第一周期

下面是一个验证`add`返回值的简单测试:

```java
@Test
public void givenEmptyList_whenElementIsAdded_thenGetReturnsThatElement() {
    List<Object> list = new CustomList<>();
    boolean succeeded = list.add(null);

    assertTrue(succeeded);
}
```

我们必须修改`add`方法来返回`true`以通过测试:

```java
@Override
public boolean add(E element) {
    internal = new Object[] { element };
    return true;
}
```

虽然测试通过了，但是`add`方法还没有覆盖所有情况。如果我们向列表中添加第二个元素，现有的元素将会丢失。

### 7.2。第二循环

下面是另一个测试，增加了列表可以包含多个元素的要求:

```java
@Test
public void givenListWithAnElement_whenAnotherIsAdded_thenGetReturnsBoth() {
    List<Object> list = new CustomList<>();
    list.add("baeldung");
    list.add(".com");
    Object element1 = list.get(0);
    Object element2 = list.get(1);

    assertEquals("baeldung", element1);
    assertEquals(".com", element2);
}
```

测试将失败，因为当前形式的`add`方法不允许添加多个元素。

让我们更改实现代码:

```java
@Override
public boolean add(E element) {
    Object[] temp = Arrays.copyOf(internal, internal.length + 1);
    temp[internal.length] = element;
    internal = temp;
    return true;
}
```

该实现足够优雅，因此我们不需要重构它。

## 8。结论

本教程通过一个测试驱动的开发过程来创建一个定制的`List`实现的一部分。使用 TDD，我们可以一步一步地实现需求，同时将测试覆盖率保持在一个非常高的水平。此外，该实现保证是可测试的，因为它是为了通过测试而创建的。

请注意，本文中创建的自定义类仅用于演示目的，不应在实际项目中采用。

本教程的完整源代码，包括为了简洁而省略的测试和实现方法，可以在 GitHub 上找到[。](https://web.archive.org/web/20220625230201/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list)