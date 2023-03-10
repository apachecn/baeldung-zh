# Jackson 中的自定义反序列化入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-deserialization>

## 1。概述

这个快速教程将演示如何使用 Jackson 2 通过一个**定制的反序列化器来反序列化 JSON。**

为了更深入地了解杰克逊 2 中我们能做的其他很酷的事情，请前往[杰克逊的主要教程](/web/20220819211945/https://www.baeldung.com/jackson "The main Jackson Tutorial")。

## 延伸阅读:

## [杰克逊对象映射器简介](/web/20220819211945/https://www.baeldung.com/jackson-object-mapper-tutorial)

The article discusses Jackson's central ObjectMapper class, basic serialization and deserialization as well as configuring the two processes.[Read more](/web/20220819211945/https://www.baeldung.com/jackson-object-mapper-tutorial) →

## [Jackson–决定序列化/反序列化哪些字段](/web/20220819211945/https://www.baeldung.com/jackson-field-serializable-deserializable-or-not)

How to control which fields get serialized/deserialized by Jackson and which fields get ignored.[Read more](/web/20220819211945/https://www.baeldung.com/jackson-field-serializable-deserializable-or-not) →

## [Jackson–定制串行器](/web/20220819211945/https://www.baeldung.com/jackson-custom-serialization)

Control your JSON output with Jackson 2 by using a Custom Serializer.[Read more](/web/20220819211945/https://www.baeldung.com/jackson-custom-serialization) →

## 2。标准反序列化

让我们从定义两个实体开始，看看 Jackson 如何在没有任何定制的情况下将 JSON 表示反序列化为这些实体:

```java
public class User {
    public int id;
    public String name;
}
public class Item {
    public int id;
    public String itemName;
    public User owner;
}
```

现在让我们定义想要反序列化的 JSON 表示:

```java
{
    "id": 1,
    "itemName": "theItem",
    "owner": {
        "id": 2,
        "name": "theUser"
    }
}
```

最后，让我们将这个 JSON 解组到 Java 实体:

```java
Item itemWithOwner = new ObjectMapper().readValue(json, Item.class);
```

## 3。`ObjectMapper` 上的自定义反序列化器

在前面的例子中，JSON 表示完美地匹配了 Java 实体。

接下来，我们将简化 JSON:

```java
{
    "id": 1,
    "itemName": "theItem",
    "createdBy": 2
}
```

默认情况下，当将其解组到完全相同的实体时，这当然会失败:

```java
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: 
Unrecognized field "createdBy" (class org.baeldung.jackson.dtos.Item), 
not marked as ignorable (3 known properties: "id", "owner", "itemName"])
 at [Source: [[email protected]](/web/20220819211945/https://www.baeldung.com/cdn-cgi/l/email-protection); line: 1, column: 43] 
 (through reference chain: org.baeldung.jackson.dtos.Item["createdBy"])
```

我们将通过使用自定义反序列化器进行**我们自己的反序列化来解决这个问题:**

```java
public class ItemDeserializer extends StdDeserializer<Item> { 

    public ItemDeserializer() { 
        this(null); 
    } 

    public ItemDeserializer(Class<?> vc) { 
        super(vc); 
    }

    @Override
    public Item deserialize(JsonParser jp, DeserializationContext ctxt) 
      throws IOException, JsonProcessingException {
        JsonNode node = jp.getCodec().readTree(jp);
        int id = (Integer) ((IntNode) node.get("id")).numberValue();
        String itemName = node.get("itemName").asText();
        int userId = (Integer) ((IntNode) node.get("createdBy")).numberValue();

        return new Item(id, itemName, new User(userId, null));
    }
}
```

正如我们所看到的，反序列化器正在使用 JSON 的标准 Jackson 表示——`JsonNode`。一旦输入的 JSON 被表示为一个`JsonNode`，我们现在就可以**从它的**中提取相关信息，并构建我们自己的`Item`实体。

简单地说，我们需要**注册这个定制的反序列化器**并正常反序列化 JSON:

```java
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule();
module.addDeserializer(Item.class, new ItemDeserializer());
mapper.registerModule(module);

Item readValue = mapper.readValue(json, Item.class);
```

## 4。类上的自定义反序列化程序

或者，我们也可以**在类**上直接注册反序列化器:

```java
@JsonDeserialize(using = ItemDeserializer.class)
public class Item {
    ...
}
```

有了在类级别定义的反序列化器，就不需要在`ObjectMapper`上注册它了——一个默认的映射器就可以了:

```java
Item itemWithOwner = new ObjectMapper().readValue(json, Item.class);
```

在我们不能直接访问原始的`ObjectMapper`来配置的情况下，这种类型的每类配置非常有用。

## 5。结论

本文展示了如何利用 Jackson 2 来**读取非标准的 JSON 输入**，以及如何将该输入映射到任何 Java 实体图，并完全控制映射。

所有这些例子的实现和代码片段**可以在 GitHubT3[上找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20220819211945/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-custom-conversions "Github Project using custom Deserializers")**