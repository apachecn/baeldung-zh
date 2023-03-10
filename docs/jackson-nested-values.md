# 用 Jackson 映射嵌套值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-nested-values>

## 1。概述

使用 JSON 的一个典型用例是执行从一个模型到另一个模型的转换。例如，我们可能希望将一个复杂的、密集嵌套的对象图解析成一个更简单的模型，以便在另一个领域中使用。

在这个快速教程中，我们将看看**如何用[杰克森](https://web.archive.org/web/20221207131427/https://github.com/FasterXML/jackson)** 映射嵌套值来展平复杂的数据结构。我们将以三种不同的方式反序列化 JSON:

*   使用`@JsonProperty`
*   使用`JsonNode`
*   使用自定义`JsonDeserializer`

## 延伸阅读:

## [与杰克逊一起使用可选](/web/20221207131427/https://www.baeldung.com/jackson-optional)

A quick overview of how we can use the Optional with Jackson.[Read more](/web/20221207131427/https://www.baeldung.com/jackson-optional) →

## [与杰克逊的传承](/web/20221207131427/https://www.baeldung.com/jackson-inheritance)

This tutorial will demonstrate how to handle inclusion of subtype metadata and ignoring properties inherited from superclasses with Jackson.[Read more](/web/20221207131427/https://www.baeldung.com/jackson-inheritance) →

## [在 Spring Boot 使用@ JSON component](/web/20221207131427/https://www.baeldung.com/spring-boot-jsoncomponent)

Learn how to use the @JsonComponent annotation in Spring Boot.[Read more](/web/20221207131427/https://www.baeldung.com/spring-boot-jsoncomponent) →

## 2。Maven 依赖关系

让我们首先给`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221207131427/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22) 上找到`jackson-databind`的最新版本。

## 3。JSON 源码

考虑下面的 JSON 作为我们例子的原始资料。

虽然结构是人为设计的，但请注意，我们包括嵌套两层的属性:

```java
{
    "id": "957c43f2-fa2e-42f9-bf75-6e3d5bb6960a",
    "name": "The Best Product",
    "brand": {
        "id": "9bcd817d-0141-42e6-8f04-e5aaab0980b6",
        "name": "ACME Products",
        "owner": {
            "id": "b21a80b1-0c09-4be3-9ebd-ea3653511c13",
            "name": "Ultimate Corp, Inc."
        }
    }  
} 
```

## 4。简化的领域模型

在下面的`Product` 类描述的扁平域模型中，我们将提取`brandName`，它嵌套在我们的源 JSON 中的一个层次。

此外，我们将提取嵌套两层的`ownerName`，它位于嵌套的`brand`对象中:

```java
public class Product {

    private String id;
    private String name;
    private String brandName;
    private String ownerName;

    // standard getters and setters
} 
```

## 5。带注释的映射

为了映射嵌套的`brandName`属性，我们首先需要将嵌套的`brand`对象解包为一个`Map`，并提取`name`属性。为了映射`ownerName`，我们将嵌套的`owner`对象解包为`Map`，并提取其`name`属性。

我们可以通过结合使用`@JsonProperty`和一些我们添加到`Product`类中的自定义逻辑，指示 Jackson**解包嵌套的属性:**

```java
public class Product {
    // ...

    @SuppressWarnings("unchecked")
    @JsonProperty("brand")
    private void unpackNested(Map<String,Object> brand) {
        this.brandName = (String)brand.get("name");
        Map<String,String> owner = (Map<String,String>)brand.get("owner");
        this.ownerName = owner.get("name");
    }
} 
```

我们的客户端代码现在可以使用一个`ObjectMapper`来转换我们的源 JSON，它作为`String`常量`SOURCE_JSON`存在于测试类中:

```java
@Test
public void whenUsingAnnotations_thenOk() throws IOException {
    Product product = new ObjectMapper()
      .readerFor(Product.class)
      .readValue(SOURCE_JSON);

    assertEquals(product.getName(), "The Best Product");
    assertEquals(product.getBrandName(), "ACME Products");
    assertEquals(product.getOwnerName(), "Ultimate Corp, Inc.");
}
```

## 6。用`JsonNode`映射

用`JsonNode`映射一个嵌套的数据结构需要更多的工作。

这里我们使用`ObjectMapper`的`readTree`解析出想要的字段:

```java
@Test
public void whenUsingJsonNode_thenOk() throws IOException {
    JsonNode productNode = new ObjectMapper().readTree(SOURCE_JSON);

    Product product = new Product();
    product.setId(productNode.get("id").textValue());
    product.setName(productNode.get("name").textValue());
    product.setBrandName(productNode.get("brand")
      .get("name").textValue());
    product.setOwnerName(productNode.get("brand")
      .get("owner").get("name").textValue());

    assertEquals(product.getName(), "The Best Product");
    assertEquals(product.getBrandName(), "ACME Products");
    assertEquals(product.getOwnerName(), "Ultimate Corp, Inc.");
}
```

## 7。用自定义`JsonDeserializer`绘制

从实现的角度来看，用定制的`JsonDeserializer`映射嵌套的数据结构与`JsonNode`方法是相同的。

我们首先创建`JsonDeserializer`:

```java
public class ProductDeserializer extends StdDeserializer<Product> {

    public ProductDeserializer() {
        this(null);
    }

    public ProductDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public Product deserialize(JsonParser jp, DeserializationContext ctxt) 
      throws IOException, JsonProcessingException {

        JsonNode productNode = jp.getCodec().readTree(jp);
        Product product = new Product();
        product.setId(productNode.get("id").textValue());
        product.setName(productNode.get("name").textValue());
        product.setBrandName(productNode.get("brand")
          .get("name").textValue());
        product.setOwnerName(productNode.get("brand").get("owner")
          .get("name").textValue());		
        return product;
    }
} 
```

### 7.1。手动注册解串器

要手动注册我们的定制反序列化器，我们的客户端代码必须将`JsonDeserializer`添加到`Module`，用`ObjectMapper`注册`Module`，并调用`readValue`:

```java
@Test
public void whenUsingDeserializerManuallyRegistered_thenOk()
 throws IOException {

    ObjectMapper mapper = new ObjectMapper();
    SimpleModule module = new SimpleModule();
    module.addDeserializer(Product.class, new ProductDeserializer());
    mapper.registerModule(module);

    Product product = mapper.readValue(SOURCE_JSON, Product.class);

    assertEquals(product.getName(), "The Best Product");
    assertEquals(product.getBrandName(), "ACME Products");
    assertEquals(product.getOwnerName(), "Ultimate Corp, Inc.");
} 
```

### 7.2。解串器的自动注册

作为手动注册`JsonDeserializer`的替代方法，我们可以**直接在类**上注册反序列化器:

```java
@JsonDeserialize(using = ProductDeserializer.class)
public class Product {
    // ...
}
```

使用这种方法，不需要手动注册。

让我们看看使用自动注册的客户机代码:

```java
@Test
public void whenUsingDeserializerAutoRegistered_thenOk()
  throws IOException {

    ObjectMapper mapper = new ObjectMapper();
    Product product = mapper.readValue(SOURCE_JSON, Product.class);

    assertEquals(product.getName(), "The Best Product");
    assertEquals(product.getBrandName(), "ACME Products");
    assertEquals(product.getOwnerName(), "Ultimate Corp, Inc.");
}
```

## 8。结论

在本文中，我们展示了几种使用 **Jackson 解析包含嵌套值的 JSON 的方法。**查看我们的主[杰克森教程](/web/20221207131427/https://www.baeldung.com/jackson)页面获取更多示例。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221207131427/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions)