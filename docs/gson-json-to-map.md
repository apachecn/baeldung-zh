# 使用 Gson 将 JSON 转换成地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gson-json-to-map>

## 1.介绍

**在这个快速教程中，我们将学习如何使用谷歌**的 [`Gson`](https://web.archive.org/web/20220625072635/https://github.com/google/gson) 将 JSON 字符串转换成`Map`。

我们将看到实现这一目标的三种不同方法，并讨论它们的优缺点——以及一些实际例子。

## 2.通过`Map.class`

一般来说， **Gson 在它的`Gson`类中提供了以下 API 来将 JSON 字符串转换成对象**:

```java
public <T> T fromJson(String json, Class<T> classOfT) throws JsonSyntaxException;
```

从签名中可以清楚地看出，第二个参数是我们希望 JSON 解析到的对象的类。在我们的例子中，应该是`Map.class`:

```java
String jsonString = "{'employee.name':'Bob','employee.salary':10000}";
Gson gson = new Gson();
Map map = gson.fromJson(jsonString, Map.class);
Assert.assertEquals(2, map.size());
Assert.assertEquals(Double.class, map.get("employee.salary").getClass());
```

这种方法将对每个属性的值类型做出最佳猜测。

例如，数字将被强制转换为`Double` s，`true `和`false`将被强制转换为`Boolean`，对象将被强制转换为`LinkedTreeMap`

**但是，如果有重复的键，强制将失败，它将抛出一个`JsonSyntaxException.`**

而且，由于`[type erasure](/web/20220625072635/https://www.baeldung.com/java-type-erasure),` ，我们也不能配置这种强制行为。因此，如果我们需要指定键或值类型，那么我们需要一种不同的方法。

## 3。使用`TypeToken`

**为了克服泛型类型的类型删除问题，`Gson`有一个重载版本的 API** :

```java
public <T> T fromJson(String json, Type typeOfT) throws JsonSyntaxException;
```

我们可以使用 Gson 的`TypeToken`用它的类型参数构造一个`Map` 。**`TypeToken`类返回一个`ParameterizedTypeImpl`的实例，该实例即使在运行时也能保持键和值的类型**:

```java
String jsonString = "{'Bob' : {'name': 'Bob Willis'},"
  + "'Jenny' : {'name': 'Jenny McCarthy'}, "
  + "'Steve' : {'name': 'Steven Waugh'}}";
Gson gson = new Gson();
Type empMapType = new TypeToken<Map<String, Employee>>() {}.getType();
Map<String, Employee> nameEmployeeMap = gson.fromJson(jsonString, empMapType);
Assert.assertEquals(3, nameEmployeeMap.size());
Assert.assertEquals(Employee.class, nameEmployeeMap.get("Bob").getClass()); 
```

现在，如果我们将我们的`Map`类型构造为`Map<String, Object>`，那么解析器将仍然默认为我们在上一节中看到的那样。

当然，对于强制原始类型，这仍然要回到 Gson。然而，这些也可以定制。

## 4。使用`JsonDeserializer`自定义

**当我们需要对我们的`Map`对象的构造进行细粒度控制时，我们可以实现一个自定义的`JsonDeserializer<Map>.`** 类型的反序列化器

为了看一个例子，让我们假设我们的 JSON 包含雇员的名字作为键，他们的雇佣日期作为值。此外，让我们假设日期的格式是`yyyy/MM/dd`，这不是`Gson`的标准格式。

我们可以配置 Gson 以不同的方式解析我们的地图，然后通过实现一个`JsonDeserializer:`

```java
public class StringDateMapDeserializer implements JsonDeserializer<Map<String, Date>> {

    private SimpleDateFormat format = new SimpleDateFormat("yyyy/MM/dd");

    @Override
    public Map<String, Date> deserialize(JsonElement elem,
          Type type,
          JsonDeserializationContext jsonDeserializationContext) {
        return elem.getAsJsonObject()
          .entrySet()
          .stream()
          .filter(e -> e.getValue().isJsonPrimitive())
          .filter(e -> e.getValue().getAsJsonPrimitive().isString())
          .collect(
            Collectors.toMap(
              Map.Entry::getKey,
              e -> formatDate(e.getValue())));
    }

    private Date formatDate(Object value) {
        try {
            return format(value.getAsString());
        } catch (ParseException ex) {
            throw new JsonParseException(ex);
        }
    }
} 
```

现在，我们必须将它注册到目标类型`Map<String, Date` >的`GsonBuilder`中，并构建一个定制的`Gson`对象。

当我们在这个`Gson`对象上调用`fromJson` API 时，解析器调用定制的反序列化器并返回期望的`Map`实例:

```java
String jsonString = "{'Bob': '2017-06-01', 'Jennie':'2015-01-03'}";
Type type = new TypeToken<Map<String, Date>>(){}.getType();
Gson gson = new GsonBuilder()
  .registerTypeAdapter(type, new StringDateMapDeserializer())
  .create();
Map<String, Date> empJoiningDateMap = gson.fromJson(jsonString, type);
Assert.assertEquals(2, empJoiningDateMap.size());
Assert.assertEquals(Date.class, empJoiningDateMap.get("Bob").getClass()); 
```

当我们的映射可能包含不同种类的值，并且我们对可能有多少不同类型的值有一个合理的想法时，这种策略也很有用。

要了解更多关于`Gson`中自定义反序列化器的信息，请随意浏览 [Gson 反序列化食谱](/web/20220625072635/https://www.baeldung.com/gson-deserialization-guide)。

## 5。结论

在这篇短文中，我们学习了几种从 JSON 格式的字符串构建地图的方法。我们还讨论了这些变化的适当用例。

GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220625072635/https://github.com/eugenp/tutorials/tree/master/json-modules/gson)