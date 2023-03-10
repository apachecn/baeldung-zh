# Java 中的 JSON 模式介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/introduction-to-json-schema-in-java>

## 1。概述

`**JSON Schema**`是一种声明性语言，用于验证`**JSON Object**`的格式和结构。它允许我们指定特殊原语的数量，以准确描述有效的`JSON Object`的样子。

`JSON Schema`规格分为三部分:

*   JSON 模式核心:JSON 模式核心规范是定义模式术语的地方。
*   [JSON 模式验证](https://web.archive.org/web/20221222053039/https://json-schema.org/latest/json-schema-validation.html):JSON 模式验证规范是定义验证约束的有效方法的文档。本文档还定义了一组关键字，可用于指定 JSON API 的验证。在接下来的例子中，我们将使用其中的一些关键字。
*   [JSON 超级模式](https://web.archive.org/web/20221222053039/https://json-schema.org/draft/2019-09/json-schema-hypermedia.html):这是 JSON 模式规范的另一个扩展，其中定义了超链接和超媒体相关的关键字。

## 2。定义一个 JSON 模式

现在我们已经定义了`JSON Schema`的用途，让我们创建一个`JSON Object`和相应的描述它的`JSON Schema`。

下面是一个简单的代表产品目录的`JSON Object`:

```java
{
    "id": 1,
    "name": "Lampshade",
    "price": 0
}
```

我们可以将其 `JSON Schema`定义如下:

```java
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Product",
    "description": "A product from the catalog",
    "type": "object",
    "properties": {
        "id": {
            "description": "The unique identifier for a product",
            "type": "integer"
        },
        "name": {
            "description": "Name of the product",
            "type": "string"
        },
        "price": {
            "type": "number",
            "minimum": 0,
            "exclusiveMinimum": true
        }
    },
    "required": ["id", "name", "price"]
}
```

正如我们所看到的，一个`JSON Schema`是一个`JSON document`，并且该文档必须是一个对象。由`JSON Schema`定义的对象成员(或属性)称为 **`keywords`** 。

让我们解释一下我们在示例中使用的关键字:

*   关键字 **`$schema`** 声明该模式是根据草案 v4 规范编写的。
*   **`title`** 和 **`description`** 关键字仅是描述性的，因为它们不对被验证的数据添加约束。模式的意图用这两个关键字来表述:描述产品。
*   **`type`** 关键字**定义了我们的`JSON`数据上的第一个约束**:它**必须是** a `JSON Object`。

另外，JSON 模式可能包含非模式关键字的属性。在我们的例子中，`**id**`、**、`name`、**、`price`、**将是`JSON Object`的成员(或属性)。**

对于每个属性，我们可以定义 **`type`** 。我们将`id`和`name`定义为**`string`**`price`为`**number**`。在`JSON Schema`中，一个数字可以有一个最小值。默认情况下这个最小值是包含的，所以我们需要指定 **`exclusiveMinimum`** 。

最后，`Schema`告诉我们`id`、`name`和`price`是`**required**`。

## 3。用 JSON 模式进行验证

随着我们的`JSON Schema`到位**，我们可以验证**我们的`JSON Object`。

有很多[库](https://web.archive.org/web/20221222053039/https://json-schema.org/implementations.html)来完成这个任务。对于我们的例子，我们选择了 Java [json-schema](https://web.archive.org/web/20221222053039/https://github.com/networknt/json-schema-validator) 库。

首先，我们需要向我们的`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>com.networknt</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>1.0.72</version>
</dependency> 
```

最后，我们可以编写几个简单的测试用例来验证我们的`JSON Object:`

```java
@Test
public void givenInvalidInput_whenValidating_thenInvalid() throws IOException {
    JsonSchemaFactory factory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V4);
    JsonSchema jsonSchema = factory.getSchema(
     JSONSchemaUnitTest.class.getResourceAsStream("/schema.json"));
    JsonNode jsonNode = mapper.readTree(
     JSONSchemaUnitTest.class.getResourceAsStream("/product_invalid.json"));
    Set<ValidationMessage> errors = jsonSchema.validate(jsonNode);
    assertThat(errors).isNotEmpty().asString().contains("price: must have a minimum value of 0");
}
```

在这种情况下，将收到验证错误。

第二个测试如下所示:

```java
@Test 
public void givenValidInput_whenValidating_thenValid() throws ValidationException { 
    JsonSchemaFactory factory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V4); 
    JsonSchema jsonSchema = factory.getSchema( 
     JSONSchemaUnitTest.class.getResourceAsStream("/schema.json")); 
    JsonNode jsonNode = mapper.readTree( 
     JSONSchemaUnitTest.class.getResourceAsStream("/product_valid.json")); 
    Set<ValidationMessage> errors = jsonSchema.validate(jsonNode); 
    assertThat(errors).isEmpty(); 
}
```

因为我们使用有效的`JSON Object`，所以不会抛出验证错误。

## 4。结论

在本文中，我们已经定义了什么是 JSON 模式，以及哪些相关的关键字 帮助我们 定义我们的模式。

耦合一个`JSON Schema`和它对应的`JSON Object`表示我们可以执行一些验证任务。 

本文的一个简单测试案例可以在 [GitHub 项目](https://web.archive.org/web/20221222053039/https://github.com/eugenp/tutorials/tree/master/json-modules/json)中找到。