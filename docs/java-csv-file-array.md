# 将 CSV 文件读入数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-csv-file-array>

## 1.概观

简单地说，CSV(逗号分隔值)文件包含由逗号分隔符分隔的有组织的信息。

在本教程中，我们将研究将 CSV 文件读入数组的不同方法。

## 2.`java.io`中的`BufferedReader`

首先，我们将使用`BufferedReader`中的`readLine()`逐行读取记录。

然后，我们将根据逗号分隔符将该行拆分成标记:

```java
List<List<String>> records = new ArrayList<>();
try (BufferedReader br = new BufferedReader(new FileReader("book.csv"))) {
    String line;
    while ((line = br.readLine()) != null) {
        String[] values = line.split(COMMA_DELIMITER);
        records.add(Arrays.asList(values));
    }
}
```

**请注意，更复杂的 CSV(例如，引用或包含逗号作为值)将不会按照此方法的意图进行解析。**

## 3.`java.util`中的`Scanner`

接下来，我们将使用一个`java.util.Scanner`遍历文件的内容，并一行接一行地检索行:

```java
List<List<String>> records = new ArrayList<>();
try (Scanner scanner = new Scanner(new File("book.csv"));) {
    while (scanner.hasNextLine()) {
        records.add(getRecordFromLine(scanner.nextLine()));
    }
}
```

然后，我们将解析这些行，并将其存储到一个数组中:

```java
private List<String> getRecordFromLine(String line) {
    List<String> values = new ArrayList<String>();
    try (Scanner rowScanner = new Scanner(line)) {
        rowScanner.useDelimiter(COMMA_DELIMITER);
        while (rowScanner.hasNext()) {
            values.add(rowScanner.next());
        }
    }
    return values;
}
```

像以前一样，更复杂的 CSV 将不会按照这种方法的意图进行解析。

## 4.OpenCSV

我们可以用 OpenCSV 处理更复杂的 CSV 文件。

OpenCSV 是一个第三方库，它提供了一个处理 CSV 文件的 API。

我们将使用`CSVReader`中的`readNext()`方法来读取文件中的记录:

```java
List<List<String>> records = new ArrayList<List<String>>();
try (CSVReader csvReader = new CSVReader(new FileReader("book.csv"));) {
    String[] values = null;
    while ((values = csvReader.readNext()) != null) {
        records.add(Arrays.asList(values));
    }
}
```

要深入挖掘并了解更多关于 OpenCSV 的信息，请查看我们的 [OpenCSV 教程](/web/20221008004712/https://www.baeldung.com/opencsv)。

## 5.结论

在这篇简短的文章中，我们探讨了将 CSV 文件读入数组的不同方法。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221008004712/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions-2)