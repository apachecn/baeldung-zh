# 从 GraphQL 返回地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-graphql-return-map>

## 1.概观

多年来， [GraphQL](/web/20220524052942/https://www.baeldung.com/graphql) 已经被广泛接受为 web 服务的通信模式之一。尽管它功能丰富，使用灵活，但在某些情况下可能会带来挑战。其中之一是从查询中返回一个`Map`，这是一个挑战，因为`Map` 不是 GraphQL 中的类型。

在本教程中，我们将学习从 GraphQL 查询返回`Map`的技术。

## 2.例子

让我们以一个产品数据库为例，该数据库具有不确定数量的定制属性。

一个`Product`作为一个数据库实体，可能有一些固定的字段，如`name`、`price`、`category`等。但是，它也可能具有因类别而异的属性。这些属性应该以保留其标识键的方式返回给客户端。

为此，我们可以使用一个`Map`作为这些属性的类型。

## 3.返回地图

为了返回地图，我们有三种选择:

*   作为 JSON 返回`String`
*   使用 GraphQL 自定义[标量类型](https://web.archive.org/web/20220524052942/https://graphql.org/learn/schema/#scalar-types)
*   作为键值对的`List`返回

对于前两个选项，我们将使用下面的 GraphQL 查询:

```
query {
    product(id:1){ 
        id 
        name 
        description 
        attributes 
    }
}
```

参数`attributes` 将以`Map`格式表示。

接下来，让我们看看所有三个选项。

### 3.1.JSON 字符串

这是最简单的选择。我们将在`Product`解析器中将`Map`序列化为 JSON `String`格式:

```
String attributes = objectMapper.writeValueAsString(product.getAttributes());
```

GraphQL 模式本身如下:

```
type Product {
    id: ID
    name: String!
    description: String
    attributes:String
}
```

下面是实现后的查询结果:

```
{
  "data": {
    "product": {
      "id": "1",
      "name": "Product 1",
      "description": "Product 1 description",
      "attributes": "{\"size\": {
                                     \"name\": \"Large\",
                                     \"description\": \"This is custom attribute description\",
                                     \"unit\": \"This is custom attribute unit\"
                                    },
                   \"attribute_1\": {
                                     \"name\": \"Attribute1 name\",
                                     \"description\": \"This is custom attribute description\",
                                     \"unit\": \"This is custom attribute unit\"
                                    }
                        }"
    }
  }
}
```

这种选择有两个问题。**第一个问题是 JSON 字符串需要在客户端处理成可行的格式。第二个问题是我们不能有属性的子查询。**

为了克服第一个问题，GraphQL 自定义标量类型的第二个选项会有所帮助。

### 3.2.GraphQL 自定义标量类型

对于实现，我们将利用 Java 中 GraphQL 的扩展标量库。

首先，我们将在`pom.xml`中包含[graph QL-Java-extended-scalar](https://web.archive.org/web/20220524052942/https://mvnrepository.com/artifact/com.graphql-java/graphql-java-extended-scalars/18.0)依赖关系:

```
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-extended-scalars</artifactId>
    <version>2022-04-06T00-10-27-a70541e</version>
</dependency>
```

然后，我们将在 GraphQL 配置组件中注册我们选择的标量类型。在这种情况下，标量类型是`JSON`:

```
@Bean
public GraphQLScalarType json() {
    return ExtendedScalars.Json;
}
```

最后，我们将相应地更新我们的 GraphQL 模式:

```
type Product {
    id: ID
    name: String!
    description: String
    attributes: JSON
}
scalar JSON
```

下面是实现后的结果:

```
{
  "data": {
    "product": {
      "id": "1",
      "name": "Product 1",
      "description": "Product 1 description",
      "attributes": {
        "size": {
          "name": "Large",
          "description": "This is custom attribute description",
          "unit": "This is a custom attribute unit"
        },
        "attribute_1": {
          "name": "Attribute1 name",
          "description": "This is custom attribute description",
          "unit": "This is a custom attribute unit"
        }
      }
    }
  }
}
```

使用这种方法，我们不需要在客户端处理属性映射。然而，标量类型有其自身的局限性。

在 GraphQL 中，标量类型是查询的叶子，这表明它们不能被进一步查询。

### 3.3.键值对列表

如果要求进一步查询`Map`，那么这是最可行的选择。我们将把`Map`对象转换成一个键值对对象列表。

下面是我们代表一个键值对的类:

```
public class AttributeKeyValueModel {
    private String key;
    private Attribute value;

    public AttributeKeyValueModel(String key, Attribute value) {
        this.key = key;
        this.value = value;
    }
} 
```

在`Product`解析器中，我们将添加以下实现:

```
List<AttributeKeyValueModel> attributeModelList = new LinkedList<>();
product.getAttributes().forEach((key, val) -> attributeModelList.add(new AttributeKeyValueModel(key, val)));
```

最后，我们将更新模式:

```
type Product {
    id: ID
    name: String!
    description: String
    attributes:[AttributeKeyValuePair]
}
type AttributeKeyValuePair {
    key:String
    value:Attribute
}
type Attribute {
    name:String
    description:String
    unit:String
}
```

既然我们已经更新了模式，我们也将更新查询:

```
query {
    product(id:1){ 
         id 
         name 
         description 
         attributes {
             key 
             value {
                 name 
                 description 
                 unit
             }
        } 
    }
}
```

现在，让我们看看结果:

```
{
  "data": {
    "product": {
      "id": "1",
      "name": "Product 1",
      "description": "Product 1 description",
      "attributes": [
        {
          "key": "size",
          "value": {
            "name": "Large",
            "description": "This is custom attribute description",
            "unit": "This is custom attribute unit"
          }
        },
        {
          "key": "attribute_1",
          "value": {
            "name": "Attribute1 name",
            "description": "This is custom attribute description",
            "unit": "This is custom attribute unit"
          }
        }
      ]
    }
  }
}
```

这种选择也有两个问题。GraphQL 查询变得有点复杂。并且对象结构需要硬编码。未知的`Map`对象在这种情况下不起作用。

## 4.结论

在本文中，我们研究了从 GraphQL 查询返回一个`Map`对象的三种不同技术。我们讨论了它们各自的局限性。因为没有一种技术是完美的，所以必须根据需求来使用它们。

和往常一样，本文的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524052942/https://github.com/eugenp/tutorials/tree/master/graphql/graphql-java)