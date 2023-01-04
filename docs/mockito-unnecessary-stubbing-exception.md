# 严格存根和不必要的存根异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-unnecessary-stubbing-exception>

## 1。概述

在这个快速教程中，我们将学习不必要的异常 T3。这个异常是我们在错误使用存根时可能会遇到的常见异常。

我们将从解释严格存根背后的哲学开始，以及为什么 Mockito 默认鼓励使用它。然后我们将看看这个异常到底意味着什么，以及在什么情况下它会发生。最后，我们将看到一个如何在测试中抑制这个异常的例子。

要了解更多关于使用 Mockito 进行测试的信息，请查看我们全面的 [Mockito 系列](/web/20221123101251/https://www.baeldung.com/tag/mockito/)。

## 2。严格杜绝

使用 1.x 版的 Mockito，可以不受限制地配置 mocks 并与之交互。这意味着，随着时间的推移，测试通常会变得过于复杂，有时更难调试。

从版本 2 开始。+， **Mockito 一直在引入新的特性，将框架推向“严格”**这背后的主要目标是:

*   检测测试代码中未使用的存根
*   减少测试代码重复和不必要的测试代码
*   通过移除“死”代码来促进更干净的测试
*   帮助提高可调试性和生产力

遵循这些原则有助于我们通过消除不必要的测试代码来创建更干净的测试。它们还帮助我们避免复制粘贴错误，以及其他开发人员的疏忽。

总而言之，严格存根报告不必要的存根，检测存根参数不匹配，并使我们的测试更加枯燥(不要重复自己)。这有助于一个干净且可维护的代码库。

### 2.1。配置严格存根

自从莫奇托 2。+，默认情况下，使用以下任一方法初始化模拟时，使用严格存根:

*   `MockitoJUnitRunner`
*   `MockitoJUnit.rule()`

**Mockito 强烈建议使用上述**中的任何一种。然而，当我们没有利用 Mockito 规则或 runner 时，还有另一种方法可以在我们的测试中实现严格的存根:

```java
Mockito.mockitoSession()
  .initMocks(this)
  .strictness(Strictness.STRICT_STUBS)
  .startMocking(); 
```

最后要强调的一点是，在 Mockito 3.0 中，默认情况下所有的存根都是“严格的”和有效的。

## 3。`UnnecessaryStubbingException`例子

简单地说，一个不必要的存根就是一个在测试执行期间从未被意识到的存根方法调用。

让我们看一个简单的例子:

```java
@Test
public void givenUnusedStub_whenInvokingGetThenThrowUnnecessaryStubbingException() {
    when(mockList.add("one")).thenReturn(true); // this won't get called
    when(mockList.get(anyInt())).thenReturn("hello");
    assertEquals("List should contain hello", "hello", mockList.get(1));
}
```

当我们运行这个单元测试时，Mockito 将检测未使用的存根并抛出一个`UnnecessaryStubbingException`:

```java
org.mockito.exceptions.misusing.UnnecessaryStubbingException: 
Unnecessary stubbings detected.
Clean & maintainable test code requires zero unnecessary code.
Following stubbings are unnecessary (click to navigate to relevant line of code):
  1\. -> at com.baeldung.mockito.misusing.MockitoUnecessaryStubUnitTest.givenUnusedStub_whenInvokingGetThenThrowUnnecessaryStubbingException(MockitoUnecessaryStubUnitTest.java:37)
Please remove unnecessary stubbings or use 'lenient' strictness. More info: javadoc for UnnecessaryStubbingException class.
```

幸运的是，从错误消息中可以很清楚地看出问题出在哪里。我们还可以看到异常消息甚至指出了导致错误的确切行。

为什么会这样？当我们使用参数`“one.”`调用`add`方法时，第一个`when`调用将我们的模拟配置为返回`true`,然而，在单元测试执行的剩余部分，我们不会调用这个方法。

**Mockito 告诉我们，我们的第一个`when`行是多余的，也许我们在配置存根时出错了。**

虽然这个例子很简单，但是很容易想象当模仿一个复杂的对象层次结构时，这种消息是如何帮助调试的，或者是非常有用的。

## 4。绕过严格的存根

**最后，我们来看看如何绕过严存根。**这也叫宽扎。

有时，我们需要将特定的存根配置为宽松的，同时保持所有其他存根和模拟使用严格的存根:

```java
@Test
public void givenLenientdStub_whenInvokingGetThenThrowUnnecessaryStubbingException() {
    lenient().when(mockList.add("one")).thenReturn(true);
    when(mockList.get(anyInt())).thenReturn("hello");
    assertEquals("List should contain hello", "hello", mockList.get(1));
}
```

在上面的例子中，**我们使用静态方法`Mockito.lenient()`来启用模拟列表的`add`方法的宽松存根。**

宽松的存根会绕过“严格的存根”验证规则。例如，当存根被声明为宽松时，将不会检查潜在的存根问题，如前面描述的不必要的存根。

## 5。结论

在这篇简短的文章中，我们在 Mockito 中介绍了严格存根的概念，详细说明了为什么引入它以及它为什么重要背后的哲学。

然后我们看了一个`UnnecessaryStubbingException,`的例子，最后看了一个如何在我们的测试中启用宽松存根的例子。

和往常一样，这篇文章的完整源代码可以在 [GitHub](https://web.archive.org/web/20221123101251/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito) 上找到。