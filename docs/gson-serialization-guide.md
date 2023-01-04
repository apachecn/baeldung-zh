# Gson 序列化食谱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gson-serialization-guide>

在本文中，我们将看看使用 Gson 库的最常见的序列化场景。

让我们从介绍一个简单的实体开始，我们将在下面的例子中使用它:

```
public class SourceClass {
    private int intValue;
    private String stringValue;

    // standard getters and setters
}
```

## 1。序列化实体数组

首先，让我们用 Gson 序列化一个对象数组:

```
@Test
public void givenArrayOfObjects_whenSerializing_thenCorrect() {
    SourceClass[] sourceArray = {new SourceClass(1, "one"), new SourceClass(2, "two")};
    String jsonString = new Gson().toJson(sourceArray);

    String expectedResult = 
      "[{"intValue":1,"stringValue":"one"},{"intValue":2,"stringValue":"two"}]";
    assertEquals(expectedResult, jsonString);
}
```

## 2。序列化实体集合

接下来，让我们对一组对象进行同样的操作:

```
@Test
public void givenCollection_whenSerializing_thenCorrect() {
    Collection<SourceClass> sourceCollection = 
      Lists.newArrayList(new SourceClass(1, "one"), new SourceClass(2, "two"));
    String jsonCollection = new Gson().toJson(sourceCollection);

    String expectedResult = 
      "[{"intValue":1,"stringValue":"one"},{"intValue":2,"stringValue":"two"}]";
    assertEquals(expectedResult, jsonCollection);
}
```

## 3。更改序列化实体的字段名称

接下来，让我们看看当我们序列化一个实体时，我们如何**改变字段**的名称。

我们将把包含字段`intValue`和`stringValue` 的实体序列化为带有`otherIntValue`和`otherStringValue`的 json:

```
@Test
public void givenUsingCustomSerializer_whenChangingNameOfFieldOnSerializing_thenCorrect() {
    SourceClass sourceObject = new SourceClass(7, "seven");
    GsonBuilder gsonBuildr = new GsonBuilder();
    gsonBuildr.registerTypeAdapter(SourceClass.class, new DifferentNameSerializer());
    String jsonString = gsonBuildr.create().toJson(sourceObject);

    String expectedResult = "{"otherIntValue":7,"otherStringValue":"seven"}";
    assertEquals(expectedResult, jsonString);
}
```

请注意，我们在这里使用自定义序列化程序来更改字段的名称:

```
public class DifferentNameSerializer implements JsonSerializer<SourceClass> {
    @Override
    public JsonElement serialize
      (SourceClass src, Type typeOfSrc, JsonSerializationContext context) {
        String otherIntValueName = "otherIntValue";
        String otherStringValueName = "otherStringValue";

        JsonObject jObject = new JsonObject();
        jObject.addProperty(otherIntValueName, src.getIntValue());
        jObject.addProperty(otherStringValueName, src.getStringValue());

        return jObject;
    }
}
```

## 4。序列化实体时忽略字段

现在让我们**在执行序列化时完全忽略字段**:

```
@Test
public void givenIgnoringAField_whenSerializingWithCustomSerializer_thenFieldIgnored() {
    SourceClass sourceObject = new SourceClass(7, "seven");
    GsonBuilder gsonBuildr = new GsonBuilder();
    gsonBuildr.registerTypeAdapter(SourceClass.class, new IgnoringFieldsSerializer());
    String jsonString = gsonBuildr.create().toJson(sourceObject);

    String expectedResult = "{"intValue":7}";
    assertEquals(expectedResult, jsonString);
}
```

与前面的例子类似，我们在这里也使用了一个定制的序列化器:

```
public class IgnoringFieldsSerializer implements JsonSerializer<SourceClass> {
    @Override
    public JsonElement serialize
      (SourceClass src, Type typeOfSrc, JsonSerializationContext context) {
        String intValue = "intValue";
        JsonObject jObject = new JsonObject();
        jObject.addProperty(intValue, src.getIntValue());

        return jObject;
    }
}
```

还要注意，在我们不能改变实体的源代码的情况下，或者如果该字段只应该在非常特殊的情况下被忽略，我们很可能需要这样做。否则，我们可以通过在实体类上直接注释来更容易地忽略该字段。

## 5。仅当字段通过自定义条件时，才对其进行序列化

最后，让我们来分析一个更高级的用例——我们希望只序列化一个通过特定自定义条件的字段。

例如，如果 int 值是正数，我们只序列化它，如果它是负数，我们就跳过它:

```
@Test
public void givenUsingCustomDeserializer_whenFieldNotMatchesCriteria_thenIgnored() {
    SourceClass sourceObject = new SourceClass(-1, "minus 1");
    GsonBuilder gsonBuildr = new GsonBuilder();
    gsonBuildr.registerTypeAdapter(SourceClass.class, 
      new IgnoringFieldsNotMatchingCriteriaSerializer());
    Gson gson = gsonBuildr.create();
    Type sourceObjectType = new TypeToken<SourceClass>() {}.getType();
    String jsonString = gson.toJson(sourceObject, sourceObjectType);

    String expectedResult = "{"stringValue":"minus 1"}";
    assertEquals(expectedResult, jsonString);
}
```

当然，我们在这里也使用了定制的序列化器:

```
public class IgnoringFieldsNotMatchingCriteriaSerializer 
  implements JsonSerializer<SourceClass> {
    @Override
    public JsonElement serialize
      (SourceClass src, Type typeOfSrc, JsonSerializationContext context) {
        JsonObject jObject = new JsonObject();

        // Criteria: intValue >= 0
        if (src.getIntValue() >= 0) {
            String intValue = "intValue";
            jObject.addProperty(intValue, src.getIntValue());
        }

        String stringValue = "stringValue";
        jObject.addProperty(stringValue, src.getStringValue());

        return jObject;
    }
}
```

这就是使用 Gson 的**序列化的 5 个常见用例。**