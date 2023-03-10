# Mockito MockSettings 概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-mocksettings>

## 1。概述

通常情况下， [Mockito](/web/20221208143832/https://www.baeldung.com/tag/mockito/) 为我们的模拟对象提供的默认设置应该已经足够了。

然而，**有时我们可能需要在模拟创建过程中提供额外的模拟设置**。这在调试、处理遗留代码或处理某些特殊情况时可能会很有用。

在之前的教程中，我们学习了如何使用[宽松的模仿](/web/20221208143832/https://www.baeldung.com/mockito-unnecessary-stubbing-exception#lenient-stubbing)。在这个快速教程中，我们将学习如何使用`MockSettings`界面提供的其他一些有用的特性。

## 2。模拟设置

简单地说，`MockSettings`接口提供了一个[流畅的 API](/web/20221208143832/https://www.baeldung.com/mockito-fluent-apis) ,允许我们在创建模拟时轻松地添加和组合额外的模拟设置。

当我们创建一个模拟对象时，我们所有的模拟都带有一组默认设置。让我们看一个简单的模拟例子:

```java
List mockedList = mock(List.class);
```

在幕后，Mockito `mock`方法委托给另一个重载方法，该重载方法具有我们的 mock 的一组默认设置:

```java
public static <T> T mock(Class<T> classToMock) {
    return mock(classToMock, withSettings());
}
```

让我们看看我们的默认设置:

```java
public static MockSettings withSettings() {
    return new MockSettingsImpl().defaultAnswer(RETURNS_DEFAULTS);
}
```

正如我们所看到的，模拟对象的标准设置非常简单。我们为模拟交互配置默认答案。通常，使用`RETURNS_DEFAULTS`会返回一些空值。

**重要的一点是，如果需要的话，我们可以为我们的模拟对象提供我们自己的自定义设置。**

在接下来的部分中，我们将看到一些例子，这些例子可能会派上用场。

## 3。 **提供不同的默认答案**

现在我们对模拟设置有了更多的了解，让我们看看如何改变模拟对象的默认返回值。

假设我们有一个非常简单的模拟设置:

```java
PizzaService service = mock(PizzaService.class);
Pizza pizza = service.orderHouseSpecial();
PizzaSize size = pizza.getSize();
```

当我们按预期运行这段代码时，我们将得到一个`NullPointerException`，因为我们的未提交方法`orderHouseSpecial`返回了`null`。

这是可以的，但是有时当处理遗留代码时，我们可能需要处理复杂的模拟对象层次结构，并且定位这些类型的异常发生的位置可能很耗时。

为了帮助我们解决这个问题，**我们可以在模拟创建过程中通过模拟设置提供不同的默认答案**:

```java
PizzaService pizzaService = mock(PizzaService.class, withSettings().defaultAnswer(RETURNS_SMART_NULLS));
```

通过使用`RETURNS_SMART_NULLS`作为我们的默认答案，Mockito 给了我们一个更有意义的错误消息，告诉我们错误的存根发生在哪里:

```java
org.mockito.exceptions.verification.SmartNullPointerException: 
You have a NullPointerException here:
-> at com.baeldung.mockito.mocksettings.MockSettingsUnitTest.whenServiceMockedWithSmartNulls_thenExceptionHasExtraInfo(MockSettingsUnitTest.java:45)
because this method call was *not* stubbed correctly:
-> at com.baeldung.mockito.mocksettings.MockSettingsUnitTest.whenServiceMockedWithSmartNulls_thenExceptionHasExtraInfo(MockSettingsUnitTest.java:44)
pizzaService.orderHouseSpecial();
```

当调试我们的测试代码时，这确实可以节省我们一些时间。`Answers`枚举还提供了一些其他预先配置的模拟答案:

*   `RETURNS_DEEP_STUBS`–返回[深度存根的答案](/web/20221208143832/https://www.baeldung.com/mockito-fluent-apis#deep-mocking)–**这在使用流畅的 API 时会很有用**
*   `RETURNS_MOCKS`–使用这个答案将返回普通值，如空集合或空字符串，此后，它将尝试返回模拟值
*   顾名思义，当我们使用这个实现时，未提交的方法将委托给真正的实现

## 4。命名模拟和详细记录

我们可以通过使用`MockSettings`的`name`方法给我们的模拟命名。这对于调试特别有用，因为我们提供的名称会在所有验证错误中使用:

```java
PizzaService service = mock(PizzaService.class, withSettings()
  .name("pizzaServiceMock")
  .verboseLogging()
  .defaultAnswer(RETURNS_SMART_NULLS));
```

在这个例子中，我们通过使用方法`verboseLogging()`将这个命名特性与详细日志记录结合起来。

**使用这个方法可以为这个模拟**上的方法调用实时记录到标准输出流。同样，它可以在测试调试过程中使用，以便发现与 mock 的错误交互。

当我们运行测试时，我们将在控制台上看到一些输出:

```java
pizzaServiceMock.orderHouseSpecial();
   invoked: -> at com.baeldung.mockito.mocksettings.MockSettingsUnitTest.whenServiceMockedWithNameAndVerboseLogging_thenLogsMethodInvocations(MockSettingsUnitTest.java:36)
   has returned: "Mock for Pizza, hashCode: 366803687" (com.baeldung.mockito.fluentapi.Pizza$MockitoMock$168951489)
```

有趣的是，如果我们使用`@Mock`注释，我们的模拟会自动将字段名作为模拟名。

## 5。模仿额外接口

有时，我们可能想要指定我们的模拟应该实现的额外接口。同样，当处理我们不能重构的遗留代码时，这可能是有用的。

假设我们有一个特殊的界面:

```java
public interface SpecialInterface {
    // Public methods
}
```

和一个使用这个接口的类:

```java
public class SimpleService {

    public SimpleService(SpecialInterface special) {
        Runnable runnable = (Runnable) special;
        runnable.run();
    }
    // More service methods
}
```

当然，这不是[干净的代码](/web/20221208143832/https://www.baeldung.com/java-clean-code)，但是如果我们被迫为此编写单元测试，我们很可能会遇到问题:

```java
SpecialInterface specialMock = mock(SpecialInterface.class);
SimpleService service = new SimpleService(specialMock);
```

当我们运行这段代码时，我们会得到一个`ClassCastException`。**为了纠正这一点，我们可以使用`extraInterfaces`方法**创建具有多种类型的模拟:

```java
SpecialInterface specialMock = mock(SpecialInterface.class, withSettings()
  .extraInterfaces(Runnable.class));
```

现在，我们的模拟创建代码不会失败，但是我们应该强调，强制转换为未声明的类型并不酷。

## 6.提供构造函数参数

在最后一个例子中，我们将看到如何使用`MockSettings`调用带有参数值的真实构造函数:

```java
@Test
public void whenMockSetupWithConstructor_thenConstructorIsInvoked() {
    AbstractCoffee coffeeSpy = mock(AbstractCoffee.class, withSettings()
      .useConstructor("espresso")
      .defaultAnswer(CALLS_REAL_METHODS));

    assertEquals("Coffee name: ", "espresso", coffeeSpy.getName());
}
```

这一次，在创建我们的`AbstractCoffee` mock 的实例时，Mockito 尝试使用带有`String`值的构造函数。我们还将默认答案配置为委托给真正的实现。

如果我们在构造函数中有一些逻辑，我们想要测试或触发它，让我们的类处于某种特定的状态，这可能是有用的。当[在抽象类上刺探](/web/20221208143832/https://www.baeldung.com/mockito-spy)时，这也很有用。

## 7.结论

在这个快速教程中，我们已经看到了如何使用附加的模拟设置来创建模拟。

然而，我们应该重申，尽管这有时是有用的并且可能是不可避免的，但是在大多数情况下，我们应该努力使用简单的模拟来编写简单的测试。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito)