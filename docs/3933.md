# 大学解析器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-univocity-parsers>

## 1.介绍

在本教程中，我们将快速浏览一下 [Univocity 解析器](https://web.archive.org/web/20220626090344/https://www.univocity.com/pages/univocity_parsers_tutorial.html)，这是一个用于在 Java 中解析 CSV、TSV 和固定宽度文件的库。

我们将从读写文件的基础开始，然后继续从 Java beans 读写文件。然后，在结束之前，我们将快速浏览一下配置选项。

## 2.设置

为了使用解析器，我们需要将最新的 [Maven 依赖项](https://web.archive.org/web/20220626090344/https://search.maven.org/artifact/com.univocity/univocity-parsers)添加到我们的项目`pom.xml`文件中:

```
<dependency>
    <groupId>com.univocity</groupId>
    <artifactId>univocity-parsers</artifactId>
    <version>2.8.4</version>
</dependency>
```

## 3.基本用法

### 3.1.阅读

在 Univocity 中，我们可以快速地将整个文件解析成一组代表文件中每一行的`String`数组。

首先，让我们通过将 CSV 文件的`Reader`提供给默认设置的`CsvParser`来解析 CSV 文件:

```
try (Reader inputReader = new InputStreamReader(new FileInputStream(
  new File("src/test/resources/productList.csv")), "UTF-8")) {
    CsvParser parser = new CsvParser(new CsvParserSettings());
    List<String[]> parsedRows = parser.parseAll(inputReader);
    return parsedRows;
} catch (IOException e) {
    // handle exception
}
```

我们可以很容易地切换这个逻辑，通过切换到`TsvParser`并为它提供一个 TSV 文件来解析一个 TSV 文件。

处理固定宽度的文件只是稍微复杂一点。主要的区别是我们需要在解析器设置中提供字段宽度。

让我们通过向我们的`FixedWidthParserSettings`提供一个`FixedWidthFields`对象来读取一个固定宽度的文件:

```
try (Reader inputReader = new InputStreamReader(new FileInputStream(
  new File("src/test/resources/productList.txt")), "UTF-8")) {
    FixedWidthFields fieldLengths = new FixedWidthFields(8, 30, 10);
    FixedWidthParserSettings settings = new FixedWidthParserSettings(fieldLengths);

    FixedWidthParser parser = new FixedWidthParser(settings);
    List<String[]> parsedRows = parser.parseAll(inputReader);
    return parsedRows;
} catch (IOException e) {
    // handle exception
}
```

### 3.2.写作

既然我们已经介绍了使用解析器读取文件，那么让我们来学习如何编写它们。

编写文件与读取文件非常相似，因为我们向解析器提供了一个`Writer`以及与我们的文件类型相匹配的设置。

让我们创建一个方法来编写所有三种可能格式的文件:

```
public boolean writeData(List<Object[]> products, OutputType outputType, String outputPath) {
    try (Writer outputWriter = new OutputStreamWriter(new FileOutputStream(new File(outputPath)),"UTF-8")){
        switch(outputType) {
            case CSV:
                CsvWriter writer = new CsvWriter(outputWriter, new CsvWriterSettings());
                writer.writeRowsAndClose(products);
                break;
            case TSV:
                TsvWriter writer = new TsvWriter(outputWriter, new TsvWriterSettings());
                writer.writeRowsAndClose(products);
                break;
            case FIXED_WIDTH:
                FixedWidthFields fieldLengths = new FixedWidthFields(8, 30, 10);
                FixedWidthWriterSettings settings = new FixedWidthWriterSettings(fieldLengths);
                FixedWidthWriter writer = new FixedWidthWriter(outputWriter, settings);
                writer.writeRowsAndClose(products);
                break;
            default:
                logger.warn("Invalid OutputType: " + outputType);
                return false;
        }
        return true;
    } catch (IOException e) {
        // handle exception
    }
}
```

与读取文件一样，编写 CSV 文件和 TSV 文件几乎是相同的。对于固定宽度的文件，我们必须为我们的设置提供字段宽度。

### 3.3.使用行处理器

Univocity 提供了许多我们可以使用的行处理器，并且还提供了我们自己创建行处理器的能力。

为了对使用行处理器有所了解，让我们使用`BatchedColumnProcessor`来处理一个较大的 CSV 文件，每批 5 行:

```
try (Reader inputReader = new InputStreamReader(new FileInputStream(new File(relativePath)), "UTF-8")) {
    CsvParserSettings settings = new CsvParserSettings();
    settings.setProcessor(new BatchedColumnProcessor(5) {
        @Override
        public void batchProcessed(int rowsInThisBatch) {}
    });
    CsvParser parser = new CsvParser(settings);
    List<String[]> parsedRows = parser.parseAll(inputReader);
    return parsedRows;
} catch (IOException e) {
    // handle exception
}
```

为了使用这个行处理器，我们在我们的`CsvParserSettings`中定义它，然后我们所要做的就是调用`parseAll`。

### 3.4.读取和写入 Java Beans

`String`数组列表没问题，但是我们经常在 Java beans 中处理数据。 **Univocity 还允许读写特别注释的 Java beans。**

让我们用 Univocity 注释定义一个`Product` bean:

```
public class Product {

    @Parsed(field = "product_no")
    private String productNumber;

    @Parsed
    private String description;

    @Parsed(field = "unit_price")
    private float unitPrice;

    // getters and setters
}
```

**主标注是`@Parsed`标注。**

如果我们的列标题与字段名匹配，我们可以使用`@Parsed`而不指定任何值。**如果我们的列标题不同于字段名称，我们可以使用`field`属性指定列标题。**

现在我们已经定义了我们的`Product` bean，让我们将 CSV 文件读入其中:

```
try (Reader inputReader = new InputStreamReader(new FileInputStream(
  new File("src/test/resources/productList.csv")), "UTF-8")) {
    BeanListProcessor<Product> rowProcessor = new BeanListProcessor<Product>(Product.class);
    CsvParserSettings settings = new CsvParserSettings();
    settings.setHeaderExtractionEnabled(true);
    settings.setProcessor(rowProcessor);
    CsvParser parser = new CsvParser(settings);
    parser.parse(inputReader);
    return rowProcessor.getBeans();
} catch (IOException e) {
    // handle exception
}
```

我们首先用带注释的类构造了一个特殊的行处理器`BeanListProcessor,`。然后，我们将它提供给`CsvParserSettings`，并用它读入一个`Product`列表

接下来，让我们把我们的`Product`列表写到一个固定宽度的文件中:

```
try (Writer outputWriter = new OutputStreamWriter(new FileOutputStream(new File(outputPath)), "UTF-8")) {
    BeanWriterProcessor<Product> rowProcessor = new BeanWriterProcessor<Product>(Product.class);
    FixedWidthFields fieldLengths = new FixedWidthFields(8, 30, 10);
    FixedWidthWriterSettings settings = new FixedWidthWriterSettings(fieldLengths);
    settings.setHeaders("product_no", "description", "unit_price");
    settings.setRowWriterProcessor(rowProcessor);
    FixedWidthWriter writer = new FixedWidthWriter(outputWriter, settings);
    writer.writeHeaders();
    for (Product product : products) {
        writer.processRecord(product);
    }
    writer.close();
    return true;
} catch (IOException e) {
    // handle exception
}
```

显著的区别是我们在设置中指定了列标题。

## 4.设置

Univocity 有许多我们可以应用于解析器的设置。正如我们前面看到的，我们可以使用设置将行处理器应用于解析器。

还有许多其他设置可以根据我们的需要进行更改。尽管这三种文件类型的许多配置是相同的，但是每个解析器也有特定于格式的设置。

让我们调整 CSV 解析器设置，对我们正在读取的数据进行一些限制:

```
CsvParserSettings settings = new CsvParserSettings();
settings.setMaxCharsPerColumn(100);
settings.setMaxColumns(50);
CsvParser parser = new CsvParser(new CsvParserSettings());
```

## 5.结论

在这个快速教程中，我们学习了使用 Univocity 库解析文件的基础知识。

我们学习了如何在字符串数组列表和 Java beans 中读写文件。在学习 Java beans 之前，我们快速看了一下使用不同的行处理器。最后，我们简要介绍了如何定制设置。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220626090344/https://github.com/eugenp/tutorials/tree/master/libraries-data-2)