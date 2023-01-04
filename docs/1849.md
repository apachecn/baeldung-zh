# 检查一个字符串在 Java 中是否是有效的 JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-validate-json-string>

## 1.概观

在 Java 中处理原始 JSON 值时，有时需要检查它是否有效。有几个库可以帮助我们做到这一点: [Gson](/web/20221025152558/https://www.baeldung.com/gson-deserialization-guide) 、 [JSON API](/web/20221025152558/https://www.baeldung.com/java-org-json) 和 [Jackson](/web/20221025152558/https://www.baeldung.com/jackson) 。每种工具都有自己的优势和局限性。

在本教程中，我们将使用它们中的每一个来实现 JSON `String`验证，并通过实际的例子来仔细观察这两种方法之间的主要区别。

## 2.用 JSON API 进行验证

最轻量级和最简单的库是 JSON API。

检查`String`是否是有效 JSON 的常用方法是异常处理。因此，**我们委托 JSON 解析，并在值不正确的情况下处理特定类型的错误，或者假设值是正确的，如果没有异常发生**。

### 2.1. **Maven 依赖**

首先，我们需要在我们的`pom.xml`中包含`[json](https://web.archive.org/web/20221025152558/https://mvnrepository.com/artifact/org.json/json) `依赖项:

```
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20211205</version>
</dependency>
```

### 2.2.**用`JSONObject`验证**

首先，为了检查`String`是否是 JSON，我们将尝试进一步创建一个`JSONObject.`，如果是无效值，我们将得到一个`JSONException:`

```
public boolean isValid(String json) {
    try {
        new JSONObject(json);
    } catch (JSONException e) {
        return false;
    }
    return true;
}
```

让我们用一个简单的例子来尝试一下:

```
String json = "{\"email\": \"[[email protected]](/web/20221025152558/https://www.baeldung.com/cdn-cgi/l/email-protection)\", \"name\": \"John\"}";
assertTrue(validator.isValid(json));
```

```
String json = "Invalid_Json"; 
assertFalse(validator.isValid(json));
```

然而，这种方法的缺点是 **`String`只能是一个对象，而不能是使用`JSONObject`的数组。**

例如，让我们看看它是如何处理数组的:

```
String json = "[{\"email\": \"[[email protected]](/web/20221025152558/https://www.baeldung.com/cdn-cgi/l/email-protection)\", \"name\": \"John\"}]";
assertFalse(validator.isValid(json));
```

### 2.3.**用`JSONArray`验证**

为了验证`String` 是对象还是数组，如果`JSONObject`创建失败，我们需要添加一个附加条件。 同样，**`JSONArray`也会抛出一个****`JSONException`****如果`String`不适合 JSON 数组**的话:

```
public boolean isValid(String json) {
    try {
        new JSONObject(json);
    } catch (JSONException e) {
        try {
            new JSONArray(json);
        } catch (JSONException ne) {
            return false;
        }
    }
    return true;
}
```

因此，我们可以验证任何值:

```
String json = "[{\"email\": \"[[email protected]](/web/20221025152558/https://www.baeldung.com/cdn-cgi/l/email-protection)\", \"name\": \"John\"}]";
assertTrue(validator.isValid(json));
```

## 3.与杰克逊确认

类似地，Jackson 库提供了一种基于`Exception`处理来验证 JSON 的方法。它是一个更复杂的工具，有许多类型的解析策略。然而，它更容易使用。

### 3.1. **Maven 依赖**

让我们添加 [`jackson-databind`](https://web.archive.org/web/20221025152558/https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind) 美芬依赖:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

### 3.2.**用`ObjectMapper`验证**

**我们使用`readTree()`方法读取整个 JSON，如果语法不正确，就得到一个`JacksonException`。**

换句话说，我们不需要提供额外的检查。它对对象和数组都有效:

```
ObjectMapper mapper = new ObjectMapper();

public boolean isValid(String json) {
    try {
        mapper.readTree(json);
    } catch (JacksonException e) {
        return false;
    }
    return true;
}
```

让我们通过例子来看看如何使用它:

```
String json = "{\"email\": \"[[email protected]](/web/20221025152558/https://www.baeldung.com/cdn-cgi/l/email-protection)\", \"name\": \"John\"}";
assertTrue(validator.isValid(json));

String json = "[{\"email\": \"[[email protected]](/web/20221025152558/https://www.baeldung.com/cdn-cgi/l/email-protection)\", \"name\": \"John\"}]";
assertTrue(validator.isValid(json));

String json = "Invalid_Json";
assertFalse(validator.isValid(json));
```

## 4.Gson 验证

Gson 是另一个公共库，它允许我们使用相同的方法验证原始 JSON 值。这是一个复杂的工具，用于不同类型的 JSON 处理的 Java 对象映射。

### 4.1. **Maven 依赖**

让我们添加 [`gson`](https://web.archive.org/web/20221025152558/https://mvnrepository.com/artifact/com.google.code.gson/gson) 美芬依赖:

```
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```

### 4.2.非严格**验证**

Gson 提供了`JsonParser`来将指定的 JSON 读入到一个由`JsonElement`组成的树中。因此，它保证了如果在读取过程中出现错误，我们会得到`JsonSyntaxException`。

因此，在 JSON 值不正确的情况下，我们可以使用`parse()`方法计算`String`并处理`Exception`:

```
public boolean isValid(String json) {
    try {
        JsonParser.parseString(json);
    } catch (JsonSyntaxException e) {
        return false;
    }
    return true;
}
```

让我们编写一些测试来检查主要情况:

```
String json = "{\"email\": \"[[email protected]](/web/20221025152558/https://www.baeldung.com/cdn-cgi/l/email-protection)\", \"name\": \"John\"}";
assertTrue(validator.isValid(json));

String json = "[{\"email\": \"[[email protected]](/web/20221025152558/https://www.baeldung.com/cdn-cgi/l/email-protection)\", \"name\": \"John\"}]";
assertTrue(validator.isValid(json));
```

**这种方法的主要区别在于，Gson 默认策略认为单独的字符串和数值作为`JsonElement`节点的一部分是有效的。换句话说，它认为单个字符串或数字也是有效的。**

例如，让我们看看它是如何处理单个字符串的:

```
String json = "Invalid_Json";
assertTrue(validator.isValid(json));
```

然而，如果我们想认为这样的值是畸形的，我们需要在我们的`JsonParser`上执行一个严格的类型策略。

### 4.3.严格的**验证**

为了实现严格的类型策略，我们创建了一个`TypeAdapter`并将`JsonElement`类定义为必需的类型匹配。因此，`JsonParser` 会抛出一个 `JsonSyntaxException`如果类型不是 JSON 对象或者数组。

我们可以调用`fromJson()`方法来使用特定的`TypeAdapter`读取原始 JSON:

```
final TypeAdapter<JsonElement> strictAdapter = new Gson().getAdapter(JsonElement.class);

public boolean isValid(String json) {
    try {
        strictAdapter.fromJson(json);
    } catch (JsonSyntaxException | IOException e) {
        return false;
    }
    return true;
}
```

最后，我们可以检查 JSON 是否有效:

```
String json = "Invalid_Json";
assertFalse(validator.isValid(json));
```

## 5.结论

在本文中，我们看到了检查`String`是否是有效 JSON 的不同方法。

每种方法都有其优点和局限性。虽然 JSON API 可以用于简单的对象验证，但是作为 JSON 对象的一部分，Gson 可以更好地扩展原始值验证。然而，杰克逊更容易使用。所以要用比较贴合的。

此外，我们应该检查一些库是否已经被使用，或者是否也适用于其余的目标。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221025152558/https://github.com/eugenp/tutorials/tree/master/json-modules/json-2)