# Cactoos 图书馆指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cactoos>

## 1.介绍

[Cactoos](https://web.archive.org/web/20221206195206/https://github.com/yegor256/cactoos) **是一个面向对象的 Java 原语类型库**。

在本教程中，我们将看看这个库中的一些类。

## 2.卡图斯

Cactoos 库的内容非常丰富，从字符串操作到数据结构。这个库提供的原语类型和它们对应的方法类似于其他库提供的，如 [Guava](/web/20221206195206/https://www.baeldung.com/category/guava/) 和 [Apache Commons](/web/20221206195206/https://www.baeldung.com/java-commons-lang-3) ，但是更侧重于面向对象的[设计原则](/web/20221206195206/https://www.baeldung.com/solid-principles)。

### 2.1.与 Apache Commons 的比较

Cactoos 库配备了一些类，这些类提供了与 Apache Commons 库的静态方法相同的功能。

让我们来看看这些静态方法，它们是`StringUtils`包的一部分，以及它们在 Cactoos 中的等价类:

| **string utils 的静态方法** | **等价仙人掌类** |
| isBlank() | IsBlank |
| 小写() | 降低 |
| 大写() | 上面的 |
| 旋转() | 旋转 |
| 交换情况() | 交换案例 |
| stripStart() | trimmedalft |
| 条纹末端() | 修剪右侧 |

关于这方面的更多信息可以在[官方文档](https://web.archive.org/web/20221206195206/https://github.com/yegor256/cactoos#our-objects-vs-their-static-methods)中找到。我们将在下一节中研究其中一些的实现。

## 3.**美芬依赖**

让我们从添加所需的 Maven 依赖项开始。该库的最新版本可以在 [Maven Central](https://web.archive.org/web/20221206195206/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%20%22org.cactoos%22%20AND%20a%3A%20%22cactoos%22) 上找到:

```java
<dependency>
    <groupId>org.cactoos</groupId>
    <artifactId>cactoos</artifactId>
    <version>0.43</version>
</dependency>
```

## 4.用线串

Cactoos 有很多用于操作`String`对象的类。

### 4.1.字符串对象创建

让我们看看如何使用`TextOf`类创建一个`String`对象:

```java
String testString = new TextOf("Test String").asString();
```

### 4.2.格式化字符串

如果需要创建格式化的`String`，我们可以使用`FormattedText`类:

```java
String formattedString = new FormattedText("Hello %s", stringToFormat).asString();
```

让我们验证这个方法实际上返回了格式化的`String`:

```java
StringMethods obj = new StringMethods();

String formattedString = obj.createdFormattedString("John");
assertEquals("Hello John", formattedString);
```

### 4.3.小写/大写字符串

*降低的*类使用其`TextOf`对象将`String`转换成小写:

```java
String lowerCaseString = new Lowered(new TextOf(testString)).asString();
```

类似地，给定的`String`可以使用`Upper`类转换成大写字母:

```java
String upperCaseString = new Upper(new TextOf(testString)).asString();
```

让我们使用一个测试字符串来验证这些方法的输出:

```java
StringMethods obj = new StringMethods();

String lowerCaseString = obj.toLowerCase("TeSt StrIng");
String upperCaseString = obj.toUpperCase("TeSt StrIng"); 

assertEquals("test string", lowerCaseString);
assertEquals("TEST STRING", upperCaseString);
```

### 4.4.检查是否有空字符串

如前所述，Cactoos 库提供了一个`IsBlank`类来检查`null`或空的`String`:

```java
new IsBlank(new TextOf(testString)) != null;
```

## 5.收集

这个库也提供了几个处理`Collections`的类。让我们来看看其中的一些。

### 5.1.迭代集合

我们可以使用实用程序类`And`迭代字符串列表:

```java
new And((String input) -> LOGGER.info(new FormattedText("%s\n", input).asString()), strings).value();
```

上面的方法是一种迭代`Strings`列表的函数方式，它将输出写入记录器。

### 5.2.过滤收藏

`Filtered`类可用于根据特定标准过滤集合:

```java
Collection<String> filteredStrings 
  = new ListOf<>(new Filtered<>(string -> string.length() == 5, new IterableOf<>(strings)));
```

让我们通过传入几个参数来测试这个方法，其中只有 3 个参数满足标准:

```java
CollectionUtils obj = new CollectionUtils(); 

List<String> strings = new ArrayList<String>() {
    add("Hello"); 
    add("John");
    add("Smith"); 
    add("Eric"); 
    add("Dizzy"); 
};

int size = obj.getFilteredList(strings).size(); 

assertEquals(3, size);
```

这个库提供的其他一些收藏类可以在官方文档中找到。

## 6.结论

在本教程中，我们已经了解了 Cactoos 库和它提供的一些用于字符串和数据结构操作的类。

除此之外，该库还为 [IO 操作](https://web.archive.org/web/20221206195206/https://github.com/yegor256/cactoos#inputoutput)以及[日期和时间](https://web.archive.org/web/20221206195206/https://github.com/yegor256/cactoos#dates-and-times)提供了其他实用程序类。

像往常一样，本教程中使用的代码示例可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221206195206/https://github.com/eugenp/tutorials/tree/master/libraries-3)