# 用 Jackson 映射动态 JSON 对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-mapping-dynamic-object>

## 1.介绍

使用 Jackson 处理预定义的 JSON 数据结构非常简单。但是，有时候我们需要处理动态的 **JSON 对象，这些对象有未知的属性**。

在这个快速教程中，我们将学习将动态 JSON 对象映射到 Java 类的多种方法。

注意，在所有的测试中，我们假设我们有类型为`com.fasterxml.jackson.databind.ObjectMapper`的字段`objectMapper`。

## 延伸阅读:

## [用 Jackson 映射嵌套值](/web/20221129015416/https://www.baeldung.com/jackson-nested-values)

Learn three ways to deserialize nested JSON values in Java using the Jackson library.[Read more](/web/20221129015416/https://www.baeldung.com/jackson-nested-values) →

## [与杰克逊一起使用可选](/web/20221129015416/https://www.baeldung.com/jackson-optional)

A quick overview of how we can use the Optional with Jackson.[Read more](/web/20221129015416/https://www.baeldung.com/jackson-optional) →

## 2.使用`JsonNode`

假设我们想在一个网络商店中处理产品规格。所有的产品都有一些共同的属性，但它们也有不同的属性，这取决于产品的类型。

例如，我们想知道手机显示屏的长宽比，但这个属性对于一只鞋来说没有太大意义。

数据结构如下所示:

```
{
    "name": "Pear yPhone 72",
    "category": "cellphone",
    "details": {
        "displayAspectRatio": "97:3",
        "audioConnector": "none"
    }
}
```

我们将动态属性存储在`details`对象中。

我们可以用下面的 Java 类映射公共属性:

```
class Product {

    String name;
    String category;

    // standard getters and setters
}
```

最重要的是，我们需要一个合适的表示来表示`details`对象。比如 **`com.fasterxml.jackson.databind.JsonNode`可以处理动态键**。

要使用它，我们必须将它作为字段添加到我们的`Product`类中:

```
class Product {

    // common fields

    JsonNode details;

    // standard getters and setters
}
```

最后，我们验证它是否有效:

```
String json = "<json object>";

Product product = objectMapper.readValue(json, Product.class);

assertThat(product.getName()).isEqualTo("Pear yPhone 72");
assertThat(product.getDetails().get("audioConnector").asText()).isEqualTo("none");
```

然而，这个解决方案有一个问题；**我们的类依赖于杰克逊库，因为我们有一个`JsonNode`字段。**

## 3.使用`Map`

我们可以通过对`details`字段使用`java.util.Map`来解决这个问题。更准确地说，我们必须使用`Map<String, Object>`。

其他一切都可以保持不变:

```
class Product {

    // common fields

    Map<String, Object> details;

    // standard getters and setters
}
```

然后我们可以通过测试来验证它:

```
String json = "<json object>";

Product product = objectMapper.readValue(json, Product.class);

assertThat(product.getName()).isEqualTo("Pear yPhone 72");
assertThat(product.getDetails().get("audioConnector")).isEqualTo("none");
```

## 4.使用`@JsonAnySetter`

当对象只包含动态属性时，前面的解决方案是很好的选择。然而，有时我们在一个 JSON 对象中混合了**的固定和动态属性。**

例如，我们可能需要简化我们的产品表示:

```
{
    "name": "Pear yPhone 72",
    "category": "cellphone",
    "displayAspectRatio": "97:3",
    "audioConnector": "none"
}
```

我们可以将这种结构视为动态对象。不幸的是，这意味着我们不能定义公共属性，我们也必须动态地对待它们。

或者，我们可以使用 **`@JsonAnySetter`来标记一个处理额外的未知属性**的方法。这种方法应该接受两个参数，即属性的名称和值:

```
class Product {

    // common fields

    Map<String, Object> details = new LinkedHashMap<>();

    @JsonAnySetter
    void setDetail(String key, Object value) {
        details.put(key, value);
    }

    // standard getters and setters
}
```

注意，我们必须实例化`details`对象来避免`NullPointerExceptions`。

由于我们将动态属性存储在一个`Map`中，我们可以像以前一样使用它:

```
String json = "<json object>";

Product product = objectMapper.readValue(json, Product.class);

assertThat(product.getName()).isEqualTo("Pear yPhone 72");
assertThat(product.getDetails().get("audioConnector")).isEqualTo("none");
```

## 5.创建自定义反序列化程序

在大多数情况下，这些解决方案工作得很好；然而，有时我们需要更多的控制。例如，我们可以在数据库中存储关于 JSON 对象的反序列化信息。

我们可以使用自定义的反序列化器来处理这些情况。由于这是一个更复杂的主题，我们将在另一篇文章中讨论它，[Jackson](/web/20221129015416/https://www.baeldung.com/jackson-deserialization)中的定制反序列化入门。

## 6.结论

在本文中，我们讨论了用 Jackson 处理动态 JSON 对象的多种方式。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221129015416/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)