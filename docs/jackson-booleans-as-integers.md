# 用 Jackson 将布尔值序列化和反序列化为整数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-booleans-as-integers>

## 1.介绍

在处理 JSON 时， [Jackson 库](/web/20220625163300/https://www.baeldung.com/jackson-object-mapper-tutorial)是 Java 世界中事实上的标准。尽管 Jackson 定义了很好的默认值，但是为了将一个`Boolean`值映射到`Integer`，我们仍然需要进行手动配置。

当然，一些开发人员想知道如何以最好的方式和最少的努力实现这一点。

在本文中，我们将解释如何将`Boolean`值序列化为`Integer`的数字字符串，反之亦然。

## 2.序列化

首先，我们将研究序列化部分。为了测试`Boolean`到`Integer`的序列化，让我们定义我们的模型，`Game`:

```java
public class Game {

    private Long id;
    private String name;
    private Boolean paused;
    private Boolean over;

    // constructors, getters and setters
}
```

像往常一样，`Game`对象的默认序列化将使用 Jackson 的`ObjectMapper`:

```java
ObjectMapper mapper = new ObjectMapper();
Game game = new Game(1L, "My Game");
game.setPaused(true);
game.setOver(false);
String json = mapper.writeValueAsString(game);
```

毫不奇怪，`Boolean`字段的输出将是默认的— `true`或`false`:

```java
{"id":1, "name":"My Game", "paused":true, "over":false}
```

然而，我们的目标是最终从我们的`Game`对象获得以下 JSON 输出:

```java
{"id":1, "name":"My Game", "paused":1, "over":0}
```

### 2.1.字段级配置

序列化为`Integer`的一个非常直接的方法是用`@JsonFormat`注释我们的`Boolean`字段，并为它设置`Shape.NUMBER`:

```java
@JsonFormat(shape = Shape.NUMBER)
private Boolean paused;

@JsonFormat(shape = Shape.NUMBER)
private Boolean over;
```

然后，让我们在一个测试方法中尝试我们的序列化:

```java
ObjectMapper mapper = new ObjectMapper();
Game game = new Game(1L, "My Game");
game.setPaused(true);
game.setOver(false);
String json = mapper.writeValueAsString(game);

assertThat(json)
  .isEqualTo("{\"id\":1,\"name\":\"My Game\",\"paused\":1,\"over\":0}");
```

正如我们在 JSON 输出中注意到的，我们的`Boolean`字段——`paused`和`over`——组成了数字`1`和`0`。我们可以看到这些值是整数格式的，因为它们没有被引号括起来。

### 2.2.全局配置

有时，注释每个字段是不实际的。例如，根据需求，我们可能需要全局配置我们的`Boolean`到`Integer`序列化。

幸运的是， **Jackson 允许我们通过覆盖`ObjectMapper`** 中的默认值来全局配置`@JsonFormat`:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.configOverride(Boolean.class)
  .setFormat(JsonFormat.Value.forShape(Shape.NUMBER));

Game game = new Game(1L, "My Game");
game.setPaused(true);
game.setOver(false);
String json = mapper.writeValueAsString(game);

assertThat(json)
  .isEqualTo("{\"id\":1,\"name\":\"My Game\",\"paused\":1,\"over\":0}");
```

## 3.反序列化

类似地，在将 JSON 字符串反序列化到我们的模型中时，我们可能还想从数字中获取`Boolean`值。

幸运的是，默认情况下，Jackson 可以将数字(只有`1`和`0`)解析为`Boolean`值。因此，我们也不需要使用`@JsonFormat`注释或任何额外的配置。

因此，在没有配置的情况下，让我们借助另一种测试方法来看看这种行为:

```java
ObjectMapper mapper = new ObjectMapper();
String json = "{\"id\":1,\"name\":\"My Game\",\"paused\":1,\"over\":0}";
Game game = mapper.readValue(json, Game.class);

assertThat(game.isPaused()).isEqualTo(true);
assertThat(game.isOver()).isEqualTo(false);
```

因此， **`Integer`到`Boolean`的反序列化在 Jackson** 中是现成的。

## 4.数字字符串而不是整数

另一个用例是使用数字字符串——`“1”`和`“0”`——而不是整数。在这种情况下，将`Boolean`值序列化为数字字符串或者反序列化回`Boolean`需要更多的努力。

### 4.1.序列化为数字字符串

为了将一个`Boolean`值序列化为等价的数字字符串，我们需要定义一个定制的序列化器。

所以，让我们通过扩展杰克逊的`JsonSerializer`来创建我们的`NumericBooleanSerializer`:

```java
public class NumericBooleanSerializer extends JsonSerializer<Boolean> {

    @Override
    public void serialize(Boolean value, JsonGenerator gen, SerializerProvider serializers)
      throws IOException {
        gen.writeString(value ? "1" : "0");
    }
}
```

旁注一下，正常情况下，`Boolean`类型可以是`null`。然而，当`value`字段为`null`时，Jackson 在内部处理这个问题，并不考虑我们的定制序列化程序。因此，我们在这里很安全。

接下来，我们将注册我们的自定义序列化程序，以便 Jackson 识别和使用它。

**如果我们只对有限数量的字段需要这种行为，我们可以选择带有`@JsonSerialize`注释的字段级配置。**

相应地，让我们注释我们的`Boolean`字段、`paused`和`over`:

```java
@JsonSerialize(using = NumericBooleanSerializer.class)
private Boolean paused;

@JsonSerialize(using = NumericBooleanSerializer.class)
private Boolean over;
```

然后，同样，我们在一个测试方法中尝试序列化:

```java
ObjectMapper mapper = new ObjectMapper();
Game game = new Game(1L, "My Game");
game.setPaused(true);
game.setOver(false);
String json = mapper.writeValueAsString(game);

assertThat(json)
  .isEqualTo("{\"id\":1,\"name\":\"My Game\",\"paused\":\"1\",\"over\":\"0\"}");
```

虽然测试方法的实现与前面的几乎相同，但是我们应该注意数字值周围的引号。当然，这表明这些值是包含数字内容的实际字符串。

最后但同样重要的是，如果我们需要在任何地方执行这种自定义序列化， **Jackson 通过 Jackson 模块**将序列化程序添加到`ObjectMapper`中，从而支持序列化程序的全局配置:

```java
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule();
module.addSerializer(Boolean.class, new NumericBooleanSerializer());
mapper.registerModule(module);

Game game = new Game(1L, "My Game");
game.setPaused(true);
game.setOver(false);
String json = mapper.writeValueAsString(game);

assertThat(json)
  .isEqualTo("{\"id\":1,\"name\":\"My Game\",\"paused\":\"1\",\"over\":\"0\"}");
```

因此，只要我们使用同一个`ObjectMapper`实例，Jackson 就会将所有`Boolean`类型的字段序列化为数字字符串。

### 4.2.从数字字符串反序列化

与序列化类似，这次我们将定义一个定制的反序列化器来将数字字符串解析成`Boolean`值。

让我们通过扩展`JsonDeserializer`来创建我们的类`NumericBooleanDeserializer`:

```java
public class NumericBooleanDeserializer extends JsonDeserializer<Boolean> {

    @Override
    public Boolean deserialize(JsonParser p, DeserializationContext ctxt)
      throws IOException {
        if ("1".equals(p.getText())) {
            return Boolean.TRUE;
        }
        if ("0".equals(p.getText())) {
            return Boolean.FALSE;
        }
        return null;
    }

}
```

接下来，我们再次注释我们的`Boolean`字段，但是这次用`@JsonDeserialize`:

```java
@JsonSerialize(using = NumericBooleanSerializer.class)
@JsonDeserialize(using = NumericBooleanDeserializer.class)
private Boolean paused;

@JsonSerialize(using = NumericBooleanSerializer.class)
@JsonDeserialize(using = NumericBooleanDeserializer.class)
private Boolean over;
```

因此，让我们编写另一个测试方法来看看我们的`NumericBooleanDeserializer`的运行情况:

```java
ObjectMapper mapper = new ObjectMapper();
String json = "{\"id\":1,\"name\":\"My Game\",\"paused\":\"1\",\"over\":\"0\"}";
Game game = mapper.readValue(json, Game.class);

assertThat(game.isPaused()).isEqualTo(true);
assertThat(game.isOver()).isEqualTo(false);
```

或者，也可以通过 Jackson 模块对我们的定制反串行化器进行全局配置:

```java
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule();
module.addDeserializer(Boolean.class, new NumericBooleanDeserializer());
mapper.registerModule(module);

String json = "{\"id\":1,\"name\":\"My Game\",\"paused\":\"1\",\"over\":\"0\"}";
Game game = mapper.readValue(json, Game.class);

assertThat(game.isPaused()).isEqualTo(true);
assertThat(game.isOver()).isEqualTo(false);
```

## 5.结论

在本文中，我们描述了如何将`Boolean`值序列化为整数和数字字符串，以及如何反序列化它们。

和往常一样，可以在 GitHub 的[上找到示例和更多内容的源代码。](https://web.archive.org/web/20220625163300/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)