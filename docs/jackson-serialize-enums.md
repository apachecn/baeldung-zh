# 如何用 Jackson 序列化和反序列化枚举

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-serialize-enums>

## 1。概述

在这个快速教程中，我们将学习如何控制用 Jackson 2 序列化和反序列化 **Java 枚举的方式。**

为了更深入地挖掘和学习**我们可以用杰克逊 2 做的其他很酷的事情，**前往[杰克逊的主要教程](/web/20220628093902/https://www.baeldung.com/jackson "The main Jackson Tutorial")。

## 2。控制枚举表示

让我们定义以下枚举:

```java
public enum Distance {
    KILOMETER("km", 1000), 
    MILE("miles", 1609.34),
    METER("meters", 1), 
    INCH("inches", 0.0254),
    CENTIMETER("cm", 0.01), 
    MILLIMETER("mm", 0.001);

    private String unit;
    private final double meters;

    private Distance(String unit, double meters) {
        this.unit = unit;
        this.meters = meters;
    }

    // standard getters and setters
}
```

## 3。将枚举序列化到 JSON

### 3.1。默认枚举表示

默认情况下，Jackson 将 Java 枚举表示为一个简单的字符串。例如:

```java
new ObjectMapper().writeValueAsString(Distance.MILE);
```

将导致:

```java
"MILE"
```

然而，当将这个**枚举封送到 JSON 对象**时，我们希望得到类似这样的结果:

```java
{"unit":"miles","meters":1609.34} 
```

### 3.2。作为 JSON 对象的枚举

从 Jackson 2.1.2 开始，现在有一个配置选项可以处理这种表示。这可以通过类级别的`@JsonFormat`注释来完成:

```java
@JsonFormat(shape = JsonFormat.Shape.OBJECT)
public enum Distance { ... }
```

当序列化此`enum`英里的`Distance.`时，这将导致期望的结果:

```java
{"unit":"miles","meters":1609.34}
```

### 3.3。`@JsonValue`列举和

控制枚举的封送输出的另一个简单方法是在 getter 上使用`@JsonValue`注释:

```java
public enum Distance { 
    ...

    @JsonValue
    public String getMeters() {
        return meters;
    }
}
```

我们在这里表达的是`getMeters()`是这个枚举的实际表示。因此序列化的结果将是:

```java
1609.34
```

### 3.4。枚举的自定义序列化程序

如果我们使用的是 2.1.2 之前的 Jackson 版本，或者如果 enum 需要更多的定制，我们可以使用**定制 Jackson 序列化程序。**首先，我们需要定义它:

```java
public class DistanceSerializer extends StdSerializer {

    public DistanceSerializer() {
        super(Distance.class);
    }

    public DistanceSerializer(Class t) {
        super(t);
    }

    public void serialize(
      Distance distance, JsonGenerator generator, SerializerProvider provider) 
      throws IOException, JsonProcessingException {
        generator.writeStartObject();
        generator.writeFieldName("name");
        generator.writeString(distance.name());
        generator.writeFieldName("unit");
        generator.writeString(distance.getUnit());
        generator.writeFieldName("meters");
        generator.writeNumber(distance.getMeters());
        generator.writeEndObject();
    }
}
```

然后，我们可以将序列化程序应用于将要序列化的类:

```java
@JsonSerialize(using = DistanceSerializer.class)
public enum TypeEnum { ... }
```

这导致:

```java
{"name":"MILE","unit":"miles","meters":1609.34}
```

## 4.将 JSON 反序列化到枚举

首先，让我们定义一个拥有`Distance` 成员的`City`类:

```java
public class City {

    private Distance distance;
    ...    
}
```

然后我们将讨论将 JSON 字符串反序列化为 Enum 的不同方法。

### 4.1.默认行为

默认情况下，Jackson 将使用 Enum 名称从 JSON 反序列化。

例如，它将反序列化 JSON:

```java
{"distance":"KILOMETER"}
```

到一个`Distance.KILOMETER`对象:

```java
City city = new ObjectMapper().readValue(json, City.class);
assertEquals(Distance.KILOMETER, city.getDistance());
```

### 4.2.使用`@JsonValue`

我们已经学习了如何使用`@JsonValue`来序列化枚举。我们也可以使用相同的注释进行反序列化。这是可能的，因为枚举值是常数。

首先，让我们将`@JsonValue`与其中一个 getter 方法`getMeters()`一起使用:

```java
public enum Distance {
    ...

    @JsonValue
    public double getMeters() {
        return meters;
    }
}
```

`getMeters()`方法的返回值表示枚举对象。因此，在反序列化示例 JSON 时:

```java
{"distance":"0.0254"}
```

Jackson 将寻找具有返回值 0.0254 的枚举对象。在这种情况下，对象是 `Distance.`英寸:

```java
assertEquals(Distance.INCH, city.getDistance()); 
```

### 4.3.使用`@JsonProperty`

`@JsonProperty`注释用于枚举实例:

```java
public enum Distance {
    @JsonProperty("distance-in-km")
    KILOMETER("km", 1000), 
    @JsonProperty("distance-in-miles")
    MILE("miles", 1609.34);

    ...
}
```

通过使用这个注释，**我们只是告诉 Jackson 将`@JsonProperty`的值映射到用这个值**注释的对象。

作为上述声明的结果，示例 JSON 字符串:

```java
{"distance": "distance-in-km"}
```

将被映射到`Distance.KILOMETER`对象:

```java
assertEquals(Distance.KILOMETER, city.getDistance());
```

### 4.4.使用`@JsonCreator`

**Jackson 调用用`@JsonCreator`标注的方法来获取封闭类的实例。**

考虑 JSON 表示:

```java
{
    "distance": {
        "unit":"miles", 
        "meters":1609.34
    }
}
```

然后我们将使用`@JsonCreator`注释定义`forValues()`工厂方法:

```java
public enum Distance {

    @JsonCreator
    public static Distance forValues(@JsonProperty("unit") String unit,
      @JsonProperty("meters") double meters) {
        for (Distance distance : Distance.values()) {
            if (
              distance.unit.equals(unit) && Double.compare(distance.meters, meters) == 0) {
                return distance;
            }
        }

        return null;
    }

    ...
}
```

注意使用了`@JsonProperty`注释来绑定 JSON 字段和方法参数。

然后，当我们反序列化 JSON 示例时，我们将得到结果:

```java
assertEquals(Distance.MILE, city.getDistance());
```

### 4.5.使用自定义的**解串器**

如果上述技术都不可用，我们可以使用定制的反序列化器。例如，我们可能无法访问 Enum 源代码，或者我们可能正在使用一个旧的 Jackson 版本，该版本不支持一个或多个到目前为止包含的注释。

根据[我们的自定义反序列化文章](/web/20220628093902/https://www.baeldung.com/jackson-deserialization)，为了反序列化前一节中提供的 JSON，我们将从创建反序列化类开始:

```java
public class CustomEnumDeserializer extends StdDeserializer<Distance> {

    @Override
    public Distance deserialize(JsonParser jsonParser, DeserializationContext ctxt)
      throws IOException, JsonProcessingException {
        JsonNode node = jsonParser.getCodec().readTree(jsonParser);

        String unit = node.get("unit").asText();
        double meters = node.get("meters").asDouble();

        for (Distance distance : Distance.values()) {

            if (distance.getUnit().equals(unit) && Double.compare(
              distance.getMeters(), meters) == 0) {
                return distance;
            }
        }

        return null;
    }
} 
```

然后，我们将使用 Enum 上的`@JsonDeserialize`注释来指定我们的自定义反序列化器:

```java
@JsonDeserialize(using = CustomEnumDeserializer.class)
public enum Distance {
   ...
}
```

我们的结果是:

```java
assertEquals(Distance.MILE, city.getDistance());
```

## 5。结论

本文展示了如何更好地控制 Java 枚举的**序列化和反序列化过程和格式。**

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628093902/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions#readme "Github Project exemplifying how to set serialize enums with Jackson 2")