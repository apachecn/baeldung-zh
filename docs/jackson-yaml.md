# 如何处理 YAML 与杰克森的关系

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-yaml>

## 1.介绍

在这个简短的教程中，我们将学习如何使用 [Jackson](/web/20220703140929/https://www.baeldung.com/jackson) 来读写 YAML 文件。

在我们检查了示例结构之后，我们将使用 [`ObjectMapper`](/web/20220703140929/https://www.baeldung.com/jackson-object-mapper-tutorial) 将 YAML 文件读入 Java 对象，并将对象写出到文件中。

## 2.属国

让我们添加杰克逊 YAML 数据格式的依赖性:

```java
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
    <version>2.13.0</version>
</dependency>
```

我们总能在 [Maven Central](https://web.archive.org/web/20220703140929/https://search.maven.org/search?q=g:com.fasterxml.jackson.dataformat%20AND%20a:jackson-dataformat-yaml&core=gav) 上找到这种依赖的最新版本。

我们的 Java 对象使用了一个`LocalDate`，所以让我们也为 JSR-310 数据类型添加一个依赖项:

```java
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.13.0</version>
</dependency>
```

同样，我们可以在 Maven Central 上查看它的最新版本。

## 3.数据和对象结构

解决了依赖关系后，我们现在将转向输入文件和将要使用的 Java 类。

让我们首先来看看我们将要读取的文件:

```java
orderNo: A001
date: 2019-04-17
customerName: Customer, Joe
orderLines:
    - item: No. 9 Sprockets
      quantity: 12
      unitPrice: 1.23
    - item: Widget (10mm)
      quantity: 4
      unitPrice: 3.45
```

然后，让我们定义一下`Order`类:

```java
public class Order {
    private String orderNo;
    private LocalDate date;
    private String customerName;
    private List<OrderLine> orderLines;

    // Constructors, Getters, Setters and toString
}
```

最后，让我们创建我们的`OrderLine`类:

```java
public class OrderLine {
    private String item;
    private int quantity;
    private BigDecimal unitPrice;

    // Constructors, Getters, Setters and toString
}
```

## 4.阅读 YAML

**我们将使用 Jackson 的`ObjectMapper`** 将我们的 YAML 文件读入一个`Order`对象，所以现在让我们来设置它:

```java
mapper = new ObjectMapper(new YAMLFactory());
```

我们需要使用`findAndRegisterModules` 方法，以便 Jackson 能够正确处理我们的`Date`:

```java
mapper.findAndRegisterModules();
```

一旦我们配置了`ObjectMapper`，**，我们只需使用`readValue` :**

```java
Order order = mapper.readValue(new File("src/main/resources/orderInput.yaml"), Order.class);
```

我们会发现我们的`Order`对象是从文件中填充的，包括`OrderLine`的列表。

## 5.写作 YAML

我们还将使用`ObjectMapper`将`Order`写到一个文件中。但是首先，让我们给它添加一些配置:

```java
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

添加这一行告诉 Jackson 将我们的日期写成`String` 而不是单个的数字部分。

默认情况下，我们的文件将以三个破折号开始。这对 YAML 格式完全有效，但我们可以通过禁用`YAMLFactory`上的功能来关闭它:

```java
mapper = new ObjectMapper(new YAMLFactory().disable(Feature.WRITE_DOC_START_MARKER));
```

完成附加设置后，让我们创建一个`Order`:

```java
List<OrderLine> lines = new ArrayList<>();
lines.add(new OrderLine("Copper Wire (200ft)", 1, 
  new BigDecimal(50.67).setScale(2, RoundingMode.HALF_UP)));
lines.add(new OrderLine("Washers (1/4\")", 24, 
  new BigDecimal(.15).setScale(2, RoundingMode.HALF_UP)));
Order order = new Order(
  "B-9910", 
  LocalDate.parse("2019-04-18", DateTimeFormatter.ISO_DATE),
  "Customer, Jane", 
  lines);
```

让我们用`writeValue`写下我们的订单:

```java
mapper.writeValue(new File("src/main/resources/orderOutput.yaml"), order);
```

当我们查看`orderOutput.yaml`时，它应该类似于:

```java
orderNo: "B-9910"
date: "2019-04-18"
customerName: "Customer, Jane"
orderLines:
- item: "Copper Wire (200ft)"
  quantity: 1
  unitPrice: 50.67
- item: "Washers (1/4\")"
  quantity: 24
  unitPrice: 0.15
```

## 6.结论

在这个快速教程中，我们学习了如何使用 Jackson 库从文件中读取和写入 YAML。我们还看了一些配置项，它们将帮助我们按照我们想要的方式获取数据。

完整的示例代码在 GitHub 上的[。](https://web.archive.org/web/20220703140929/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)