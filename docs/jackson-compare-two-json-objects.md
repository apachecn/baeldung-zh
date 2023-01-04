# 用 Jackson 比较两个 JSON 对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-compare-two-json-objects>

## 1。概述

在本教程中，我们将使用 Java 的 JSON 处理库 [Jackson](/web/20221129022100/https://www.baeldung.com/jackson) 来比较两个 JSON 对象。

## 2。Maven 依赖关系

首先，让我们添加 `[jackson-databind](https://web.archive.org/web/20221129022100/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)` Maven 依赖项:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency> 
```

## 3。使用 Jackson 比较两个 JSON 对象

我们将使用 [`ObjectMapper`](https://web.archive.org/web/20221129022100/https://static.javadoc.io/com.fasterxml.jackson.core/jackson-databind/2.9.8/com/fasterxml/jackson/databind/ObjectMapper.html) 类来读取一个对象作为`[JsonNode](https://web.archive.org/web/20221129022100/https://static.javadoc.io/com.fasterxml.jackson.core/jackson-databind/2.9.8/com/fasterxml/jackson/databind/JsonNode.html)`。

让我们创建一个`ObjectMapper`:

```java
ObjectMapper mapper = new ObjectMapper();
```

### 3.1.比较两个简单的 JSON 对象

让我们从使用 [`JsonNode.equals`](https://web.archive.org/web/20221129022100/https://static.javadoc.io/com.fasterxml.jackson.core/jackson-databind/2.9.8/com/fasterxml/jackson/databind/JsonNode.html#equals-java.lang.Object-) 方法开始。**`equals()`方法执行全面(深入)的比较。**

假设我们有一个定义为`s1`变量的 JSON 字符串:

```java
{
    "employee":
    {
        "id": "1212",
        "fullName": "John Miles",
        "age": 34
    }
}
```

我们想将它与另一个 JSON 进行比较，`s2`:

```java
{   
    "employee":
    {
        "id": "1212",
        "age": 34,
        "fullName": "John Miles"
    }
}
```

让我们将输入的 JSON 读作`JsonNode`并进行比较:

```java
assertEquals(mapper.readTree(s1), mapper.readTree(s2));
```

需要注意的是**即使输入 JSON 变量`s1` 和`s2`中属性的顺序不同，`equals()`方法也会忽略顺序，将它们视为相同。**

### 3.2.用嵌套元素比较两个 JSON 对象

接下来，我们将看到如何比较两个具有嵌套元素的 JSON 对象。

让我们从定义为`s1`变量的 JSON 开始:

```java
{ 
    "employee":
    {
        "id": "1212",
        "fullName":"John Miles",
        "age": 34,
        "contact":
        {
            "email": "[[email protected]](/web/20221129022100/https://www.baeldung.com/cdn-cgi/l/email-protection)",
            "phone": "9999999999"
        }
    }
}
```

正如我们所见，JSON 包含一个嵌套元素`contact`。

我们想将它与另一个由`s2`定义的 JSON 进行比较:

```java
{
    "employee":
    {
        "id": "1212",
        "age": 34,
        "fullName": "John Miles",
        "contact":
        {
            "email": "[[email protected]](/web/20221129022100/https://www.baeldung.com/cdn-cgi/l/email-protection)",
            "phone": "9999999999"
        }
    }
}
```

让我们将输入的 JSON 读作`JsonNode`并进行比较:

```java
assertEquals(mapper.readTree(s1), mapper.readTree(s2)); 
```

同样，我们应该注意到 **`equals()`也可以比较两个带有嵌套元素的输入 JSON 对象。**

### 3.3.比较两个包含列表元素的 JSON 对象

类似地，我们也可以比较两个包含列表元素的 JSON 对象。

让我们把这个 JSON 定义为`s1`:

```java
{
    "employee":
    {
        "id": "1212",
        "fullName": "John Miles",
        "age": 34,
        "skills": ["Java", "C++", "Python"]
    }
}
```

我们正在将其与另一个 JSON 进行比较，`s2`:

```java
{
    "employee":
    {
        "id": "1212",
        "age": 34,
        "fullName": "John Miles",
        "skills": ["Java", "C++", "Python"] 
    } 
}
```

让我们将输入的 JSON 读作`JsonNode`并进行比较:

```java
assertEquals(mapper.readTree(s1), mapper.readTree(s2));
```

重要的是要知道，只有当两个列表元素具有完全相同顺序的相同值时，它们才会被比较为相等。

## 4.用自定义比较器比较两个 JSON 对象

[`JsonNode.equals`](https://web.archive.org/web/20221129022100/https://static.javadoc.io/com.fasterxml.jackson.core/jackson-databind/2.9.8/com/fasterxml/jackson/databind/JsonNode.html#equals-java.lang.Object-) 在大多数情况下效果相当好。Jackson 还提供了 [`JsonNode.equals(comparator, JsonNode)`](https://web.archive.org/web/20221129022100/https://static.javadoc.io/com.fasterxml.jackson.core/jackson-databind/2.9.8/com/fasterxml/jackson/databind/JsonNode.html#equals-java.util.Comparator-com.fasterxml.jackson.databind.JsonNode-) 来配置一个定制的 Java `Comparator`对象。

让我们了解一下如何使用自定义`Comparator`。

### 4.1.用于比较数值的自定义比较器

让我们看看如何使用自定义的`Comparator `来比较两个具有数值的 JSON 元素。

我们将使用这个 JSON 作为输入`s1`:

```java
{
    "name": "John",
    "score": 5.0
}
```

我们来对比另一个定义为`s2`的 JSON:

```java
{
    "name": "John",
    "score": 5
}
```

我们需要观察输入`s1` 和`s2`中属性`score`的值**是不同的。**

让我们将输入的 JSON 读作`JsonNode`并进行比较:

```java
JsonNode actualObj1 = mapper.readTree(s1);
JsonNode actualObj2 = mapper.readTree(s2);

assertNotEquals(actualObj1, actualObj2);
```

请注意，这两个对象不相等。标准`equals()` 方法认为值 5.0 和 5 是不同的。

然而，**我们可以使用一个自定义的`Comparator`来比较值 5 和 5.0，并将它们视为相等。**

让我们首先创建一个`Comparator`来比较两个`NumericNode` 对象:

```java
public class NumericNodeComparator implements Comparator<JsonNode> 
{
    @Override
    public int compare(JsonNode o1, JsonNode o2)
    {
        if (o1.equals(o2)){
           return 0;
        }
        if ((o1 instanceof NumericNode) && (o2 instanceof NumericNode)){
            Double d1 = ((NumericNode) o1).asDouble();
            Double d2 = ((NumericNode) o2).asDouble(); 
            if (d1.compareTo(d2) == 0) {
               return 0;
            }
        }
        return 1;
    }
}
```

接下来，我们来看看如何使用这个`Comparator`:

```java
NumericNodeComparator cmp = new NumericNodeComparator();
assertTrue(actualObj1.equals(cmp, actualObj2));
```

### 4.2.用于比较文本值的自定义比较器

让我们来看另一个自定义`Comparator`的**例子，用于两个 JSON 值的不区分大小写的比较**。

我们将使用这个 JSON 作为输入`s1`:

```java
{
    "name": "john", 
    "score": 5 
}
```

我们来对比另一个定义为`s2`的 JSON:

```java
{ 
    "name": "JOHN", 
    "score": 5 
}
```

正如我们所见，属性`name`在输入`s1`中是小写的，在`s2`中是大写的。

让我们首先创建一个`Comparator`来比较两个`TextNode` 对象:

```java
public class TextNodeComparator implements Comparator<JsonNode> 
{
    @Override
    public int compare(JsonNode o1, JsonNode o2) {
        if (o1.equals(o2)) {
            return 0;
        }
        if ((o1 instanceof TextNode) && (o2 instanceof TextNode)) {
            String s1 = ((TextNode) o1).asText();
            String s2 = ((TextNode) o2).asText();
            if (s1.equalsIgnoreCase(s2)) {
                return 0;
            }
        }
        return 1;
    }
}
```

让我们看看如何使用`TextNodeComparator`来比较`s1`和`s2`:

```java
JsonNode actualObj1 = mapper.readTree(s1);
JsonNode actualObj2 = mapper.readTree(s2);

TextNodeComparator cmp = new TextNodeComparator();

assertNotEquals(actualObj1, actualObj2);
assertTrue(actualObj1.equals(cmp, actualObj2));
```

最后，我们可以看到，当输入的 JSON 元素值不完全相同，但我们仍然希望将它们视为相等时，在比较两个 JSON 对象**时使用定制的 comparator 对象会非常有用。**

## 5.结论

在这篇简短的文章中，我们看到了如何使用 Jackson 来比较两个 JSON 对象，并使用一个定制的比较器`. `

和往常一样，这里讨论的所有例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129022100/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-core)