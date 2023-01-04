# 用 Mockito 模拟最终类和方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-final>

## 1。概述

在这个简短的教程中，我们将关注如何使用 Mockito 模拟 final 类和方法。

正如关注 Mockito 框架的其他文章一样(例如 [Mockito 验证](/web/20221026103456/https://www.baeldung.com/mockito-verify)、 [Mockito When/Then](/web/20221026103456/https://www.baeldung.com/mockito-behavior) 和 [Mockito 的模拟方法](/web/20221026103456/https://www.baeldung.com/mockito-mock-methods))，我们将使用下面所示的`MyList`类作为测试用例中的协作者。

我们将为本教程添加一个新方法:

```java
public class MyList extends AbstractList {
    final public int finalMethod() {
        return 0;
    }
}
```

我们还将用一个`final`子类来扩展它:

```java
public final class FinalList extends MyList {

    @Override
    public int size() {
        return 1;
    }
} 
```

## 2。为最终方法和类配置 Mockito】

在我们使用 Mockito 模仿 final 类和方法之前，我们必须对它进行配置。

我们需要向项目的`src/test/resources/mockito-extensions`目录添加一个名为`org.mockito.plugins.MockMaker`的文本文件，并添加一行文本:

```java
mock-maker-inline 
```

加载时，Mockito 检查`extensions` 目录中的配置文件。这个文件支持模拟最终的方法和类。

## 3。模拟最终方法

**一旦我们正确配置了 Mockito，我们就可以像模拟其他方法一样模拟 final 方法了**:

```java
@Test
public void whenMockFinalMethodMockWorks() {

    MyList myList = new MyList();

    MyList mock = mock(MyList.class);
    when(mock.finalMethod()).thenReturn(1);

    assertThat(mock.finalMethod()).isNotEqualTo(myList.finalMethod());
} 
```

通过创建一个具体实例和一个`MyList`的，我们可以比较两个版本的`finalMethod()`返回的值，并验证模拟被调用。

## 4。模拟最后一堂课

嘲笑最后一个职业和嘲笑其他职业一样简单:

```java
@Test
public void whenMockFinalClassMockWorks() {

    FinalList finalList = new FinalList();

    FinalList mock = mock(FinalList.class);
    when(mock.size()).thenReturn(2);

    assertThat(mock.size()).isNotEqualTo(finalList.size());
} 
```

与上面的测试类似，我们为最终类创建一个具体实例和一个模拟实例，模拟一个方法，并验证模拟实例的行为是否不同。

## 5。结论

在这篇简短的文章中，我们介绍了如何通过使用 Mockito 扩展用 Mockito 模拟最终类和方法。

完整的例子，一如既往，可以在 GitHub 上找到[。](https://web.archive.org/web/20221026103456/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-simple)