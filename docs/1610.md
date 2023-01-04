# Mockito 中 when()和 doXxx()方法的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mockito-when-vs-do>

## 1.介绍

[Mockito](/web/20221208143832/https://www.baeldung.com/mockito-series) 是一个流行的 Java 嘲讽框架。有了它，[创建模仿对象](/web/20221208143832/https://www.baeldung.com/mockito-mock-methods)，[配置模仿行为](/web/20221208143832/https://www.baeldung.com/mockito-behavior)，[捕获方法参数](/web/20221208143832/https://www.baeldung.com/mockito-argument-matchers)，[验证与模仿](/web/20221208143832/https://www.baeldung.com/mockito-verify)的交互就变得简单了。

**现在，我们将关注指定模仿行为。我们有两种方法可以做到这一点:`when().thenDoSomething()`和`doSomething().when()`语法。**

在这个简短的教程中，我们将看到为什么我们有这两个。

## 2.`when()`方法

让我们考虑下面的`Employee`界面:

```
interface Employee {
    String greet();
    void work(DayOfWeek day);
}
```

在我们的测试中，我们使用了这个接口的模拟。假设我们想要配置 mock 的`greet()`方法来返回字符串`“Hello”`。使用莫克托的`when()`方法很简单:

```
@Test
void givenNonVoidMethod_callingWhen_shouldConfigureBehavior() {
    // given
    when(employee.greet()).thenReturn("Hello");

    // when
    String greeting = employee.greet();

    // then
    assertThat(greeting, is("Hello"));
}
```

会发生什么？对象是一个模仿对象。**当我们调用它的任何方法时，Mockito 会注册这个调用。通过调用`when()`方法，Mockito 知道这个调用不是业务逻辑的交互。这是一个声明，我们想给模拟对象分配一些行为。之后，使用`thenXxx()`方法之一，我们指定预期的行为。**

直到这一点，这是很好的老嘲讽。同样，我们想配置`work()`方法，当我们用 Sunday 参数调用它时抛出一个异常:

```
@Test
void givenVoidMethod_callingWhen_wontCompile() {
    // given
    when(employee.work(DayOfWeek.SUNDAY)).thenThrow(new IAmOnHolidayException());

    // when
    Executable workCall = () -> employee.work(DayOfWeek.SUNDAY);

    // then
    assertThrows(IAmOnHolidayException.class, workCall);
}
```

**不幸的是，这段代码不会编译，因为在`work(employee.work(…))`调用中，`work()`方法有一个`void`返回类型；因此，我们不能将它包装到另一个方法调用中。**这是否意味着我们不能模仿 void 方法？当然可以。`doXxx`救援方法！

## 3.`doXxx()`方法

让我们看看如何用`doThrow()`方法配置异常抛出:

```
@Test
void givenVoidMethod_callingDoThrow_shouldConfigureBehavior() {
    // given
    doThrow(new IAmOnHolidayException()).when(employee).work(DayOfWeek.SUNDAY);

    // when
    Executable workCall = () -> employee.work(DayOfWeek.SUNDAY);

    // then
    assertThrows(IAmOnHolidayException.class, workCall);
}
```

这个语法与前一个略有不同:我们不试图将一个`void`方法调用包装在另一个方法调用中。因此，这段代码可以编译。

让我们看看刚刚发生了什么。首先，我们声明我们想要抛出一个异常。接下来，我们调用了`when()`方法，并传递了模拟对象。之后，我们指定了想要配置哪个模拟交互的行为。

注意，这与我们之前使用的`when()`方法不同。另外，请注意，我们在调用`when().`之后链接了模拟交互，同时，用第一个语法在括号内定义了它。

当第一个`when().thenXxx()`不能完成配置`void`调用这样的普通任务时，为什么我们要有第一个`when().thenXxx()`？与`doXxx().when()`语法相比，它有很多优点。

首先，**对于开发者来说，编写和阅读类似“当一些交互时，然后做一些事情”这样的语句比“做一些事情，当一些交互时”更符合逻辑。**

第二，我们可以用链接向同一个交互添加多个行为。那是因为`when()`返回类 [`OngoingStubbing<T>`](https://web.archive.org/web/20221208143832/https://javadoc.io/static/org.mockito/mockito-core/3.5.10/org/mockito/stubbing/OngoingStubbing.html) 的一个实例，也就是`thenXxx()`方法返回相同的类型。

另一方面，`doXxx()`方法返回一个`[Stubber](https://web.archive.org/web/20221208143832/https://javadoc.io/static/org.mockito/mockito-core/3.5.10/org/mockito/stubbing/Stubber.html)`实例，`Stubber.when(T mock)`返回`T`，所以我们可以指定我们想要配置哪种方法调用。但是`T`是我们应用程序的一部分，例如，我们代码片段中的`Employee`。但是`T`不会返回一个 Mockito 类，所以我们不能用链式添加多个行为。

## 4.BDDMockito

[BDDMockito](/web/20221208143832/https://www.baeldung.com/bdd-mockito) 使用了一种我们之前讨论过的替代语法。这非常简单:在我们的模拟配置中，我们必须将关键字“`when”`替换为“`given`”，将关键字“`do`”替换为“`will`”。除此之外，我们的代码保持不变:

```
@Test
void givenNonVoidMethod_callingGiven_shouldConfigureBehavior() {
    // given
    given(employee.greet()).willReturn("Hello");

    // when
    String greeting = employee.greet();

    // then
    assertThat(greeting, is("Hello"));
}

@Test
void givenVoidMethod_callingWillThrow_shouldConfigureBehavior() {
    // given
    willThrow(new IAmOnHolidayException()).given(employee).work(DayOfWeek.SUNDAY);

    // when
    Executable workCall = () -> employee.work(DayOfWeek.SUNDAY);

    // then
    assertThrows(IAmOnHolidayException.class, workCall);
}
```

## 5.结论

我们看到了用`when().thenXxx()`或`doXxx().when()`方式配置模拟对象的优点和缺点。此外，我们看到了这些语法是如何工作的，以及为什么我们有这两种语法。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito)