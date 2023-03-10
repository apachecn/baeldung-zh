# 接受字符输入的 Java 扫描器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-scanner-character-input>

## 1.概观

在本教程中，我们将看到如何从`[Scanner](/web/20221004100110/https://www.baeldung.com/java-scanner)`类中获取字符输入。

## 2.扫描字符

**Java [`Scanner`](https://web.archive.org/web/20221004100110/https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Scanner.html) 没有提供任何类似于`nextInt()`、`nextLine(),`等文字输入的方法。**

有几种方法我们可以使用`Scanner`进行字符输入。

让我们首先创建一个输入字符串:

```java
String input = new StringBuilder().append("abc\n")
  .append("mno\n")
  .append("xyz\n")
  .toString();
```

## 3.使用`next()`

让我们看看**如何使用`Scanner`的`next()`方法和`String`类的`charAt()`方法来获取一个字符作为输入**:

```java
@Test
public void givenInputSource_whenScanCharUsingNext_thenOneCharIsRead() {
    Scanner sc = new Scanner(input);
    char c = sc.next().charAt(0);
    assertEquals('a', c);
}
```

**Java Scanner 的`next()`方法返回一个字符串对象**。我们在这里使用`String`类的`charAt()`方法从字符串对象中获取字符。

## 4.使用`findInLine()`

这个方法接受一个字符串模式作为输入，我们将传递“.”(点)仅匹配单个字符。然而，这将返回一个字符串形式的字符，所以我们将使用`charAt()`方法来获取字符:

```java
@Test
public void givenInputSource_whenScanCharUsingFindInLine_thenOneCharIsRead() {
    Scanner sc = new Scanner(input);
    char c = sc.findInLine(".").charAt(0);
    assertEquals('a', c);
}
```

## 5.使用`useDelimeter()`

这个方法也只扫描一个字符，但是作为一个类似于`findInLine()` API 的字符串对象。我们可以类似地使用`charAt()`方法来获取字符值:

```java
@Test
public void givenInputSource_whenScanCharUsingUseDelimiter_thenOneCharIsRead() {
    Scanner sc = new Scanner(input);
    char c = sc.useDelimiter("").next().charAt(0);
    assertEquals('a', c);
}
```

## 6.结论

在本教程中，我们学习了如何使用 Java `Scanner`获取 char 输入。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221004100110/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis-2)