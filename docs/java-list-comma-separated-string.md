# 将列表转换为逗号分隔的字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-comma-separated-string>

## 1.介绍

[`List`转换](/web/20221017061307/https://www.baeldung.com/java-string-with-separator-to-list)仍然是一个热门话题，因为这是我们作为 Java 开发人员经常做的事情。在本教程中，我们将学习如何使用四种不同的方法将`String`的`List`转换成逗号分隔的`String` 。

## 2.使用 Java 8+

我们将使用三个不同的类和它们的方法进行转换，从 Java 8 开始就有了。

让我们将下面的列表作为即将到来的示例的输入:

```java
List<String> arraysAsList = Arrays.asList("ONE", "TWO", "THREE");
```

### 2.1.`String`

**首先，我们将使用`String`类，它有许多用于`String`处理的实用程序，并提供了转换方法`join()` :**

```java
String commaSeparatedString = String.join(",", arraysAsList);

assertThat(commaSeparatedString).isEqualTo("ONE,TWO,THREE");
```

### 2.2.`StringJoiner`

**其次，我们将使用 [`StringJoiner`](/web/20221017061307/https://www.baeldung.com/java-string-joiner) 类，它有一个构造函数，接受一个`CharSequence`分隔符作为参数:**

```java
StringJoiner stringJoiner = new StringJoiner(",");
arraysAsList.stream()
  .forEach(v -> stringJoiner.add(v));
String commaSeparatedString = stringJoiner.toString();

assertThat(commaSeparatedString).isEqualTo("ONE,TWO,THREE");
```

**还有另一个构造函数，它采用一个`CharSequence`分隔符，一个`CharSequence` 作为前缀，另一个作为后缀:**

```java
StringJoiner stringJoinerWithDelimiterPrefixSuffix = new StringJoiner(",", "[", "]");
arraysAsList.stream()
  .forEach(v -> stringJoinerWithDelimiterPrefixSuffix.add(v));
String commaSeparatedStringWithDelimiterPrefixSuffix = stringJoinerWithDelimiterPrefixSuffix.toString();

assertThat(commaSeparatedStringWithDelimiterPrefixSuffix).isEqualTo("[ONE,TWO,THREE]");
```

### 2.3.`Collectors`

**第三， [`Collectors`](/web/20221017061307/https://www.baeldung.com/java-list-to-string#custom-implementation-using-collectors) 类提供了各种带有不同签名的实用程序和`joining()`方法。**

首先让我们看看如何通过使用`Collectors.joining()`方法将`collect()`方法应用到`Stream`中，将`CharSequence`分隔符作为输入:

```java
String commaSeparatedUsingCollect = arraysAsList.stream()
  .collect(Collectors.joining(","));

assertThat(commaSeparatedUsingCollect).isEqualTo("ONE,TWO,THREE");
```

在下一个例子中，我们将看到如何使用`map()`方法将列表中的每个对象转换为`String`，然后应用`collect()`和`Collectors.` `joining()`方法:

```java
String commaSeparatedObjectToString = arraysAsList.stream()
  .map(Object::toString)
  .collect(Collectors.joining(","));

assertThat(commaSeparatedObjectToString).isEqualTo("ONE,TWO,THREE");
```

接下来，我们将使用`map()`方法将列表元素转换成`String`，然后应用方法`collect()`和 `Collectors.joining()`:

```java
String commaSeparatedStringValueOf = arraysAsList.stream()
  .map(String::valueOf)
  .collect(Collectors.joining(","));

assertThat(commaSeparatedStringValueOf).isEqualTo("ONE,TWO,THREE");
```

现在，让我们像上面一样使用`map()`，然后使用`Collectors.` `joining()`方法输入一个`CharSequence`分隔符，一个`CharSequence`作为前缀，一个`CharSequence`作为后缀:

```java
String commaSeparatedStringValueOfWithDelimiterPrefixSuffix = arraysAsList.stream()
  .map(String::valueOf)
  .collect(Collectors.joining(",", "[", "]"));

assertThat(commaSeparatedStringValueOfWithDelimiterPrefixSuffix).isEqualTo("[ONE,TWO,THREE]");
```

最后，我们将看到如何使用`reduce()`方法来转换列表，而不是使用`collect()`:

```java
String commaSeparatedUsingReduce = arraysAsList.stream()
  .reduce((x, y) -> x + "," + y)
  .get();

assertThat(commaSeparatedUsingReduce).isEqualTo("ONE,TWO,THREE");
```

## 3.使用 Apache Commons Lang

或者，我们也可以使用由 [Apache Commons Lang](/web/20221017061307/https://www.baeldung.com/java-list-to-string#using-an-external-library) 库提供的实用程序类来代替 Java 类。

**我们必须在我们的`pom.xml`文件中添加一个[依赖项](https://web.archive.org/web/20221017061307/https://search.maven.org/search?q=g:org.apache.commons%20a:commons-lang3)，以使用 Apache 的`StringUtils`类:**

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.0</version>
</dependency>
```

**`join()`方法有多种实现，接受输入，比如一系列元素、一个值的`Iterator`，以及多种形式下的分隔符，比如`String`或`char` :**

```java
String commaSeparatedString = StringUtils.join(arraysAsList, ",");

assertThat(commaSeparatedString).isEqualTo("ONE,TWO,THREE");
```

**如果传递给`join()`的信息是一个*对象*的数组，它也将一个`int`作为 startIndex，一个`int`作为 endIndex:**

```java
String commaSeparatedStringIndex = StringUtils.join(arraysAsList.toArray(), ",", 0, 3);

assertThat(commaSeparatedStringIndex).isEqualTo("ONE,TWO,THREE");
```

## 4.使用弹簧芯

Spring Core library 同样提供了一个具有这种转换方法的实用程序类。

**我们首先在我们的`pom.xml`文件中添加一个[依赖项](https://web.archive.org/web/20221017061307/https://search.maven.org/search?q=g:org.springframework%20a:spring-core)，以使用 Spring 的核心`StringUtils`类:**

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.22</version>
</dependency>
```

**Spring 的核心`StringUtils`类提供了一个方法`collectionToCommaDelimitedString(),`，它将一个逗号作为默认分隔符，将一个`Collection`作为参数进行转换:**

```java
String collectionToCommaDelimitedString = StringUtils.collectionToCommaDelimitedString(arraysAsList);

assertThat(collectionToCommaDelimitedString).isEqualTo("ONE,TWO,THREE");
```

**第二种方法，`collectionToDelimitedString(),`将一个要转换的`Collection`和一个`String` 分隔符:**作为参数

```java
String collectionToDelimitedString = StringUtils.collectionToDelimitedString(arraysAsList, ",");

assertThat(collectionToDelimitedString).isEqualTo("ONE,TWO,THREE");
```

## 5.使用谷歌番石榴

最后，我们将使用谷歌番石榴图书馆。

**我们必须在我们的`pom.xml`文件中添加一个[依赖项](https://web.archive.org/web/20221017061307/https://search.maven.org/search?q=g:com.google.guava%20a:guava)来使用 Google 的番石榴`Joiner`类:**

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

Google 的 Guava `Joiner` 类提供了我们可以应用的各种方法。

**第一种方法是`on(),`，它将一个`String`作为定界符参数，然后第二种方法是`join()`方法，它将具有要转换的值的`Iterable`作为参数:**

```java
String commaSeparatedString = Joiner.on(",")
  .join(arraysAsList);

assertThat(commaSeparatedString).isEqualTo("ONE,TWO,THREE");
```

让我们为下一个例子取另一个包含一些`null`值的列表:

```java
List<String> arraysAsListWithNull = Arrays.asList("ONE", null, "TWO", null, "THREE");
```

**鉴于此，我们可以在`on()`和`join()`之间使用其他方法，其中之一就是`skipNulls()`方法。我们可以用它来避免从`Iterable:`到**的`null` 值的转换

```java
String commaSeparatedStringSkipNulls = Joiner.on(",")
  .skipNulls()
  .join(arraysAsListWithNull);

assertThat(commaSeparatedStringSkipNulls).isEqualTo("ONE,TWO,THREE");
```

**另一种选择是使用`useForNull(),`，它以一个`String`值为参数，替换`Iterable`中的`null`值来转换:**

```java
String commaSeparatedStringUseForNull = Joiner.on(",")
  .useForNull(" ")
  .join(arraysAsListWithNull);

assertThat(commaSeparatedStringUseForNull).isEqualTo("ONE, ,TWO, ,THREE");
```

## 6.结论

在本文中，我们已经看到了各种将`String`的`List`转换成逗号分隔的`String`的例子。最后，由我们来选择哪个库和实用程序类更适合我们的目的。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20221017061307/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-4)