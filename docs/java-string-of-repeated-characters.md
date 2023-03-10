# 生成 N 个重复字符的 Java 字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-of-repeated-characters>

## 1.概观

在本教程中，我们将熟悉**生成 N 个重复字符**的不同选项。当我们需要添加填充空白、生成 [ASCII 艺术](/web/20220815152727/https://www.baeldung.com/ascii-art-in-java)等时，这很方便。

这个问题在 JDK11 中很容易解决，但是如果我们使用的是早期版本，那么还有许多其他的解决方案。我们将从最常见的方法开始，并添加一些库中的其他方法。

## 2.例子

让我们定义将在所有解决方案中用来验证生成的字符串的常数:

```java
private static final String EXPECTED_STRING = "aaaaaaa";
private static final int N = 7;
```

因此， *EXPECTED_STRING* 常量表示我们需要在解决方案中生成的字符串。 *N* 常量用于定义字符重复的次数。

现在，让我们检查用于生成 N 个重复字符的字符串`a`的选项。

## 3.JDK11 `String.repeat`功能

Java 有一个`repeat`函数来构建源字符串的副本:

```java
String newString = "a".repeat(N);
assertEquals(EXPECTED_STRING, newString);
```

这允许我们重复单个字符或多字符串:

```java
String newString = "-->".repeat(5);
assertEquals("-->-->-->-->-->", newString);
```

这背后的算法使用循环来非常有效地填充字符数组。

如果我们没有 JDK11，那么我们将不得不自己创建一个算法，或者使用第三方库中的算法。其中最好的不太可能比 JDK11 原生解决方案更快或更容易使用。

## 4.构建`String`的常用方法

### 4.1.`StringBuilder`带有一个`for`回路

先说`StringBuilder`类。我们将遍历一个`for`循环 N 次，添加重复的字符:

```java
StringBuilder builder = new StringBuilder(N);
for (int i = 0; i < N; i++) {
    builder.append("a");
}
String newString = builder.toString();
assertEquals(EXPECTED_STRING, newString);
```

通过这种方法，我们得到了想要的字符串。**这可能是理解**、**最简单的方法，但在运行时不一定最快**。

### 4.2.`char`带有`for`循环的数组

我们可以用我们想要的字符填充一个固定大小的`char`数组，并将其转换成一个字符串:

```java
char[] charArray = new char[N];
for (int i = 0; i < N; i++) {
    charArray[i] = 'a';
}
String newString = new String(charArray);
assertEquals(EXPECTED_STRING, newString);
```

这应该会更快，因为**在我们构建**时，它不需要动态大小的结构来存储我们的字符串，Java 可以有效地将`char`数组转换为`String.`

### 4.3.`Arrays fill`方法

我们可以使用库函数来填充数组，而不是使用循环:

```java
char charToAppend = 'a';
char[] charArray = new char[N];
Arrays.fill(charArray, charToAppend);
String newString = new String(charArray);
assertEquals(EXPECTED_STRING, newString);
```

与之前的解决方案相比，这种解决方案在运行时时间更短、效率更高。

## 5.用`repeat`方法生成字符串

### 5.1.阿帕奇`repeat `方法

这个解决方案需要为 [Apache Commons 库](https://web.archive.org/web/20220815152727/https://search.maven.org/artifact/org.apache.commons/commons-lang3/3.12.0/jar)添加一个新的依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

添加这个依赖项后，我们可以使用来自`StringUtils`类的`repeat`方法。**它将一个重复字符和该字符应该重复的次数作为参数**:

```java
char charToAppend = 'a';
String newString = StringUtils.repeat(charToAppend, N);
assertEquals(EXPECTED_STRING, newString);
```

### 5.2.番石榴`repeat`法

与前一种方法一样，这种方法需要为 [Guava](https://web.archive.org/web/20220815152727/https://search.maven.org/artifact/com.google.guava/guava/31.0.1-jre/bundle) 库添加一个新的依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

除了它来自不同的库之外，这个解决方案与 Apache Commons 解决方案完全相同:

```java
String charToAppend = "a";
String newString = Strings.repeat(charToAppend, N);
assertEquals(EXPECTED_STRING, newString);
```

## 6.用`nCopies`方法生成字符串

如果我们认为我们的目标字符串是重复子字符串的集合，那么我们可以使用一个`List`实用程序来构建列表，然后将结果列表转换成我们的最终`String`。为此，我们可以使用`java.util`包中`Collections`类的`nCopies`方法:

```java
public static <T> List<T> nCopies(int n, T o);
```

虽然构建子字符串列表不如我们使用固定字符数组的解决方案有效，但重复字符模式而不仅仅是单个字符会很有帮助。

### 6.1.`String` `join `和`nCopies` 的方法

让我们用`nCopies`方法创建一个单字符字符串列表，并使用`String.join`将其转换为我们的结果:

```java
String charToAppend = "a";
String newString = String.join("", Collections.nCopies(N, charToAppend));
assertEquals(EXPECTED_STRING, newString);
```

`String.join`方法需要一个分隔符，为此我们使用空字符串。

### 6.2.番石榴`Joiner`和`nCopies` 的方法

番石榴提供了一种替代的串连接器，我们也可以使用:

```java
String charToAppend = "a";
String newString = Joiner.on("").join(Collections.nCopies(N, charToAppend));
assertEquals(EXPECTED_STRING, newString);
```

## 7.用`Stream generate`方法生成`String`

创建子字符串列表的缺点是，在构造最终字符串之前，我们创建了一个可能很大的临时列表对象。

但是，从 Java 8 开始，我们可以使用来自 [`Stream` API](/web/20220815152727/https://www.baeldung.com/java-8-streams-introduction) 的`generate`方法。**结合`limit`方法(用于定义长度)和`collect` 方法，我们可以生成一串 N 个重复的字符**:

```java
String charToAppend = "a";
String newString = generate(() -> charToAppend)
  .limit(length)
  .collect(Collectors.joining());
assertEquals(exampleString, newString);
```

## 8.用 Apache 的`RandomStringUtils`生成`String`

**来自`Apache Commons`库的`RandomStringUtils`类支持使用`random`方法**生成一串 N 个重复的字符。我们必须定义一个字符和重复的次数:

```java
String charToAppend = "a";
String newString = RandomStringUtils.random(N, charToAppend);
assertEquals(EXPECTED_STRING, newString);
```

## 9.结论

在本文中，我们看到了生成 N 个重复字符的字符串的各种解决方案。其中最简单的是`String.repeat`，从 JDK 11 号开始就有了。

对于 Java 的早期版本，有许多其他可能的可用选项。最佳选择将取决于我们在运行时效率、易于编码和库可用性方面的需求。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220815152727/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-3)