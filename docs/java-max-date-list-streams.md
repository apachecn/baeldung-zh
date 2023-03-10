# 使用流查找列表中的最大日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-max-date-list-streams>

## 1.概观

在本文中，我们将首先创建一个带有日期的对象。然后，我们将看到如何使用 [`Streams`](/web/20221112223030/https://www.baeldung.com/java-8-streams) 在那些对象的列表中找到最大日期。

## 2.示例设置

**Java 最初的 [`Date`](/web/20221112223030/https://www.baeldung.com/java-get-the-current-date-legacy) API 仍然被广泛使用**，所以我们将展示一个使用它的例子。但是自从 Java 8 推出了 [`LocalDate`](/web/20221112223030/https://www.baeldung.com/java-8-date-time-intro) ，大部分`Date`方法都被弃用了。因此，**我们也将展示一个使用`LocalDate`的例子。**

首先，让我们创建一个基本的`Event`对象，它包含一个单独的`Date`属性:

```java
public class Event {

    Date date;

    // constructor, getter and setter
}
```

我们现在可以定义三个`Event`的列表:第一个发生在今天，第二个发生在明天，第三个发生在一周内。**为了给`Date,`添加天数，我们将使用 [Apache Commons 的](/web/20221112223030/https://www.baeldung.com/java-commons-lang-3) `DateUtils`方法`addDays()`** :

```java
Date TODAY = new Date();
Event TODAYS_EVENT = new Event(TODAY);
Date TOMORROW = DateUtils.addDays(TODAY, 1);
Event TOMORROWS_EVENT = new Event(TOMORROW);
Date NEXT_WEEK = DateUtils.addDays(TODAY, 7);
Event NEXT_WEEK_EVENT = new Event(NEXT_WEEK);
List<Event> events = List.of(TODAYS_EVENT, TOMORROWS_EVENT, NEXT_WEEK_EVENT);
```

我们现在的目标是编写一个方法，能够确定`NEXT_WEEK_EVENT`是这个`Event`列表中的最大日期。我们也会用`LocalDate`代替`Date`来做同样的事情。我们的`LocalEvent`会是这样的:

```java
public class LocalEvent {

    LocalDate date;

    // constructor, getter and setter
}
```

构建`Event`列表更加简单，因为`LocalDate`已经有了一个内置的`plusDays()`方法:

```java
LocalDate TODAY_LOCAL = LocalDate.now();
LocalEvent TODAY_LOCAL_EVENT = new LocalEvent(TODAY_LOCAL);
LocalDate TOMORROW_LOCAL = TODAY_LOCAL.plusDays(1);
LocalEvent TOMORROW_LOCAL_EVENT = new LocalEvent(TOMORROW_LOCAL);
LocalDate NEXT_WEEK_LOCAL = TODAY_LOCAL.plusWeeks(1);
LocalEvent NEXT_WEEK_LOCAL_EVENT = new LocalEvent(NEXT_WEEK_LOCAL);
List<LocalEvent> localEvents = List.of(TODAY_LOCAL_EVENT, TOMORROW_LOCAL_EVENT, NEXT_WEEK_LOCAL_EVENT);
```

## 3.获取最大日期

首先，**我们将使用 [`Stream API`](/web/20221112223030/https://www.baeldung.com/java-8-streams) 来流式传输我们的`Event`列表**。然后，我们需要将`Date` getter 应用于`Stream`的每个元素。因此，我们将获得一个包含事件日期的`Stream`。**我们现在可以使用`max()`功能。这将返回关于所提供的`[Comparator](/web/20221112223030/https://www.baeldung.com/java-comparator-comparable)`的`Stream`中的最大值`Date`。**

`Date`类实现了`Comparable<Date>`。因此，`compareTo()`方法定义了自然日期顺序。简而言之，可以等价地调用`max()`中的以下两个方法:

*   `Date`的`compareTo()`可以通过方法引用来引用
*   `Comparator`的`naturalOrder()`可以直接使用

最后，让我们注意，如果给定的`Event`列表为 null 或空，我们可以直接返回 null。这将确保我们在传输列表时不会遇到问题。

该方法最终看起来像这样:

```java
Date findMaxDateOf(List<Event> events) {
    if (events == null || events.isEmpty()) {
        return null;
    }
    return events.stream()
      .map(Event::getDate)
      .max(Date::compareTo)
      .get();
}
```

或者，使用`naturalOrder(),`,它将显示为:

```java
Date findMaxDateOf(List<Event> events) {
    if (events == null || events.isEmpty()) {
        return null;
    }
    return events.stream()
      .map(Event::getDate)
      .max(Comparator.naturalOrder())
      .get();
}
```

最后，我们现在可以快速测试我们的方法是否为我们的列表返回了正确的结果:

```java
assertEquals(NEXT_WEEK, findMaxDateOf(List.of(TODAYS_EVENT, TOMORROWS_EVENT, NEXT_WEEK_EVENT);
```

用`LocalDate`，推理完全一样。`LocalDate`确实实现了`ChronoLocalDate` [接口](/web/20221112223030/https://www.baeldung.com/java-interfaces)，它扩展了`Comparable<ChronoLocalDate>`。因此，`LocalDate`的自然顺序是由`ChronoLocalDate`的`compareTo()`方法定义的。

因此，该方法可以写成:

```java
LocalDate findMaxDateOf(List<LocalEvent> events) {
    if (events == null || events.isEmpty()) {
        return null;
    }
    return events.stream()
      .map(LocalEvent::getDate)
      .max(LocalDate::compareTo)
      .get();
}
```

或者，以完全等同的方式:

```java
LocalDate findMaxDateOf(List<LocalEvent> events) {
    if (events == null || events.isEmpty()) {
        return null;
    }
    return events.stream()
      .map(LocalEvent::getDate)
      .max(Comparator.naturalOrder())
      .get();
}
```

我们可以编写下面的测试来确认它的工作:

```java
assertEquals(NEXT_WEEK_LOCAL, findMaxDateOf(List.of(TODAY_LOCAL_EVENT, TOMORROW_LOCAL_EVENT, NEXT_WEEK_LOCAL_EVENT)));
```

## 4.结论

在本教程中，我们已经看到了如何获取对象列表中的最大日期。我们已经使用了`Date`和`LocalDate`对象。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221112223030/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-4)