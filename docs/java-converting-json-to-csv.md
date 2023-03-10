# 用 Java 将 JSON 转换成 CSV

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-converting-json-to-csv>

## 1。简介

在这个简短的教程中，我们将看到如何使用 Jackson 将 JSON 转换成 CSV，反之亦然。

还有其他可用的库，比如来自 org.json 的 [CDL 类，但是这里我们只关注 Jackson 库。](/web/20220926191006/https://www.baeldung.com/java-org-json#cdl)

在我们查看了示例数据结构之后，我们将使用 [`ObjectMapper`](/web/20220926191006/https://www.baeldung.com/jackson-object-mapper-tutorial) 和 CSVMapper 的组合在 JSON 和 CSV 之间进行转换。

## 2。依赖性

让我们为 Jackson CSV 数据格式化程序添加依赖项:

```java
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-csv</artifactId>
    <version>2.13.0</version>
</dependency>
```

我们总能在 [Maven Central](https://web.archive.org/web/20220926191006/https://search.maven.org/search?q=g:com.fasterxml.jackson.dataformat%20AND%20a:jackson-dataformat-csv&core=gav) 上找到这种依赖的最新版本。

我们还将添加核心 Jackson 数据绑定的依赖项:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

同样，我们可以在 [Maven Central](https://web.archive.org/web/20220926191006/https://search.maven.org/search?q=g:com.fasterxml.jackson.core%20AND%20a:jackson-databind&core=gav) 上找到这种依赖的最新版本。

## 3。数据结构

在我们将 JSON 文档重新格式化为 CSV 之前，我们需要考虑我们的数据模型在两种格式之间的映射有多好。

首先，让我们考虑不同格式支持哪些数据:

*   我们使用 JSON 来表示各种对象结构，包括包含数组和嵌套对象的对象结构
*   我们使用 CSV 来表示来自对象列表的数据，列表中的每个对象出现在一个新行上

这意味着，如果我们的 JSON 文档有一个对象数组，我们可以将每个对象重新格式化为 CSV 文件的一个新行。因此，作为一个例子，让我们使用一个 JSON 文档，其中包含订单中的以下商品列表:

```java
[ {
  "item" : "No. 9 Sprockets",
  "quantity" : 12,
  "unitPrice" : 1.23
}, {
  "item" : "Widget (10mm)",
  "quantity" : 4,
  "unitPrice" : 3.45
} ]
```

我们将使用 JSON 文档中的字段名作为列标题，并将其重新格式化为下面的 CSV 文件:

```java
item,quantity,unitPrice
"No. 9 Sprockets",12,1.23
"Widget (10mm)",4,3.45
```

## 4。读取 JSON 并写入 CSV

首先，我们使用 Jackson 的`ObjectMapper`将示例 JSON 文档读入一个由`JsonNode`对象组成的树:

```java
JsonNode jsonTree = new ObjectMapper().readTree(new File("src/main/resources/orderLines.json"));
```

接下来，让我们创建一个`CsvSchema`。这决定了 CSV 文件中的列标题、类型和列顺序。为此，我们创建一个`CsvSchema Builder`并设置列标题以匹配 JSON 字段名称:

```java
Builder csvSchemaBuilder = CsvSchema.builder();
JsonNode firstObject = jsonTree.elements().next();
firstObject.fieldNames().forEachRemaining(fieldName -> {csvSchemaBuilder.addColumn(fieldName);} );
CsvSchema csvSchema = csvSchemaBuilder.build().withHeader();
```

**然后，我们用我们的`CsvSchema`创建一个`CsvMapper`，最后，我们将`jsonTree`写入我们的 CSV 文件**:

```java
CsvMapper csvMapper = new CsvMapper();
csvMapper.writerFor(JsonNode.class)
  .with(csvSchema)
  .writeValue(new File("src/main/resources/orderLines.csv"), jsonTree);
```

当我们运行这个示例代码时，我们的示例 JSON 文档被转换成预期的 CSV 文件。

## 5。读取 CSV 并写入 JSON

现在，让我们使用 Jackson 的`CsvMapper`将我们的 CSV 文件读入到`OrderLine`对象的`List`中。为此，我们首先创建一个简单的 POJO 形式的`OrderLine`类:

```java
public class OrderLine {
    private String item;
    private int quantity;
    private BigDecimal unitPrice;

    // Constructors, Getters, Setters and toString
}
```

我们将使用 CSV 文件中的列标题来定义我们的`CsvSchema`。**然后，** **我们使用`CsvMapper`将数据从 CSV** 读取到`OrderLine`对象的`MappingIterator`中:

```java
CsvSchema orderLineSchema = CsvSchema.emptySchema().withHeader();
CsvMapper csvMapper = new CsvMapper();
MappingIterator<OrderLine> orderLines = csvMapper.readerFor(OrderLine.class)
  .with(orderLineSchema)
  .readValues(new File("src/main/resources/orderLines.csv"));
```

接下来，我们将使用`MappingIterator`来获得`OrderLine`对象的`List`。然后，我们使用 Jackson 的`ObjectMapper`将列表写成一个 JSON 文档:

```java
new ObjectMapper()
  .configure(SerializationFeature.INDENT_OUTPUT, true)
  .writeValue(new File("src/main/resources/orderLinesFromCsv.json"), orderLines.readAll());
```

当我们运行这个示例代码时，我们的示例 CSV 文件被转换成预期的 JSON 文档。

## 6。配置 CSV 文件格式

让我们使用 Jackson 的一些注释来调整 CSV 文件的格式。我们将把`‘item'`列标题改为`‘name'`，把`‘quantity'`列标题改为`‘count'`，去掉`‘unitPrice'`列，把`‘count'`列作为第一列。

因此，我们预期的 CSV 文件变成了:

```java
count,name
12,"No. 9 Sprockets"
4,"Widget (10mm)"
```

我们将创建一个新的抽象类来定义 CSV 文件所需的格式:

```java
@JsonPropertyOrder({
    "count",
    "name"
})
public abstract class OrderLineForCsv {

    @JsonProperty("name")
    private String item; 

    @JsonProperty("count")
    private int quantity; 

    @JsonIgnore
    private BigDecimal unitPrice;

}
```

然后，我们使用我们的`OrderLineForCsv`类创建一个`CsvSchema`:

```java
CsvMapper csvMapper = new CsvMapper();
CsvSchema csvSchema = csvMapper
  .schemaFor(OrderLineForCsv.class)
  .withHeader(); 
```

我们也使用`OrderLineForCsv`作为杰克逊混音。这告诉 Jackson 在处理一个`OrderLine`对象时使用我们添加到`OrderLineForCsv`类的注释:

```java
csvMapper.addMixIn(OrderLine.class, OrderLineForCsv.class); 
```

最后，我们使用一个`ObjectMapper`将我们的 JSON 文档读入一个`OrderLine`数组，并使用我们的`csvMapper`将它写入一个 CSV 文件:

```java
OrderLine[] orderLines = new ObjectMapper()
    .readValue(new File("src/main/resources/orderLines.json"), OrderLine[].class);

csvMapper.writerFor(OrderLine[].class)
    .with(csvSchema)
    .writeValue(new File("src/main/resources/orderLinesReformated.csv"), orderLines); 
```

当我们运行这个示例代码时，我们的示例 JSON 文档被转换成预期的 CSV 文件。

## 7。结论

在这个快速教程中，我们学习了如何使用 Jackson 数据格式库读写 CSV 文件。我们还研究了一些配置选项，这些选项有助于我们获得我们想要的数据。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926191006/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)