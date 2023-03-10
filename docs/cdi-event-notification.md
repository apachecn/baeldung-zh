# CDI 2.0 中事件通知模型的介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cdi-event-notification>

## 1。概述

[CDI](/web/20220524004826/https://www.baeldung.com/java-ee-cdi) (上下文和依赖注入)是 Jakarta EE 平台的标准依赖注入框架。

在本教程中，我们将看一看 [CDI 2.0](https://web.archive.org/web/20220524004826/http://www.cdi-spec.org/news/2017/05/15/CDI_2_is_released/) ，以及它如何通过**添加一个改进的、全功能的事件通知模型来构建 CDI 1.x 的强大的、类型安全的注入机制。**

## 2。美芬依赖

首先，我们将构建一个简单的 Maven 项目。

**我们需要一个符合 CDI 2.0 的容器，CDI 的参考实现 [Weld](https://web.archive.org/web/20220524004826/http://weld.cdi-spec.org/) 是一个很好的选择:**

```java
<dependencies>
    <dependency>
        <groupId>javax.enterprise</groupId>
        <artifactId>cdi-api</artifactId>
        <version>2.0.SP1</version>
    </dependency>
    <dependency>
        <groupId>org.jboss.weld.se</groupId>
        <artifactId>weld-se-core</artifactId>
        <version>3.0.5.Final</version>
    </dependency>
</dependencies> 
```

和往常一样，我们可以从 Maven Central 上拉最新版本的[`cdi-api`](https://web.archive.org/web/20220524004826/https://search.maven.org/search?q=g:javax.enterprise%20AND%20a:cdi-api&core=gav)`[weld-se-core](https://web.archive.org/web/20220524004826/https://search.maven.org/search?q=g:org.jboss.weld.se%20AND%20a:weld-se-core&core=gav)`。

## 3。观察和处理自定义事件

简单来说，**CDI 2.0 事件通知模型是[观察者模式](/web/20220524004826/https://www.baeldung.com/java-observer-pattern)** 的经典实现，基于 [`@Observes`](https://web.archive.org/web/20220524004826/https://docs.jboss.org/cdi/api/1.0/javax/enterprise/event/Observes.html) 方法——参数标注。因此，它允许我们轻松地定义观察器方法，可以自动调用这些方法来响应一个或多个事件。

例如，我们可以定义一个或多个 bean，这将触发一个或多个特定的事件，而其他 bean 将被通知这些事件并做出相应的反应。

为了更清楚地演示这是如何工作的，我们将构建一个简单的示例，包括一个基本服务类、一个自定义事件类和一个对自定义事件做出反应的 observer 方法。

### 3.1。基本服务类别

让我们从创建一个简单的`TextService`类开始:

```java
public class TextService {

    public String parseText(String text) {
        return text.toUpperCase();
    }
} 
```

### 3.2。自定义事件类

接下来，让我们定义一个示例事件类，它在其构造函数中接受一个`String`参数:

```java
public class ExampleEvent {

    private final String eventMessage;

    public ExampleEvent(String eventMessage) {
        this.eventMessage = eventMessage;
    }

    // getter
}
```

### 3.3。用`@Observes`注解定义一个观察者方法

现在我们已经定义了我们的服务和事件类，让我们使用`@Observes`注释为我们的`ExampleEvent`类创建一个 observer 方法:

```java
public class ExampleEventObserver {

    public String onEvent(@Observes ExampleEvent event, TextService textService) {
        return textService.parseText(event.getEventMessage());
    } 
}
```

虽然乍一看， `onEvent()`方法的实现看起来相当简单，但它实际上通过`@Observes`注释封装了很多功能。

正如我们所见，`onEvent()`方法是一个事件处理程序，它将`ExampleEvent`和`TextService`对象作为参数。

**让我们记住，在`@Observes`注释之后指定的所有参数都是标准注入点。**因此，CDI 将为我们创建完全初始化的实例，并将它们注入到 observer 方法中。

### 3.4。初始化我们的 CDI 2.0 容器

至此，我们已经创建了服务和事件类，并且定义了一个简单的 observer 方法来对事件做出反应。但是我们如何指示 CDI 在运行时注入这些实例呢？

这是事件通知模型最充分展示其功能的地方。我们只需初始化新的 [`SeContainer`](https://web.archive.org/web/20220524004826/https://docs.jboss.org/cdi/api/2.0/javax/enterprise/inject/se/SeContainer.html) 实现，并通过`fireEvent()`方法触发一个或多个事件:

```java
SeContainerInitializer containerInitializer = SeContainerInitializer.newInstance(); 
try (SeContainer container = containerInitializer.initialize()) {
    container.getBeanManager().fireEvent(new ExampleEvent("Welcome to Baeldung!")); 
}
```

**注意，我们使用了`SeContainerInitializer`和`SeContainer`对象，因为我们在 Java SE 环境中使用 CDI，而不是在 Jakarta EE 中。**

当通过传播事件本身触发`ExampleEvent`时，所有附加的观察器方法都将得到通知。

因为所有在`@Observes`注释后作为参数传递的对象都将被完全初始化，所以 CDI 会在将它注入到`onEvent()`方法之前，负责为我们连接整个`TextService`对象图。

简而言之，**我们拥有类型安全 IoC 容器的优势，以及功能丰富的事件通知模型**。

## 4。`ContainerInitialized`事件

在前面的例子中，我们使用了一个自定义事件将一个事件传递给一个 observer 方法，并获得一个完全初始化的`TextService`对象。

当然，当我们真的需要跨应用程序的多个点传播一个或多个事件时，这是很有用的。

有时，我们只需要获得一堆完全初始化的对象，这些对象可以在我们的应用程序类中使用，而不需要执行额外的事件。

为此， **CDI 2.0 提供了 [`ContainerInitialized`](https://web.archive.org/web/20220524004826/http://javadox.com/org.jboss.weld.se/weld-se-core/2.2.6.Final/org/jboss/weld/environment/se/events/ContainerInitialized.html) 事件类，在焊接容器初始化**时自动触发。

让我们看看如何使用`ContainerInitialized`事件将控制权转移给`ExampleEventObserver`类:

```java
public class ExampleEventObserver {
    public String onEvent(@Observes ContainerInitialized event, TextService textService) {
        return textService.parseText(event.getEventMessage());
    }    
} 
```

记住**`ContainerInitialized`事件类是特定于焊接的**。因此，如果我们使用不同的 CDI 实现，我们需要重构我们的 observer 方法。

## 5。条件观察器方法

在当前的实现中，我们的`ExampleEventObserver`类默认定义了一个无条件的观察者方法。这意味着**观察者方法将总是被通知所提供的事件**，不管该类的实例是否存在于当前上下文中。

同样，我们可以通过将`notifyObserver=IF_EXISTS`指定为`@Observes`注释的参数来**定义一个条件观察器方法**:

```java
public String onEvent(@Observes(notifyObserver=IF_EXISTS) ExampleEvent event, TextService textService) { 
    return textService.parseText(event.getEventMessage());
} 
```

当我们使用一个条件观察者方法时，**只有当定义观察者方法的类的一个实例存在于当前上下文**中时，该方法才会被通知匹配事件。

## 6。事务观察者方法

**我们还可以在一个事务中触发事件，比如数据库更新或删除操作**。为此，我们可以通过在`@Observes`注释中添加`during`参数来定义事务观察者方法。

`during`参数的每个可能值对应于事务的一个特定阶段:

*   `BEFORE_COMPLETION`
*   `AFTER_COMPLETION`
*   `AFTER_SUCCESS`
*   `AFTER_FAILURE`

如果我们在一个事务中触发`ExampleEvent`事件，我们需要相应地重构`onEvent()`方法，以便在所需的阶段处理该事件:

```java
public String onEvent(@Observes(during=AFTER_COMPLETION) ExampleEvent event, TextService textService) { 
    return textService.parseText(event.getEventMessage());
}
```

只有在给定事务的匹配阶段，事务观察者方法才会被告知所提供的事件。

## 7。观察者方法排序

CDI 2.0 的事件通知模型中包含的另一个很好的改进是能够为调用给定事件的观察者设置顺序或优先级。

通过在`@Observes.`后指定 [`@Priority`](https://web.archive.org/web/20220524004826/https://docs.oracle.com/javaee/7/api/javax/annotation/Priority.html) 注释，我们可以很容易地定义观察者方法的调用顺序

为了理解这个特性是如何工作的，除了`ExampleEventObserver`实现的方法之外，让我们定义另一个 observer 方法:

```java
public class AnotherExampleEventObserver {

    public String onEvent(@Observes ExampleEvent event) {
        return event.getEventMessage();
    }   
}
```

在这种情况下，默认情况下，两种观察器方法具有相同的优先级。因此，CDI 调用它们的顺序是不可预测的。

我们可以通过`@Priority`注释为每个方法分配一个调用优先级来轻松解决这个问题:

```java
public String onEvent(@Observes @Priority(1) ExampleEvent event, TextService textService) {
    // ... implementation
} 
```

```java
public String onEvent(@Observes @Priority(2) ExampleEvent event) {
    // ... implementation
}
```

**优先级遵循自然排序。**因此，CDI 将首先调用优先级为`1`的 observer 方法，然后调用优先级为`2`的方法。

同样，**如果我们在两个或更多方法中使用相同的优先级，那么顺序也是不确定的**。

## 8。异步事件

在我们到目前为止所学的所有例子中，我们都是同步触发事件的。然而，CDI 2.0 也允许我们轻松地触发异步事件。异步观察器方法可以在不同的线程中处理这些异步事件。

我们可以用`fireAsync()`方法异步触发一个事件:

```java
public class ExampleEventSource {

    @Inject
    Event<ExampleEvent> exampleEvent;

    public void fireEvent() {
        exampleEvent.fireAsync(new ExampleEvent("Welcome to Baeldung!"));
    }   
}
```

**bean 触发事件，这些事件是`[Event](https://web.archive.org/web/20220524004826/https://docs.jboss.org/cdi/api/2.0/javax/enterprise/event/Event.html)`接口的实现。因此，我们可以像注射其他常规豆一样注射它们**。

为了处理我们的异步事件，我们需要用 [`@ObservesAsync`](https://web.archive.org/web/20220524004826/https://docs.jboss.org/cdi/api/2.0.EDR2/javax/enterprise/event/ObservesAsync.html) 注释定义一个或多个异步观察器方法:

```java
public class AsynchronousExampleEventObserver {

    public void onEvent(@ObservesAsync ExampleEvent event) {
        // ... implementation
    }
}
```

## 9。结论

在本文中，**我们了解了如何开始使用 CDI 2.0 附带的改进的事件通知模型。**

像往常一样，本教程中显示的所有代码示例都可以在 [GitHub](https://web.archive.org/web/20220524004826/https://github.com/eugenp/tutorials/tree/master/cdi/src/main/java/com/baeldung/cdi2observers) 上获得。