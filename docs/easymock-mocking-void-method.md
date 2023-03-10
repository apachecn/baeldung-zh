# 用 EasyMock 模拟 Void 方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/easymock-mocking-void-method>

## 1。概述

模仿框架被用来模仿与依赖项的交互，以便孤立地测试我们的类。通常，我们模拟依赖关系来返回各种可能的值。这样，我们可以确保我们的类能够处理这些值。

但是，有时我们可能不得不模仿不返回任何东西的依赖方法。

在本教程中，**我们将看到何时以及如何使用 EasyMock 模仿`void`方法。**

## 2。Maven 依赖关系

首先，让我们将 EasyMock 依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.easymock</groupId>
    <artifactId>easymock</artifactId>
    <version>4.0.2</version>
    <scope>test</scope>
</dependency>
```

## 3。什么时候嘲弄一个`void`法

当我们测试具有依赖关系的类时，我们通常希望覆盖依赖关系返回的所有值。但是有时，依赖方法不返回值。所以，**如果没有返回任何东西，我们为什么要模仿一个`void`方法呢？**

即使 void 方法不返回值，**它们也可能有副作用。**这方面的一个例子是`Session.save()`方法。当我们保存一个新实体时，`save()`方法会生成一个 id，并在传递的实体上设置它。

出于这个原因，我们必须模仿 void 方法来模拟各种处理结果。

模拟可能派上用场的另一个场合是测试 void 方法抛出的异常时。

## 4。如何嘲弄一个`void`法

现在，让我们看看如何使用 EasyMock 模拟 void 方法。

让我们假设，我们必须模拟一个`WeatherService`类的 void 方法，它接受一个位置并设置最低和最高温度:

```java
public interface WeatherService {
    void populateTemperature(Location location);
}
```

### 4.1。创建模拟对象

让我们从为`WeatherService`创建一个模拟开始:

```java
@Mock
private WeatherService mockWeatherService;
```

这里，我们已经使用 EasyMock 注释`@Mock`完成了这项工作。但是，我们也可以使用`EasyMock.mock()`方法做到这一点。

接下来，我们将通过调用`populateTemperature()`来记录与 mock 的预期交互:

```java
mockWeatherService.populateTemperature(EasyMock.anyObject(Location.class));
```

现在，如果我们不想模拟这个方法的处理，这个调用本身就足以模拟这个方法。

### 4.2。抛出异常

首先，让我们看一下我们想要测试我们的类是否可以处理 void 方法抛出的**异常的情况。为此，我们必须模拟这个方法，让它抛出这些异常。**

在我们的例子中，该方法抛出`ServiceUnavailableException`:

```java
EasyMock.expectLastCall().andThrow(new ServiceUnavailableException());
```

如上所述，这包括简单地调用`andThrow(Throwable)`方法。

### 4.3。模拟方法行为

如前所述，我们有时可能需要**模拟 void 方法的行为。**

在我们的例子中，这将涉及填充所经过位置的最低和最高温度:

```java
EasyMock.expectLastCall()
  .andAnswer(() -> {
      Location passedLocation = (Location) EasyMock.getCurrentArguments()[0];
      passedLocation.setMaximumTemparature(new BigDecimal(MAX_TEMP));
      passedLocation.setMinimumTemperature(new BigDecimal(MAX_TEMP - 10));
      return null;
  });
```

这里，我们使用了`andAnswer(IAnswer)`方法来定义`populateTemperature()`方法被调用时的行为。然后，我们使用了`EasyMock.getCurrentArguments()`方法——它返回传递给模拟方法的参数——来修改传递的位置。

注意，我们已经在最后将**返回了`null`。**这是因为我们在嘲讽一种虚无的方法。

同样值得注意的是，这种方法不仅限于模仿 void 方法。我们也可以将它用于返回值的方法。在这里，当我们想要模拟方法根据传递的参数返回值时，它就派上了用场。

### 4.4。重放被模仿的方法

最后，我们将使用`EasyMock.replay()`方法将 mock 更改为“replay”模式，以便记录的动作可以在被调用时重放:

```java
EasyMock.replay(mockWeatherService);
```

因此，当我们调用测试方法时，应该执行所定义的自定义行为。

## 5。结论

在本教程中，我们看到了如何使用 EasyMock 模拟 void 方法。

当然，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628083632/https://github.com/eugenp/tutorials/tree/master/testing-modules/easymock)