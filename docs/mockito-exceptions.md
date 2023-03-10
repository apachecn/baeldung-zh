# 使用 Mockito 模拟异常抛出

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-exceptions>

## 1。概述

在这个快速教程中，我们将重点关注如何配置一个方法调用来用 Mockito 抛出一个异常。

想了解更多关于图书馆的信息，也可以看看我们的 [Mockito 系列](/web/20221129011032/https://www.baeldung.com/mockito-final)。

下面是我们将使用的简单字典类:

```java
class MyDictionary {
    private Map<String, String> wordMap = new HashMap<>();

    public void add(String word, String meaning) {
        wordMap.put(word, meaning);
    }

    public String getMeaning(String word) {
        return wordMap.get(word);
    }
}
```

## 延伸阅读:

## [断言 JUnit 4 和 5 中抛出异常](/web/20221129011032/https://www.baeldung.com/junit-assert-exception)

Have a look at how to test if an exception was thrown using JUnit.[Read more](/web/20221129011032/https://www.baeldung.com/junit-assert-exception) →

## [Mockito 的 Java 8 特性](/web/20221129011032/https://www.baeldung.com/mockito-java-8)

Overview of Java 8 support in Mockito framework, including Streams and default interface methods[Read more](/web/20221129011032/https://www.baeldung.com/mockito-java-8) →

## [AssertJ 异常断言](/web/20221129011032/https://www.baeldung.com/assertj-exception-assertion)

Learn how to use AssertJ for performing assertions on exceptions.[Read more](/web/20221129011032/https://www.baeldung.com/assertj-exception-assertion) →

## 2。非`Void`返回类型

首先，如果我们的方法返回类型不是`void`，我们可以使用`when().thenThrow()`:

```java
@Test(expected = NullPointerException.class)
public void whenConfigNonVoidRetunMethodToThrowEx_thenExIsThrown() {
    MyDictionary dictMock = mock(MyDictionary.class);
    when(dictMock.getMeaning(anyString()))
      .thenThrow(NullPointerException.class);

    dictMock.getMeaning("word");
} 
```

注意，我们配置了`getMeaning()`方法——它返回一个类型为`String`的值——在被调用时抛出一个`NullPointerException`。

## 3。`Void`返回类型

现在，如果我们的方法返回`void,`，我们将使用`doThrow()`:

```java
@Test(expected = IllegalStateException.class)
public void whenConfigVoidRetunMethodToThrowEx_thenExIsThrown() {
    MyDictionary dictMock = mock(MyDictionary.class);
    doThrow(IllegalStateException.class)
      .when(dictMock)
      .add(anyString(), anyString());

    dictMock.add("word", "meaning");
}
```

这里，我们配置了一个`add()`方法——它返回`void`——在被调用时抛出`IllegalStateException`。

我们不能将`when().thenThrow()`与`void`返回类型一起使用，因为编译器不允许在括号内使用`void`方法。

## 4。作为对象的异常

为了配置异常本身，我们可以像前面的例子一样传递异常的类，或者作为一个对象传递:

```java
@Test(expected = NullPointerException.class)
public void whenConfigNonVoidRetunMethodToThrowExWithNewExObj_thenExIsThrown() {
    MyDictionary dictMock = mock(MyDictionary.class);
    when(dictMock.getMeaning(anyString()))
      .thenThrow(new NullPointerException("Error occurred"));

    dictMock.getMeaning("word");
}
```

我们可以用 `doThrow()`做同样的事情:

```java
@Test(expected = IllegalStateException.class)
public void whenConfigVoidRetunMethodToThrowExWithNewExObj_thenExIsThrown() {
    MyDictionary dictMock = mock(MyDictionary.class);
    doThrow(new IllegalStateException("Error occurred"))
      .when(dictMock)
      .add(anyString(), anyString());

    dictMock.add("word", "meaning");
}
```

## 5。间谍

我们还可以配置`Spy`抛出一个异常，就像我们对 mock 所做的一样:

```java
@Test(expected = NullPointerException.class)
public void givenSpy_whenConfigNonVoidRetunMethodToThrowEx_thenExIsThrown() {
    MyDictionary dict = new MyDictionary();
    MyDictionary spy = Mockito.spy(dict);
    when(spy.getMeaning(anyString()))
      .thenThrow(NullPointerException.class);

    spy.getMeaning("word");
}
```

## 6。结论

在本文中，我们探讨了如何在 Mockito 中配置方法调用来抛出异常。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129011032/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-simple)