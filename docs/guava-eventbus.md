# 番石榴活动指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-eventbus>

## 1。概述

Guava 库提供了允许组件间发布-订阅通信的`EventBus`。在本教程中，我们将了解如何使用`EventBus`的一些功能。

## 2。设置

首先，我们在`pom.xml:`中添加谷歌番石榴库依赖项

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

The latest version can be found [here](https://web.archive.org/web/20220630153823/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22).

## 3。使用 `EventBus`

让我们从一个简单的例子开始。

### 3.1。设置

我们从查看`EventBus`对象开始。它可以注册侦听器和发布事件。使用它就像实例化该类一样简单:

```java
EventBus eventBus = new EventBus();
```

Guava library 给了你以最适合你的开发需求的方式使用`EventBus`的自由。

### 3.2。创建监听器

We create a listener class that has handler methods to receive specific events. We annotate the handler methods with `@Subscribe`. The method accepts as an argument an object of the same type as the event being posted:

```java
public class EventListener {

    private static int eventsHandled;

    @Subscribe
    public void stringEvent(String event) {
        eventsHandled++;
    }
}
```

### 3.3。注册监听器

We can subscribe to an event by registering our `EventListener` class on the `EventBus`:

```java
EventListener listener = new EventListener();
eventBus.register(listener);
```

### 3.4。取消注册监听器

如果出于某种原因，我们想从`EventBus`中注销一个类，这也很容易做到:

```java
eventBus.unregister(listener);
```

### 3.5。发布事件

We can post events as well with the `EventBus`:

```java
@Test
public void givenStringEvent_whenEventHandled_thenSuccess() {
    eventBus.post("String Event");
    assertEquals(1, listener.getEventsHandled());
}
```

### 3.6。发布自定义事件

We can also specify a custom event class and post that event. We start by creating a custom event:

```java
public class CustomEvent {
    private String action;

    // standard getters/setters and constructors
}
```

在该事件的`EventListener`类中添加一个处理程序方法:

```java
@Subscribe
public void someCustomEvent(CustomEvent customEvent) {
    eventsHandled++;
}
```

我们现在可以发布我们的自定义事件:

```java
@Test
public void givenCustomEvent_whenEventHandled_thenSuccess() {
    CustomEvent customEvent = new CustomEvent("Custom Event");
    eventBus.post(customEvent);

    assertEquals(1, listener.getEventsHandled());
}
```

### 3.7。处理一个`Unsubscribed`事件

我们提供了一个`DeadEvent`类，允许我们处理任何没有监听器的事件。我们可以添加一个方法来处理`DeadEvent`类:

```java
@Subscribe
public void handleDeadEvent(DeadEvent deadEvent) {
    eventsHandled++;
}
```

## 4。结论

在本教程中，我们用一个简单的例子来指导如何使用番石榴。

你可以在 GitHub 上找到本文[的完整源代码和所有代码片段。](https://web.archive.org/web/20220630153823/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-utilities)