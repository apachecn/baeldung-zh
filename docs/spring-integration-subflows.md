# 在 Spring 集成中使用子流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-integration-subflows>

## 1。概述

Spring Integration 使得使用一些[企业集成模式](/web/20220627171759/https://www.baeldung.com/spring-integration)变得很容易。其中一种方式是通过[它的 DSL](/web/20220627171759/https://www.baeldung.com/spring-integration-java-dsl) 。

在本教程中，我们将看看 DSL 对子流的支持，以简化我们的一些配置。

## 2。我们的任务

假设我们有一个整数序列，我们想把它分成三个不同的桶。

如果我们想使用 Spring Integration 来实现这一点，我们可以从创建三个输出通道开始:

*   像 0、3、6 和 9 这样的数字将会进入`multipleOfThreeChannel`
*   像 1、4、7 和 10 这样的数字将会进入`remainderIsOneChannel`
*   而像 2、5、8 和 11 这样的数字则归入`remainderIsTwoChannel`

为了了解子流有多有用，让我们从没有子流的情况开始。

然后，我们将使用子流程来简化我们的配置:

*   `publishSubscribeChannel`
*   `routeToRecipients`
*   `Filter` s，来配置我们的`if-then`逻辑
*   *路由器* s，来配置我们的`switch`逻辑

## 3。先决条件

现在，在配置我们的子流之前，让我们创建那些输出通道。

我们将制作这些`QueueChannel`,因为这样更容易演示:

```java
@EnableIntegration
@IntegrationComponentScan
public class SubflowsConfiguration {

    @Bean
    QueueChannel multipleOfThreeChannel() {
        return new QueueChannel();
    }

    @Bean
    QueueChannel remainderIsOneChannel() {
        return new QueueChannel();
    }

    @Bean
    QueueChannel remainderIsTwoChannel() {
        return new QueueChannel();
    }

    boolean isMultipleOfThree(Integer number) {
       return number % 3 == 0;
    }

    boolean isRemainderIOne(Integer number) {
        return number % 3 == 1;
    }

    boolean isRemainderTwo(Integer number) {
        return number % 3 == 2;
    }
}
```

最终，这些是我们的分组数字将结束的地方。

还要注意，Spring 集成很容易看起来很复杂，所以为了可读性，我们将添加一些助手方法。

## 4。无子流求解

现在我们需要定义我们的流程。

如果没有子流程，简单的想法就是定义三个独立的集成流程，每个流程对应一种类型的数字。

**我们将向每个`IntegrationFlow` 组件发送相同的消息序列，但是每个组件的输出消息将会不同。**

### 4.1。定义`IntegrationFlow`组件

首先，让我们定义我们的`SubflowConfiguration `类中的每个`IntegrationFlow` bean:

```java
@Bean
public IntegrationFlow multipleOfThreeFlow() {
    return flow -> flow.split()
      .<Integer> filter(this::isMultipleOfThree)
      .channel("multipleOfThreeChannel");
}
```

我们的流程包含两个端点——一个`Splitter `后面跟着一个`Filt` `er`。

该过滤器做的就像它听起来那样。但是为什么我们还需要一个分离器呢？我们一会儿就会看到这一点，但基本上，它将一个输入`Collection` 分割成单独的消息。

当然，我们可以用同样的方式再定义两个`IntegrationFlow`bean。

### 4.2。消息网关

对于每个流，我们还需要一个`Message Gateway`。

简单地说，这些抽象了调用者的 Spring 集成消息 API，类似于 REST 服务如何抽象 HTTP:

```java
@MessagingGateway
public interface NumbersClassifier {

    @Gateway(requestChannel = "multipleOfThreeFlow.input")
    void multipleOfThree(Collection<Integer> numbers);

    @Gateway(requestChannel = "remainderIsOneFlow.input")
    void remainderIsOne(Collection<Integer> numbers);

    @Gateway(requestChannel = "remainderIsTwoFlow.input")
    void remainderIsTwo(Collection<Integer> numbers);

}
```

对于每一个，我们需要使用`@Gateway `注释并指定输入通道的隐式名称，它只是 bean 的名称后跟`“.input”`。**注意，我们可以使用这个约定，因为我们使用的是基于 lambda 的流。**

这些方法是我们心流的切入点。

### 4.3。发送消息并检查输出

现在，我们来测试一下:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { SeparateFlowsConfiguration.class })
public class SeparateFlowsUnitTest {

    @Autowired
    private QueueChannel multipleOfThreeChannel;

    @Autowired
    private NumbersClassifier numbersClassifier; 
```

```java
 @Test
    public void whenSendMessagesToMultipleOf3Flow_thenOutputMultiplesOf3() {
        numbersClassifier.multipleOfThree(Arrays.asList(1, 2, 3, 4, 5, 6));
        Message<?> outMessage = multipleOfThreeChannel.receive(0);
        assertEquals(outMessage.getPayload(), 3);
        outMessage = multipleOfThreeChannel.receive(0);
        assertEquals(outMessage.getPayload(), 6);
        outMessage = multipleOfThreeChannel.receive(0);
        assertNull(outMessage);
    }
}
```

请注意，我们已经将消息作为`List`发送，这就是为什么我们需要拆分器，将单个“列表消息”转换成几个“数字消息”。

我们用`o`调用`receive`来获取下一条可用消息，而无需等待。因为我们的列表中有两个 3 的倍数，我们希望能够调用它两次。对`receive `的第三次调用返回`null`。

`receive, `当然会返回一个`Message`，所以我们调用`getPayload` 来提取这个数字。

同样，我们可以对另外两个做同样的事情。

这就是没有分流的解决方案。我们需要维护三个独立的流和三个独立的网关方法。

我们现在要做的是用一个 bean 替换三个`IntegrationFlow `bean，用一个 bean 替换三个网关方法。

## 5。使用`publishSubscribeChannel`

`publishSubscribeChannel()`方法向所有订阅子流广播消息。这样，我们可以创建一个流，而不是三个。

```java
@Bean
public IntegrationFlow classify() {
    return flow -> flow.split()
        .publishSubscribeChannel(subscription -> 
           subscription
             .subscribe(subflow -> subflow
               .<Integer> filter(this::isMultipleOfThree)
               .channel("multipleOfThreeChannel"))
             .subscribe(subflow -> subflow
                .<Integer> filter(this::isRemainderOne)
                .channel("remainderIsOneChannel"))
             .subscribe(subflow -> subflow
                .<Integer> filter(this::isRemainderTwo)
                .channel("remainderIsTwoChannel")));
}
```

通过这种方式，**子流是匿名的，这意味着它们不能被独立地寻址。**

现在，我们只有一个流，所以让我们也编辑我们的`NumbersClassifier `:

```java
@Gateway(requestChannel = "classify.input")
void classify(Collection<Integer> numbers);
```

现在，**因为我们只有一个`IntegrationFlow` bean 和一个网关方法，我们只需要发送一次我们的列表:**

```java
@Test
public void whenSendMessagesToFlow_thenNumbersAreClassified() {
    numbersClassifier.classify(Arrays.asList(1, 2, 3, 4, 5, 6));

    // same assertions as before
}
```

注意，从现在开始，只有集成流定义会改变，因此我们不会再显示测试。

## 6。使用`routeToRecipients`

**另一种实现相同功能的方法是`routeToRecipients`，这很好，因为它内置了过滤功能。**

使用这种方法，**我们可以为广播指定频道和子流。**

### 6.1。`recipient`

在下面的代码中，我们将根据我们的条件指定`multipleof3Channel`、`remainderIs1Channel,` 和`remainderIsTwoChannel`为接收者:

```java
@Bean
public IntegrationFlow classify() {
    return flow -> flow.split()
        .routeToRecipients(route -> route
          .<Integer> recipient("multipleOfThreeChannel", 
            this::isMultipleOfThree)       
          .<Integer> recipient("remainderIsOneChannel", 
            this::isRemainderOne)
          .<Integer> recipient("remainderIsTwoChannel", 
            this::isRemainderTwo));
}
```

我们也可以无条件调用`recipient `，`routeToRecipients `会无条件发布到那个目的地。

### 6.2.`recipientFlow`

注意`routeToRecipients`允许我们定义一个完整的流程，就像`publishSubscribeChannel. `一样

让我们修改上面的代码，并将一个**匿名子流指定为第一个接收者**:

```java
.routeToRecipients(route -> route
  .recipientFlow(subflow -> subflow
      .<Integer> filter(this::isMultipleOfThree)
      .channel("mutipleOfThreeChannel"))
  ...);
```

**这个子流将接收整个消息序列，**因此我们需要像之前一样进行过滤，以获得相同的行为。

又一次，一颗豆子对我们来说足够了。

现在让我们继续讨论`if-else`组件。其中一个就是`Filter`。

## 7。使用`if-then` 流程

我们已经在前面的例子中使用了`Filter`。好消息是，我们不仅可以指定进一步处理的条件，还可以为被丢弃的消息指定**通道或**流。

**我们可以把丢弃的流和通道想象成一个`else `块:**

```java
@Bean
public IntegrationFlow classify() {
    return flow -> flow.split()
        .<Integer> filter(this::isMultipleOfThree, 
           notMultiple -> notMultiple
             .discardFlow(oneflow -> oneflow
               .<Integer> filter(this::isRemainderOne,
                 twoflow -> twoflow
                   .discardChannel("remainderIsTwoChannel"))
               .channel("remainderIsOneChannel"))
        .channel("multipleofThreeChannel");
}
```

在本例中，我们已经实现了我们的`if-else`路由逻辑:

*   `If`数量不是三的倍数，`then`将这些消息丢弃到丢弃流；**我们在这里使用流，因为需要更多的逻辑来了解它的目的地通道。**
*   在丢弃流中，`if`数字不是余数 1，`then`将这些消息丢弃到丢弃通道。

## 8。根据计算值

最后，让我们尝试一下`route `方法，这种方法比`routeToRecipients. `给了我们更多的控制。这很好，因为`Router`可以将气流分成任意数量的部分，而`Filter `只能做两部分。

### 8.1.`channelMapping`

让我们定义一下我们的`IntegrationFlow` bean:

```java
@Bean
public IntegrationFlow classify() {
    return classify -> classify.split()
      .<Integer, Integer> route(number -> number % 3, 
        mapping -> mapping
         .channelMapping(0, "multipleOfThreeChannel")
         .channelMapping(1, "remainderIsOneChannel")
         .channelMapping(2, "remainderIsTwoChannel"));
}
```

在上面的代码中，我们通过执行除法来计算路由关键字:

```java
route(p -> p % 3,...
```

基于这个密钥，我们路由消息:

```java
channelMapping(0, "multipleof3Channel")
```

### 8.2.`subFlowMapping`

现在，和其他流程一样，我们可以通过指定一个子流程来进行更多的控制，用`subFlowMapping`替换`channelMapping`:

```java
.subFlowMapping(1, subflow -> subflow.channel("remainderIsOneChannel"))
```

或者通过调用`handle `方法而不是`channel `方法来获得更多的控制:

```java
.subFlowMapping(2, subflow -> subflow
  .<Integer> handle((payload, headers) -> {
      // do extra work on the payload
     return payload;
  }))).channel("remainderIsTwoChannel");
```

在这种情况下，子流将在 `route()` 方法之后返回到主流，因此我们需要指定通道`remainderIsTwoChannel.`

## 9.结论

在本教程中，我们探索了如何使用子流以某种方式过滤和路由消息。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627171759/https://github.com/eugenp/tutorials/tree/master/spring-integration)