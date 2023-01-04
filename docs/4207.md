# Jackson–自定义串行器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-custom-serialization>

## 1。概述

这个快速教程将展示如何使用一个定制的序列化器来序列化一个 Java 实体。

如果你想更深入地了解和学习其他很酷的事情，你可以用杰克逊 2 号来做——直接去[杰克逊的主要教程](/web/20220628085846/https://www.baeldung.com/jackson "The main Jackson Tutorial")。

## 2。对象图的标准序列化

让我们定义两个简单的实体，看看 Jackson 如何在没有任何定制逻辑的情况下序列化它们:

```
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

现在，让我们用一个`User`实体序列化一个`Item`实体:

```
Item myItem = new Item(1, "theItem", new User(2, "theUser"));
String serialized = new ObjectMapper().writeValueAsString(myItem);
```

这将为两个实体产生一个完整的 JSON 表示:

```
{
    "id": 1,
    "itemName": "theItem",
    "owner": {
        "id": 2,
        "name": "theUser"
    }
}
```

## 3。`ObjectMapper`上的自定义序列化程序

现在，**让我们简化上面的 JSON 输出**，只序列化`User`的`id`，而不是整个`User`对象；我们希望得到以下更简单的 JSON:

```
{
    "id": 25,
    "itemName": "FEDUfRgS",
    "owner": 15
}
```

简单地说，我们必须**为`Item`对象定义一个定制的序列化器**:

```
public class ItemSerializer extends StdSerializer<Item> {

    public ItemSerializer() {
        this(null);
    }

    public ItemSerializer(Class<Item> t) {
        super(t);
    }

    @Override
    public void serialize(
      Item value, JsonGenerator jgen, SerializerProvider provider) 
      throws IOException, JsonProcessingException {

        jgen.writeStartObject();
        jgen.writeNumberField("id", value.id);
        jgen.writeStringField("itemName", value.itemName);
        jgen.writeNumberField("owner", value.owner.id);
        jgen.writeEndObject();
    }
}
```

现在，我们需要用`Item`类的`ObjectMapper`注册这个定制序列化程序，并执行序列化:

```
Item myItem = new Item(1, "theItem", new User(2, "theUser"));
ObjectMapper mapper = new ObjectMapper();

SimpleModule module = new SimpleModule();
module.addSerializer(Item.class, new ItemSerializer());
mapper.registerModule(module);

String serialized = mapper.writeValueAsString(myItem);
```

就这样——我们现在有了一个更简单的自定义 JSON 序列化的`Item->User`实体。

## 4。类上的自定义序列化程序

我们也可以直接在类上注册序列化器，而不是在类`ObjectMapper`上注册:

```
@JsonSerialize(using = ItemSerializer.class)
public class Item {
    ...
}
```

现在，当执行**标准序列化**时:

```
Item myItem = new Item(1, "theItem", new User(2, "theUser"));
String serialized = new ObjectMapper().writeValueAsString(myItem);
```

我们将获得定制的 JSON 输出，由序列化程序创建，通过`@JsonSerialize`指定:

```
{
    "id": 25,
    "itemName": "FEDUfRgS",
    "owner": 15
}
```

当不能直接访问和配置`ObjectMapper`时，这很有帮助。

## 5。结论

本文展示了如何通过使用序列化器，在 Jackson 2 中获得定制的 JSON 输出。

所有这些例子和代码片段**的实现都可以在[GitHub](https://web.archive.org/web/20220628085846/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-custom-conversions#readme "Github Project exemplifying how to use Custom Serializers")T3 上找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。**