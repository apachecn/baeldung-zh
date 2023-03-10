# 获取 JSONObject 中的值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jsonobject-get-value>

## 1.介绍

在本教程中，我们将深入研究在`JSONObject`实例中获取值的细节。

关于 Java 中 JSON 支持的一般介绍，请查看[JSON-Java 简介](/web/20221221184057/https://www.baeldung.com/java-org-json "Introduction to JSON-Java")。

## 2.`JSONObject`结构

**`JSONObject`是一个类似地图的结构**。它将其数据保存为一组键值对。**虽然键属于`String`类型，但值可能属于几种类型**。**此外，值类型可以是原始的或复合的**。原语是`String`、`Number, and`、`Boolean types,`或`JSONObject.NULL`对象。化合物是`JSONObject`和`JSONArray`类型。因此，JSON 数据可能具有任意的复杂性和嵌套。

因此，在嵌套结构中获取值需要更多的工作。从这一点开始，我们来参考以下一个假想员工的 JSON 数据:

```java
{
  "name" : "Bob",
  "profession" : "Software engineer",
  "department" : "Research",
  "age" : 40,
  "family" : [
    {
      "wife" : {
        "name" : "Alice",
        "profession" : "Doctor",
        "age" : 38
      }
    },
    {
      "son" : {
        "name" : "Peter",
        "occupation" : "Schoolboy",
        "age" : 11
      }
    }
  ],
  "performance" : [
    {
      "2020" : 4.5
    },
    {
      "2021" : 4.8
    }
  ]
}
```

## 3.`JSONObject`的获取方法

首先，让我们看看`JSONObject`类提供了什么 getter API。**有两组方法——`get()`和`opt()`方法**。这两个组的区别在于，`get()`方法在找不到键时抛出，而`opt()`方法不抛出，根据方法返回 null 或特定值。

此外，每个组都有一个泛型方法和几个特定的类型转换方法。**泛型方法返回一个`Object`实例，而特定方法返回一个已经转换的实例**。让我们使用通用的`get()`方法获取 JSON 数据的“family”字段。我们假设 JSON 数据已经预先加载到`jsonObject`变量中，变量的类型为`JSONObject`:

```java
 JSONArray family = (JSONArray) jsonObject.get("family");
```

我们可以用一种更易读的方式来做同样的事情，为`JSONArray`使用特定的 getter:

```java
 JSONArray family = jsonObject.getJSONArray("family");
```

## 4.直接获取值

**在这种方法中，我们通过获取所需值**路径上的每个中间值来直接获取值。下面的代码显示了如何直接获得雇员儿子的姓名:

```java
 JSONArray family = jsonObject.getJSONArray("family");
    JSONObject sonObject = family.getJSONObject(1);
    JSONObject sonData = sonObject.getJSONObject("son");
    String sonName = sonData.getString("name");
    Assertions.assertEquals(sonName, "Peter");
```

**我们可以看到，直接获取数值的方法有局限性**。首先，我们需要知道 JSON 数据的确切结构。其次，我们需要知道每个值的数据类型，以便使用正确的 getter 方法`JSONObject.`

**此外，当 JSON 数据的结构是动态的**时，我们需要在代码中添加彻底的检查。例如，对于所有的`get()`方法，我们需要将代码放在`try-catch`块中。此外，对于`opt()`方法，我们需要添加空值或特定值检查。

## 5.递归获取值

相比之下，在 JSON 数据中获取值的递归方法更灵活，更不容易出错。在实现这种方法时，我们需要考虑 JSON 数据的嵌套结构。

首先，当一个键的值是`JSONObject`或`JSONArray`类型时，我们需要在该值中向下传播递归搜索。第二，当在当前递归调用中找到键时，我们需要将它的映射值添加到返回的结果中，而不管该值是否属于基元类型。

以下方法实现递归搜索:

```java
 public List<String> getValuesInObject(JSONObject jsonObject, String key) {
        List<String> accumulatedValues = new ArrayList<>();
        for (String currentKey : jsonObject.keySet()) {
            Object value = jsonObject.get(currentKey);
            if (currentKey.equals(key)) {
                accumulatedValues.add(value.toString());
            }

            if (value instanceof JSONObject) {
                accumulatedValues.addAll(getValuesInObject((JSONObject) value, key));
            } else if (value instanceof JSONArray) {
                accumulatedValues.addAll(getValuesInArray((JSONArray) value, key));
            }
        }

        return accumulatedValues;
    }

    public List<String> getValuesInArray(JSONArray jsonArray, String key) {
        List<String> accumulatedValues = new ArrayList<>();
        for (Object obj : jsonArray) {
            if (obj instanceof JSONArray) {
                accumulatedValues.addAll(getValuesInArray((JSONArray) obj, key));
            } else if (obj instanceof JSONObject) {
                accumulatedValues.addAll(getValuesInObject((JSONObject) obj, key));
            }
        }

        return accumulatedValues;
    }
```

为了简单起见，我们提供了两个独立的方法:一个用于在`JSONObject`实例中进行递归搜索，另一个用于在`JSONArray`实例中进行递归搜索。`JSONObject`是类似地图的结构，而`JSONArray`是类似阵列的结构。因此，迭代对于他们是不同的。因此，将所有逻辑放在一个方法中会使代码因类型转换和 if-else 分支而变得复杂。

最后，让我们为``getValuesInObject()`` 方法编写测试代码:

```java
 @Test
    public void getAllAssociatedValuesRecursively() {
        List<String> values = jsonObjectValueGetter.getValuesInObject(jsonObject, "son");
        Assertions.assertEquals(values.size(), 1);

        String sonString = values.get(0);
        Assertions.assertTrue(sonString.contains("Peter"));
        Assertions.assertTrue(sonString.contains("Schoolboy"));
        Assertions.assertTrue(sonString.contains("11"));

        values = jsonObjectValueGetter.getValuesInObject(jsonObject, "name");
        Assertions.assertEquals(values.size(), 3);

        Assertions.assertEquals(values.get(0), "Bob");
        Assertions.assertEquals(values.get(1), "Alice");
        Assertions.assertEquals(values.get(2), "Peter");
    }
```

## 6.结论

在本文中，我们已经讨论了在`JSONObject. `中获取值，讨论中使用的片段的完整代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20221221184057/https://github.com/eugenp/tutorials/tree/master/json-modules/json-2)