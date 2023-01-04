# 春季活动

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-events>

## 1。概述

在本教程中，我们将讨论如何在 Spring 中使用事件。

事件是框架中最容易被忽略的功能之一，但也是最有用的功能之一。和 Spring 中的许多其他东西一样，事件发布是`ApplicationContext`提供的功能之一。

有几个简单的准则可以遵循:

*   如果我们使用的是 Spring Framework 4.2 之前的版本，event 类应该扩展`ApplicationEvent`。[从 4.2 版本](https://web.archive.org/web/20221101065528/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationEventPublisher.html#publishEvent-java.lang.Object-)开始，事件类不再需要扩展`ApplicationEvent`类。
*   发布者应该注入一个`ApplicationEventPublisher` 对象。
*   监听器应该实现`ApplicationListener`接口。

## 延伸阅读:

## [Spring 应用上下文事件](/web/20221101065528/https://www.baeldung.com/spring-context-events)

Learn about the built-in events for the Spring application context[Read more](/web/20221101065528/https://www.baeldung.com/spring-context-events) →

## [春天如何做@ Async](/web/20221101065528/https://www.baeldung.com/spring-async)

How to enable and use @Async in Spring - from the very simple config and basic usage to the more complex executors and exception handling strategies.[Read more](/web/20221101065528/https://www.baeldung.com/spring-async) →

## [春天表情语言指南](/web/20221101065528/https://www.baeldung.com/spring-expression-language)

This article explores Spring Expression Language (SpEL), a powerful expression language that supports querying and manipulating object graphs at runtime.[Read more](/web/20221101065528/https://www.baeldung.com/spring-expression-language) →

## 2。自定义事件

Spring 允许我们创建和发布默认同步的定制事件。这有几个好处，比如监听器能够参与发布者的事务上下文。

### 2.1。一个简单的应用事件

让我们创建一个简单的事件类——只是一个存储事件数据的占位符。

在这种情况下，事件类保存一条字符串消息:

```java
public class CustomSpringEvent extends ApplicationEvent {
    private String message;

    public CustomSpringEvent(Object source, String message) {
        super(source);
        this.message = message;
    }
    public String getMessage() {
        return message;
    }
}
```

### 2.2。一个出版商

现在让我们创建一个事件的发布者。发布者构建事件对象，并将其发布给任何正在收听的人。

为了发布事件，发布者可以简单地注入`ApplicationEventPublisher` 并使用`publishEvent()` API:

```java
@Component
public class CustomSpringEventPublisher {
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    public void publishCustomEvent(final String message) {
        System.out.println("Publishing custom event. ");
        CustomSpringEvent customSpringEvent = new CustomSpringEvent(this, message);
        applicationEventPublisher.publishEvent(customSpringEvent);
    }
}
```

或者，publisher 类可以实现`ApplicationEventPublisherAware`接口，这也将在应用程序启动时注入事件发布器。通常情况下，只给发行商注入`@Autowire`会更简单。

从 Spring Framework 4.2 开始，`ApplicationEventPublisher`接口为接受任何对象作为事件的`[publishEvent(Object event)](https://web.archive.org/web/20221101065528/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationEventPublisher.html#publishEvent-java.lang.Object-)`方法提供了一个新的重载。**因此，Spring 事件不再需要扩展`ApplicationEvent`类。**

### 2.3。一个听众

最后，让我们创建监听器。

对监听器的唯一要求是成为一个 bean 并实现`ApplicationListener`接口:

```java
@Component
public class CustomSpringEventListener implements ApplicationListener<CustomSpringEvent> {
    @Override
    public void onApplicationEvent(CustomSpringEvent event) {
        System.out.println("Received spring custom event - " + event.getMessage());
    }
}
```

注意我们的定制监听器是如何用定制事件的一般类型来参数化的，这使得`onApplicationEvent()`方法是类型安全的。这也避免了必须检查对象是否是特定事件类的实例并强制转换它。

而且，正如已经讨论过的(默认情况下 **Spring 事件是同步的**),`doStuffAndPublishAnEvent()`方法会一直阻塞，直到所有的监听器都处理完事件。

## 3。创建异步事件

在某些情况下，同步发布事件并不是我们真正想要的——我们可能需要异步处理我们的事件。

我们可以在配置中通过创建一个带有执行器的`ApplicationEventMulticaster` bean 来打开它。

对于我们这里的目的，`SimpleAsyncTaskExecutor` 工作得很好:

```java
@Configuration
public class AsynchronousSpringEventsConfig {
    @Bean(name = "applicationEventMulticaster")
    public ApplicationEventMulticaster simpleApplicationEventMulticaster() {
        SimpleApplicationEventMulticaster eventMulticaster =
          new SimpleApplicationEventMulticaster();

        eventMulticaster.setTaskExecutor(new SimpleAsyncTaskExecutor());
        return eventMulticaster;
    }
}
```

事件、发布者和监听器实现与以前一样，但是现在**监听器将在一个单独的线程中异步处理事件。**

## 4。现有框架事件

Spring 本身发布各种现成的事件。例如，`ApplicationContext` 将触发各种框架事件:`ContextRefreshedEvent`、 `ContextStartedEvent`、 `RequestHandledEvent` 等。

这些事件为应用程序开发人员提供了一个选项，可以挂钩到应用程序的生命周期和上下文中，并在需要的地方添加他们自己的定制逻辑。

下面是一个监听器监听上下文刷新的快速示例:

```java
public class ContextRefreshedListener 
  implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent cse) {
        System.out.println("Handling context re-freshed event. ");
    }
}
```

要了解更多关于现有框架事件的信息，请看我们的下一个教程。

## 5。注释驱动的事件监听器

从 Spring 4.2 开始，事件侦听器不需要成为实现`ApplicationListener`接口的 bean——它可以通过`@EventListener`注释注册到受管 bean 的任何`public`方法上:

```java
@Component
public class AnnotationDrivenEventListener {
    @EventListener
    public void handleContextStart(ContextStartedEvent cse) {
        System.out.println("Handling context started event.");
    }
}
```

和以前一样，方法签名声明它使用的事件类型。

默认情况下，侦听器是同步调用的。然而，我们可以通过添加一个`@Async`注释轻松地使其异步。我们只需要记住在应用程序中[启用`Async`支持](/web/20221101065528/https://www.baeldung.com/spring-async#enable-async-support)。

## 6。泛型支持

也可以用事件类型中的泛型信息来调度事件。

### 6.1。通用应用程序事件

**让我们创建一个通用的事件类型。**

在我们的示例中，事件类保存任何内容和一个`success`状态指示器:

```java
public class GenericSpringEvent<T> {
    private T what;
    protected boolean success;

    public GenericSpringEvent(T what, boolean success) {
        this.what = what;
        this.success = success;
    }
    // ... standard getters
}
```

注意`GenericSpringEvent`和`CustomSpringEvent`的区别。我们现在可以灵活地发布任意事件，不再需要从`ApplicationEvent`扩展。

### 6.2。一个听众

现在让我们创建一个事件的监听器。

我们可以像以前一样通过实现 `ApplicationListener`接口来定义监听器:

```java
@Component
public class GenericSpringEventListener 
  implements ApplicationListener<GenericSpringEvent<String>> {
    @Override
    public void onApplicationEvent(@NonNull GenericSpringEvent<String> event) {
        System.out.println("Received spring generic event - " + event.getWhat());
    }
}
```

但不幸的是，这个定义要求我们从`ApplicationEvent`类继承`GenericSpringEvent`。因此，对于本教程，让我们利用前面[讨论过的](#annotation-driven)注释驱动的事件监听器。

还可以通过在`@EventListener`注释上定义一个布尔 SpEL 表达式来使**事件监听器有条件的**。

在这种情况下，只有在`String`的`GenericSpringEvent`成功时，才会调用事件处理程序:

```java
@Component
public class AnnotationDrivenEventListener {
    @EventListener(condition = "#event.success")
    public void handleSuccessful(GenericSpringEvent<String> event) {
        System.out.println("Handling generic event (conditional).");
    }
}
```

Spring Expression Language(SpEL)是一种强大的表达式语言，在另一个教程中会详细介绍。

### 6.3。一个出版商

事件发布者类似于上面描述的。但是由于类型擦除，我们需要发布一个事件来解析我们要过滤的泛型参数，例如，`class GenericStringSpringEvent extends GenericSpringEvent<String>`。

此外，还有另外一种发布事件的方式。如果我们从一个用`@EventListener`标注的方法返回一个非空值作为结果，Spring Framework 会把这个结果作为一个新事件发送给我们。此外，我们可以发布多个新事件，方法是将它们作为事件处理的结果返回到集合中。

## 7 .**。事务绑定事件**

这一节是关于使用`@TransactionalEventListener`注释的。要了解更多关于事务管理的信息，请查看 Spring 和 JPA 的[事务。](/web/20221101065528/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)

从 Spring 4.2 开始，框架提供了一个新的`@TransactionalEventListener`注释，它是`@EventListener`的扩展，允许将事件的监听器绑定到事务的一个阶段。

可以绑定到以下事务阶段:

*   `AFTER_COMMIT`(默认值)用于在交易成功完成**时触发事件。**
*   `AFTER_ROLLBACK`——如果交易已经**回退**
*   `AFTER_COMPLETION`——如果交易已经**完成**(`AFTER_COMMIT`和`AFTER_ROLLBACK`的别名)
*   `BEFORE_COMMIT`用于在事务**提交之前触发事件权限**。****

下面是一个事务性事件侦听器的简单示例:

```java
@TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
public void handleCustom(CustomSpringEvent event) {
    System.out.println("Handling event inside a transaction BEFORE COMMIT.");
}
```

只有当有一个事务正在运行并且将要提交时，才会调用这个侦听器。

如果没有事务正在运行，则根本不会发送事件，除非我们通过将`fallbackExecution`属性设置为`true`来覆盖它。

## 8。结论

在这篇简短的文章中，我们回顾了在 Spring 中处理事件的基础知识，包括创建一个简单的定制事件，发布它，然后在一个监听器中处理它。

我们还简要了解了如何在配置中启用事件的异步处理。

然后我们了解了 Spring 4.2 中引入的改进，比如注释驱动的监听器、更好的泛型支持和绑定到事务阶段的事件。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221101065528/https://github.com/eugenp/tutorials/tree/master/spring-core-2)