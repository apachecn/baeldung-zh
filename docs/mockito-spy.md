# 利用间谍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-spy>

## 1。概述

在本教程中，我们将说明如何充分利用 Mockito 中的**间谍。**

我们将讨论`@Spy`注释以及如何阻止间谍。最后，我们将探讨一下`Mock`和`Spy`的区别。

当然，更多的 mock ITO good，[看看这里的系列](/web/20221201145617/https://www.baeldung.com/tag/mockito/)。

## 延伸阅读:

## [莫奇托验证食谱](/web/20221201145617/https://www.baeldung.com/mockito-verify)

**Mockito Verify** examples, usage and best practices.[Read more](/web/20221201145617/https://www.baeldung.com/mockito-verify) →

## [将 Mockito Mocks 注入春豆](/web/20221201145617/https://www.baeldung.com/injecting-mocks-in-spring)

This article will show how to use dependency injection to insert Mockito mocks into Spring Beans for unit testing.[Read more](/web/20221201145617/https://www.baeldung.com/injecting-mocks-in-spring) →

## [莫克托的模仿方法](/web/20221201145617/https://www.baeldung.com/mockito-mock-methods)

This tutorial illustrates various uses of the standard static mock methods of the Mockito API.[Read more](/web/20221201145617/https://www.baeldung.com/mockito-mock-methods) →

## 2。简单的`Spy`例子

让我们先来看一个简单的**如何使用`spy`** 的例子。

简单来说，API 就是`Mockito.spy()`到**窥探一个真实的对象**。

这将允许我们调用对象的所有正常方法，同时仍然跟踪每个交互，就像我们对 mock 所做的那样。

现在让我们来做一个简单的例子，我们将窥探一个现有的`ArrayList`对象:

```java
@Test
public void whenSpyingOnList_thenCorrect() {
    List<String> list = new ArrayList<String>();
    List<String> spyList = Mockito.spy(list);

    spyList.add("one");
    spyList.add("two");

    Mockito.verify(spyList).add("one");
    Mockito.verify(spyList).add("two");

    assertThat(spyList).hasSize(2);
}
```

注意**真正的方法`add()`是如何被实际调用的**以及`spyList`的大小是如何变成 2 的。

## 3。`@Spy`注解

接下来，让我们看看如何使用`@Spy`注释。我们可以使用`@Spy`注释来代替`spy()`:

```java
@Spy
List<String> spyList = new ArrayList<String>();

@Test
public void whenUsingTheSpyAnnotation_thenObjectIsSpied() {
    spyList.add("one");
    spyList.add("two");

    Mockito.verify(spyList).add("one");
    Mockito.verify(spyList).add("two");

    assertThat(aSpyList).hasSize(2);
}
```

为了**启用 Mockito 注解**(比如`@Spy`、`@Mock`、…)，我们需要执行以下操作之一:

*   调用方法`MockitoAnnotations.initMocks(this)`初始化带注释的字段
*   使用内置滑槽`@RunWith(MockitoJUnitRunner.class)`

## 4。`Spy`捻断一根

现在让我们看看如何存根一个`Spy`。我们可以使用与 mock 相同的语法来配置/覆盖方法的行为。

这里我们将使用`doReturn()`来覆盖`size()`方法:

```java
@Test
public void whenStubASpy_thenStubbed() {
    List<String> list = new ArrayList<String>();
    List<String> spyList = Mockito.spy(list);

    assertEquals(0, spyList.size());

    Mockito.doReturn(100).when(spyList).size();
    assertThat(spyList).hasSize(100);
}
```

## 5。`Mock`vs`Spy in Mockito` 

我们来讨论一下《莫克托》中`Mock`和`Spy`的区别。我们不会检查这两个概念之间的理论差异，只是它们在 Mockito 本身内部有什么不同。

当 Mockito 创建一个 mock 时，它从一个类型的`Class`开始，而不是从一个实际的实例开始。mock 只是创建了这个类的一个简单的外壳实例,完全用来跟踪与它的交互。

另一方面，**间谍将包装一个现有的实例**。它仍将以与正常实例相同的方式运行；唯一不同的是，它还将被用来跟踪所有与之交互的内容。

这里我们将创建一个`ArrayList`类的`mock`:

```java
@Test
public void whenCreateMock_thenCreated() {
    List mockedList = Mockito.mock(ArrayList.class);

    mockedList.add("one");
    Mockito.verify(mockedList).add("one");

    assertThat(mockedList).hasSize(0);
}
```

正如我们所看到的，在嘲讽列表中添加一个元素实际上并没有添加任何东西；它只是调用方法，没有其他副作用。

另一方面，间谍的行为会有所不同；它实际上会调用`add`方法的实际实现，并将元素添加到底层列表中:

```java
@Test
public void whenCreateSpy_thenCreate() {
    List spyList = Mockito.spy(new ArrayList());

    spyList.add("one");
    Mockito.verify(spyList).add("one");

    assertThat(spyList).hasSize(1);
}
```

## 6。`NotAMockException`了解莫奇托

在这最后一部分，我们将了解 Mockito `NotAMockException`。这个异常是我们在误用模仿或间谍时可能会遇到的常见异常之一。

让我们先来了解这种异常可能发生的环境:

```java
List<String> list = new ArrayList<String>();
Mockito.doReturn(100).when(list).size();
```

当我们运行这段代码时，我们会得到以下错误:

```java
org.mockito.exceptions.misusing.NotAMockException: 
Argument passed to when() is not a mock!
Example of correct stubbing:
    doThrow(new RuntimeException()).when(mock).someMethod(); 
```

幸运的是，从 Mockito 错误消息中可以很清楚地看出问题出在哪里。在我们的例子中，`list`对象不是一个模仿对象。**mock ITO`when()`方法期望一个 mock 或 spy 对象作为参数**。

我们还可以看到，异常消息甚至描述了正确的调用应该是什么样子。现在我们对问题有了更好的理解，让我们按照建议来修复它:

```java
final List<String> spyList = Mockito.spy(new ArrayList<>());
assertThatNoException().isThrownBy(() -> Mockito.doReturn(100).when(spyList).size());
```

我们的例子现在如预期的那样运行，我们不再看到 Mockito `NotAMockException.`

## 7。结论

在这篇简短的文章中，我们讨论了使用 Mockito 间谍的最有用的例子。

我们学习了如何创建一个`spy`，使用@Spy 注释，存根一个`spy,`，最后是`Mock`和`Spy`的区别。

所有这些例子的实现**可以在 GitHub** 上找到[。](https://web.archive.org/web/20221201145617/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-simple)

这是一个 Maven 项目，因此应该很容易导入和运行。

最后，更多 mock ITO good，[看看这里的系列](/web/20221201145617/https://www.baeldung.com/tag/mockito/)。