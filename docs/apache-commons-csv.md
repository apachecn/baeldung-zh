# Apache Commons CSV 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-csv>

## 1。概述

[Apache Commons CSV 库](https://web.archive.org/web/20221021075025/https://commons.apache.org/proper/commons-csv/)有许多创建和读取 CSV 文件的有用特性。

在这个快速教程中，我们将通过展示一个简单的例子来了解如何利用这个库。

## 2。Maven 依赖关系

首先，我们将使用 Maven 导入这个库的最新版本:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-csv</artifactId>
    <version>1.4</version>
</dependency> 
```

要查看本库的最新版本，请点击这里。

## 3。读取 CSV 文件

考虑以下名为 book.csv 的 CSV 文件，该文件包含书籍的属性:

```
author,title
Dan Simmons,Hyperion
Douglas Adams,The Hitchhiker's Guide to the Galaxy
```

让我们看看如何阅读它:

```
Map<String, String> AUTHOR_BOOK_MAP = new HashMap<>() {
    {
        put("Dan Simmons", "Hyperion");
        put("Douglas Adams", "The Hitchhiker's Guide to the Galaxy");
    }
});
String[] HEADERS = { "author", "title"};

@Test
public void givenCSVFile_whenRead_thenContentsAsExpected() throws IOException {
    Reader in = new FileReader("book.csv");
    Iterable<CSVRecord> records = CSVFormat.DEFAULT
      .withHeader(HEADERS)
      .withFirstRecordAsHeader()
      .parse(in);
    for (CSVRecord record : records) {
        String author = record.get("author");
        String title = record.get("title");
        assertEquals(AUTHOR_BOOK_MAP.get(author), title);
    }
}
```

我们在跳过第一行后读取 CSV 文件的记录，因为它是文件头。

有不同类型的`CSVFormat` 指定 CSV 文件的格式，您可以在下一段中看到一个例子。

## 4。创建 CSV 文件

让我们看看如何创建与上面相同的 CSV 文件:

```
public void createCSVFile() throws IOException {
    FileWriter out = new FileWriter("book_new.csv");
    try (CSVPrinter printer = new CSVPrinter(out, CSVFormat.DEFAULT
      .withHeader(HEADERS))) {
        AUTHOR_BOOK_MAP.forEach((author, title) -> {
            printer.printRecord(author, title);
        });
    }
}
```

新的 CSV 文件将使用适当的头文件创建，因为我们已经在我们的`CSVFormat` 声明中指定了它们。

## 5。标题&阅读栏目

读写头有不同的方法。同样，读取列值也有不同的方法。

让我们一个一个地看一下:

### 5.1。通过索引访问列

这是读取列值的最基本方式。当 CSV 文件的文件头未知时，可以使用这种方法:

```
Reader in = new FileReader("book.csv");
Iterable<CSVRecord> records = CSVFormat.DEFAULT.parse(in);
for (CSVRecord record : records) {
    String columnOne = record.get(0);
    String columnTwo = record.get(1);
}
```

### 5.2。通过预定义标题访问列

与按索引访问相比，这是一种更直观的访问列的方式:

```
Iterable<CSVRecord> records = CSVFormat.DEFAULT
  .withHeader("author", "title").parse(in);
for (CSVRecord record : records) {
    String author = record.get("author");
    String title = record.get("title");
}
```

### 5.3。使用枚举作为标题

使用`Strings`访问列值可能容易出错。使用枚举代替字符串将使代码更加标准化，也更容易理解:

```
enum BookHeaders {
    author, title
}

Iterable<CSVRecord> records = CSVFormat.DEFAULT
  .withHeader(BookHeaders.class).parse(in);
for (CSVRecord record : records) {
    String author = record.get(BookHeaders.author);
    String title = record.get(BookHeaders.title);
}
```

### 5.4。跳过标题行

通常，CSV 文件在第一行包含标题。因此，在大多数情况下，跳过它并从第二行开始读取是安全的。

这将自动检测标题访问列值:

```
Iterable<CSVRecord> records = CSVFormat.DEFAULT
  .withFirstRowAsHeader().parse(in);
for (CSVRecord record : records) {
    String author = record.get("author");
    String title = record.get("title");
}
```

### 5.5。创建带有标题的文件

类似地，我们可以创建一个第一行包含标题的 CSV 文件:

```
FileWriter out = new FileWriter("book_new.csv");
CSVPrinter printer = CSVFormat.DEFAULT
  .withHeader("author", "title").print(out);
```

## 6。结论

我们通过一个简单的例子展示了 Apache 的 Commons CSV 库的使用。你可以在这里阅读更多关于图书馆的信息。

这篇文章的代码可以在 Github 的[上找到。](https://web.archive.org/web/20221021075025/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons-io)