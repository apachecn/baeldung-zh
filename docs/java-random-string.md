# Java——生成随机字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-random-string>

## 1.介绍

在本教程中，我们将学习如何用 Java 生成随机字符串，首先使用标准 Java 库，然后使用 Java 8 变体，最后使用[Apache Commons Lang 库](https://web.archive.org/web/20221003190651/https://commons.apache.org/proper/commons-lang/)。

本文是 Baeldung 网站上的[“Java—回到基础”系列文章](/web/20221003190651/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 2。用普通 Java 生成随机无界字符串

让我们从简单开始，生成一个 7 个字符的随机`String`:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomStringUnbounded_thenCorrect() {
    byte[] array = new byte[7]; // length is bounded by 7
    new Random().nextBytes(array);
    String generatedString = new String(array, Charset.forName("UTF-8"));

    System.out.println(generatedString);
}
```

请记住，新字符串将不会是任何远程字母数字。

## 延伸阅读:

## [Java 中高效的词频计算器](/web/20221003190651/https://www.baeldung.com/java-word-frequency)

Explore various ways of counting words in Java and see how they perform.[Read more](/web/20221003190651/https://www.baeldung.com/java-word-frequency) →

## [Java–随机长整型、浮点型、整型和双精度型](/web/20221003190651/https://www.baeldung.com/java-generate-random-long-float-integer-double)

Learn how to generate random numbers in Java - both unbounded as well as within a given interval.[Read more](/web/20221003190651/https://www.baeldung.com/java-generate-random-long-float-integer-double) →

## [Java 字符串池指南](/web/20221003190651/https://www.baeldung.com/java-string-pool)

Learn how the JVM optimizes the amount of memory allocated to String storage in the Java String Pool.[Read more](/web/20221003190651/https://www.baeldung.com/java-string-pool) →

## 3。用普通 Java 生成随机有界字符串

接下来让我们看看如何创建一个更受约束的随机字符串；我们将使用小写字母和设定的长度生成一个随机的`String`:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomStringBounded_thenCorrect() {

    int leftLimit = 97; // letter 'a'
    int rightLimit = 122; // letter 'z'
    int targetStringLength = 10;
    Random random = new Random();
    StringBuilder buffer = new StringBuilder(targetStringLength);
    for (int i = 0; i < targetStringLength; i++) {
        int randomLimitedInt = leftLimit + (int) 
          (random.nextFloat() * (rightLimit - leftLimit + 1));
        buffer.append((char) randomLimitedInt);
    }
    String generatedString = buffer.toString();

    System.out.println(generatedString);
}
```

## 4。用 Java 8 生成随机字母串

现在让我们使用 JDK 8 中添加的`Random.ints,` ，来生成一个字母`String:`

```java
@Test
public void givenUsingJava8_whenGeneratingRandomAlphabeticString_thenCorrect() {
    int leftLimit = 97; // letter 'a'
    int rightLimit = 122; // letter 'z'
    int targetStringLength = 10;
    Random random = new Random();

    String generatedString = random.ints(leftLimit, rightLimit + 1)
      .limit(targetStringLength)
      .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
      .toString();

    System.out.println(generatedString);
}
```

## 5。用 Java 8 生成随机字母数字字符串

然后，我们可以扩大我们的字符集，以获得一个字母数字`String:`

```java
@Test
public void givenUsingJava8_whenGeneratingRandomAlphanumericString_thenCorrect() {
    int leftLimit = 48; // numeral '0'
    int rightLimit = 122; // letter 'z'
    int targetStringLength = 10;
    Random random = new Random();

    String generatedString = random.ints(leftLimit, rightLimit + 1)
      .filter(i -> (i <= 57 || i >= 65) && (i <= 90 || i >= 97))
      .limit(targetStringLength)
      .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
      .toString();

    System.out.println(generatedString);
}
```

我们使用了上面的`filter`方法来省略 65 到 90 之间的 Unicode 字符，以避免超出范围的字符。

## 6。用 Apache Commons Lang 生成有界随机字符串

Apache 的 Commons Lang 库在随机字符串生成方面帮助很大。让我们看看**只使用字母**生成有界的`String`:

```java
@Test
public void givenUsingApache_whenGeneratingRandomStringBounded_thenCorrect() {

    int length = 10;
    boolean useLetters = true;
    boolean useNumbers = false;
    String generatedString = RandomStringUtils.random(length, useLetters, useNumbers);

    System.out.println(generatedString);
}
```

因此，这一个用简单的一行代码代替了 Java 示例中的所有低级代码。

## 7 .**。用 Apache Commons Lang 生成字母字符串**

下面是另一个非常简单的例子，这次是一个只有字母字符的有界的`String`，但是没有将布尔标志传递给 API:

```java
@Test
public void givenUsingApache_whenGeneratingRandomAlphabeticString_thenCorrect() {
    String generatedString = RandomStringUtils.randomAlphabetic(10);

    System.out.println(generatedString);
}
```

## 8。用 Apache Commons Lang 生成字母数字字符串

最后，我们有相同的随机有界的`String,`,但这次是数字:

```java
@Test
public void givenUsingApache_whenGeneratingRandomAlphanumericString_thenCorrect() {
    String generatedString = RandomStringUtils.randomAlphanumeric(10);

    System.out.println(generatedString);
}
```

现在我们有了它，**用普通 Java、Java 8 变体或 Apache Commons 库创建有界和无界字符串**。

## 9.结论

通过不同的实现方法，我们能够使用普通 Java、Java 8 变体或 Apache Commons 库生成绑定和非绑定字符串。

在这些 Java 例子中，我们使用了`java.util.Random`，但是有一点值得一提的是，它不是密码安全的。**对于安全敏感的应用，考虑使用 [`java.security.SecureRandom`](/web/20221003190651/https://www.baeldung.com/java-secure-random) 来代替。**

所有这些例子和片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221003190651/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。