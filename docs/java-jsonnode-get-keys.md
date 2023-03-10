# 使用 JsonNode 获取 JSON 字符串中的所有键

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jsonnode-get-keys>

## 1.概观

在本文中，我们将探索使用`JsonNode`从 JSON 中提取所有嵌套键的不同方法。我们的目标是遍历一个 JSON 字符串并在一个列表中收集键名。

## 2.介绍

[Jackson](/web/20220524024809/https://www.baeldung.com/jackson) 库使用树模型来表示 [JSON](/web/20220524024809/https://www.baeldung.com/java-json) 数据。树模型为我们提供了一种与分层数据交互的有效方式。

**JSON 对象被表示为树模型**中的节点。这使得对 JSON 内容执行 CRUD 操作变得更加容易。

### 2.1.`ObjectMapper`

我们使用`ObjectMapper`类方法来读取 JSON 内容。 **`ObjectMapper.readTree()`方法反序列化 JSON 并构建一个`JsonNode`实例**的树。它将一个 JSON 源作为输入，并返回所创建的树模型的根节点。随后，我们可以使用根节点来遍历整个 JSON 树。

树模型不仅限于读取常规的 Java 对象。JSON 字段和树模型之间存在一对一的映射。因此，每个对象，无论是否是 POJO，都可以表示为一个节点。因此，我们喜欢用灵活的方法将 JSON 内容表示为通用节点。

要了解更多信息，请参考我们关于[杰克森 `ObjectMapper.`的文章](/web/20220524024809/https://www.baeldung.com/jackson-object-mapper-tutorial)

### 2.2.`JsonNode`

`JsonNode`类表示 JSON 树模型中的一个节点。它可以用以下数据类型表示 JSON 数据:`Array, Binary, Boolean, Missing, Null, Number, Object, POJO, String.`这些数据类型在`JsonNodeType`中定义了`enum`。

## 3.从 JSON 获取密钥

我们在本文中使用以下 JSON 作为输入:

```java
{
   "Name":"Craig",
   "Age":10,
   "BookInterests":[
      {
         "Book":"The Kite Runner",
         "Author":"Khaled Hosseini"
      },
      {
         "Book":"Harry Potter",
         "Author":"J. K. Rowling"
      }
   ],
   "FoodInterests":{
      "Breakfast":[
         {
            "Bread":"Whole wheat",
            "Beverage":"Fruit juice"
         },
         {
            "Sandwich":"Vegetable Sandwich",
            "Beverage":"Coffee"
         }
      ]
   }
}
```

这里，我们使用一个`String`对象作为输入，但是我们可以从不同的来源读取 JSON 内容，比如`File`、`byte[]`、`URL`、`InputStream`、`JsonParser`等等。

现在，让我们讨论从 JSON 获取键的不同方法。

### 3.1.使用`fieldNames`

我们可以在一个`JsonNode `实例上使用`fieldNames()`方法来获取嵌套的字段名。它**只返回直接嵌套字段的名称**。

让我们用一个简单的例子来尝试一下:

```java
public List<String> getKeysInJsonUsingJsonNodeFieldNames(String json, ObjectMapper mapper) throws JsonMappingException, JsonProcessingException {

    List<String> keys = new ArrayList<>();
    JsonNode jsonNode = mapper.readTree(json);
    Iterator<String> iterator = jsonNode.fieldNames();
    iterator.forEachRemaining(e -> keys.add(e));
    return keys;
}
```

我们得到以下密钥:

```java
[Name, Age, BookInterests, FoodInterests]
```

为了得到所有的内部嵌套节点，**我们需要在每一层的节点上调用`fieldNames()`方法**:

```java
public List<String> getAllKeysInJsonUsingJsonNodeFieldNames(String json, ObjectMapper mapper) throws JsonMappingException, JsonProcessingException {

    List<String> keys = new ArrayList<>();
    JsonNode jsonNode = mapper.readTree(json);
    getAllKeysUsingJsonNodeFieldNames(jsonNode, keys);
    return keys;
} 
```

```java
private void getAllKeysUsingJsonNodeFields(JsonNode jsonNode, List<String> keys) {

    if (jsonNode.isObject()) {
        Iterator<Entry<String, JsonNode>> fields = jsonNode.fields();
        fields.forEachRemaining(field -> {
            keys.add(field.getKey());
            getAllKeysUsingJsonNodeFieldNames((JsonNode) field.getValue(), keys);
        });
    } else if (jsonNode.isArray()) {
        ArrayNode arrayField = (ArrayNode) jsonNode;
        arrayField.forEach(node -> {
            getAllKeysUsingJsonNodeFieldNames(node, keys);
        });
    }
}
```

首先，我们检查 JSON 值是对象还是数组。如果是，我们也遍历值对象来获取内部节点。结果，我们得到了 JSON 中出现的所有键名:

```java
[Name, Age, BookInterests, Book, Author,
  Book, Author, FoodInterests, Breakfast, Bread, Beverage, Sandwich, Beverage]
```

在上面的例子中，我们还可以使用`JsonNode `类的`fields()`方法来获取字段对象，而不仅仅是字段名:

```java
public List<String> getAllKeysInJsonUsingJsonNodeFields(String json, ObjectMapper mapper) throws JsonMappingException, JsonProcessingException {

    List<String> keys = new ArrayList<>();
    JsonNode jsonNode = mapper.readTree(json);
    getAllKeysUsingJsonNodeFields(jsonNode, keys);
    return keys;
}

private void getAllKeysUsingJsonNodeFields(JsonNode jsonNode, List<String> keys) {

    if (jsonNode.isObject()) {
        Iterator<Entry<String, JsonNode>> fields = jsonNode.fields();
        fields.forEachRemaining(field -> {
            keys.add(field.getKey());
            getAllKeysUsingJsonNodeFieldNames((JsonNode) field.getValue(), keys);
        });
    } else if (jsonNode.isArray()) {
        ArrayNode arrayField = (ArrayNode) jsonNode;
        arrayField.forEach(node -> {
            getAllKeysUsingJsonNodeFieldNames(node, keys);
        });
    }
}
```

### 3.2.使用`JsonParser`

我们也可以使用 **`JsonParser`类进行低级的 JSON 解析**。`JsonParser`从给定的 JSON 内容中创建一系列可迭代的令牌。令牌类型在下面列出的`JsonToken`类中被指定为`enums `:

*   `NOT_AVAILABLE`
*   `START_OBJECT`
*   `END_OBJECT`
*   `START_ARRAY`
*   `FIELD_NAME`
*   `VALUE_EMBEDDED_OBJECT`
*   `VALUE_STRING`
*   `VALUE_NUMBER_INT`
*   `VALUE_NUMBER_FLOAT`
*   `VALUE_TRUE`
*   `VALUE_FALSE`
*   `VALUE_NULL`

当使用`JsonParser`进行迭代时，我们可以检查令牌类型并执行所需的操作。让我们获取示例 JSON 字符串的所有字段名:

```java
public List<String> getKeysInJsonUsingJsonParser(String json, ObjectMapper mapper) throws IOException {

    List<String> keys = new ArrayList<>();
    JsonNode jsonNode = mapper.readTree(json);
    JsonParser jsonParser = jsonNode.traverse();
    while (!jsonParser.isClosed()) {
        if (jsonParser.nextToken() == JsonToken.FIELD_NAME) {
            keys.add((jsonParser.getCurrentName()));
        }
    }
    return keys;
}
```

这里，我们使用了`JsonNode`类的`traverse()`方法来获取`JsonParser`对象。类似地，我们也可以使用`JsonFactory` 创建一个`JsonParser`对象:

```java
public List<String> getKeysInJsonUsingJsonParser(String json) throws JsonParseException, IOException {

    List<String> keys = new ArrayList<>();
    JsonFactory factory = new JsonFactory();
    JsonParser jsonParser = factory.createParser(json);
    while (!jsonParser.isClosed()) {
        if (jsonParser.nextToken() == JsonToken.FIELD_NAME) {
            keys.add((jsonParser.getCurrentName()));
        }
    }
    return keys;
}
```

结果，我们得到了从示例 JSON 内容中提取的所有键名:

```java
[Name, Age, BookInterests, Book, Author,
  Book, Author, FoodInterests, Breakfast, Bread, Beverage, Sandwich, Beverage]
```

请注意，与本教程中介绍的其他方法相比，这段代码是多么的简洁。

### 3.3.使用`Map`

我们可以**使用`ObjectMapper `类的`readValue()`方法将 JSON 内容反序列化到一个`Map`T5。因此，我们可以在迭代 `Map` 对象时提取 JSON 元素。让我们尝试使用这种方法从示例 JSON 中获取所有键:**

```java
public List<String> getKeysInJsonUsingMaps(String json, ObjectMapper mapper) throws JsonMappingException, JsonProcessingException {
    List<String> keys = new ArrayList<>();
    Map<String, Object> jsonElements = mapper.readValue(json, new TypeReference<Map<String, Object>>() {
    });
    getAllKeys(jsonElements, keys);
    return keys;
}

private void getAllKeys(Map<String, Object> jsonElements, List<String> keys) {

    jsonElements.entrySet()
        .forEach(entry -> {
            keys.add(entry.getKey());
            if (entry.getValue() instanceof Map) {
                Map<String, Object> map = (Map<String, Object>) entry.getValue();
                getAllKeys(map, keys);
            } else if (entry.getValue() instanceof List) {
                List<?> list = (List<?>) entry.getValue();
                list.forEach(listEntry -> {
                    if (listEntry instanceof Map) {
                        Map<String, Object> map = (Map<String, Object>) listEntry;
                        getAllKeys(map, keys);
                    }
                });
            }
        });
}
```

在这种情况下，在获得顶级节点后，我们遍历 JSON 对象，这些对象的值要么是对象(映射)，要么是数组，以获得嵌套节点。

## 4.结论

我们已经看到了从 JSON 内容中读取键名的不同方法。此后，我们可以扩展本文中讨论的遍历逻辑，根据需要对 JSON 元素执行其他操作。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524024809/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson)