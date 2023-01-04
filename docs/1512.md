# Mockito 参数匹配器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-argument-matchers>

## 1.概观

在本教程中，我们将学习**如何使用 `ArgumentMatcher,`，并讨论它与`ArgumentCaptor`** 有何不同。

关于 Mockito 框架的介绍，请参考本文。

## 2.Maven 依赖性

我们需要添加一个工件:

```
<dependency>
    <groupId>org.mockito</groupId> 
    <artifactId>mockito-core</artifactId>
    <version>2.21.0</version> 
    <scope>test</scope>
</dependency>
```

最新版本的 [`Mockito`](https://web.archive.org/web/20220706105622/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.mockito%22%20AND%20a%3A%22mockito-core%22) 可以在`Maven Central`上找到。

## 3.`ArgumentMatchers`

我们可以用各种方式配置一个模拟方法。一种选择是返回一个固定值:

```
doReturn("Flower").when(flowerService).analyze("poppy");
```

在上面的例子中，只有当分析服务接收到`String`“poppy”时，才会返回`String`“Flower”

但是可能会有这样的情况，我们需要对更大范围的值或未知值做出响应。

在这些场景中，**我们可以用`argument`** `**matchers**:`配置我们的模仿方法

```
when(flowerService.analyze(anyString())).thenReturn("Flower");
```

现在，因为有了`anyString`参数匹配器，无论我们传递什么值来分析，结果都是一样的。`ArgumentMatchers`允许我们灵活验证或存根。

**如果一个方法有不止一个参数，我们不能只对某些参数使用`ArgumentMatchers`。** `Mockito`要求我们通过`matchers`或精确值提供所有参数。

这里我们可以看到一个不正确方法的例子:

```
abstract class FlowerService {
    public abstract boolean isABigFlower(String name, int petals);
}

FlowerService mock = mock(FlowerService.class);

when(mock.isABigFlower("poppy", anyInt())).thenReturn(true);
```

为了解决这个问题，并根据需要保留名称“poppy ”,我们将使用`eq matcher`:

```
when(mock.isABigFlower(eq("poppy"), anyInt())).thenReturn(true);
```

当我们使用`matchers`时，还有两点需要注意:

*   **我们不能用它们作为返回值；**我们需要一个精确的值。
*   **我们不能在验证或存根之外使用参数`matchers`。**

根据第二点，`Mockito` 将检测放错位置的参数并抛出一个`InvalidUseOfMatchersException`。

一个不好的例子是:

```
String orMatcher = or(eq("poppy"), endsWith("y"));
verify(mock).analyze(orMatcher);
```

我们实现上述代码的方式是:

```
verify(mock).analyze(or(eq("poppy"), endsWith("y")));
```

`Mockito`还提供了`AdditionalMatchers`来实现对`ArgumentMatchers`的通用逻辑运算(“非”、“与”、“或”)，这些运算同时匹配基本类型和非基本类型:

```
verify(mock).analyze(or(eq("poppy"), endsWith("y")));
```

## 4.自定义参数匹配器

创建我们自己的`matcher`允许我们为一个给定的场景选择最好的方法，并且产生干净和可维护的高质量测试。

例如，我们可以有一个`MessageController`来传递消息。它将接收一个`MessageDTO`，并由此创建一个`MessageService`将交付的`Message`。

我们的验证将很简单；我们将验证我们用`any Message:`调用了`MessageService`1 次

```
verify(messageService, times(1)).deliverMessage(any(Message.class));
```

因为**中的`Message`是在被测方法**中构造的，所以我们必须使用`any`作为`matcher`。

这种方法不允许我们验证`Message`、**中的数据，这些数据可能不同于`MessageDTO`、**中的数据。

出于这个原因，我们将实现一个自定义的参数匹配器:

```
public class MessageMatcher implements ArgumentMatcher<Message> {

    private Message left;

    // constructors

    @Override
    public boolean matches(Message right) {
        return left.getFrom().equals(right.getFrom()) &&
          left.getTo().equals(right.getTo()) &&
          left.getText().equals(right.getText()) &&
          right.getDate() != null &&
          right.getId() != null;
    }
}
```

**要使用我们的匹配器，我们需要修改我们的测试，用`argThat`** 替换`any`:

```
verify(messageService, times(1)).deliverMessage(argThat(new MessageMatcher(message)));
```

现在我们知道我们的`Message`实例将拥有与我们的`MessageDTO`相同的数据。

## 5.自定义参数匹配器与`ArgumentCaptor`

两种技术，`custom argument matchers`和`ArgumentCaptor,`都可以用来确保某些参数被传递给模拟。

然而，如果我们需要用 **`ArgumentCaptor`断言参数值**来完成验证，那么**或我们的`custom argument matcher`可能不会被重用**，那么`ArgumentCaptor`可能会更合适。

`Custom argument matchers` via `ArgumentMatcher`通常比较适合打闷棍。

## 6.结论

在本文中，我们探索了`Mockito`的一个特性`ArgumentMatcher`。我们还讨论了它与`ArgumentCaptor`T3 有什么不同

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220706105622/https://github.com/eugenp/tutorials/tree/master/spring-mockito)