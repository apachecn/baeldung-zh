# 在 Java 中查找一个字符串中的所有数字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-find-numbers-in-string>

## 1.概观

有时我们需要在字符串中找到数字或完整的数字。我们可以通过正则表达式或某些库函数来实现这一点。

在本文中，我们将使用正则表达式来查找和提取字符串中的数字。我们还将介绍一些计算数字的方法。

## 2.计算数字位数

让我们从计算字符串中的位数开始。

### 2.1.使用正则表达式

我们可以使用 [Java 正则表达式](/web/20221208143830/https://www.baeldung.com/regular-expressions-java)到[计算一个数字的匹配次数](/web/20221208143830/https://www.baeldung.com/java-count-regex-matches)。

在正则表达式中，**`\d``“`匹配“任意个位数”**。让我们用这个表达式来计算字符串中的位数:

```java
int countDigits(String stringToSearch) {
    Pattern digitRegex = Pattern.compile("\\d");
    Matcher countEmailMatcher = digitRegex.matcher(stringToSearch);

    int count = 0;
    while (countEmailMatcher.find()) {
        count++;
    }

    return count;
}
```

一旦我们为正则表达式定义了一个`Matcher`,我们就可以在一个循环中使用它到`find` ,并对所有匹配进行计数。让我们来测试一下:

```java
int count = countDigits("64x6xxxxx453xxxxx9xx038x68xxxxxx95786xxx7986");

assertThat(count, equalTo(21));
```

### 2.2.使用谷歌番石榴`CharMatcher`

要使用 [Guava](https://web.archive.org/web/20221208143830/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) ，我们首先需要添加 Maven 依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

番石榴提供了 [`CharMatcher.inRange​`](https://web.archive.org/web/20221208143830/https://guava.dev/releases/30.0-jre/api/docs/com/google/common/base/CharMatcher.html#inRange(char,char) "CharMatcher.inRange​") 计数位数的方法:

```java
int count = CharMatcher.inRange('0', '9')
  .countIn("64x6xxxxx453xxxxx9xx038x68xxxxxx95786xxx7986");

assertThat(count, equalTo(21));
```

## 3.查找数字

计算数字需要捕获有效数字表达式的所有数字的模式。

### 3.1.寻找整数

为了构造一个表达式来识别整数，我们必须考虑到**它们可以是正数或负数，并且由一个或多个数字序列**组成。我们还注意到负整数前面有一个减号。

因此，我们可以通过将正则表达式扩展到“`-?\d+`”来查找整数。这种模式意味着“一个可选的减号，后跟一个或多个数字”。

让我们创建一个示例方法，使用这个正则表达式来查找字符串中的整数:

```java
List<String> findIntegers(String stringToSearch) {
    Pattern integerPattern = Pattern.compile("-?\\d+");
    Matcher matcher = integerPattern.matcher(stringToSearch);

    List<String> integerList = new ArrayList<>();
    while (matcher.find()) {
        integerList.add(matcher.group());
    }

    return integerList;
}
```

一旦我们在正则表达式上创建了一个`Matcher`，我们就在一个循环中使用它来`find`一个字符串中的所有整数。我们在每次匹配时调用`group`来获取所有的整数。

让我们来测试一下`findIntegers`:

```java
List<String> integersFound = 
  findIntegers("646xxxx4-53xxx34xxxxxxxxx-35x45x9xx3868xxxxxx-95786xxx79-86");

assertThat(integersFound)
  .containsExactly("646", "4", "-53", "34", "-35", "45", "9", "3868", "-95786", "79", "-86"); 
```

### 3.2.寻找十进制数

要创建一个正则表达式让[找到十进制数](/web/20221208143830/https://www.baeldung.com/java-check-string-number#regex)，我们需要考虑书写时使用的字符模式。

如果十进制数是负数，它以负号开始。其后是一个或多个数字和一个可选的小数部分。这个小数部分以小数点开始，其后是一个或多个数字的序列。

我们可以使用正则表达式`“-?\d+(\.\d+)?`来定义它:

```java
List<String> findDecimalNums(String stringToSearch) {
    Pattern decimalNumPattern = Pattern.compile("-?\\d+(\\.\\d+)?");
    Matcher matcher = decimalNumPattern.matcher(stringToSearch);

    List<String> decimalNumList = new ArrayList<>();
    while (matcher.find()) {
        decimalNumList.add(matcher.group());
    }

    return decimalNumList;
}
```

现在我们将测试`findDecimalNums`:

```java
List<String> decimalNumsFound = 
  findDecimalNums("x7854.455xxxxxxxxxxxx-3x-553.00x53xxxxxxxxxxxxx3456xxxxxxxx3567.4xxxxx");

assertThat(decimalNumsFound)
  .containsExactly("7854.455", "-3", "-553.00", "53", "3456", "3567.4"); 
```

## 4.将找到的字符串转换为数值

我们可能还希望将找到的数字转换成它们的 Java 类型。

让我们使用 [`Stream`映射](/web/20221208143830/https://www.baeldung.com/java-streams)将整数转换成`Long`:

```java
LongStream integerValuesFound = findIntegers("x7854x455xxxxxxxxxxxx-3xxxxxx34x56")
  .stream()
  .mapToLong(Long::valueOf);

assertThat(integerValuesFound)
  .containsExactly(7854L, 455L, -3L, 34L, 56L);
```

接下来，我们将以同样的方式将十进制数转换为`Double`:

```java
DoubleStream decimalNumValuesFound = findDecimalNums("x7854.455xxxxxxxxxxxx-3xxxxxx34.56")
  .stream()
  .mapToDouble(Double::valueOf);

assertThat(decimalNumValuesFound)
  .containsExactly(7854.455, -3.0, 34.56);
```

## 5.寻找其他类型的数字

数字可以用其他格式表示，我们可以通过调整正则表达式来检测。

### 5.1.科学符号

让我们找一些用科学记数法格式化的数字:

```java
String strToSearch = "xx1.25E-3xxx2e109xxx-70.96E+105xxxx-8.7312E-102xx919.3822e+31xxx";

Matcher matcher = Pattern.compile("-?\\d+(\\.\\d+)?[eE][+-]?\\d+")
  .matcher(strToSearch);

// loop over the matcher

assertThat(sciNotationNums)
  .containsExactly("1.25E-3", "2e109", "-70.96E+105", "-8.7312E-102", "919.3822e+31"); 
```

### 5.2.十六进制的

现在我们将在字符串中找到十六进制数:

```java
String strToSearch = "xaF851Bxxx-3f6Cxx-2Ad9eExx70ae19xxx";

Matcher matcher = Pattern.compile("-?[0-9a-fA-F]+")
  .matcher(strToSearch);

// loop over the matcher

assertThat(hexNums)
  .containsExactly("aF851B", "-3f6C", "-2Ad9eE", "70ae19"); 
```

## 6.结论

在本文中，我们首先讨论了如何使用正则表达式和 Google Guava 的`CharMatcher`类计算字符串中的位数。

然后，我们探索了使用正则表达式来查找整数和小数。

最后，我们讨论了寻找其他格式的数字，比如科学记数法和十六进制。

一如既往，本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex)