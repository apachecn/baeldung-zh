# 在 Jackson 中使用树模型节点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-json-node-tree-model>

## 1。概述

本教程将重点介绍如何在 Jackson 中使用**树模型节点。**

我们将使用`JsonNode`进行各种转换，以及添加、修改和删除节点。

## 2。创建一个节点

创建节点的第一步是使用默认构造函数实例化一个`ObjectMapper`对象:

```java
ObjectMapper mapper = new ObjectMapper();
```

由于创建一个`ObjectMapper`对象是昂贵的，建议我们在多个操作中重用同一个对象。

接下来，一旦我们有了`ObjectMapper`，我们有三种不同的方法来创建树节点。

### 2.1。从头开始构建一个节点

这是无中生有地创建节点的最常见方式:

```java
JsonNode node = mapper.createObjectNode();
```

或者，我们也可以通过`JsonNodeFactory`创建一个节点:

```java
JsonNode node = JsonNodeFactory.instance.objectNode();
```

### 2.2。从 JSON 源解析

这种方法在[Jackson–Marshall String to JSON node](/web/20220707143834/https://www.baeldung.com/jackson-json-to-jsonnode)一文中有很好的介绍。更多信息请参考。

### 2.3。从一个对象转换

通过调用`ObjectMapper`上的`valueToTree(Object fromValue)`方法，可以从 Java 对象转换节点:

```java
JsonNode node = mapper.valueToTree(fromValue);
```

这里的`convertValue` API 也很有帮助:

```java
JsonNode node = mapper.convertValue(fromValue, JsonNode.class);
```

让我们看看它在实践中是如何工作的。

假设我们有一个名为`NodeBean`的类:

```java
public class NodeBean {
    private int id;
    private String name;

    public NodeBean() {
    }

    public NodeBean(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // standard getters and setters
}
```

让我们编写一个测试来确保转换正确进行:

```java
@Test
public void givenAnObject_whenConvertingIntoNode_thenCorrect() {
    NodeBean fromValue = new NodeBean(2016, "baeldung.com");

    JsonNode node = mapper.valueToTree(fromValue);

    assertEquals(2016, node.get("id").intValue());
    assertEquals("baeldung.com", node.get("name").textValue());
}
```

## 3。转换一个节点

### 3.1。写成 JSON

这是将树节点转换成 JSON 字符串的基本方法，其中目的地可以是`File`、`OutputStream`或`Writer`:

```java
mapper.writeValue(destination, node);
```

通过重用第 2.3 节中声明的类`NodeBean`，一个测试确保该方法按预期工作:

```java
final String pathToTestFile = "node_to_json_test.json";

@Test
public void givenANode_whenModifyingIt_thenCorrect() throws IOException {
    String newString = "{\"nick\": \"cowtowncoder\"}";
    JsonNode newNode = mapper.readTree(newString);

    JsonNode rootNode = ExampleStructure.getExampleRoot();
    ((ObjectNode) rootNode).set("name", newNode);

    assertFalse(rootNode.path("name").path("nick").isMissingNode());
    assertEquals("cowtowncoder", rootNode.path("name").path("nick").textValue());
}
```

### 3.2。转换成一个对象

将`JsonNode`转换成 Java 对象最方便的方法是使用`treeToValue` API:

```java
NodeBean toValue = mapper.treeToValue(node, NodeBean.class);
```

这在功能上等同于以下内容:

```java
NodeBean toValue = mapper.convertValue(node, NodeBean.class)
```

我们也可以通过令牌流来实现:

```java
JsonParser parser = mapper.treeAsTokens(node);
NodeBean toValue = mapper.readValue(parser, NodeBean.class);
```

最后，让我们实现一个测试来验证转换过程:

```java
@Test
public void givenANode_whenConvertingIntoAnObject_thenCorrect()
  throws JsonProcessingException {
    JsonNode node = mapper.createObjectNode();
    ((ObjectNode) node).put("id", 2016);
    ((ObjectNode) node).put("name", "baeldung.com");

    NodeBean toValue = mapper.treeToValue(node, NodeBean.class);

    assertEquals(2016, toValue.getId());
    assertEquals("baeldung.com", toValue.getName());
}
```

## 4。操作树节点

我们将使用包含在名为`example.json`的文件中的以下 JSON 元素，作为将要采取的操作的基础结构:

```java
{
    "name": 
        {
            "first": "Tatu",
            "last": "Saloranta"
        },

    "title": "Jackson founder",
    "company": "FasterXML"
}
```

这个位于类路径上的 JSON 文件被解析成一个模型树:

```java
public class ExampleStructure {
    private static ObjectMapper mapper = new ObjectMapper();

    static JsonNode getExampleRoot() throws IOException {
        InputStream exampleInput = 
          ExampleStructure.class.getClassLoader()
          .getResourceAsStream("example.json");

        JsonNode rootNode = mapper.readTree(exampleInput);
        return rootNode;
    }
}
```

请注意，在下面的小节中说明节点上的操作时，将使用树的根。

### 4.1。定位节点

在处理任何节点之前，我们需要做的第一件事是定位它并将其赋给一个变量。

如果我们事先知道到节点的路径，这是很容易做到的。

假设我们想要一个名为`last`的节点，它位于`name`节点之下:

```java
JsonNode locatedNode = rootNode.path("name").path("last");
```

或者，也可以使用`get`或`with`API 来代替`path`。

如果路径未知，搜索当然会变得更加复杂和反复。

我们可以在第 5 节[中看到一个迭代所有节点的例子——迭代节点](#iterating)。

### 4.2。添加新节点

可以将一个节点添加为另一个节点的子节点:

```java
ObjectNode newNode = ((ObjectNode) locatedNode).put(fieldName, value);
```

`put`的许多重载变体可以用来添加不同值类型的新节点。

其他类似的方法还有很多，包括`putArray`、`putObject`、`PutPOJO`、`putRawValue`、`putNull`。

最后，让我们看一个例子，我们将整个结构添加到树的根节点:

```java
"address":
{
    "city": "Seattle",
    "state": "Washington",
    "country": "United States"
}
```

下面是通过所有这些操作并验证结果的完整测试:

```java
@Test
public void givenANode_whenAddingIntoATree_thenCorrect() throws IOException {
    JsonNode rootNode = ExampleStructure.getExampleRoot();
    ObjectNode addedNode = ((ObjectNode) rootNode).putObject("address");
    addedNode
      .put("city", "Seattle")
      .put("state", "Washington")
      .put("country", "United States");

    assertFalse(rootNode.path("address").isMissingNode());

    assertEquals("Seattle", rootNode.path("address").path("city").textValue());
    assertEquals("Washington", rootNode.path("address").path("state").textValue());
    assertEquals(
      "United States", rootNode.path("address").path("country").textValue();
}
```

### 4.3。编辑节点

可以通过调用`set(String fieldName, JsonNode value)`方法来修改`ObjectNode`实例:

```java
JsonNode locatedNode = locatedNode.set(fieldName, value);
```

对相同类型的对象使用`replace`或`setAll`方法可能会得到类似的结果。

为了验证该方法是否按预期工作，我们将在测试中将根节点下的字段`name`的值从对象`first`和`last`更改为另一个仅包含`nick`字段的值:

```java
@Test
public void givenANode_whenModifyingIt_thenCorrect() throws IOException {
    String newString = "{\"nick\": \"cowtowncoder\"}";
    JsonNode newNode = mapper.readTree(newString);

    JsonNode rootNode = ExampleStructure.getExampleRoot();
    ((ObjectNode) rootNode).set("name", newNode);

    assertFalse(rootNode.path("name").path("nick").isMissingNode());
    assertEquals("cowtowncoder", rootNode.path("name").path("nick").textValue());
}
```

### 4.4。删除一个节点

可以通过调用父节点上的`remove(String fieldName)` API 来删除节点:

```java
JsonNode removedNode = locatedNode.remove(fieldName);
```

为了一次移除多个节点，我们可以使用`Collection<String>`类型的参数调用一个重载方法，该方法返回父节点而不是要移除的节点:

```java
ObjectNode locatedNode = locatedNode.remove(fieldNames);
```

在极端情况下，当我们想要删除给定节点的所有子节点时， `removeAll`API 就派上了用场。

下面的测试将着重于上面提到的第一种方法，这是最常见的情况:

```java
@Test
public void givenANode_whenRemovingFromATree_thenCorrect() throws IOException {
    JsonNode rootNode = ExampleStructure.getExampleRoot();
    ((ObjectNode) rootNode).remove("company");

    assertTrue(rootNode.path("company").isMissingNode());
}
```

## 5。迭代节点

让我们遍历 JSON 文档中的所有节点，并将它们重新格式化为 YAML。

JSON 有三种类型的节点，分别是值、对象和数组。

因此，让我们通过添加一个`Array`来确保我们的样本数据具有所有三种不同的类型:

```java
{
    "name": 
        {
            "first": "Tatu",
            "last": "Saloranta"
        },

    "title": "Jackson founder",
    "company": "FasterXML",
    "pets" : [
        {
            "type": "dog",
            "number": 1
        },
        {
            "type": "fish",
            "number": 50
        }
    ]
}
```

现在让我们看看我们想要生产的 YAML:

```java
name: 
  first: Tatu
  last: Saloranta
title: Jackson founder
company: FasterXML
pets: 
- type: dog
  number: 1
- type: fish
  number: 50
```

我们知道 JSON 节点有一个层次树结构。因此，迭代整个 JSON 文档最简单的方法是从顶部开始，向下遍历所有子节点。

我们将把根节点传递给一个递归方法。然后，该方法将使用所提供节点的每个子节点调用自身。

### 5.1。测试迭代

我们将从创建一个简单的测试开始，该测试检查我们能否成功地将 JSON 转换成 YAML。

我们的测试将 JSON 文档的根节点提供给我们的`toYaml`方法，并断言返回值是我们所期望的:

```java
@Test
public void givenANodeTree_whenIteratingSubNodes_thenWeFindExpected() throws IOException {
    JsonNode rootNode = ExampleStructure.getExampleRoot();

    String yaml = onTest.toYaml(rootNode);

    assertEquals(expectedYaml, yaml); 
}

public String toYaml(JsonNode root) {
    StringBuilder yaml = new StringBuilder(); 
    processNode(root, yaml, 0); 
    return yaml.toString(); }
}
```

### 5.2。处理不同的节点类型

我们需要稍微不同地处理不同类型的节点。

我们将在我们的`processNode`方法中这样做:

```java
private void processNode(JsonNode jsonNode, StringBuilder yaml, int depth) {
    if (jsonNode.isValueNode()) {
        yaml.append(jsonNode.asText());
    }
    else if (jsonNode.isArray()) {
        for (JsonNode arrayItem : jsonNode) {
            appendNodeToYaml(arrayItem, yaml, depth, true);
        }
    }
    else if (jsonNode.isObject()) {
        appendNodeToYaml(jsonNode, yaml, depth, false);
    }
}
```

首先，让我们考虑一个值节点。我们简单地调用节点的`asText`方法来获得值的`String`表示。

接下来，让我们看一个数组节点。数组节点中的每一项本身都是一个`JsonNode`，所以我们迭代数组并将每个节点传递给`appendNodeToYaml`方法。我们还需要知道这些节点是数组的一部分。

不幸的是，节点本身不包含任何告诉我们这一点的东西，所以我们将传递一个标志到我们的`appendNodeToYaml`方法中。

最后，我们希望迭代每个对象节点的所有子节点。一种选择是使用`JsonNode.elements`。

但是，我们不能从元素中确定字段名，因为它只包含字段值:

```java
Object  {"first": "Tatu", "last": "Saloranta"}
Value  "Jackson Founder"
Value  "FasterXML"
Array  [{"type": "dog", "number": 1},{"type": "fish", "number": 50}]
```

相反，我们将使用`JsonNode.fields`,因为这使我们能够访问字段名和值:

```java
Key="name", Value=Object  {"first": "Tatu", "last": "Saloranta"}
Key="title", Value=Value  "Jackson Founder"
Key="company", Value=Value  "FasterXML"
Key="pets", Value=Array  [{"type": "dog", "number": 1},{"type": "fish", "number": 50}]
```

对于每个字段，我们将字段名添加到输出中，然后通过将值传递给`processNode`方法，将该值作为子节点进行处理:

```java
private void appendNodeToYaml(
  JsonNode node, StringBuilder yaml, int depth, boolean isArrayItem) {
    Iterator<Entry<String, JsonNode>> fields = node.fields();
    boolean isFirst = true;
    while (fields.hasNext()) {
        Entry<String, JsonNode> jsonField = fields.next();
        addFieldNameToYaml(yaml, jsonField.getKey(), depth, isArrayItem && isFirst);
        processNode(jsonField.getValue(), yaml, depth+1);
        isFirst = false;
    }

}
```

从节点上看不出来它有多少祖先。

因此，我们将一个名为 depth 的字段传递到`processNode`方法中进行跟踪，并且我们在每次获得一个子节点时增加这个值，以便我们可以在 YAML 输出中正确地缩进这些字段:

```java
private void addFieldNameToYaml(
  StringBuilder yaml, String fieldName, int depth, boolean isFirstInArray) {
    if (yaml.length()>0) {
        yaml.append("\n");
        int requiredDepth = (isFirstInArray) ? depth-1 : depth;
        for(int i = 0; i < requiredDepth; i++) {
            yaml.append("  ");
        }
        if (isFirstInArray) {
            yaml.append("- ");
        }
    }
    yaml.append(fieldName);
    yaml.append(": ");
}
```

既然我们已经有了迭代节点并创建 YAML 输出的所有代码，我们可以运行测试来证明它是可行的。

## 6。结论

本文介绍了在 Jackson 中使用树模型时常见的 API 和场景。

和往常一样，所有这些例子和代码片段的实现可以在 GitHub 上找到 **[。](https://web.archive.org/web/20220707143834/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson)**