# 在 Java 中将逗号分隔的字符串转换为列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-with-separator-to-list>

## 1.概观

在本教程中，我们将看看如何将逗号分隔的`String`转换成字符串的`List`。此外，我们将把整数的逗号分隔的`String`转换成`Integers`的`List`。

## 2.属国

我们将用于转换的一些方法需要 [Apache Commons Lang 3](https://web.archive.org/web/20221117064052/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3) 和 [Guava](https://web.archive.org/web/20221117064052/https://search.maven.org/search?q=a:guava%20AND%20g:com.google.guava) 库。因此，让我们将它们添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

## 3.定义我们的例子

在我们开始之前，让我们定义两个输入字符串，我们将在我们的例子中使用。第一个字符串`countries,`包含由逗号分隔的几个字符串，第二个字符串`ranks,`包含由逗号分隔的数字:

```java
String countries = "Russia,Germany,England,France,Italy";
String ranks = "1,2,3,4,5,6,7";
```

在本教程中，我们将把上述字符串转换成字符串和整数的列表，并存储在:

```java
List<String> convertedCountriesList;
List<Integer> convertedRankList;
```

最后，在我们执行转换后，预期输出将是:

```java
List<String> expectedCountriesList = Arrays.asList("Russia", "Germany", "England", "France", "Italy");
List<Integer> expectedRanksList = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
```

## 4.核心 Java

在我们的第一个解决方案中，我们将使用核心 Java 把一个字符串转换成一个字符串和整数的列表。

首先，我们将使用`split, `一个`String`类实用方法[把我们的字符串分割成一个字符串数组](/web/20221117064052/https://www.baeldung.com/java-split-string)。然后，我们将在新的字符串数组上使用`Arrays.asList`,将其转换成一个字符串列表:

```java
List<String> convertedCountriesList = Arrays.asList(countries.split(",", -1));
```

现在让我们把数字串转换成一个整数列表。

我们将使用`split`方法将数字字符串转换成字符串数组。然后，我们将把新数组中的每个字符串转换成一个整数，并将其添加到我们的列表`:`

```java
String[] convertedRankArray = ranks.split(",");
List<Integer> convertedRankList = new ArrayList<Integer>();
for (String number : convertedRankArray) {
    convertedRankList.add(Integer.parseInt(number.trim()));
}
```

在这两种情况下，我们使用来自`String`类的 [`split`实用程序方法将逗号分隔的字符串拆分成一个字符串数组。](/web/20221117064052/https://www.baeldung.com/string/split)

注意，用于转换我们的`countries`字符串的[重载`split`方法](/web/20221117064052/https://www.baeldung.com/string/split)包含第二个参数`limit`，我们为其提供的值为-1。这指定应该尽可能多地应用分隔符模式。

**我们用来分割整数字符串的`split`方法(`ranks`)使用零作为`limit`，因此它忽略了空字符串，而用于`countries`字符串的`split`在返回的数组中保留了空字符串。**

## 5.Java 流

现在，我们将使用 [Java Stream API](/web/20221117064052/https://www.baeldung.com/java-streams) 实现相同的转换。

首先，我们将使用`String`类中的 [`split`方法将我们的`countries`字符串转换成一个字符串数组。然后，我们将使用`Stream`类将数组转换成字符串列表:](/web/20221117064052/https://www.baeldung.com/string/split)

```java
List<String> convertedCountriesList = Stream.of(countries.split(",", -1))
  .collect(Collectors.toList());
```

让我们看看如何使用`Stream.`将我们的数字串转换成一个整数列表

同样，我们将首先使用`split` 方法将数字串转换为字符串数组，并使用`Stream`类中的`of()`方法将结果数组转换为`String`的`Stream`。

然后，我们将使用`map(String:`:`trim)`**来修剪`Stream`上每个`String`的前导和尾随空格。**

接下来，我们将在流中应用`map(Integer::parseInt)`，将`Stream`中的每个字符串转换为`Integer`。

最后，我们将调用`Stream`上的`collect(Collectors.toList())`,将其转换为一个整数列表:

```java
List<Integer> convertedRankList = Stream.of(ranks.split(","))
  .map(String::trim)
  .map(Integer::parseInt)
  .collect(Collectors.toList());
```

## 6.Apache Commons Lang

在这个解决方案中，我们将使用 [Apache Commons Lang3](/web/20221117064052/https://www.baeldung.com/java-commons-lang-3) 库来执行转换。Apache Commons Lang3 提供了几个助手函数来操作核心 Java 类。

首先，我们将使用`StringUtils.splitPreserveAllTokens` `.`把我们的字符串分割成一个字符串数组，然后，我们将使用`Arrays.asList` 方法把我们的新字符串数组转换成一个列表:

```java
List<String> convertedCountriesList = Arrays.asList(StringUtils.splitPreserveAllTokens(countries, ","));
```

现在让我们把数字串转换成一个整数列表。

我们将再次使用`StringUtils.split`方法从我们的字符串创建一个字符串数组。然后，我们将使用`Integer.parseInt `将新数组中的每个字符串转换成一个整数，并将转换后的整数添加到我们的列表中:

```java
String[] convertedRankArray = StringUtils.split(ranks, ",");
List<Integer> convertedRankList = new ArrayList<Integer>();
for (String number : convertedRankArray) {
    convertedRankList.add(Integer.parseInt(number.trim()));
} 
```

在这个例子中，我们使用了`splitPreserveAllTokens`方法来分割我们的`countries `字符串，而我们使用了`split`方法来分割我们的`ranks`字符串。

尽管这两个函数都将字符串拆分成一个数组，但是 **`splitPreserveAllTokens`保留了所有标记，包括由相邻分隔符创建的空字符串，而`split`方法忽略了空字符串**。

因此，如果我们有空的字符串，我们希望它们包含在我们的列表中，那么我们应该使用`splitPreserveAllTokens`而不是`split`。

## 7.番石榴

最后，我们将使用 [Guava](/web/20221117064052/https://www.baeldung.com/category/guava/) 库将我们的字符串转换成它们适当的列表。

为了转换我们的`countries `字符串，我们将首先调用`Splitter.on` ,用一个逗号作为参数来指定我们的字符串应该在哪个字符上分割。

然后，我们将在我们的`Splitter`实例上使用`trimResults `方法。这将忽略创建的子字符串中的所有前导和尾随空格。

最后，我们将使用`splitToList `方法来分割我们的输入字符串，并将其转换为一个列表:

```java
List<String> convertedCountriesList = Splitter.on(",")
  .trimResults()
  .splitToList(countries);
```

现在，让我们将数字串转换成整数列表`.`

我们将再次使用与上面相同的过程将数字串转换成字符串列表。

然后，我们再用 **`Lists`** 。 **`transform`方法，它接受我们的字符串列表作为第一个参数，接受`Function`接口的实现作为第二个参数**。

`Function`接口实现将我们列表中的每个字符串转换成一个整数:

```java
List<Integer> convertedRankList = Lists.transform(Splitter.on(",")
  .trimResults()
  .splitToList(ranks), new Function<String, Integer>() {
      @Override
      public Integer apply(String input) {
          return Integer.parseInt(input.trim());
      }
  }); 
```

## 8.结论

在本文中，我们将逗号分隔的`Strings`转换成一个字符串列表和一个整数列表。然而，我们可以按照类似的过程将一个`String`转换成任何原始数据类型的列表。

和往常一样，这篇文章的代码可以在 Github 的[上找到。](https://web.archive.org/web/20221117064052/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)