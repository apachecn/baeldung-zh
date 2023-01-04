# 数组到字符串的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-to-string>

## 1.概观

在这个简短的教程中，我们将看看如何将一个字符串或整数数组转换成一个字符串，然后再转换回来。

我们可以通过常用库中的普通 Java 和 Java 实用程序类来实现这一点。

## 2.将数组转换为字符串

有时我们需要将字符串或整数的数组转换成字符串，但不幸的是，没有直接的方法来执行这种转换。

数组上的`toString()`方法的默认实现返回类似于`Ljava.lang.String;@74a10858`的东西，它只通知我们对象的类型和散列码。

然而， [`java.util.Arrays`](/web/20220714102915/https://www.baeldung.com/java-util-arrays) 实用程序类支持数组和字符串操作，包括一个用于数组的`toString()`方法。

**`Arrays.toString()`返回一个带有输入数组内容的字符串。**新创建的字符串是一个逗号分隔的数组元素列表，用方括号(“[]”)括起来:

```java
String[] strArray = { "one", "two", "three" };
String joinedString = Arrays.toString(strArray);
assertEquals("[one, two, three]", joinedString);
```

```java
int[] intArray = { 1,2,3,4,5 }; 
joinedString = Arrays.toString(intArray);
assertEquals("[1, 2, 3, 4, 5]", joinedString);
```

虽然`Arrays.toString(int[])`方法很好地为我们完成了这项任务，但是让我们将它与我们自己可以实现的不同方法进行比较。

### 2.1.`StringBuilder.append()`

首先，让我们看看如何使用`StringBuilder.append()`进行这种转换:

```java
String[] strArray = { "Convert", "Array", "With", "Java" };
StringBuilder stringBuilder = new StringBuilder();
for (int i = 0; i < strArray.length; i++) {
    stringBuilder.append(strArray[i]);
}
String joinedString = stringBuilder.toString();
assertEquals("ConvertArrayWithJava", joinedString);
```

此外，为了转换整数数组，我们可以使用相同的方法，但是在追加到我们的`StringBuilder`时调用`Integer.valueOf(intArray[i]) `。

### 2.2.Java 流 API

Java 8 和更高版本提供了`String.join()`方法，通过连接元素并用指定的分隔符分隔它们来产生新的字符串，在我们的例子中只是空字符串:

```java
String joinedString = String.join("", new String[]{ "Convert", "With", "Java", "Streams" });
assertEquals("ConvertWithJavaStreams", joinedString);
```

此外，我们可以使用 Java Streams API 中的`Collectors.joining()`方法，该方法按照与源数组相同的顺序连接来自`Stream`的字符串:

```java
String joinedString = Arrays
    .stream(new String[]{ "Convert", "With", "Java", "Streams" })
    .collect(Collectors.joining());
assertEquals("ConvertWithJavaStreams", joinedString);
```

### 2.3.`StringUtils.join()`

Apache Commons Lang 永远不会被排除在这样的任务之外。

`StringUtils`类有几个`StringUtils.join()`方法，可以用来将字符串数组变成单个字符串:

```java
String joinedString = StringUtils.join(new String[]{ "Convert", "With", "Apache", "Commons" });
assertEquals("ConvertWithApacheCommons", joinedString);
```

### 2.4.`Joiner.join()`

并且不甘示弱，`Guava`用它的`[Joiner](/web/20220714102915/https://www.baeldung.com/guava-joiner-and-splitter-tutorial) `类容纳了同样的。`Joiner `类提供了一个流畅的 API，并提供了一些辅助方法来连接数据。

例如，我们可以添加一个分隔符或跳过空值:

```java
String joinedString = Joiner.on("")
        .skipNulls()
        .join(new String[]{ "Convert", "With", "Guava", null });
assertEquals("ConvertWithGuava", joinedString);
```

## 3.将字符串转换为字符串数组

类似地，我们有时需要将一个字符串分割成一个数组，该数组包含由指定分隔符分割的输入字符串的某个子集，让我们来看看如何实现这一点。

### 3.1.`String.split()`

首先，让我们从使用没有分隔符的`String.split()`方法分割空白开始:

```java
String[] strArray = "loremipsum".split("");
```

它产生:

```java
["l", "o", "r", "e", "m", "i", "p", "s", "u", "m"]
```

### 3.2.`StringUtils.split()`

其次，我们再来看看来自 Apache 的 Commons Lang 库的`StringUtils`类。

在字符串对象上的许多空安全方法中，我们可以找到默认情况下的`StringUtils.split().` ,它假定了一个空白分隔符:

```java
String[] splitted = StringUtils.split("lorem ipsum dolor sit amet");
```

这导致:

```java
["lorem", "ipsum", "dolor", "sit", "amet"]
```

但是，如果我们愿意，我们也可以提供一个分隔符。

### 3.3.`Splitter.split()`

最后，我们还可以将`Guava`与它的`Splitter` fluent API 一起使用:

```java
List<String> resultList = Splitter.on(' ')
    .trimResults()
    .omitEmptyStrings()
    .splitToList("lorem ipsum dolor sit amet");   
String[] strArray = resultList.toArray(new String[0]);
```

这会产生:

```java
["lorem", "ipsum", "dolor", "sit", "amet"]
```

## 4.结论

在本文中，我们展示了如何使用核心 Java 和流行的实用程序库将数组转换成字符串，以及如何将字符串转换成数组。

当然，所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220714102915/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions-2)