# 杰克逊日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-serialize-dates>

## 1。概述

在本教程中，我们将序列化杰克逊的日期。我们将首先序列化一个简单的 java.util. `Date`，然后是 Joda-Time，最后是 Java 8 `DateTime`。

## 2。将`Date`序列化为时间戳

首先，让我们看看如何用杰克森序列化一个简单的 **`java.util.Date`。**

在下面的例子中，我们将序列化一个“`Event,`”的实例，它有一个`Date`字段“`eventDate`”:

```java
@Test
public void whenSerializingDateWithJackson_thenSerializedToTimestamp()
  throws JsonProcessingException, ParseException {

    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));

    Date date = df.parse("01-01-1970 01:00");
    Event event = new Event("party", date);

    ObjectMapper mapper = new ObjectMapper();
    mapper.writeValueAsString(event);
}
```

需要注意的是，默认情况下，Jackson 会将日期序列化为时间戳格式(自 UTC 1970 年 1 月 1 日起的毫秒数)。

“`event`”序列化的实际输出是:

```java
{
   "name":"party",
   "eventDate":3600000
}
```

## 3。将`Date`序列化为 ISO-8601

序列化为这种简洁的时间戳格式并不是最佳选择。相反，让我们将`Date`序列化为`ISO-8601`格式:

```java
@Test
public void whenSerializingDateToISO8601_thenSerializedToText()
  throws JsonProcessingException, ParseException {

    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));

    String toParse = "01-01-1970 02:30";
    Date date = df.parse(toParse);
    Event event = new Event("party", date);

    ObjectMapper mapper = new ObjectMapper();
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    // StdDateFormat is ISO8601 since jackson 2.9
    mapper.setDateFormat(new StdDateFormat().withColonInTimeZone(true));
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString("1970-01-01T02:30:00.000+00:00"));
}
```

我们可以看到日期的表示现在可读性更好了。

## 4。配置`ObjectMapper` `DateFormat`

以前的解决方案仍然缺乏选择精确格式来表示`java.util.Date`实例的充分灵活性。

相反，让我们来看一个配置，它允许我们**设置表示日期**的格式:

```java
@Test
public void whenSettingObjectMapperDateFormat_thenCorrect()
  throws JsonProcessingException, ParseException {

    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm");

    String toParse = "20-12-2014 02:30";
    Date date = df.parse(toParse);
    Event event = new Event("party", date);

    ObjectMapper mapper = new ObjectMapper();
    mapper.setDateFormat(df);

    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
```

请注意，尽管我们现在在日期格式方面更加灵活，但我们仍然在整个`ObjectMapper`级别使用全局配置。

## 5。使用`@JsonFormat`格式化`Date`

接下来让我们来看看对整个应用程序来说，**控制单个类的日期格式的`@JsonFormat`注释，**而不是全局的:

```java
public class Event {
    public String name;

    @JsonFormat
      (shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;
}
```

现在我们来测试一下:

```java
@Test
public void whenUsingJsonFormatAnnotationToFormatDate_thenCorrect()
  throws JsonProcessingException, ParseException {

    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));

    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    Event event = new Event("party", date);

    ObjectMapper mapper = new ObjectMapper();
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
```

## 6。自定义`Date`串行器

接下来，为了获得对输出的完全控制，我们将利用一个定制的日期序列化器:

```java
public class CustomDateSerializer extends StdSerializer<Date> {

    private SimpleDateFormat formatter 
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateSerializer() {
        this(null);
    }

    public CustomDateSerializer(Class t) {
        super(t);
    }

    @Override
    public void serialize (Date value, JsonGenerator gen, SerializerProvider arg2)
      throws IOException, JsonProcessingException {
        gen.writeString(formatter.format(value));
    }
}
```

现在我们将使用它作为我们的“`eventDate`”字段的序列化器:

```java
public class Event {
    public String name;

    @JsonSerialize(using = CustomDateSerializer.class)
    public Date eventDate;
}
```

最后，我们将测试它:

```java
@Test
public void whenUsingCustomDateSerializer_thenCorrect()
  throws JsonProcessingException, ParseException {

    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    Event event = new Event("party", date);

    ObjectMapper mapper = new ObjectMapper();
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
```

## 延伸阅读:

## [如何用 Jackson 序列化和反序列化枚举](/web/20220911051535/https://www.baeldung.com/jackson-serialize-enums)

How to serialize and deserialize an Enum as a JSON Object using Jackson 2.[Read more](/web/20220911051535/https://www.baeldung.com/jackson-serialize-enums) →

## [Jackson–定制串行器](/web/20220911051535/https://www.baeldung.com/jackson-custom-serialization)

Control your JSON output with Jackson 2 by using a Custom Serializer.[Read more](/web/20220911051535/https://www.baeldung.com/jackson-custom-serialization) →

## [Jackson 中的自定义反序列化入门](/web/20220911051535/https://www.baeldung.com/jackson-deserialization)

Use Jackson to map custom JSON to any java entity graph with full control over the deserialization process.[Read more](/web/20220911051535/https://www.baeldung.com/jackson-deserialization) →

## 7 .**。与杰克逊连载 Joda-Time**

日期并不总是`java.util.Date.`的实例，事实上，越来越多的日期由其他类表示，一个常见的是 Joda-Time 库中的`DateTime`实现。

让我们来看看如何将杰克逊和**连载。**

我们将利用 [`jackson-datatype-joda`](https://web.archive.org/web/20220911051535/https://search.maven.org/artifact/com.fasterxml.jackson.datatype/jackson-datatype-joda) 模块来获得现成的 Joda-Time 支持:

```java
<dependency>
  <groupId>com.fasterxml.jackson.datatype</groupId>
  <artifactId>jackson-datatype-joda</artifactId>
  <version>2.9.7</version>
</dependency>
```

然后我们可以简单地注册`JodaModule`并完成:

```java
@Test
public void whenSerializingJodaTime_thenCorrect() 
  throws JsonProcessingException {
    DateTime date = new DateTime(2014, 12, 20, 2, 30, 
      DateTimeZone.forID("Europe/London"));

    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(new JodaModule());
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

    String result = mapper.writeValueAsString(date);
    assertThat(result, containsString("2014-12-20T02:30:00.000Z"));
}
```

## 8。用自定义序列化器序列化 Joda `DateTime`

如果我们不想要额外的 Joda-Time Jackson 依赖项，我们也可以利用定制序列化器(类似于前面的例子)来干净地序列化`DateTime`实例:

```java
public class CustomDateTimeSerializer extends StdSerializer<DateTime> {

    private static DateTimeFormatter formatter = 
      DateTimeFormat.forPattern("yyyy-MM-dd HH:mm");

    public CustomDateTimeSerializer() {
        this(null);
    }

     public CustomDateTimeSerializer(Class<DateTime> t) {
         super(t);
     }

    @Override
    public void serialize
      (DateTime value, JsonGenerator gen, SerializerProvider arg2)
      throws IOException, JsonProcessingException {
        gen.writeString(formatter.print(value));
    }
}
```

然后我们可以把它作为我们的属性"`eventDate`"序列化器:

```java
public class Event {
    public String name;

    @JsonSerialize(using = CustomDateTimeSerializer.class)
    public DateTime eventDate;
}
```

最后，我们可以将所有东西放在一起进行测试:

```java
@Test
public void whenSerializingJodaTimeWithJackson_thenCorrect() 
  throws JsonProcessingException {

    DateTime date = new DateTime(2014, 12, 20, 2, 30);
    Event event = new Event("party", date);

    ObjectMapper mapper = new ObjectMapper();
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString("2014-12-20 02:30"));
}
```

## 9。用杰克森连载 Java 8 `Date`

现在让我们看看如何使用杰克森序列化本例中的 Java 8`DateTime,` **`LocalDateTime,` 。我们可以利用 [`jackson-datatype-jsr310`](https://web.archive.org/web/20220911051535/https://search.maven.org/artifact/com.fasterxml.jackson.datatype/jackson-datatype-jsr310) 模块:**

```java
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.9.7</version>
</dependency>
```

然后我们需要做的就是注册`JavaTimeModule` ( `JSR310Module`已被否决)，杰克逊会处理剩下的事情:

```java
@Test
public void whenSerializingJava8Date_thenCorrect()
  throws JsonProcessingException {
    LocalDateTime date = LocalDateTime.of(2014, 12, 20, 2, 30);

    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(new JavaTimeModule());
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

    String result = mapper.writeValueAsString(date);
    assertThat(result, containsString("2014-12-20T02:30"));
}
```

## 10。序列化 Java 8 `Date`没有任何额外的依赖

如果我们不想要额外的依赖，我们总是可以使用定制的序列化器**将 Java 8 `DateTime`写出到 JSON** :

```java
public class CustomLocalDateTimeSerializer 
  extends StdSerializer<LocalDateTime> {

    private static DateTimeFormatter formatter = 
      DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    public CustomLocalDateTimeSerializer() {
        this(null);
    }

    public CustomLocalDateTimeSerializer(Class<LocalDateTime> t) {
        super(t);
    }

    @Override
    public void serialize(
      LocalDateTime value,
      JsonGenerator gen,
      SerializerProvider arg2)
      throws IOException, JsonProcessingException {

        gen.writeString(formatter.format(value));
    }
}
```

然后，我们将为我们的“`eventDate`”字段使用序列化程序:

```java
public class Event {
    public String name;

    @JsonSerialize(using = CustomLocalDateTimeSerializer.class)
    public LocalDateTime eventDate;
}
```

最后，我们将测试它:

```java
@Test
public void whenSerializingJava8DateWithCustomSerializer_thenCorrect()
  throws JsonProcessingException {

    LocalDateTime date = LocalDateTime.of(2014, 12, 20, 2, 30);
    Event event = new Event("party", date);

    ObjectMapper mapper = new ObjectMapper();
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString("2014-12-20 02:30"));
}
```

## 11。反序列化`Date`

现在让我们看看如何用**杰克森**反序列化一个`Date`。在下面的例子中，我们将反序列化一个包含日期的“`Event`”实例:

```java
@Test
public void whenDeserializingDateWithJackson_thenCorrect()
  throws JsonProcessingException, IOException {

    String json = "{"name":"party","eventDate":"20-12-2014 02:30:00"}";

    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    ObjectMapper mapper = new ObjectMapper();
    mapper.setDateFormat(df);

    Event event = mapper.readerFor(Event.class).readValue(json);
    assertEquals("20-12-2014 02:30:00", df.format(event.eventDate));
}
```

## 12。反序列化 Joda `ZonedDateTime`并保留时区

在其默认配置中，Jackson 将 Joda `ZonedDateTime`的时区调整为本地上下文的时区。因为本地上下文的时区不是默认设置的，必须手动配置，所以 Jackson 将时区调整为 GMT:

```java
@Test
public void whenDeserialisingZonedDateTimeWithDefaults_thenNotCorrect()
  throws IOException {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.findAndRegisterModules();
    objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    ZonedDateTime now = ZonedDateTime.now(ZoneId.of("Europe/Berlin"));
    String converted = objectMapper.writeValueAsString(now);

    ZonedDateTime restored = objectMapper.readValue(converted, ZonedDateTime.class);
    System.out.println("serialized: " + now);
    System.out.println("restored: " + restored);
    assertThat(now, is(restored));
}
```

测试用例将失败，并输出:

```java
serialized: 2017-08-14T13:52:22.071+02:00[Europe/Berlin]
restored: 2017-08-14T11:52:22.071Z[UTC]
```

幸运的是，对于这种奇怪的默认行为有一个快速简单的修复方法；我们只要告诉杰克逊不要调整时区。

这可以通过将下面一行代码添加到上面的测试用例中来实现:

```java
objectMapper.disable(DeserializationFeature.ADJUST_DATES_TO_CONTEXT_TIME_ZONE);
```

注意，为了保留时区，我们还必须禁用将日期序列化为时间戳的默认行为。

## 13。定制`Date`解串器

我们也可以使用定制的`Date`反串行化器**。**我们将为属性“`eventDate`”编写一个定制的反序列化器:

```java
public class CustomDateDeserializer extends StdDeserializer<Date> {

    private SimpleDateFormat formatter = 
      new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateDeserializer() {
        this(null);
    }

    public CustomDateDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public Date deserialize(JsonParser jsonparser, DeserializationContext context)
      throws IOException, JsonProcessingException {
        String date = jsonparser.getText();
        try {
            return formatter.parse(date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}
```

接下来，我们将把它用作“`eventDate`”反序列化器:

```java
public class Event {
    public String name;

    @JsonDeserialize(using = CustomDateDeserializer.class)
    public Date eventDate;
}
```

最后，我们将测试它:

```java
@Test
public void whenDeserializingDateUsingCustomDeserializer_thenCorrect()
  throws JsonProcessingException, IOException {

    String json = "{"name":"party","eventDate":"20-12-2014 02:30:00"}";

    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    ObjectMapper mapper = new ObjectMapper();

    Event event = mapper.readerFor(Event.class).readValue(json);
    assertEquals("20-12-2014 02:30:00", df.format(event.eventDate));
}
```

## 14.固定`InvalidDefinition` `Exception`

当创建一个`LocalDate`实例时，我们可能会遇到一个异常:

```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance
of `java.time.LocalDate`(no Creators, like default construct, exist): no String-argument
constructor/factory method to deserialize from String value ('2014-12-20') at [Source:
(String)"2014-12-20"; line: 1, column: 1]
```

出现这个问题是因为 JSON 本身没有日期格式，所以它将日期表示为`String`。

日期的`String`表示与内存中类型为`LocalDate` 的对象不同，因此我们需要一个外部反序列化器从`String`中读取该字段，并需要一个序列化器将日期呈现为`String`格式。

这些方法也适用于`LocalDateTime,` ，唯一的变化是为`LocalDateTime`使用一个等价的类。

### 14.1.杰克逊依赖

杰克逊允许我们用几种方法来解决这个问题。首先，我们必须确保 [`jsr310`依赖项](https://web.archive.org/web/20220911051535/https://search.maven.org/artifact/com.fasterxml.jackson.datatype/jackson-datatype-jsr310)在我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.11.0</version>
</dependency>
```

### 14.2.序列化为单个日期对象

为了能够处理`LocalDate`，我们需要用我们的`ObjectMapper`注册`JavaTimeModule`。

我们还需要禁用`ObjectMapper`中的特性`WRITE_DATES_AS_TIMESTAMPS `,以防止 Jackson 向 JSON 输出中添加时间数字:

```java
@Test
public void whenSerializingJava8DateAndReadingValue_thenCorrect() throws IOException {
    String stringDate = "\"2014-12-20\"";

    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(new JavaTimeModule());
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

    LocalDate result = mapper.readValue(stringDate, LocalDate.class);
    assertThat(result.toString(), containsString("2014-12-20"));
}
```

这里我们使用了 Jackson 的原生支持来序列化和反序列化日期。

### 14.3.POJO 中的注释

处理这个问题的另一种方法是在实体级别使用`LocalDateDeserializer`和`JsonFormat`注释:

```java
public class EventWithLocalDate {

    @JsonDeserialize(using = LocalDateDeserializer.class)
    @JsonSerialize(using = LocalDateSerializer.class)
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy")
    public LocalDate eventDate;
}
```

`@JsonDeserialize`注释用于指定一个定制的反序列化器来解组 JSON 对象。类似地，`@JsonSerialize`表示在封送实体时使用的自定义序列化程序。

此外，注释`@JsonFormat`允许我们指定序列化日期值的格式。因此，这个 POJO 可以用来读写 JSON:

```java
@Test
public void whenSerializingJava8DateAndReadingFromEntity_thenCorrect() throws IOException {
    String json = "{\"name\":\"party\",\"eventDate\":\"20-12-2014\"}";

    ObjectMapper mapper = new ObjectMapper();

    EventWithLocalDate result = mapper.readValue(json, EventWithLocalDate.class);
    assertThat(result.getEventDate().toString(), containsString("2014-12-20"));
}
```

虽然这种方法比使用默认的`JavaTimeModule`花费更多的工作，但是它更加可定制。

## 15。结论

在这篇内容丰富的`Date`文章中，我们研究了 **Jackson 使用一种我们可以控制的合理格式来帮助将日期编组和解组到 JSON** 的几种方法。

和往常一样，示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220911051535/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions "Github Project covering all Jackson examples")