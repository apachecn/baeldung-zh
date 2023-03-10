# 在 Java 中用分隔符连接原语数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-join-primitive-array>

## 1.介绍

在这个快速教程中，我们将学习如何在 Java 中用单字符分隔符**连接一个原语数组。在我们的例子中，我们将考虑两个数组:一个数组`int`和一个数组`char`。**

## 2.定义问题

让我们首先为示例定义一个数组`int`和一个数组`char`，以及我们将用来连接它们的内容的分隔符:

```java
int[] intArray = {1, 2, 3, 4, 5, 6, 7, 8, 9};
char[] charArray = {'a', 'b', 'c', 'd', 'e', 'f'};
char separatorChar = '-';
String separator = String.valueOf(separatorChar); 
```

注意，我们已经包含了一个`char`和`String`分隔符，因为**我们将展示的一些方法需要一个`char`参数，而其他的需要一个`String`参数**。

连接操作的结果将包含`int`数组的`“1-2-3-4-5-6-7-8-9”`，以及`char`数组的`“a-b-c-d-e-f”`。

## 3.`Collectors.joining()`

让我们从 Java 8 Stream API 中的一个可用方法开始— `Collectors.joining()`。

首先，我们使用`java.util`包中的`Arrays.stream()`方法从原语数组中创建一个`Stream`。接下来，我们将每个元素映射到`String`。最后，我们用给定的分隔符连接元素。

让我们从我们的`int`数组开始:

```java
String joined = Arrays.stream(intArray)
  .mapToObj(String::valueOf)
  .collect(Collectors.joining(separator));
```

**用这种方法加入我们的`char`数组时，必须先把`char`数组包装成`CharBuffer`，然后再投射到`char`。**这是因为`chars()`方法返回一个`int`值的`Stream`。

**不幸的是，Java Stream API 没有提供一个本机方法来包装`char`的`Stream`。**

让我们加入我们的`char`阵列:

```java
String joined = CharBuffer.wrap(charArray).chars()
  .mapToObj(intValue -> String.valueOf((char) intValue))
  .collect(Collectors.joining(separator));
```

## 4.`StringJoiner`

与`Collectors.joining()`类似，这种方法利用了流 API，但是它不是收集元素，而是遍历元素并将它们添加到一个`StringJoiner`实例中:

```java
StringJoiner intStringJoiner = new StringJoiner(separator);
Arrays.stream(intArray)
  .mapToObj(String::valueOf)
  .forEach(intStringJoiner::add);
String joined = intStringJoiner.toString();
```

**同样，在使用流 API 时，我们必须将`char`数组包装成`CharBuffer`** :

```java
StringJoiner charStringJoiner = new StringJoiner(separator);
CharBuffer.wrap(charArray).chars()
  .mapToObj(intChar -> String.valueOf((char) intChar))
  .forEach(charStringJoiner::add);
String joined = charStringJoiner.toString();
```

## 5.Apache Commons Lang

[Apache Commons Lang](/web/20220629000226/https://www.baeldung.com/java-commons-lang-3) 库在`StringUtils`和`ArrayUtils`类中提供了一些方便的方法，我们可以用它们来加入我们的原始数组。

要使用这个库，我们需要将`[commons-lang3](https://web.archive.org/web/20220629000226/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)` [依赖](https://web.archive.org/web/20220629000226/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

当使用`String`分离器时，我们将同时使用`StringUtils`和`ArrayUtils`。

让我们一起使用这些来加入我们的`int`数组:

```java
String joined = StringUtils.join(ArrayUtils.toObject(intArray), separator);
```

或者，如果我们使用原始的`char`类型作为分隔符，我们可以简单地写:

```java
String joined = StringUtils.join(intArray, separatorChar);
```

加入我们的`char`数组的实现非常相似:

```java
String joined = StringUtils.join(ArrayUtils.toObject(charArray), separator);
```

当使用`char`分离器时:

```java
String joined = StringUtils.join(charArray, separatorChar);
```

## 6.番石榴

[谷歌的番石榴](/web/20220629000226/https://www.baeldung.com/guava-joiner-and-splitter-tutorial)库提供了`Joiner`类，我们可以用它来加入我们的数组。为了在我们的项目中使用番石榴，我们需要添加[`guava`Maven 依赖](https://web.archive.org/web/20220629000226/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22):

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

让我们使用`Joiner`类加入我们的`int`数组:

```java
String joined = Joiner.on(separator).join(Ints.asList(intArray));
```

在这个例子中，我们还使用了来自 Guava 的`Ints.asList()`方法，它很好地将基元数组转换成了`Integer`的`List`。

Guava 提供了一个类似的方法将一个`char`数组转换成一个`Character.`的`List`数组。结果，加入我们的`char`数组看起来非常像上面使用`int`数组的例子:

```java
String joined = Joiner.on(separator).join(Chars.asList(charArray));
```

## 7.`StringBuilder`

最后，**如果我们既不能使用 Java 8，也不能使用第三方库，我们可以用`StringBuilder`** 手动加入一个元素数组。在这种情况下，两种类型的阵列的实现是相同的:

```java
if (array.length == 0) {
    return "";
}
StringBuilder stringBuilder = new StringBuilder();
for (int i = 0; i < array.length - 1; i++) {
    stringBuilder.append(array[i]);
    stringBuilder.append(separator);
}
stringBuilder.append(array[array.length - 1]);
String joined = stringBuilder.toString();
```

## 8。结论

这篇简短的文章展示了用给定的分隔符或字符串连接原语数组的多种方法。我们展示了使用原生 JDK 解决方案的示例，以及使用两个第三方库——Apache Commons Lang 和 Guava——的其他解决方案。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220629000226/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)