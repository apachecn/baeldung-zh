# 用 Jackson 实现映射序列化和反序列化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-map>

## 1。概述

在这个快速教程中，我们将使用[Jackson](https://web.archive.org/web/20220912110321/https://github.com/FasterXML/jackson)来查看 Java 映射的**序列化和反序列化。**

我们将说明如何在 JSON 格式的`Strings.`之间序列化和反序列化`Map<String, String>`、`Map<Object, String>,`和`Map<Object, Object>`

## 延伸阅读:

## [Jackson–使用地图和空值](/web/20220912110321/https://www.baeldung.com/jackson-map-null-values-or-null-key)

How to serialize Maps with a null key or null values using Jackson.[Read more](/web/20220912110321/https://www.baeldung.com/jackson-map-null-values-or-null-key) →

## [如何用 Jackson 序列化和反序列化枚举](/web/20220912110321/https://www.baeldung.com/jackson-serialize-enums)

How to serialize and deserialize an Enum as a JSON Object using Jackson 2.[Read more](/web/20220912110321/https://www.baeldung.com/jackson-serialize-enums) →

## [使用 Jackson 进行 XML 序列化和反序列化](/web/20220912110321/https://www.baeldung.com/jackson-xml-serialization-and-deserialization)

This short tutorial shows how the Jackson library can be used to serialize Java object to XML and deserialize them back to objects.[Read more](/web/20220912110321/https://www.baeldung.com/jackson-xml-serialization-and-deserialization) →

## 2。Maven 配置

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

我们可以在这里得到杰克逊的最新版本。

## 3。序列化

序列化将 Java 对象转换成字节流，可以根据需要持久化或共享。Java `Maps`是将键`Object`映射到值`Object,`的集合，通常是最不直观的序列化对象。

### 3.1。`Map<String, String>`连载

对于一个简单的例子，让我们创建一个`Map<String, String>`并将其序列化为 JSON:

```java
Map<String, String> map = new HashMap<>();
map.put("key", "value");

ObjectMapper mapper = new ObjectMapper();
String jsonResult = mapper.writerWithDefaultPrettyPrinter()
  .writeValueAsString(map);
```

`ObjectMapper` 是杰克逊的序列化映射器。它允许我们序列化我们的`map,`，并使用`String`中的`toString()`方法将它写成一个漂亮的 JSON `String`:

```java
{
  "key" : "value"
}
```

### 3.2。 `Map<Object, String>`连载

通过一些额外的步骤，我们还可以序列化一个包含定制 Java 类的映射。让我们创建一个`MyPair`类来表示一对相关的`String`对象。

注意:getter/setter 应该是公共的，我们用`@JsonValue` 注释`toString()` 以确保 Jackson 在序列化时使用这个自定义的`toString()`:

```java
public class MyPair {

    private String first;
    private String second;

    @Override
    @JsonValue
    public String toString() {
        return first + " and " + second;
    }

    // standard getter, setters, equals, hashCode, constructors
}
```

然后我们会告诉 Jackson 如何通过扩展 Jackson 的`JsonSerializer`来序列化`MyPair`:

```java
public class MyPairSerializer extends JsonSerializer<MyPair> {

    private ObjectMapper mapper = new ObjectMapper();

    @Override
    public void serialize(MyPair value, 
      JsonGenerator gen,
      SerializerProvider serializers) 
      throws IOException, JsonProcessingException {

        StringWriter writer = new StringWriter();
        mapper.writeValue(writer, value);
        gen.writeFieldName(writer.toString());
    }
}
```

`JsonSerializer`顾名思义，使用`MyPair`的`toString()`方法将`MyPair`序列化为 JSON。此外，Jackson 提供了许多[序列化器类](https://web.archive.org/web/20220912110321/https://github.com/FasterXML/jackson-databind/blob/master/docs/javadoc/2.3/com/fasterxml/jackson/databind/ser/std/package-summary.html)来满足我们的序列化需求。

接下来，我们用`@JsonSerialize`注释将`MyPairSerializer`应用到我们的`Map<MyPair, String>`。注意，我们只告诉 Jackson 如何序列化`MyPair` ，因为它已经知道如何序列化`String:`

```java
@JsonSerialize(keyUsing = MyPairSerializer.class) 
Map<MyPair, String> map;
```

然后让我们测试我们的地图序列化:

```java
map = new HashMap<>();
MyPair key = new MyPair("Abbott", "Costello");
map.put(key, "Comedy");

String jsonResult = mapper.writerWithDefaultPrettyPrinter()
  .writeValueAsString(map);
```

序列化的 JSON 输出是:

```java
{
  "Abbott and Costello" : "Comedy"
}
```

### 3.3。`Map<Object, Object>`连载

最复杂的情况是序列化一个`Map<Object, Object>`，但是大部分工作已经完成了。让我们使用杰克森的`MapSerializer` 作为我们的映射，使用上一节的`MyPairSerializer,`作为映射的键和值类型:

```java
@JsonSerialize(keyUsing = MapSerializer.class)
Map<MyPair, MyPair> map;

@JsonSerialize(keyUsing = MyPairSerializer.class)
MyPair mapKey;

@JsonSerialize(keyUsing = MyPairSerializer.class)
MyPair mapValue;
```

然后让我们测试序列化我们的`Map<MyPair, MyPair>`:

```java
mapKey = new MyPair("Abbott", "Costello");
mapValue = new MyPair("Comedy", "1940s");
map.put(mapKey, mapValue);

String jsonResult = mapper.writerWithDefaultPrettyPrinter()
  .writeValueAsString(map);
```

使用`MyPair`的`toString()`方法的序列化 JSON 输出是:

```java
{
  "Abbott and Costello" : "Comedy and 1940s"
}
```

## 4。反序列化

反序列化将字节流转换成我们可以在代码中使用的 Java 对象。在这一节中，我们将把 JSON 输入反序列化到不同签名的`Map` `s` 中。

### 4.1。`Map<String, String>`反序列化

举个简单的例子，让我们取一个 JSON 格式的输入字符串，并将其转换成一个`Map<String, String>` Java 集合:

```java
String jsonInput = "{\"key\": \"value\"}";
TypeReference<HashMap<String, String>> typeRef 
  = new TypeReference<HashMap<String, String>>() {};
Map<String, String> map = mapper.readValue(jsonInput, typeRef);
```

我们使用 Jackson 的`ObjectMapper,`作为序列化，使用`readValue()` 处理输入。此外，请注意我们对 Jackson 的`TypeReference`的使用，我们将在所有的反序列化示例中使用它来描述目的地的类型`Map`。这是我们地图的`toString()` 表示:

```java
{key=value}
```

### 4.2。`Map<Object, String>`反序列化

现在让我们将输入 JSON 和目的地的`TypeReference` 改为 `Map<MyPair, String>`:

```java
String jsonInput = "{\"Abbott and Costello\" : \"Comedy\"}";

TypeReference<HashMap<MyPair, String>> typeRef 
  = new TypeReference<HashMap<MyPair, String>>() {};
Map<MyPair,String> map = mapper.readValue(jsonInput, typeRef);
```

我们需要为`MyPair`创建一个构造函数，它接受带有两个元素的`String`,并将它们解析为`MyPair`元素:

```java
public MyPair(String both) {
    String[] pairs = both.split("and");
    this.first = pairs[0].trim();
    this.second = pairs[1].trim();
}
```

我们的`Map<MyPair,String>`对象的`toString()`是:

```java
{Abbott and Costello=Comedy}
```

**当我们反序列化为包含`Map;` 的 Java 类时，还有另一种选择，我们可以使用 Jackson 的`KeyDeserializer`类**，Jackson 提供的众多[反序列化](https://web.archive.org/web/20220912110321/https://github.com/FasterXML/jackson-databind/blob/master/docs/javadoc/2.3/com/fasterxml/jackson/databind/deser/package-summary.html)类中的一个。让我们用`@JsonCreator`、`@JsonProperty`和`@JsonDeserialize:`来注释我们的`ClassWithAMap`

```java
public class ClassWithAMap {

  @JsonProperty("map")
  @JsonDeserialize(keyUsing = MyPairDeserializer.class)
  private Map<MyPair, String> map;

  @JsonCreator
  public ClassWithAMap(Map<MyPair, String> map) {
    this.map = map;
  }

  // public getters/setters omitted
}
```

在这里，我们告诉 Jackson 对包含在`ClassWithAMap`中的`Map<MyPair, String>` 进行反序列化，所以我们需要扩展`KeyDeserializer` 来描述如何从输入`String`中反序列化 map 的键，一个`MyPair` 对象:

```java
public class MyPairDeserializer extends KeyDeserializer {

  @Override
  public MyPair deserializeKey(
    String key, 
    DeserializationContext ctxt) throws IOException, 
    JsonProcessingException {

      return new MyPair(key);
    }
}
```

然后我们可以使用`readValue`测试反序列化:

```java
String jsonInput = "{\"Abbott and Costello\":\"Comedy\"}";

ClassWithAMap classWithMap = mapper.readValue(jsonInput,
  ClassWithAMap.class);
```

同样，我们的`ClassWithAMap's`映射的`toString()`方法给出了我们期望的输出:

```java
{Abbott and Costello=Comedy}
```

### 4.3。`Map<Object,Object>`反序列化

最后，让我们将输入 JSON 和目的地的`TypeReference`改为`Map<MyPair, MyPair>`:

```java
String jsonInput = "{\"Abbott and Costello\" : \"Comedy and 1940s\"}";
TypeReference<HashMap<MyPair, MyPair>> typeRef 
  = new TypeReference<HashMap<MyPair, MyPair>>() {};
Map<MyPair,MyPair> map = mapper.readValue(jsonInput, typeRef);
```

我们的`Map<MyPair, MyPair>`对象的`toString()`是:

```java
{Abbott and Costello=Comedy and 1940s}
```

## 5。结论

在这篇简短的文章中，我们学习了如何在 JSON 格式的字符串之间序列化和反序列化 Java `Maps`。

和往常一样，本文提供的例子可以在 [GitHub 库](https://web.archive.org/web/20220912110321/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions)中找到。