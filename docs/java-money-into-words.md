# 用文字显示金额

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-money-into-words>

## 1。概述

在本教程中，我们将看到如何在 Java 中将货币金额转换成单词表示。

我们还将通过一个外部库—[Tradukisto](https://web.archive.org/web/20221126234556/https://github.com/allegro/tradukisto)来看看定制实现是什么样子的。

## 2。实施

先从我们自己的实现说起。**第一步是用以下元素声明两个`String`数组**:

```java
public static String[] ones = { 
  "", "one", "two", "three", "four", 
  "five", "six", "seven", "eight", 
  "nine", "ten", "eleven", "twelve", 
  "thirteen", "fourteen", "fifteen", 
  "sixteen", "seventeen", "eighteen", 
  "nineteen" 
};

public static String[] tens = {
  "",          // 0
  "",          // 1
  "twenty",    // 2
  "thirty",    // 3
  "forty",     // 4
  "fifty",     // 5
  "sixty",     // 6
  "seventy",   // 7
  "eighty",    // 8
  "ninety"     // 9
};
```

当我们收到一个输入时，我们需要处理无效值(零和负值)。**一旦收到有效的输入，我们就可以将美元和美分的数量提取到变量中:**

```java
 long dollars = (long) money;
 long cents = Math.round((money - dollars) * 100);
```

如果给定的数字小于 20，那么我们将根据索引从数组中获取适当的`ones'` 元素:

```java
if (n < 20) {
    return ones[(int) n];
}
```

我们将对小于 100 的数字使用类似的方法，但是现在我们也必须使用`tens` 数组:

```java
if (n < 100) {
    return tens[(int) n / 10] 
      + ((n % 10 != 0) ? " " : "") 
      + ones[(int) n % 10];
}
```

对于小于 1000 的数字，我们也是这样做的。

接下来，我们使用递归调用来处理小于一百万的数字，如下所示:

```java
if (n < 1_000_000) {
    return convert(n / 1000) + " thousand" + ((n % 1000 != 0) ? " " : "") 
      + convert(n % 1000);
}
```

同样的方法也用于小于 10 亿的数字，以此类推。

下面是可以调用来完成此转换的主要方法:

```java
 public static String getMoneyIntoWords(double money) {
    long dollars = (long) money;
    long cents = Math.round((money - dollars) * 100);
    if (money == 0D) {
        return "";
    }
    if (money < 0) {
        return INVALID_INPUT_GIVEN;
    }
    String dollarsPart = "";
    if (dollars > 0) {
        dollarsPart = convert(dollars) 
          + " dollar" 
          + (dollars == 1 ? "" : "s");
    }
    String centsPart = "";
    if (cents > 0) {
        if (dollarParts.length() > 0) {
            centsPart = " and ";
        }
        centsPart += convert(cents) + " cent" + (cents == 1 ? "" : "s");
    }
    return dollarsPart + centsPart;
}
```

让我们测试我们的代码，以确保它正常工作:

```java
@Test
public void whenGivenDollarsAndCents_thenReturnWords() {
    String expectedResult
     = "nine hundred twenty four dollars and sixty cents";

    assertEquals(
      expectedResult, 
      NumberWordConverter.getMoneyIntoWords(924.6));
}

@Test
public void whenTwoBillionDollarsGiven_thenReturnWords() {
    String expectedResult 
      = "two billion one hundred thirty three million two hundred" 
        + " forty seven thousand eight hundred ten dollars";

    assertEquals(
      expectedResult, 
      NumberWordConverter.getMoneyIntoWords(2_133_247_810));
}

@Test
public void whenThirtyMillionDollarsGiven_thenReturnWords() {
    String expectedResult 
      = "thirty three million three hundred forty eight thousand nine hundred seventy eight dollars";
    assertEquals(
      expectedResult, 
      NumberWordConverter.getMoneyIntoWords(33_348_978));
}
```

让我们也测试一些边缘情况，并确保我们也涵盖了它们:

```java
@Test
public void whenZeroDollarsGiven_thenReturnEmptyString() {
    assertEquals("", NumberWordConverter.getMoneyIntoWords(0));
}

@Test
public void whenNoDollarsAndNineFiveNineCents_thenCorrectRounding() {
    assertEquals(   
      "ninety six cents", 
      NumberWordConverter.getMoneyIntoWords(0.959));
}

@Test
public void whenNoDollarsAndOneCent_thenReturnCentSingular() {
    assertEquals(
      "one cent", 
      NumberWordConverter.getMoneyIntoWords(0.01));
} 
```

## 3。使用库

现在我们已经实现了自己的算法，让我们通过使用现有的库来进行这种转换。

[`Tradukisto`](https://web.archive.org/web/20221126234556/https://github.com/allegro/tradukisto) 是 Java 8+的一个库，可以帮助我们将数字转换成它们的文字表示。首先，我们需要将它导入到我们的项目中(这个库的最新版本可以在[这里](https://web.archive.org/web/20221126234556/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22pl.allegro.finance%22%20AND%20a%3A%22tradukisto%22)找到):

```java
<dependency>
    <groupId>pl.allegro.finance</groupId>
    <artifactId>tradukisto</artifactId>
    <version>1.0.1</version>
</dependency>
```

我们现在可以使用`MoneyConverters`的`asWords()`方法来完成这个转换:

```java
public String getMoneyIntoWords(String input) {
    MoneyConverters converter = MoneyConverters.ENGLISH_BANKING_MONEY_VALUE;
    return converter.asWords(new BigDecimal(input));
}
```

让我们用一个简单的测试用例来测试这个方法:

```java
@Test
public void whenGivenDollarsAndCents_thenReturnWordsVersionTwo() {
    assertEquals(
      "three hundred ten £ 00/100", 
      NumberWordConverter.getMoneyIntoWords("310"));
}
```

我们也可以使用 [ICU4J](https://web.archive.org/web/20221126234556/http://site.icu-project.org/home) 库来实现这一点，但是这个库很大，并且附带了许多其他特性，这超出了本文的范围。

但是，如果需要 Unicode 和全球化支持，请查看一下。

## 4。结论

在这篇简短的文章中，我们看到了两种将一笔钱转换成文字的方法。

这里解释的所有例子的代码，以及更多可以在 GitHub 上找到的[。](https://web.archive.org/web/20221126234556/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-2)