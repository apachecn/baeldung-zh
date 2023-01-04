# JMockit 期望指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jmockit-expectations>

## 1。简介

本文是 JMockit 系列的第二部分。您可能希望阅读第一篇文章，因为我们假设您已经熟悉 JMockit 的基础知识。

今天，我们将深入探讨，重点关注期望值。我们将展示如何定义更具体或更通用的参数匹配，以及定义值的更高级的方法。

## 2。参数值匹配

以下方法既适用于`Expectations`也适用于`Verifications`。

### 2.1。“任何”字段

JMockit 提供了一组实用字段，使参数匹配更加通用。这些实用程序之一是`anyX`字段。

这些函数将检查是否传递了任何值，每个原语类型(和相应的包装类)都有一个值，一个字符串值，还有一个类型为`Object`的“通用”值。

让我们看一个例子:

```java
public interface ExpectationsCollaborator {
    String methodForAny1(String s, int i, Boolean b);
    void methodForAny2(Long l, List<String> lst);
}

@Test
public void test(@Mocked ExpectationsCollaborator mock) throws Exception {
    new Expectations() {{
        mock.methodForAny1(anyString, anyInt, anyBoolean); 
        result = "any";
    }};

    Assert.assertEquals("any", mock.methodForAny1("barfooxyz", 0, Boolean.FALSE));
    mock.methodForAny2(2L, new ArrayList<>());

    new FullVerifications() {{
        mock.methodForAny2(anyLong, (List<String>) any);
    }};
}
```

您必须考虑到，当使用`any`字段时，您需要将其转换为预期的类型。字段的完整列表在[文档](https://web.archive.org/web/20220126104623/https://jmockit.github.io/tutorial/Mocking.html#expectation)中。

### 2.2。“用”的方法

JMockit 还提供了几种方法来帮助进行泛型参数匹配。这些是`withX`方法。

这些字段允许比`anyX`字段更高级的匹配。我们可以在这里看到一个例子，我们将定义一个方法的期望，该方法将由一个包含`foo`、一个不等于 1 的整数、一个非空的`Boolean`和`List`类的任何实例的字符串触发:

```java
public interface ExpectationsCollaborator {
    String methodForWith1(String s, int i);
    void methodForWith2(Boolean b, List<String> l);
}

@Test
public void testForWith(@Mocked ExpectationsCollaborator mock) throws Exception {
    new Expectations() {{
        mock.methodForWith1(withSubstring("foo"), withNotEqual(1));
        result = "with";
    }};

    assertEquals("with", mock.methodForWith1("barfooxyz", 2));
    mock.methodForWith2(Boolean.TRUE, new ArrayList<>());

    new Verifications() {{
        mock.methodForWith2(withNotNull(), withInstanceOf(List.class));
    }};
}
```

你可以在 JMockit 的[文档](https://web.archive.org/web/20220126104623/https://jmockit.github.io/tutorial/Mocking.html#expectation)中看到`withX`方法的完整列表。

考虑到特殊`with(Delegate)`和`withArgThat(Matcher)`将在其各自的小节中介绍。

### 2.3。Null 不为 Null

有一点很好理解，那就是`null`并不用于定义一个`null`已经被传递给模拟的参数。

实际上，`null`是作为**语法糖**来定义任何对象都会被传递(所以只能用于引用类型的参数)。为了具体验证给定的参数接收到了`null`引用，可以使用`withNull()`匹配器。

对于下一个例子，我们将定义一个模拟的行为，当传递的参数是:任何字符串、任何列表和一个`null`引用时，将触发该行为:

```java
public interface ExpectationsCollaborator {
    String methodForNulls1(String s, List<String> l);
    void methodForNulls2(String s, List<String> l);
}

@Test
public void testWithNulls(@Mocked ExpectationsCollaborator mock){
    new Expectations() {{
        mock.methodForNulls1(anyString, null); 
        result = "null";
    }};

    assertEquals("null", mock.methodForNulls1("blablabla", new ArrayList<String>()));
    mock.methodForNulls2("blablabla", null);

    new Verifications() {{
        mock.methodForNulls2(anyString, (List<String>) withNull());
    }};
}
```

注意区别:`null`表示任何列表，`withNull()`表示对列表的`null`引用。特别是，这避免了将值强制转换为已声明的参数类型的需要(请注意，必须强制转换第三个参数，而不是第二个)。

能够使用它的唯一条件是至少有一个显式的参数匹配器被用于期望(一个`with`方法或者一个`any`字段)。

### 2.4。“时间”字段

有时，我们希望**约束** **被模仿方法预期的调用次数**。为此，JMockit 有保留字`times`、`minTimes`和`maxTimes`(三者只允许非负整数)。

```java
public interface ExpectationsCollaborator {
    void methodForTimes1();
    void methodForTimes2();
    void methodForTimes3();
}

@Test
public void testWithTimes(@Mocked ExpectationsCollaborator mock) {
    new Expectations() {{
        mock.methodForTimes1(); times = 2;
        mock.methodForTimes2();
    }};

    mock.methodForTimes1();
    mock.methodForTimes1();
    mock.methodForTimes2();
    mock.methodForTimes3();
    mock.methodForTimes3();
    mock.methodForTimes3();

    new Verifications() {{
        mock.methodForTimes3(); minTimes = 1; maxTimes = 3;
    }};
}
```

在这个例子中，我们已经定义了应该使用行`times = 2;`对`methodForTimes1()`进行两次调用(不是一次，不是三次，而是两次)。

然后，我们使用默认行为(如果没有给定重复约束，则使用`minTimes = 1;`)来定义至少对`methodForTimes2().`进行一次调用

最后，使用`minTimes = 1;`后跟`maxTimes = 3;`我们定义了`methodForTimes3()`会发生一到三次调用。

考虑到可以为同一个期望指定`minTimes`和`maxTimes`，只要先指定`minTimes`。另一方面，`times`只能单独使用。

### 2.5。自定义参数匹配

有时候，参数匹配不像简单地指定一个值或使用一些预定义的实用程序(`anyX`或`withX`)那样直接。

在这种情况下，JMockit 依赖于 [Hamcrest](https://web.archive.org/web/20220126104623/http://hamcrest.org/) 的`Matcher`接口。您只需要为特定的测试场景定义一个匹配器，并通过一个`withArgThat()`调用使用该匹配器。

让我们看一个将特定的类与传递的对象相匹配的例子:

```java
public interface ExpectationsCollaborator {
    void methodForArgThat(Object o);
}

public class Model {
    public String getInfo(){
        return "info";
    }
}

@Test
public void testCustomArgumentMatching(@Mocked ExpectationsCollaborator mock) {
    new Expectations() {{
        mock.methodForArgThat(withArgThat(new BaseMatcher<Object>() {
            @Override
            public boolean matches(Object item) {
                return item instanceof Model && "info".equals(((Model) item).getInfo());
            }

            @Override
            public void describeTo(Description description) { }
        }));
    }};
    mock.methodForArgThat(new Model());
}
```

## 3。返回值

现在让我们看看返回值；请记住，以下方法仅适用于`Expectations`，因为不能为`Verifications`定义返回值。

### 3.1。结果和返回(…)

当使用 JMockit 时，有三种不同的方法来定义被模仿方法调用的预期结果。在这三种方法中，我们现在将讨论前两种(最简单的),这两种方法肯定会涵盖 90%的日常用例。

这两个是`result`字段和`returns(Object…)`方法:

*   通过`result`字段，您可以为任何非空返回模拟方法定义**一个**返回值。这个返回值也可以是一个抛出的异常(这次对非 void 和 void 返回方法都有效)。
    *   可以进行几次`result`字段分配，以便为多个方法调用返回**多个值**(可以混合返回值和要抛出的错误)。
    *   当给`result`分配一个列表或一个数组的值时，将实现相同的行为(与被模仿方法的返回类型相同，这里没有例外)。
*   `returns(Object…)`方法是用于返回同一时间的几个值的**语法糖**。

下面的代码片段更容易显示这一点:

```java
public interface ExpectationsCollaborator{
    String methodReturnsString();
    int methodReturnsInt();
}

@Test
public void testResultAndReturns(@Mocked ExpectationsCollaborator mock) {
    new Expectations() {{
        mock.methodReturnsString();
        result = "foo";
        result = new Exception();
        result = "bar";
        returns("foo", "bar");
        mock.methodReturnsInt();
        result = new int[]{1, 2, 3};
        result = 1;
    }};

    assertEquals("Should return foo", "foo", mock.methodReturnsString());
    try {
        mock.methodReturnsString();
        fail("Shouldn't reach here");
    } catch (Exception e) {
        // NOOP
    }
    assertEquals("Should return bar", "bar", mock.methodReturnsString());
    assertEquals("Should return 1", 1, mock.methodReturnsInt());
    assertEquals("Should return 2", 2, mock.methodReturnsInt());
    assertEquals("Should return 3", 3, mock.methodReturnsInt());
    assertEquals("Should return foo", "foo", mock.methodReturnsString());
    assertEquals("Should return bar", "bar", mock.methodReturnsString());
    assertEquals("Should return 1", 1, mock.methodReturnsInt());
}
```

在这个例子中，我们已经定义了对`methodReturnsString()`的前三次调用的预期回报是(按顺序)`“foo”`、异常和`“bar”`。我们通过对`result`字段使用三种不同的赋值来实现这一点。

然后在**第 14 行**上，我们定义了对于第四次和第五次调用，`“foo”` 和`“bar”`应该使用`returns(Object…)`方法返回。

对于`methodReturnsInt()`,我们在**行 13** 上定义了返回 1、2 和最后 3，方法是将不同结果的数组分配给`result`字段；在**行 15** 上，我们定义了通过简单分配给`result`字段返回 1。

正如你所看到的，有几种方法来定义模拟方法的返回值。

### 3.2。委托人

在文章的最后，我们将讨论定义返回值的第三种方式:`Delegate`接口。该接口用于在定义模拟方法时定义更复杂的返回值。

我们将看到一个简单解释的例子:

```java
public interface ExpectationsCollaborator {
    int methodForDelegate(int i);
}

@Test
public void testDelegate(@Mocked ExpectationsCollaborator mock) {
    new Expectations() {{
        mock.methodForDelegate(anyInt);

        result = new Delegate() {
            int delegate(int i) throws Exception {
                if (i < 3) {
                    return 5;
                } else {
                    throw new Exception();
                }
            }
        };
    }};

    assertEquals("Should return 5", 5, mock.methodForDelegate(1));
    try {
        mock.methodForDelegate(3);
        fail("Shouldn't reach here");
    } catch (Exception e) {
    }
} 
```

使用委托者的方法是为它创建一个新实例，并将其分配给一个`returns`字段。在这个新实例中，您应该创建一个新方法，它具有与模拟方法相同的参数和返回类型(您可以为它使用任何名称)。在这个新方法中，使用您想要的任何实现来返回所需的值。

在这个例子中，我们做了一个实现，其中当传递给模拟方法的值小于`3`时，应该返回`5`，否则抛出一个异常(注意，我们必须使用`times = 2;`，这样第二次调用是预期的，因为我们通过定义一个返回值失去了默认行为)。

这看起来像是相当多的代码，但在某些情况下，这将是实现我们想要的结果的唯一方法。

## 4。结论

这样，我们实际上展示了为我们的日常测试创建期望和验证所需的一切。

我们当然会发布更多关于 JMockit 的文章，所以请继续关注以了解更多。

和往常一样，本教程的完整实现可以在 GitHub 项目中找到。

### 4.1。系列文章

该系列的所有文章:

*   [JMockit 101](/web/20220126104623/https://www.baeldung.com/jmockit-101)
*   [JMockit 期望指南](/web/20220126104623/https://www.baeldung.com/jmockit-expectations)
*   [JMockit 高级用法](/web/20220126104623/https://www.baeldung.com/jmockit-advanced-usage)