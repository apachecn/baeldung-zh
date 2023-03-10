# Spring 应用程序上下文事件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-context-events>

## 1。简介

在本教程中，我们将学习 Spring 框架提供的事件支持机制。我们将探索框架提供的各种内置事件，然后看看如何使用事件。

要了解如何创建和发布自定义事件，请点击这里查看我们之前的教程。

Spring 有一个围绕`ApplicationContext.` 构建的事件机制，它可以用来在不同的 beans 之间交换信息。我们可以通过监听事件和执行自定义代码来利用应用程序事件。

例如，这里的一个场景是在`ApplicationContext`完全启动时执行定制逻辑。

## 2。标准上下文事件

事实上，Spring 中有各种各样的内置事件，**让开发人员挂钩到应用程序的生命周期和上下文**中，并进行一些定制操作。

尽管我们很少在应用程序中手动使用这些事件，但是框架本身会大量使用这些事件。让我们从探索春天的各种内置事件开始。

### 2.1。`ContextRefreshedEvent`

在**初始化或刷新** `**ApplicationContext**,` 时，弹簧升起`ContextRefreshedEvent`。通常，只要上下文没有关闭，刷新就可以被触发多次。

注意，我们也可以通过调用`ConfigurableApplicationContext`接口上的`refresh()`方法来手动触发事件。

### 2.2。`ContextStartedEvent`

**通过调用`ConfigurableApplicationContext,` 上的`start()`方法**，我们触发此事件并启动`ApplicationContext`。事实上，该方法通常用于在显式停止后重启 beans。我们还可以使用该方法来处理没有自动启动配置的组件。

**这里，需要注意的是对`start()`的调用总是显式的，与** `**refresh**` **`().`** 相反

### 2.3。`ContextStoppedEvent`

一个`ContextStoppedEvent` 被发布**当`ApplicationContext`被停止**时，通过调用前面讨论过的`ConfigurableApplicationContext.` 上的`stop()`方法，我们可以通过使用`start()`方法重启一个停止的事件。

### 2.4。`ContextClosedEvent`

当`ApplicationContext`关闭时，使用`ConfigurableApplicationContext`中的`close()`方法发布该事件**。
实际上，在关闭一个上下文后，我们无法重新启动它。**

一个上下文在关闭时达到了它的生命终点，因此我们不能像在`ContextStoppedEvent.`中那样重启它

## 3。`@EventListener`

接下来，让我们探索如何消费发布的事件。从 4.2 版本开始，Spring 支持注释驱动的事件监听器—`@EventListener.`

特别是，我们可以利用这个注释来让**根据方法**的签名自动注册一个`ApplicationListener` :

```java
@EventListener
public void handleContextRefreshEvent(ContextStartedEvent ctxStartEvt) {
    System.out.println("Context Start Event received.");
}
```

值得注意的是， **`@EventListener`是一个核心注释，因此不需要任何额外的配置**。事实上，现有的`<context:annotation-driven/>`元素提供了对它的完全支持。

用`@EventListener`标注的方法可以返回非 void 类型。如果返回值为非空，事件机制将为其发布一个新事件。

### 3.1。收听多个事件

现在，可能会出现这样的情况，我们需要侦听器来消费多个事件。

对于这样的场景，我们可以利用[类的](https://web.archive.org/web/20220628063256/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/event/EventListener.html#classes)属性:

```java
@EventListener(classes = { ContextStartedEvent.class, ContextStoppedEvent.class })
public void handleMultipleEvents() {
    System.out.println("Multi-event listener invoked");
}
```

## 4。应用程序事件监听器

如果我们使用早期版本的 Spring ( <4.2), we'll have to introduce a [自定义`ApplicationEventListener`并覆盖方法`onApplicationEvent`](/web/20220628063256/https://www.baeldung.com/spring-events) 来监听事件。

## 5。结论

在本文中，我们探索了 Spring 中的各种内置事件。此外，我们还看到了收听已发布事件的各种方式。

和往常一样，本文中使用的代码片段可以在 Github 上找到[。](https://web.archive.org/web/20220628063256/https://github.com/eugenp/tutorials/tree/master/spring-core)