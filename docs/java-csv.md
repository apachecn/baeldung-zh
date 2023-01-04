# 如何用 Java 写 CSV 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-csv>

## 1.概观

在这个快速教程中，我们将学习如何使用 Java 编写 CSV 文件。CSV 代表逗号分隔值，是系统间批量数据传输的常用格式。

为了编写我们的 CSV 文件，我们将使用`java.io`包中的类。

我们将讨论特殊字符以及如何处理它们。我们将把我们的输出文件**设定为在微软 Excel 和谷歌工作表中打开。**

在我们的 Java 示例之后，我们将简要地看一下**一些可用于处理 CSV 文件的第三方库。**

## 2.用`PrintWriter`书写

我们将使用一个`PrintWriter`来编写我们的 CSV 文件。要更详细地了解如何使用`java.io`写入文件，请参阅我们的[关于写入文件](/web/20221206132944/https://www.baeldung.com/java-write-to-file)的文章。

### 2.1.撰写 CSV

首先，让我们创建一个方法来格式化表示为一个数组`String`的单行数据:

```java
public String convertToCSV(String[] data) {
    return Stream.of(data)
      .map(this::escapeSpecialCharacters)
      .collect(Collectors.joining(","));
}
```

在我们调用这个方法之前，让我们构建一些示例数据:

```java
List<String[]> dataLines = new ArrayList<>();
dataLines.add(new String[] 
  { "John", "Doe", "38", "Comment Data\nAnother line of comment data" });
dataLines.add(new String[] 
  { "Jane", "Doe, Jr.", "19", "She said \"I'm being quoted\"" });
```

有了这些数据，让我们用`convertToCSV,`转换每一行，并将其写入一个文件:

```java
public void givenDataArray_whenConvertToCSV_thenOutputCreated() throws IOException {
    File csvOutputFile = new File(CSV_FILE_NAME);
    try (PrintWriter pw = new PrintWriter(csvOutputFile)) {
        dataLines.stream()
          .map(this::convertToCSV)
          .forEach(pw::println);
    }
    assertTrue(csvOutputFile.exists());
}
```

### 2.2.处理特殊字符

在 CSV 文件中，某些字符是有问题的，作为开发人员，我们很少能完全控制我们数据的质量。现在我们来看看如何处理特殊字符。

对于我们的例子，我们将关注逗号、引号和换行符。**包含逗号或引号的字段会用双引号括起来，双引号会用双引号转义。**我们将删除新行，并用空白替换它们。

有问题的字符以及如何处理它们可能会因用例而异。

我们的`convertToCSV`方法在构建`String.`时对每个数据块调用`escapeSpecialCharacters`方法

现在让我们实现我们的`escapeSpecialCharacters`方法:

```java
public String escapeSpecialCharacters(String data) {
    String escapedData = data.replaceAll("\\R", " ");
    if (data.contains(",") || data.contains("\"") || data.contains("'")) {
        data = data.replace("\"", "\"\"");
        escapedData = "\"" + data + "\"";
    }
    return escapedData;
}
```

## 3.第三方库

正如我们在示例中看到的，当我们开始考虑特殊字符以及如何处理它们时，编写 CSV 文件会变得复杂。

幸运的是，**有许多第三方库可用于处理 CSV 文件，**其中许多库处理这些特殊字符和其他可能出现的异常情况。

让我们来看看其中的几个:

*   Apache Commons CSV : Apache 的 CSV 产品，用于处理 CSV 文件
*   Open CSV :另一个流行且被积极维护的 CSV 库
*   Flatpack :一个正在积极开发的开源 CSV 库
*   [CSVeed](https://web.archive.org/web/20221206132944/http://csveed.org/) :开源，主动维护

## 4.结论

在这篇简短的文章中，我们讨论了如何使用 Java 的`PrintWriter`类编写 CSV 文件。接下来，我们讨论并处理了输出数据中的特殊字符。

在简单的 Java 示例之后，我们查看了可用的第三方库的概述。

GitHub 上有示例代码[。](https://web.archive.org/web/20221206132944/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions-2)