# Java 扫描仪使用示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-scanner-usedelimiter>

## 1.概观

在本教程中，我们将看到如何使用 [`Scanner`](/web/20220627154627/https://www.baeldung.com/java-scanner) 类的`useDelimiter`方法。

## 2.`java.util.Scanner`简介

[`Scanner`](/web/20220627154627/https://www.baeldung.com/java-scanner) API 提供了一个简单的文本扫描器。

**默认情况下，`Scanner`使用空格作为分隔符将其输入分割成标记。**让我们写一个函数，它将:

*   将输入传递给 a `Scanner`
*   遍历`Scanner`来收集列表中的令牌

让我们来看看基本实现:

```java
public static List<String> baseScanner(String input) {
    try (Scanner scan = new Scanner(input)) {
        List<String> result = new ArrayList<String>();
        scan.forEachRemaining(result::add);
        return result;
    }
}
```

请注意，在这段代码中，我们使用了一个`[try-with-resources](/web/20220627154627/https://www.baeldung.com/java-try-with-resources)` 来创建我们的`Scanner`。这是可能的，因为`Scanner`类实现了`AutoCloseable` 接口。该块负责自动关闭扫描仪资源。在 Java 7 之前，我们不能使用`try-with-resources`，因此必须手动处理它。

我们还可以注意到，为了迭代`Scanner`元素，我们使用了`forEachRemaining`方法。这个方法是在 Java 8 中引入的。Scanner 实现了`[Iterator](/web/20220627154627/https://www.baeldung.com/java-iterator)`，如果我们使用旧版本的 Java，我们必须利用它来遍历元素。

正如我们所说的，`Scanner`将默认使用空格来解析它的输入。例如，用下面的输入调用我们的`baseScanner`方法:“Welcome to Baeldung”，应该返回一个包含以下有序元素的列表:“Welcome”、“to”、“Baeldung”。

让我们编写一个测试来检查我们的方法的行为是否符合预期:

```java
@Test
void whenBaseScanner_ThenWhitespacesAreUsedAsDelimiters() {
    assertEquals(List.of("Welcome", "to", "Baeldung"), baseScanner("Welcome to Baeldung"));
}
```

## 3.使用自定义分隔符

现在让我们设置我们的扫描仪使用自定义分隔符。**我们将传入一个`String` ，它将被`Scanner`用来中断输入。**

让我们看看如何做到这一点:

```java
public static List<String> scannerWithDelimiter(String input, String delimiter) {
    try (Scanner scan = new Scanner(input)) {
        scan.useDelimiter(delimiter); 
        List<String> result = new ArrayList<String>();
        scan.forEachRemaining(result::add);
        return result;
    }
}
```

让我们来评论几个例子:

*   我们可以使用单个字符作为分隔符:如果需要，该字符必须是被[转义的](/web/20220627154627/https://www.baeldung.com/java-regexp-escape-char)。例如，如果我们想要模仿基本行为并使用空格作为分隔符，我们将使用“\ \ s”
*   我们可以使用任何单词/短语作为分隔符
*   我们可以使用多个可能的字符作为分隔符:为此，我们必须用|分隔它们。例如，如果我们想在每个空格和每个换行符之间分割输入，我们将使用以下分隔符:" \n|\\s "
*   简而言之，我们可以使用任何类型的正则表达式作为分隔符:例如，“a+”就是一个有效的分隔符

让我们看看如何测试第一种情况:

```java
@Test
void givenSimpleCharacterDelimiter_whenScannerWithDelimiter_ThenInputIsCorrectlyParsed() {
    assertEquals(List.of("Welcome", "to", "Baeldung"), scannerWithDelimiter("Welcome to Baeldung", "\\s"));
}
```

**实际上，在场景下，`useDelimiter`方法会将其输入转换为封装在`Pattern`对象中的[正则表达式](/web/20220627154627/https://www.baeldung.com/regular-expressions-java)。另外，我们也可以自己处理`Pattern`的实例化。为此，我们需要使用覆盖的`useDelimiter(Pattern pattern)`，如下所示:**

```java
public static List<String> scannerWithDelimiterUsingPattern(String input, Pattern delimiter) {
    try (Scanner scan = new Scanner(input)) {
        scan.useDelimiter(delimiter); 
        List<String> result = new ArrayList<String>();
        scan.forEachRemaining(result::add);
        return result;
    }
}
```

要实例化一个`Pattern`，我们可以使用下面测试中的`compile`方法:

```java
@Test
void givenStringDelimiter_whenScannerWithDelimiterUsingPattern_ThenInputIsCorrectlyParsed() {
    assertEquals(List.of("Welcome", "to", "Baeldung"), DelimiterDemo.scannerWithDelimiterUsingPattern("Welcome to Baeldung", Pattern.compile("\\s")));
}
```

## 4.结论

在本文中，我们展示了几个可以用来调用`useDelimiter`函数的模式示例。我们注意到默认情况下，`Scanner`使用空白分隔符，我们指出我们可以在这里使用任何类型的正则表达式。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627154627/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9)