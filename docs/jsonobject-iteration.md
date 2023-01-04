# 迭代 org.json.JSONObject 的实例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jsonobject-iteration>

## 1.介绍

在本教程中，我们将研究几种迭代一个简单的 Java JSON 表示`[JSONObject](/web/20221113082518/https://www.baeldung.com/java-org-json)`的方法。

我们将从一个简单的解决方案开始，然后看一些更强大的东西。

## 2.遍历一个`JSONObject`

让我们从迭代名称-值对的 JSON 的简单情况开始:

```
{
  "name": "Cake",
  "cakeId": "0001",
  "cakeShape": "Heart"
}
```

为此，我们可以使用`keys()`方法:简单地**遍历这些键**

```
void handleJSONObject(JSONObject jsonObject) {
    jsonObject.keys().forEachRemaining(key -> {
        Object value = jsonObject.get(key);
        logger.info("Key: {0}\tValue: {1}", key, value);
    }
}
```

我们的输出将是:

```
Key: name      Value: Cake
Key: cakeId    Value: 0001
Key: cakeShape Value: Heart
```

## 3.遍历一个`JSONObject`

但是假设我们有一个更复杂的结构:

```
{
  "batters": [
    {
      "type": "Regular",
      "id": "1001"
    },
    {
      "type": "Chocolate",
      "id": "1002"
    },
    {
      "type": "BlueBerry",
      "id": "1003"
    }
  ],
  "name": "Cake",
  "cakeId": "0001"
}
```

在这种情况下，遍历这些键意味着什么？

让我们看看我们天真的方法会给我们带来什么:

```
Key: batters    Value: [{"type":"Regular","id":"1001"},{"type":"Chocolate","id":"1002"},
  {"type":"BlueBerry","id":"1003"}]
Key: name       Value: Cake
Key: cakeId     Value: 0001
```

这可能没什么帮助。在这种情况下，我们想要的似乎不是迭代，而是遍历。

遍历一个`JSONObject`不同于迭代一个`JSONObject`的键集。

为此，**我们实际上也需要检查值类型。**假设我们用一种不同的方法来做这件事:

```
void handleValue(Object value) {
    if (value instanceof JSONObject) {
        handleJSONObject((JSONObject) value);
    } else if (value instanceof JSONArray) {
        handleJSONArray((JSONArray) value);
    } else {
        logger.info("Value: {0}", value);
    }
}
```

然后，我们的方法仍然相当相似:

```
void handleJSONObject(JSONObject jsonObject) {
    jsonObject.keys().forEachRemaining(key -> {
        Object value = jsonObject.get(key);
        logger.info("Key: {0}", key);
        handleValue(value);
    });
}
```

唯一的问题是我们需要考虑如何处理数组。

## 4.遍历一个`JSONArray`

让我们尝试保持使用迭代器的类似方法。**我们不叫`keys()`，而是叫`iterator()` :**

```
void handleJSONArray(JSONArray jsonArray) {
    jsonArray.iterator().forEachRemaining(element -> {
        handleValue(element)
    });
}
```

现在，这个解决方案是有局限性的，因为**我们将遍历与我们想要采取的动作**结合起来。区分这两者的一个常见方法是使用[访问者模式](/web/20221113082518/https://www.baeldung.com/java-visitor-pattern)。

## 5.结论

在本文中，我们看到了一种对简单的名称-值对遍历`JSONObject`的方法、与复杂结构相关的问题以及解决该问题的遍历技术。

当然，**这是一个深度优先的遍历方法，但是我们也可以用类似的方式来做广度优先的**。

该示例的完整代码可在 Github 上的[处获得。](https://web.archive.org/web/20221113082518/https://github.com/eugenp/tutorials/tree/master/json-modules/json)