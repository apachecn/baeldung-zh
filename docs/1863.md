# Gson 反序列化指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gson-deserialization-guide>

在这本食谱中，我们正在探索使用流行的 [Gson 库](https://web.archive.org/web/20221107090238/https://code.google.com/p/google-gson/)将 JSON 解组为 Java 对象的各种方法。

## 1。将 JSON 反序列化为单个基本对象

让我们从简单的开始——我们将**把一个简单的 json 解组到一个 Java 对象—`Foo`**:

```
public class Foo {
    public int intValue;
    public String stringValue;

    // + standard equals and hashCode implementations
}
```

解决方案是:

```
@Test
public void whenDeserializingToSimpleObject_thenCorrect() {
    String json = "{"intValue":1,"stringValue":"one"}";

    Foo targetObject = new Gson().fromJson(json, Foo.class);

    assertEquals(targetObject.intValue, 1);
    assertEquals(targetObject.stringValue, "one");
}
```

## 延伸阅读:

## [从 Gson 中的序列化中排除字段](/web/20221107090238/https://www.baeldung.com/gson-exclude-fields-serialization)

Explore the options available to exclude fields from serialization in Gson.[Read more](/web/20221107090238/https://www.baeldung.com/gson-exclude-fields-serialization) →

## [Gson 连载食谱](/web/20221107090238/https://www.baeldung.com/gson-serialization-guide)

Learn how to serialize entities using the Gson library.[Read more](/web/20221107090238/https://www.baeldung.com/gson-serialization-guide) →

## [杰克逊 vs Gson](/web/20221107090238/https://www.baeldung.com/jackson-vs-gson)

Quick and practical guide to serialization with Jackson and Gson.[Read more](/web/20221107090238/https://www.baeldung.com/jackson-vs-gson) →

## 2。将 JSON 反序列化为通用对象

接下来，让我们使用泛型定义一个对象:

```
public class GenericFoo<T> {
    public T theValue;
}
```

并将一些 json 解组到这种类型的对象中:

```
@Test
public void whenDeserializingToGenericObject_thenCorrect() {
    Type typeToken = new TypeToken<GenericFoo<Integer>>() { }.getType();
    String json = "{"theValue":1}";

    GenericFoo<Integer> targetObject = new Gson().fromJson(json, typeToken);

    assertEquals(targetObject.theValue, new Integer(1));
}
```

## 3。将带有额外未知字段的 JSON 反序列化到对象

接下来，让我们反序列化一些复杂的 json，它包含额外的**未知字段**:

```
@Test
public void givenJsonHasExtraValues_whenDeserializing_thenCorrect() {
    String json = 
      "{"intValue":1,"stringValue":"one","extraString":"two","extraFloat":2.2}";
    Foo targetObject = new Gson().fromJson(json, Foo.class);

    assertEquals(targetObject.intValue, 1);
    assertEquals(targetObject.stringValue, "one");
}
```

如你所见， **Gson 将忽略未知字段**并简单地匹配它能够匹配的字段。

## 4。将带有不匹配字段名的 JSON 反序列化到对象

现在，让我们看看 Gson 如何处理包含与我们的`Foo`对象的字段完全不匹配的字段的 json 字符串:

```
@Test
public void givenJsonHasNonMatchingFields_whenDeserializingWithCustomDeserializer_thenCorrect() {
    String json = "{"valueInt":7,"valueString":"seven"}";

    GsonBuilder gsonBldr = new GsonBuilder();
    gsonBldr.registerTypeAdapter(Foo.class, new FooDeserializerFromJsonWithDifferentFields());
    Foo targetObject = gsonBldr.create().fromJson(json, Foo.class);

    assertEquals(targetObject.intValue, 7);
    assertEquals(targetObject.stringValue, "seven");
}
```

注意，我们注册了一个定制的反序列化器——它能够正确地解析出 json 字符串中的字段，并将它们映射到我们的`Foo`:

```
public class FooDeserializerFromJsonWithDifferentFields implements JsonDeserializer<Foo> {

    @Override
    public Foo deserialize
      (JsonElement jElement, Type typeOfT, JsonDeserializationContext context) 
      throws JsonParseException {
        JsonObject jObject = jElement.getAsJsonObject();
        int intValue = jObject.get("valueInt").getAsInt();
        String stringValue = jObject.get("valueString").getAsString();
        return new Foo(intValue, stringValue);
    }
}
```

## 5。将 JSON 数组反序列化为 Java 对象数组

接下来，我们将把 json 数组反序列化为对象的 Java 数组 T2:

```
@Test
public void givenJsonArrayOfFoos_whenDeserializingToArray_thenCorrect() {
    String json = "[{"intValue":1,"stringValue":"one"}," +
      "{"intValue":2,"stringValue":"two"}]";
    Foo[] targetArray = new GsonBuilder().create().fromJson(json, Foo[].class);

    assertThat(Lists.newArrayList(targetArray), hasItem(new Foo(1, "one")));
    assertThat(Lists.newArrayList(targetArray), hasItem(new Foo(2, "two")));
    assertThat(Lists.newArrayList(targetArray), not(hasItem(new Foo(1, "two"))));
}
```

## 6。将 JSON 数组反序列化为 Java 集合

接下来，将 json 数组**直接放入 Java 集合**:

```
@Test
public void givenJsonArrayOfFoos_whenDeserializingCollection_thenCorrect() {
    String json = 
      "[{"intValue":1,"stringValue":"one"},{"intValue":2,"stringValue":"two"}]";
    Type targetClassType = new TypeToken<ArrayList<Foo>>() { }.getType();

    Collection<Foo> targetCollection = new Gson().fromJson(json, targetClassType);
    assertThat(targetCollection, instanceOf(ArrayList.class));
}
```

## 7。将 JSON 反序列化为嵌套对象

接下来，让我们定义我们的嵌套对象——**`FooWithInner`**:

```
public class FooWithInner {
    public int intValue;
    public String stringValue;
    public InnerFoo innerFoo;

    public class InnerFoo {
        public String name;
    }
}
```

下面是如何反序列化包含该嵌套对象的输入:

```
@Test
public void whenDeserializingToNestedObjects_thenCorrect() {
    String json = "{\"intValue\":1,\"stringValue\":\"one\",\"innerFoo\":{\"name\":\"inner\"}}";

    FooWithInner targetObject = new Gson().fromJson(json, FooWithInner.class);

    assertEquals(targetObject.intValue, 1);
    assertEquals(targetObject.stringValue, "one");
    assertEquals(targetObject.innerFoo.name, "inner");
}
```

## 8。使用自定义构造函数反序列化 JSON

最后，让我们看看如何在反序列化期间强制使用特定的构造函数，而不是默认的——无参数构造函数——使用 **`InstanceCreator`** :

```
public class FooInstanceCreator implements InstanceCreator<Foo> {

    @Override
    public Foo createInstance(Type type) {
        return new Foo("sample");
    }
}
```

下面是如何在反序列化中使用我们的`FooInstanceCreator`:

```
@Test
public void whenDeserializingUsingInstanceCreator_thenCorrect() {
    String json = "{\"intValue\":1}";

    GsonBuilder gsonBldr = new GsonBuilder();
    gsonBldr.registerTypeAdapter(Foo.class, new FooInstanceCreator());
    Foo targetObject = gsonBldr.create().fromJson(json, Foo.class);

    assertEquals(targetObject.intValue, 1);
    assertEquals(targetObject.stringValue, "sample");
}
```

注意，当我们使用下面的构造函数时，`Foo.stringValue`等于`sample`,而不是 null:

```
public Foo(String stringValue) {
    this.stringValue = stringValue;
}
```

## 9。结论

本文展示了如何利用 Gson 库**解析 JSON 输入**——回顾了单个和多个对象的最常见用例。

所有这些示例和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20221107090238/https://github.com/eugenp/tutorials/tree/master/json-modules/gson "Github Project containing all the Gson samples") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。