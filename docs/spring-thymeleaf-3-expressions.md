# 春天和百里香叶 3:表情

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-3-expressions>

## 1。简介

[百里叶](https://web.archive.org/web/20221129020940/http://www.thymeleaf.org/)是一个 Java 模板引擎，用于处理和创建 HTML、XML、JavaScript、CSS 和纯文本。关于百里香和春天的介绍，请看[这篇文章](/web/20221129020940/https://www.baeldung.com/thymeleaf-in-spring-mvc)。

除了这些基本功能之外，Thymeleaf 还为我们提供了一组实用程序对象，帮助我们在应用程序中执行常见的任务。

在本文中，我们将讨论百里香叶 3.0 的一个核心特性 Spring MVC 应用程序中的表达式实用程序对象。更具体地说，我们将讨论处理日期、日历、字符串、对象等主题。

## 2。Maven 依赖关系

首先，让我们看看将百里香与 Spring 集成所需的配置。在我们的依赖关系中需要`thymeleaf-spring`库:

```java
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

注意，对于 Spring 4 项目，必须使用`thymeleaf-spring4`库来代替`thymeleaf-spring5`。最新版本的依赖可以在[这里](https://web.archive.org/web/20221129020940/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.thymeleaf%22%20AND%20a%3A%22thymeleaf-spring5%22)找到。

## 3。表达式工具对象

在看这篇文章的核心焦点之前，如果你想后退一步，看看如何在你的 web 应用程序项目中配置百里香 3.0，看看这篇[教程](/web/20221129020940/https://www.baeldung.com/spring-thymeleaf-3)。

为了本文的目的，我们创建了一个 Spring 控制器和 HTML 文件——来测试我们将要讨论的所有特性。以下是可用辅助对象及其功能的完整列表:

*   **`#dates`** :针对 `java.util.Date`对象的实用方法
*   `**#calendars**`:类似于`#dates`，用于 `java.util.Calendar`对象
*   `**#numbers**`:格式化数字对象的实用方法
*   `**#strings**`:用于`String`对象的实用方法
*   `**#objects**`:一般 Java `Object`类的实用方法
*   `**#bools**`:评估`boolean`的实用方法
*   `**#arrays**`:数组的实用方法
*   `**#lists**`:列表的实用方法
*   `**#sets**`:器械包的实用方法
*   `**#maps**`:地图的实用方法
*   `**#aggregates**`:在数组或集合上创建聚集的实用方法
*   `**#messages**`:在变量表达式中获取外部化消息的实用方法

### 3.1。日期对象

我们要讨论的第一个函数是对`java.util.Date` 对象的处理。负责`date`处理的表达式实用程序对象从`#dates.functionName().`开始。我们要介绍的第一个函数是`Date`对象的格式化(它被添加到弹簧模型参数中)。

假设我们想使用`ISO8601`格式:

```java
<p th:text="${#dates.formatISO(date)}"></p>
```

无论我们的`date`在后端如何设置，都需要按照这个标准来显示。更重要的是，如果我们想对格式进行具体说明，我们可以手动指定:

```java
<p th:text="${#dates.format(date, 'dd-MM-yyyy HH:mm')}"></p>
```

该函数以两个变量作为参数:`Date`及其格式。

最后，这里有几个我们可以使用的类似的有用函数:

```java
<p th:text="${#dates.dayOfWeekName(date)}"></p>
<p th:text="${#dates.createNow()}"></p>
<p th:text="${#dates.createToday()}"></p>
```

在第一个示例中，我们将收到一周中的某一天的名称，在第二个示例中，我们将创建一个新的`Date`对象，最后我们将创建一个时间设置为 00:00 的新的`Date`。

### 3.2。日历对象

日历实用程序与日期处理非常相似，除了我们使用的是`java.util.Calendar`对象的实例:

```java
<p th:text="${#calendars.formatISO(calendar)}"></p>
<p th:text="${#calendars.format(calendar, 'dd-MM-yyyy HH:mm')}"></p>
<p th:text="${#calendars.dayOfWeekName(calendar)}"></p>
```

唯一的区别是当我们想要创建新的`Calendar`实例时:

```java
<p th:text="${#calendars.createNow().getTime()}"></p>
<p th:text="${#calendars.createToday().getFirstDayOfWeek()}"></p>
```

请注意，我们可以使用任何`Calendar`类方法来获取请求的数据。

### 3.3。数字处理

另一个非常少的特性是数字处理。让我们关注一个用`double` 类型随机创建的`num`变量:

```java
<p th:text="${#numbers.formatDecimal(num,2,3)}"></p>
<p th:text="${#numbers.formatDecimal(num,2,3,'COMMA')}"></p>
```

在第一行中，我们通过设置最小整数位数和精确小数位数来格式化十进制数。在第二个例子中，除了整数和小数，我们还指定了小数分隔符。选项有`POINT`、`COMMA`、`WHITESPACE`、`NONE`或`DEFAULT`(根据地区)。

这一段我们还想介绍一个功能。它是整数序列的创建:

```java
<p th:each="number: ${#numbers.sequence(0,2)}">
    <span th:text="${number}"></span>
</p>
<p th:each="number: ${#numbers.sequence(0,4,2)}">
    <span th:text="${number}"></span>
</p>
```

在第一个例子中，我们让百里香叶生成一个从 0 到 2 的序列，而在第二个例子中，除了最小和最大值之外，我们还提供了一个步长的定义(在这个例子中，值将按 2 变化)。

请注意，间隔两边都是封闭的。

### 3.4。字符串操作

这是表达式工具对象最全面的特性。

我们可以从检查空或`null`对象的实用程序开始描述。开发人员经常会在百里香标签中使用 Java 方法来实现这一点，这对`null`对象来说可能不安全。

相反，我们可以这样做:

```java
<p th:text="${#strings.isEmpty(string)}"></p>
<p th:text="${#strings.isEmpty(nullString)}"></p>
<p th:text="${#strings.defaultString(emptyString,'Empty String')}"></p>
```

第一个`String`不为空，所以方法会返回`false.` 第二个`String` 是`null`，所以我们会得到`true`。最后，如果`String`为空，我们可以使用`#strings.defaultString(…)`方法指定一个默认值。

还有很多方法。所有这些不仅适用于字符串，也适用于`Java.Collections.` ,例如使用子串相关的操作:

```java
<p th:text="${#strings.indexOf(name,frag)}"></p>
<p th:text="${#strings.substring(name,3,5)}"></p>
<p th:text="${#strings.substringAfter(name,prefix)}"></p>
<p th:text="${#strings.substringBefore(name,suffix)}"></p>
<p th:text="${#strings.replace(name,'las','ler')}"></p>
```

或者使用空安全比较和连接:

```java
<p th:text="${#strings.equals(first, second)}"></p>
<p th:text="${#strings.equalsIgnoreCase(first, second)}"></p>
<p th:text="${#strings.concat(values...)}"></p>
<p th:text="${#strings.concatReplaceNulls(nullValue, values...)}"></p>
```

最后，还有与文本样式相关的功能，这些功能将保持语法始终相同:

```java
<p th:text="${#strings.abbreviate(string,5)} "></p>
<p th:text="${#strings.capitalizeWords(string)}"></p>
```

在第一种方法中，缩写文本的最大尺寸为`n`。如果文本较大，它将被裁剪并以“…”结束。

在第二种方法中，我们将大写单词。

### 3.5。总量

这里我们要讨论的最后一个功能是`aggregates`。它们是安全的，并且提供了从数组或任何其他集合中计算平均值或总和的工具:

```java
<p th:text="${#aggregates.sum(array)}"></p>
<p th:text="${#aggregates.avg(array)}"></p>
<p th:text="${#aggregates.sum(set)}"></p>
<p th:text="${#aggregates.avg(set)}"></p>
```

## 4。结论

在本文中，我们讨论了在百里香框架 3.0 版中实现的表达式工具对象特性。

本教程的完整实现可以在 GitHub 项目中找到。

**怎么考？**我们的建议是先使用浏览器，然后检查现有的 JUnit 测试。

请注意，这些例子并没有涵盖所有可用的效用表达式。如果你想了解所有类型的公用事业，看看这里的。