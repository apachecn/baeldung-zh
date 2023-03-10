# 用 Mockito 模仿 Void 方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-void-methods>

## 1。概述

在这个简短的教程中，我们重点关注用 Mockito 模仿`void`方法。

## 延伸阅读:

## [Mockito 的 Java 8 特性](/web/20220630141603/https://www.baeldung.com/mockito-2-java-8)

Overview of Java 8 support in Mockito framework, including Streams and default interface methods[Read more](/web/20220630141603/https://www.baeldung.com/mockito-2-java-8) →

## [使用 Mockito 模拟异常抛出](/web/20220630141603/https://www.baeldung.com/mockito-exceptions)

Learn to configure a method call to throw an exception in Mockito.[Read more](/web/20220630141603/https://www.baeldung.com/mockito-exceptions) →

与其他关注 Mockito 框架的文章一样(如 [Mockito 验证](/web/20220630141603/https://www.baeldung.com/mockito-verify)、 [Mockito When/Then](/web/20220630141603/https://www.baeldung.com/mockito-behavior) 和 [Mockito 的模拟方法](/web/20220630141603/https://www.baeldung.com/mockito-mock-methods))，下面显示的`MyList`类将被用作测试用例中的协作者。

我们将为本教程添加一个新方法:

```java
public class MyList extends AbstractList<String> {

    @Override
    public void add(int index, String element) {
        // no-op
    }
}
```

## 2。简单的嘲讽和验证

`Void`方法可以和 Mockito 的`doNothing()`、 `doThrow()`、 `and doAnswer()`方法一起使用，使嘲讽和验证变得直观:

```java
@Test
public void whenAddCalledVerified() {
    MyList myList = mock(MyList.class);
    doNothing().when(myList).add(isA(Integer.class), isA(String.class));
    myList.add(0, "");

    verify(myList, times(1)).add(0, "");
}
```

**然而，`doNothing()`是 Mockito 对`void`方法的默认行为。**

这个版本的`whenAddCalledVerified()`完成了与上面版本相同的事情:

```java
@Test
public void whenAddCalledVerified() {
    MyList myList = mock(MyList.class);
    myList(0, "");

    verify(myList, times(1)).add(0, "");
} 
```

`DoThrow()`生成异常:

```java
@Test(expected = Exception.class)
public void givenNull_AddThrows() {
    MyList myList = mock(MyList.class);
    doThrow().when(myList).add(isA(Integer.class), isNull());

    myList.add(0, null);
} 
```

我们将在下面讨论`doAnswer()`。

## 3。参数捕获

**用`doNothing()` 覆盖默认行为的一个原因是为了捕获参数。**

在上面的例子中，我们使用了`verify()`方法来检查传递给`add()`的参数。

然而，我们可能需要捕捉这些论点，并对它们做更多的事情。

在这些情况下，我们像上面一样使用`doNothing()`，但是带有一个`ArgumentCaptor`:

```java
@Test
public void whenAddCalledValueCaptured() {
    MyList myList = mock(MyList.class);
    ArgumentCaptor<String> valueCapture = ArgumentCaptor.forClass(String.class);
    doNothing().when(myList).add(any(Integer.class), valueCapture.capture());
    myList.add(0, "captured");

    assertEquals("captured", valueCapture.getValue());
} 
```

## 4。`Void`接听来电

方法可以执行比仅仅添加或设置值更复杂的行为。

对于这些情况，我们可以使用 Mockito 的`Answer`来添加我们需要的行为:

```java
@Test
public void whenAddCalledAnswered() {
    MyList myList = mock(MyList.class);
    doAnswer(invocation -> {
        Object arg0 = invocation.getArgument(0);
        Object arg1 = invocation.getArgument(1);

        assertEquals(3, arg0);
        assertEquals("answer me", arg1);
        return null;
    }).when(myList).add(any(Integer.class), any(String.class));
    myList.add(3, "answer me");
} 
```

正如在 [Mockito 的 Java 8 特性](/web/20220630141603/https://www.baeldung.com/mockito-2-java-8)中所解释的，我们使用带有`Answer`的 lambda 来定义`add()`的定制行为。

## 5。部分嘲讽

部分模拟也是一种选择。Mockito 的`doCallRealMethod()`可用于`void`方法:

```java
@Test
public void whenAddCalledRealMethodCalled() {
    MyList myList = mock(MyList.class);
    doCallRealMethod().when(myList).add(any(Integer.class), any(String.class));
    myList.add(1, "real");

    verify(myList, times(1)).add(1, "real");
} 
```

**这样，我们可以调用实际的方法，同时验证它。**

## 6。结论

在这篇简短的文章中，我们介绍了使用 Mockito 进行测试时接近`void`方法的四种不同方式。

和往常一样，GitHub 项目的[中提供了示例。](https://web.archive.org/web/20220630141603/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito)