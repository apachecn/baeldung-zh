# 莫克托的模仿方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-mock-methods>

## 1。概述

在本教程中，我们将说明`Mockito` API 的标准静态`mock`方法的各种用途。

正如在关注 Mockito 框架的其他文章中一样(如 [Mockito Verify](/web/20221026103513/https://www.baeldung.com/mockito-verify) 或 [Mockito When/Then](/web/20221026103513/https://www.baeldung.com/mockito-behavior) )，下面显示的`MyList`类将被用作测试用例中被模仿的协作者:

```
public class MyList extends AbstractList<String> {
    @Override
    public String get(int index) {
        return null;
    }

    @Override
    public int size() {
        return 1;
    }
}
```

## 2。简单嘲讽

`mock`方法最简单的重载变体是为要模仿的类提供一个参数:

```
public static <T> T mock(Class<T> classToMock)
```

我们将使用这个方法模拟一个类并设置一个期望:

```
MyList listMock = mock(MyList.class);
when(listMock.add(anyString())).thenReturn(false);
```

然后我们将在 mock 上执行一个方法:

```
boolean added = listMock.add(randomAlphabetic(6));
```

下面的代码确认我们调用了 mock 上的`add`方法。调用返回一个与我们之前设置的期望值相匹配的值:

```
verify(listMock).add(anyString());
assertThat(added).isFalse();
```

## 3。用 Mock 的名字嘲讽

在这一节中，我们将介绍`mock`方法的另一种变体，它提供了一个指定模拟名称的参数:

```
public static <T> T mock(Class<T> classToMock, String name)
```

一般来说，mock 的名字与工作代码无关。然而，它可能有助于调试，因为我们使用 mock 的名称来跟踪验证错误。

为了确保不成功的验证抛出的异常消息包含所提供的模拟名称，我们将在下面的代码中使用`assertThatThrownBy.`
，我们将为`MyList`类创建一个模拟，并将其命名为`myMock`:

```
MyList listMock = mock(MyList.class, "myMock");
```

然后，我们将对 mock 的方法设置一个期望值，并执行它:

```
when(listMock.add(anyString())).thenReturn(false);
listMock.add(randomAlphabetic(6));
```

接下来，我们将调用`assertThatThrownBy `内部的验证，并验证抛出的异常的实例:

```
assertThatThrownBy(() -> verify(listMock, times(2)).add(anyString()))
    .isInstanceOf(TooFewActualInvocations.class)
```

此外，我们还可以验证异常的消息，它应该包含关于 mock 的信息:

```
assertThatThrownBy(() -> verify(listMock, times(2)).add(anyString()))
    .isInstanceOf(TooFewActualInvocations.class)
    .hasMessageContaining("myMock.add");
```

下面是抛出异常的消息:

```
org.mockito.exceptions.verification.TooLittleActualInvocations:
myMock.add(<any>);
Wanted 2 times:
at com.baeldung.mockito.MockitoMockTest
  .whenUsingMockWithName_thenCorrect(MockitoMockTest.java:...)
but was 1 time:
at com.baeldung.mockito.MockitoMockTest
  .whenUsingMockWithName_thenCorrect(MockitoMockTest.java:...)
```

正如我们所看到的，异常消息包括 mock 的名称，这对于在验证不成功的情况下找到故障点很有用。

## 4。`Answer`嘲讽着

这里我们将演示一个`mock`变体的使用，在这个变体中，我们将在创建时为模拟的交互响应配置策略。Mockito 文档中的这个`mock`方法的签名如下所示:

```
public static <T> T mock(Class<T> classToMock, Answer defaultAnswer)
```

让我们从`Answer`接口实现的定义开始:

```
class CustomAnswer implements Answer<Boolean> {
    @Override
    public Boolean answer(InvocationOnMock invocation) throws Throwable {
        return false;
    }
}
```

我们将使用上面的`CustomAnswer`类来生成模拟:

```
MyList listMock = mock(MyList.class, new CustomAnswer());
```

如果我们不对一个方法设置一个期望值，那么默认的答案，由`CustomAnswer`类型配置的，将会发挥作用。为了证明这一点，我们将跳过期望值设置步骤，直接跳到方法执行:

```
boolean added = listMock.add(randomAlphabetic(6));
```

下面的验证和断言确认了带有`Answer`参数的`mock`方法按预期工作:

```
verify(listMock).add(anyString());
assertThat(added).isFalse();
```

## 5。`MockSettings`嘲讽着

我们将在本文中介绍的最后一个`mock`方法是带有一个`MockSettings`类型参数的变体。我们使用这个重载的方法来提供一个非标准的模拟。

`MockSettings`接口的方法支持几个自定义设置，比如用`invocationListeners`注册当前 mock 上方法调用的监听器，用`serializable`配置序列化，用`spiedInstance`指定要监视的实例，用`useConstructor`配置 Mockito 在实例化 mock 时尝试使用构造函数，等等。

为了方便起见，我们将重用上一节中介绍的`CustomAnswer`类来创建一个定义默认答案的`MockSettings`实现。

一个`MockSettings`对象被一个工厂方法实例化:

```
MockSettings customSettings = withSettings().defaultAnswer(new CustomAnswer());
```

我们将在创建新模拟时使用设置对象:

```
MyList listMock = mock(MyList.class, customSettings);
```

与上一节类似，我们将调用一个`MyList`实例的`add`方法，并验证带有`MockSettings`参数的`mock`方法是否按预期工作:

```
boolean added = listMock.add(randomAlphabetic(6));
verify(listMock).add(anyString());
assertThat(added).isFalse();
```

## 6。结论

在本文中，我们详细介绍了 Mockito 的`mock`方法。这些例子和代码片段的实现可以在 GitHub 项目中找到。