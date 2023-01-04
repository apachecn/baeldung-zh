# 大使介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mbassador>

## 1。概述

简单地说， [MBassador](https://web.archive.org/web/20220629002728/https://github.com/bennidi/mbassador) 是**一个利用[发布-订阅](https://web.archive.org/web/20220629002728/https://en.wikipedia.org/wiki/Publish–subscribe_pattern)语义的高性能事件总线。**

消息被广播给一个或多个对等体，而事先不知道有多少订户，或者他们如何使用该消息。

## 2。Maven 依赖关系

在我们可以使用这个库之前，我们需要添加 [mbassador](https://web.archive.org/web/20220629002728/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22mbassador%22) 依赖项:

```
<dependency>
    <groupId>net.engio</groupId>
    <artifactId>mbassador</artifactId>
    <version>1.3.1</version>
</dependency>
```

## 3。基本事件处理

### 3.1。简单的例子

我们将从发布消息的简单示例开始:

```
private MBassador<Object> dispatcher = new MBassador<>();
private String messageString;

@Before
public void prepareTests() {
    dispatcher.subscribe(this);
}

@Test
public void whenStringDispatched_thenHandleString() {
    dispatcher.post("TestString").now();

    assertNotNull(messageString);
    assertEquals("TestString", messageString);
}

@Handler
public void handleString(String message) {
    messageString = message;
} 
```

在这个测试类的顶部，我们看到用默认构造函数创建了一个`MBassador`。接下来，在 `@Before`方法中，我们调用`subscribe()`并传入对类本身的引用。

在`subscribe(),`中，调度程序检查订户的`@Handler`注释。

并且，在第一个测试中，我们调用了`dispatcher.post(…).now()`来发送消息——这导致了`handleString()`被调用。

这个初始测试演示了几个重要的概念。**任何`Object`都可以是订阅者，只要它有一个或多个用`@Handler`标注的方法。**一个用户可以拥有任意数量的处理程序。

为了简单起见，我们使用了订阅自身的测试对象，但是在大多数生产场景中，消息调度程序与消费者在不同的类中。

处理程序方法只有一个输入参数——消息，并且不能抛出任何检查过的异常。

类似于`subscribe()`方法，post 方法接受任何`Object`。这个`Object`是交付给订户的。

**当消息被发布时，它被传递给任何订阅了该消息类型的收听者。**

让我们添加另一个消息处理程序并发送不同的消息类型:

```
private Integer messageInteger; 

@Test
public void whenIntegerDispatched_thenHandleInteger() {
    dispatcher.post(42).now();

    assertNull(messageString);
    assertNotNull(messageInteger);
    assertTrue(42 == messageInteger);
}

@Handler
public void handleInteger(Integer message) {
    messageInteger = message;
} 
```

不出所料，当我们调度一个`Integer`，`handleInteger()`被调用，`handleString()`不被调用。一个调度程序可以用于发送多种消息类型。

### 3.2。无效消息

那么，如果没有消息的处理程序，消息会去哪里呢？让我们添加一个新的事件处理程序，然后发送第三种消息类型:

```
private Object deadEvent; 

@Test
public void whenLongDispatched_thenDeadEvent() {
    dispatcher.post(42L).now();

    assertNull(messageString);
    assertNull(messageInteger);
    assertNotNull(deadEvent);
    assertTrue(deadEvent instanceof Long);
    assertTrue(42L == (Long) deadEvent);
} 

@Handler
public void handleDeadEvent(DeadMessage message) {
    deadEvent = message.getMessage();
} 
```

在这个测试中，我们分派了一个`Long`而不是一个`Integer.``handleInteger()`和`handleString()`都没有被调用，但是`handleDeadEvent()`被调用了。

**当消息没有处理程序时，它被包装在一个`DeadMessage`对象中。**因为我们为`Deadmessage`添加了一个处理程序，所以我们捕获了它。

`DeadMessage`可以放心忽略；如果应用程序不需要跟踪失效消息，可以允许它们无处可去。

## 4。使用事件层级

发送`String`和`Integer`事件受到限制。让我们创建几个消息类:

```
public class Message {}

public class AckMessage extends Message {}

public class RejectMessage extends Message {
    int code;

    // setters and getters
}
```

我们有一个简单的基类和两个扩展它的类。

### 4.1。发送基类`Message`

我们从`Message`事件开始:

```
private MBassador<Message> dispatcher = new MBassador<>();

private Message message;
private AckMessage ackMessage;
private RejectMessage rejectMessage;

@Before
public void prepareTests() {
    dispatcher.subscribe(this);
}

@Test
public void whenMessageDispatched_thenMessageHandled() {
    dispatcher.post(new Message()).now();
    assertNotNull(message);
    assertNull(ackMessage);
    assertNull(rejectMessage);
}

@Handler
public void handleMessage(Message message) {
    this.message = message;
}

@Handler
public void handleRejectMessage(RejectMessage message) {
   rejectMessage = message;
}

@Handler
public void handleAckMessage(AckMessage message) {
    ackMessage = message;
}
```

探索 MBassador——一辆高性能的公共活动巴士。这限制了我们使用`Messages` ，但是增加了一层额外的类型安全。

当我们发送一个`Message`，`handleMessage()`接收它。其他两个处理程序没有。

### 4.2。发送子类消息

先发个`RejectMessage`:

```
@Test
public void whenRejectDispatched_thenMessageAndRejectHandled() {
    dispatcher.post(new RejectMessage()).now();

    assertNotNull(message);
    assertNotNull(rejectMessage);
    assertNull(ackMessage);
}
```

当我们发送一个`RejectMessage`时，`handleRejectMessage()`和`handleMessage()`都会收到。

由于`RejectMessage` 扩展了`Message,`，`Message`处理程序收到了它，此外还有`R`，`ejectMessage` 处理程序。

让我们用一个`AckMessage`来验证这个行为:

```
@Test
public void whenAckDispatched_thenMessageAndAckHandled() {
    dispatcher.post(new AckMessage()).now();

    assertNotNull(message);
    assertNotNull(ackMessage);
    assertNull(rejectMessage);
}
```

正如我们所料，当我们发送一个`AckMessage`时，`handleAckMessage()`和`handleMessage()`都会收到。

## 5。过滤信息

按类型组织消息已经是一个强大的功能，但是我们可以对它们进行更多的过滤。

### 5.1。类别和子类的过滤器

当我们发布一个`RejectMessage`或`AckMessage`时，我们在特定类型的事件处理程序和基类中都收到了事件。

我们可以通过将`Message`抽象化并创建一个像`GenericMessage`这样的类来解决这个类型层次问题。但是如果我们没有这种奢侈呢？

我们可以使用消息过滤器:

```
private Message baseMessage;
private Message subMessage;

@Test
public void whenMessageDispatched_thenMessageFiltered() {
    dispatcher.post(new Message()).now();

    assertNotNull(baseMessage);
    assertNull(subMessage);
}

@Test
public void whenRejectDispatched_thenRejectFiltered() {
    dispatcher.post(new RejectMessage()).now();

    assertNotNull(subMessage);
    assertNull(baseMessage);
}

@Handler(filters = { @Filter(Filters.RejectSubtypes.class) })
public void handleBaseMessage(Message message) {
    this.baseMessage = message;
}

@Handler(filters = { @Filter(Filters.SubtypesOnly.class) })
public void handleSubMessage(Message message) {
    this.subMessage = message;
}
```

**注释`@Handler`的`filters`参数接受一个实现`IMessageFilter`** 的`Class`。本库提供了两个例子:

`Filters.RejectSubtypes`顾名思义:它会过滤掉任何子类型。在这种情况下，我们看到`RejectMessage`不是由`handleBaseMessage().`处理的

`Filters.SubtypesOnly`也顾名思义:它会过滤掉任何基本类型。在这种情况下，我们看到`Message`不是由`handleSubMessage().`处理的

### 5.2。`IMessageFilter`

`Filters.RejectSubtypes`和`Filters.SubtypesOnly`都实现了`IMessageFilter`。

`RejectSubTypes` 将消息的类别与其定义的消息类型进行比较，只允许等于其类型之一的消息通过，而不是任何子类。

### 5.3。条件过滤

幸运的是，有一种更简单的过滤信息的方法。 **MBassador 支持 [Java EL 表达式](https://web.archive.org/web/20220629002728/https://en.wikipedia.org/wiki/Unified_Expression_Language)的子集作为过滤消息的条件。**

让我们根据消息的长度来过滤一条`String`消息:

```
private String testString;

@Test
public void whenLongStringDispatched_thenStringFiltered() {
    dispatcher.post("foobar!").now();

    assertNull(testString);
}

@Handler(condition = "msg.length() < 7")
public void handleStringMessage(String message) {
    this.testString = message;
}
```

“foobar！”消息长度为七个字符，并被过滤。让我们发一个更短的`String`:

```
 @Test
public void whenShortStringDispatched_thenStringHandled() {
    dispatcher.post("foobar").now();

    assertNotNull(testString);
}
```

现在，“foobar”只有六个字符长，并且是通过。

我们的`RejectMessage`包含一个带有访问器的字段。让我们为此编写一个过滤器:

```
private RejectMessage rejectMessage;

@Test
public void whenWrongRejectDispatched_thenRejectFiltered() {

    RejectMessage testReject = new RejectMessage();
    testReject.setCode(-1);

    dispatcher.post(testReject).now();

    assertNull(rejectMessage);
    assertNotNull(subMessage);
    assertEquals(-1, ((RejectMessage) subMessage).getCode());
}

@Handler(condition = "msg.getCode() != -1")
public void handleRejectMessage(RejectMessage rejectMessage) {
    this.rejectMessage = rejectMessage;
}
```

同样，我们可以在对象上查询一个方法，并决定是否过滤消息。

### 5.4。捕获过滤的消息

与`DeadEvents,`类似，我们可能希望捕获和处理过滤后的消息。还有一个专门的机制来捕获过滤的事件。**过滤事件与“死亡”事件的处理方式不同。**

让我们编写一个测试来说明这一点:

```
private String testString;
private FilteredMessage filteredMessage;
private DeadMessage deadMessage;

@Test
public void whenLongStringDispatched_thenStringFiltered() {
    dispatcher.post("foobar!").now();

    assertNull(testString);
    assertNotNull(filteredMessage);
    assertTrue(filteredMessage.getMessage() instanceof String);
    assertNull(deadMessage);
}

@Handler(condition = "msg.length() < 7")
public void handleStringMessage(String message) {
    this.testString = message;
}

@Handler
public void handleFilterMessage(FilteredMessage message) {
    this.filteredMessage = message;
}

@Handler
public void handleDeadMessage(DeadMessage deadMessage) {
    this.deadMessage = deadMessage;
} 
```

通过添加一个`FilteredMessage`处理程序，我们可以跟踪因为长度而被过滤的`Strings`。`filterMessage` 包含了我们太长的`String` ，而`deadMessage` 仍然是`null.`

## 6。异步消息调度和处理

到目前为止，我们所有的例子都使用了同步消息调度；当我们调用`post.now()`时，消息被传递到我们调用`post()`的同一个线程中的每个处理程序。

### 6.1。异步调度

`MBassador.post()`返回一个 [SyncAsyncPostCommand](https://web.archive.org/web/20220629002728/https://bennidi.github.io/mbassador/net/engio/mbassy/bus/publication/SyncAsyncPostCommand.html) 。该类提供了几种方法，包括:

*   `now()`——同步发送消息；呼叫将被阻塞，直到所有消息都已传递
*   `asynchronously()`–异步执行消息发布

让我们在一个示例类中使用异步调度。我们将在这些测试中使用[可用性](/web/20220629002728/https://www.baeldung.com/awaitlity-testing)来简化代码:

```
private MBassador<Message> dispatcher = new MBassador<>();
private String testString;
private AtomicBoolean ready = new AtomicBoolean(false);

@Test
public void whenAsyncDispatched_thenMessageReceived() {
    dispatcher.post("foobar").asynchronously();

    await().untilAtomic(ready, equalTo(true));
    assertNotNull(testString);
}

@Handler
public void handleStringMessage(String message) {
    this.testString = message;
    ready.set(true);
}
```

我们在这个测试中调用了`asynchronously()`，并使用一个`AtomicBoolean`作为带有`await()`的标志来等待交付线程交付消息。

如果我们注释掉对`await()`的调用，我们就冒着测试失败的风险，因为我们在交付线程完成之前检查了`testString` 。

### 6.2。异步处理程序调用

异步调度允许消息提供者在消息被传递到每个处理程序之前返回到消息处理，但是它仍然按顺序调用每个处理程序，并且每个处理程序必须等待上一个处理程序完成。

如果一个处理程序执行一个昂贵的操作，这可能会导致问题。

MBassador 为异步处理程序调用提供了一种机制。为此配置的处理程序在其线程中接收消息:

```
private Integer testInteger;
private String invocationThreadName;
private AtomicBoolean ready = new AtomicBoolean(false);

@Test
public void whenHandlerAsync_thenHandled() {
    dispatcher.post(42).now();

    await().untilAtomic(ready, equalTo(true));
    assertNotNull(testInteger);
    assertFalse(Thread.currentThread().getName().equals(invocationThreadName));
}

@Handler(delivery = Invoke.Asynchronously)
public void handleIntegerMessage(Integer message) {

    this.invocationThreadName = Thread.currentThread().getName();
    this.testInteger = message;
    ready.set(true);
}
```

处理程序可以使用`Handler`注释上的`delivery = Invoke.Asynchronously`属性请求异步调用。我们在测试中通过比较分派方法和处理程序中的`Thread`名称来验证这一点。

## 7。定制 MBassador

到目前为止，我们一直使用默认配置的 MBassador 实例。调度员的行为可以用注释来修改，类似于我们目前看到的那些；我们将讨论更多的内容来结束本教程。

### 7.1。异常处理

处理程序不能定义已检查的异常。相反，可以向调度程序提供一个`IPublicationErrorHandler`作为其构造函数的参数:

```
public class MBassadorConfigurationTest
  implements IPublicationErrorHandler {

    private MBassador dispatcher;
    private String messageString;
    private Throwable errorCause;

    @Before
    public void prepareTests() {
        dispatcher = new MBassador<String>(this);
        dispatcher.subscribe(this);
    }

    @Test
    public void whenErrorOccurs_thenErrorHandler() {
        dispatcher.post("Error").now();

        assertNull(messageString);
        assertNotNull(errorCause);
    }

    @Test
    public void whenNoErrorOccurs_thenStringHandler() {
        dispatcher.post("Error").now();

        assertNull(errorCause);
        assertNotNull(messageString);
    }

    @Handler
    public void handleString(String message) {
        if ("Error".equals(message)) {
            throw new Error("BOOM");
        }
        messageString = message;
    }

    @Override
    public void handleError(PublicationError error) {
        errorCause = error.getCause().getCause();
    }
}
```

当 `handleString()` 抛出一个`Error,`时，它被保存到`errorCause.`

### 7.2。处理器优先级

**`Handlers`的调用顺序与它们被添加的顺序相反，但这不是我们想要依赖的行为。**即使能够在线程中调用处理程序，我们可能仍然需要知道它们将被调用的顺序。

我们可以显式设置处理程序优先级:

```
private LinkedList<Integer> list = new LinkedList<>();

@Test
public void whenRejectDispatched_thenPriorityHandled() {
    dispatcher.post(new RejectMessage()).now();

    // Items should pop() off in reverse priority order
    assertTrue(1 == list.pop());
    assertTrue(3 == list.pop());
    assertTrue(5 == list.pop());
}

@Handler(priority = 5)
public void handleRejectMessage5(RejectMessage rejectMessage) {
    list.push(5);
}

@Handler(priority = 3)
public void handleRejectMessage3(RejectMessage rejectMessage) {
    list.push(3);
}

@Handler(priority = 2, rejectSubtypes = true)
public void handleMessage(Message rejectMessage) 
    logger.error("Reject handler #3");
    list.push(3);
}

@Handler(priority = 0)
public void handleRejectMessage0(RejectMessage rejectMessage) {
    list.push(1);
} 
```

从最高优先级到最低优先级调用处理程序。最后调用默认优先级为零的处理程序。我们看到处理程序以相反的顺序编号`pop()`。

### 7.3。拒绝子类型，最简单的方法

上面测试中的`handleMessage()`怎么了？我们不必使用`RejectSubTypes.class`来过滤我们的子类型。

`RejectSubTypes`是一个布尔标志，它提供与类相同的过滤，但性能比`IMessageFilter`实现更好。

但是，我们仍然需要使用基于过滤器的实现来只接受子类型。

## 8。结论

MBassador 是一个简单明了的库，用于在对象之间传递消息。消息可以用多种方式组织，可以同步或异步发送。

和往常一样，这个例子可以在 GitHub 项目中找到。