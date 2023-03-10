# 杰克逊注释示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-annotations>

## 1。概述

在本教程中，我们将深入研究**杰克逊注释**。

我们将看到如何使用现有的注释，如何创建定制的注释，最后，如何禁用它们。

## 延伸阅读:

## [更多杰克逊注解](/web/20221230230642/http://www.baeldung.com/jackson-advanced-annotations)

This article covers some lesser-known JSON processing annotations provided by Jackson.[Read more](/web/20221230230642/http://www.baeldung.com/jackson-advanced-annotations) →

## [杰克逊-双向关系](/web/20221230230642/http://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion)

How to use Jackson to break the infinite recursion problem on bidirectional relationships.[Read more](/web/20221230230642/http://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion) →

## [Jackson 中的自定义反序列化入门](/web/20221230230642/http://www.baeldung.com/jackson-deserialization)

Use Jackson to map custom JSON to any java entity graph with full control over the deserialization process.[Read more](/web/20221230230642/http://www.baeldung.com/jackson-deserialization) →

## 2。杰克逊连载注释

首先，我们来看看序列化注释。

### 2.1。`@JsonAnyGetter`

`@JsonAnyGetter`注释允许使用`Map`字段作为标准属性的灵活性。

例如，`ExtendableBean`实体具有`name`属性和一组键/值对形式的可扩展属性:

```java
public class ExtendableBean {
    public String name;
    private Map<String, String> properties;

    @JsonAnyGetter
    public Map<String, String> getProperties() {
        return properties;
    }
}
```

当我们序列化这个实体的一个实例时，我们在`Map`中得到所有的键值作为标准的普通属性:

```java
{
    "name":"My bean",
    "attr2":"val2",
    "attr1":"val1"
}
```

下面是这个实体的序列化在实践中的样子:

```java
@Test
public void whenSerializingUsingJsonAnyGetter_thenCorrect()
  throws JsonProcessingException {

    ExtendableBean bean = new ExtendableBean("My bean");
    bean.add("attr1", "val1");
    bean.add("attr2", "val2");

    String result = new ObjectMapper().writeValueAsString(bean);

    assertThat(result, containsString("attr1"));
    assertThat(result, containsString("val1"));
}
```

我们也可以使用可选参数`enabled`作为`false`来禁用`@JsonAnyGetter().`在这种情况下，`Map`将被转换为 JSON，并在序列化后出现在`properties`变量下。

### 2.2。`@JsonGetter`

**`@JsonGetter`注释是`@JsonProperty`注释的替代，它将一个方法标记为 getter 方法。**

在下面的例子中，我们将方法`getTheName()`指定为`MyBean`实体的`name`属性的 getter 方法:

```java
public class MyBean {
    public int id;
    private String name;

    @JsonGetter("name")
    public String getTheName() {
        return name;
    }
}
```

这在实践中是如何工作的:

```java
@Test
public void whenSerializingUsingJsonGetter_thenCorrect()
  throws JsonProcessingException {

    MyBean bean = new MyBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));
}
```

### 2.3。`@JsonPropertyOrder`

我们可以使用`@JsonPropertyOrder`注释来指定**序列化**属性的顺序。

让我们为一个`MyBean`实体的属性设置一个定制的顺序:

```java
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```

以下是序列化的结果:

```java
{
    "name":"My bean",
    "id":1
}
```

然后我们可以做一个简单的测试:

```java
@Test
public void whenSerializingUsingJsonPropertyOrder_thenCorrect()
  throws JsonProcessingException {

    MyBean bean = new MyBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));
}
```

我们也可以使用`@JsonPropertyOrder(alphabetic=true)`按字母顺序排列属性。在这种情况下，序列化的输出将是:

```java
{
    "id":1,
    "name":"My bean"
}
```

### 2.4。`@JsonRawValue`

`@JsonRawValue`注释可以**指示 Jackson 完全按照**的方式序列化一个属性。

在下面的例子中，我们使用`@JsonRawValue`来嵌入一些定制的 JSON 作为一个实体的值:

```java
public class RawBean {
    public String name;

    @JsonRawValue
    public String json;
}
```

序列化实体的输出是:

```java
{
    "name":"My bean",
    "json":{
        "attr":false
    }
}
```

接下来是一个简单的测试:

```java
@Test
public void whenSerializingUsingJsonRawValue_thenCorrect()
  throws JsonProcessingException {

    RawBean bean = new RawBean("My bean", "{\"attr\":false}");

    String result = new ObjectMapper().writeValueAsString(bean);
    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("{\"attr\":false}"));
}
```

我们还可以使用可选的布尔参数`value`来定义这个注释是否是活动的。

### 2.5。`@JsonValue`

`@JsonValue`表示库将用来序列化整个实例的单个方法。

例如，在一个枚举中，我们用`@JsonValue` 来注释`getName`，这样任何这样的实体都可以通过它的名字来序列化:

```java
public enum TypeEnumWithValue {
    TYPE1(1, "Type A"), TYPE2(2, "Type 2");

    private Integer id;
    private String name;

    // standard constructors

    @JsonValue
    public String getName() {
        return name;
    }
}
```

这是我们的测试:

```java
@Test
public void whenSerializingUsingJsonValue_thenCorrect()
  throws JsonParseException, IOException {

    String enumAsString = new ObjectMapper()
      .writeValueAsString(TypeEnumWithValue.TYPE1);

    assertThat(enumAsString, is(""Type A""));
}
```

### 2.6。`@JsonRootName`

如果启用了包装，则使用`@JsonRootName`注释来指定要使用的根包装器的名称。

包装意味着不再将`User`序列化为类似于:

```java
{
    "id": 1,
    "name": "John"
}
```

它会被包装成这样:

```java
{
    "User": {
        "id": 1,
        "name": "John"
    }
}
```

让我们来看一个例子。**W**我们将使用`@JsonRootName` 注释来表示这个潜在包装器实体的名称:

```java
@JsonRootName(value = "user")
public class UserWithRoot {
    public int id;
    public String name;
}
```

默认情况下，包装器的名称是类名–`UserWithRoot`。通过使用注释，我们得到了看起来更干净的`user:`

```java
@Test
public void whenSerializingUsingJsonRootName_thenCorrect()
  throws JsonProcessingException {

    UserWithRoot user = new User(1, "John");

    ObjectMapper mapper = new ObjectMapper();
    mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
    String result = mapper.writeValueAsString(user);

    assertThat(result, containsString("John"));
    assertThat(result, containsString("user"));
}
```

以下是序列化的结果:

```java
{
    "user":{
        "id":1,
        "name":"John"
    }
}
```

从 Jackson 2.4 开始，一个新的可选参数`namespace`可用于 XML 等数据格式。如果我们添加它，它将成为完全限定名的一部分:

```java
@JsonRootName(value = "user", namespace="users")
public class UserWithRootNamespace {
    public int id;
    public String name;

    // ...
}
```

如果我们用`XmlMapper,`将其序列化，输出将是:

```java
<user >
    <id xmlns="">1</id>
    <name xmlns="">John</name>
    <items xmlns=""/>
</user>
```

### 2.7。`@JsonSerialize`

**`@JsonSerialize` 表示当**编组**实体时使用的自定义序列化程序。**

让我们看一个简单的例子。我们将使用`@JsonSerialize`来序列化带有`CustomDateSerializer`的`eventDate`属性:

```java
public class EventWithSerializer {
    public String name;

    @JsonSerialize(using = CustomDateSerializer.class)
    public Date eventDate;
}
```

下面是简单的定制 Jackson 序列化程序:

```java
public class CustomDateSerializer extends StdSerializer<Date> {

    private static SimpleDateFormat formatter 
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateSerializer() { 
        this(null); 
    } 

    public CustomDateSerializer(Class<Date> t) {
        super(t); 
    }

    @Override
    public void serialize(
      Date value, JsonGenerator gen, SerializerProvider arg2) 
      throws IOException, JsonProcessingException {
        gen.writeString(formatter.format(value));
    }
}
```

现在让我们在测试中使用这些:

```java
@Test
public void whenSerializingUsingJsonSerialize_thenCorrect()
  throws JsonProcessingException, ParseException {

    SimpleDateFormat df
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    EventWithSerializer event = new EventWithSerializer("party", date);

    String result = new ObjectMapper().writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
```

## 3。杰克逊反序列化注释

接下来让我们探索 Jackson 反序列化注释。

### 3.1。`@JsonCreator`

**我们可以使用 `@JsonCreator`注释来调整反序列化中使用的构造函数/工厂。**

当我们需要反序列化一些与我们需要得到的目标实体不完全匹配的 JSON 时，这非常有用。

让我们看一个例子。假设我们需要反序列化以下 JSON:

```java
{
    "id":1,
    "theName":"My bean"
}
```

然而，在我们的目标实体中没有`theName`字段，只有一个`name`字段。现在我们不想改变实体本身，我们只需要通过用`@JsonCreator,`注释构造函数和使用`@JsonProperty`注释来对解组过程有更多的控制:

```java
public class BeanWithCreator {
    public int id;
    public String name;

    @JsonCreator
    public BeanWithCreator(
      @JsonProperty("id") int id, 
      @JsonProperty("theName") String name) {
        this.id = id;
        this.name = name;
    }
}
```

让我们来看看实际情况:

```java
@Test
public void whenDeserializingUsingJsonCreator_thenCorrect()
  throws IOException {

    String json = "{\"id\":1,\"theName\":\"My bean\"}";

    BeanWithCreator bean = new ObjectMapper()
      .readerFor(BeanWithCreator.class)
      .readValue(json);
    assertEquals("My bean", bean.name);
}
```

### 3.2。`@JacksonInject`

**`@JacksonInject`表示属性将从注入而不是从 JSON 数据中获取值。**

在下面的例子中，我们使用`@JacksonInject` 来注入属性`id`:

```java
public class BeanWithInject {
    @JacksonInject
    public int id;

    public String name;
}
```

它是这样工作的:

```java
@Test
public void whenDeserializingUsingJsonInject_thenCorrect()
  throws IOException {

    String json = "{\"name\":\"My bean\"}";

    InjectableValues inject = new InjectableValues.Std()
      .addValue(int.class, 1);
    BeanWithInject bean = new ObjectMapper().reader(inject)
      .forType(BeanWithInject.class)
      .readValue(json);

    assertEquals("My bean", bean.name);
    assertEquals(1, bean.id);
}
```

### 3.3。`@JsonAnySetter`

`@JsonAnySetter`允许我们灵活地使用`Map`作为标准属性。在反序列化时，来自 JSON 的属性将被简单地添加到映射中。

首先，我们将使用`@JsonAnySetter` 来反序列化实体`ExtendableBean`:

```java
public class ExtendableBean {
    public String name;
    private Map<String, String> properties;

    @JsonAnySetter
    public void add(String key, String value) {
        properties.put(key, value);
    }
}
```

这是我们需要反序列化的 JSON:

```java
{
    "name":"My bean",
    "attr2":"val2",
    "attr1":"val1"
}
```

下面是所有这些联系在一起的方式:

```java
@Test
public void whenDeserializingUsingJsonAnySetter_thenCorrect()
  throws IOException {
    String json
      = "{\"name\":\"My bean\",\"attr2\":\"val2\",\"attr1\":\"val1\"}";

    ExtendableBean bean = new ObjectMapper()
      .readerFor(ExtendableBean.class)
      .readValue(json);

    assertEquals("My bean", bean.name);
    assertEquals("val2", bean.getProperties().get("attr2"));
}
```

### 3.4。`@JsonSetter`

`@JsonSetter`是`@JsonProperty`的替代，它将该方法标记为 setter 方法。

当我们需要读取一些 JSON 数据时，这是非常有用的，但是**目标实体类并不完全匹配那些数据**，因此我们需要调整这个过程以使它适合。

在下面的例子中，我们将指定方法 s `etTheName()`作为我们的`MyBean`实体中`name`属性的设置者:

```java
public class MyBean {
    public int id;
    private String name;

    @JsonSetter("name")
    public void setTheName(String name) {
        this.name = name;
    }
}
```

现在，当我们需要解组一些 JSON 数据时，这非常好:

```java
@Test
public void whenDeserializingUsingJsonSetter_thenCorrect()
  throws IOException {

    String json = "{\"id\":1,\"name\":\"My bean\"}";

    MyBean bean = new ObjectMapper()
      .readerFor(MyBean.class)
      .readValue(json);
    assertEquals("My bean", bean.getTheName());
}
```

### 3.5。`@JsonDeserialize`

**`@JsonDeserialize`表示使用自定义的反串行化器。**

首先，我们将使用`@JsonDeserialize`通过`CustomDateDeserializer`反序列化`eventDate`属性:

```java
public class EventWithSerializer {
    public String name;

    @JsonDeserialize(using = CustomDateDeserializer.class)
    public Date eventDate;
}
```

下面是自定义的反序列化程序:

```java
public class CustomDateDeserializer
  extends StdDeserializer<Date> {

    private static SimpleDateFormat formatter
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateDeserializer() { 
        this(null); 
    } 

    public CustomDateDeserializer(Class<?> vc) { 
        super(vc); 
    }

    @Override
    public Date deserialize(
      JsonParser jsonparser, DeserializationContext context) 
      throws IOException {

        String date = jsonparser.getText();
        try {
            return formatter.parse(date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}
```

接下来是背靠背测试:

```java
@Test
public void whenDeserializingUsingJsonDeserialize_thenCorrect()
  throws IOException {

    String json
      = "{"name":"party","eventDate":"20-12-2014 02:30:00"}";

    SimpleDateFormat df
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    EventWithSerializer event = new ObjectMapper()
      .readerFor(EventWithSerializer.class)
      .readValue(json);

    assertEquals(
      "20-12-2014 02:30:00", df.format(event.eventDate));
}
```

### 3.6.`@JsonAlias`

在反序列化期间， `@JsonAlias` 为属性定义了**一个或多个可选名称。**

让我们通过一个简单的例子来看看这个注释是如何工作的:

```java
public class AliasBean {
    @JsonAlias({ "fName", "f_name" })
    private String firstName;   
    private String lastName;
}
```

这里我们有一个 POJO，我们希望将 JSON 的值如`fName`、`f_name`和`firstName`反序列化到 POJO 的`firstName`变量中。

下面是一个测试，确保此注释按预期工作:

```java
@Test
public void whenDeserializingUsingJsonAlias_thenCorrect() throws IOException {
    String json = "{\"fName\": \"John\", \"lastName\": \"Green\"}";
    AliasBean aliasBean = new ObjectMapper().readerFor(AliasBean.class).readValue(json);
    assertEquals("John", aliasBean.getFirstName());
}
```

## 4。杰克逊财产包含注释

### 4.1。`@JsonIgnoreProperties`

**`@JsonIgnoreProperties`是一个类级注释，它标记了 Jackson 将忽略的一个属性或属性列表。**

让我们看一个简单的例子，忽略序列化的属性`id`:

```java
@JsonIgnoreProperties({ "id" })
public class BeanWithIgnore {
    public int id;
    public String name;
}
```

下面是确保忽略发生的测试:

```java
@Test
public void whenSerializingUsingJsonIgnoreProperties_thenCorrect()
  throws JsonProcessingException {

    BeanWithIgnore bean = new BeanWithIgnore(1, "My bean");

    String result = new ObjectMapper()
      .writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

为了无一例外地忽略 JSON 输入中的任何未知属性，我们可以设置`@JsonIgnoreProperties `注释的`ignoreUnknown=true`。

### 4.2。`@JsonIgnore`

**相反， `@JsonIgnore`注释用于在字段级别标记要忽略的属性。**

让我们使用`@JsonIgnore`来忽略序列化的属性`id`:

```java
public class BeanWithIgnore {
    @JsonIgnore
    public int id;

    public String name;
}
```

然后，我们将测试以确保成功忽略了`id`:

```java
@Test
public void whenSerializingUsingJsonIgnore_thenCorrect()
  throws JsonProcessingException {

    BeanWithIgnore bean = new BeanWithIgnore(1, "My bean");

    String result = new ObjectMapper()
      .writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

### 4.3。`@JsonIgnoreType`

**`@JsonIgnoreType`标记要忽略的注释类型的所有属性。**

我们可以使用注释来标记要忽略的类型为`Name` 的所有属性:

```java
public class User {
    public int id;
    public Name name;

    @JsonIgnoreType
    public static class Name {
        public String firstName;
        public String lastName;
    }
}
```

我们还可以测试以确保忽略正常工作:

```java
@Test
public void whenSerializingUsingJsonIgnoreType_thenCorrect()
  throws JsonProcessingException, ParseException {

    User.Name name = new User.Name("John", "Doe");
    User user = new User(1, name);

    String result = new ObjectMapper()
      .writeValueAsString(user);

    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("name")));
    assertThat(result, not(containsString("John")));
}
```

### 4.4。`@JsonInclude`

**我们可以使用`@JsonInclude` 来排除空值/null 值/默认值的属性。**

让我们看一个从序列化中排除空值的例子:

```java
@JsonInclude(Include.NON_NULL)
public class MyBean {
    public int id;
    public String name;
}
```

下面是完整的测试:

```java
public void whenSerializingUsingJsonInclude_thenCorrect()
  throws JsonProcessingException {

    MyBean bean = new MyBean(1, null);

    String result = new ObjectMapper()
      .writeValueAsString(bean);

    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("name")));
}
```

### 4.5。`@JsonAutoDetect`

`@JsonAutoDetect`可以覆盖**的默认语义，哪些属性可见，哪些属性不可见**。

首先，让我们通过一个简单的例子来看看注释是如何非常有帮助的；让我们启用私有属性序列化:

```java
@JsonAutoDetect(fieldVisibility = Visibility.ANY)
public class PrivateBean {
    private int id;
    private String name;
}
```

然后是测试:

```java
@Test
public void whenSerializingUsingJsonAutoDetect_thenCorrect()
  throws JsonProcessingException {

    PrivateBean bean = new PrivateBean(1, "My bean");

    String result = new ObjectMapper()
      .writeValueAsString(bean);

    assertThat(result, containsString("1"));
    assertThat(result, containsString("My bean"));
}
```

## 5。杰克逊多态类型处理注释

接下来让我们看看 Jackson 多态类型处理注释:

*   `@JsonTypeInfo`–表示在序列化中包含何种类型信息的详细信息
*   `@JsonSubTypes`–表示注释类型的子类型
*   `@JsonTypeName`–定义用于注释类的逻辑类型名

让我们看一个更复杂的例子，使用所有三个——`@JsonTypeInfo`、`@JsonSubTypes,` 和 `@JsonTypeName –` 来序列化/反序列化实体`Zoo`:

```java
public class Zoo {
    public Animal animal;

    @JsonTypeInfo(
      use = JsonTypeInfo.Id.NAME, 
      include = As.PROPERTY, 
      property = "type")
    @JsonSubTypes({
        @JsonSubTypes.Type(value = Dog.class, name = "dog"),
        @JsonSubTypes.Type(value = Cat.class, name = "cat")
    })
    public static class Animal {
        public String name;
    }

    @JsonTypeName("dog")
    public static class Dog extends Animal {
        public double barkVolume;
    }

    @JsonTypeName("cat")
    public static class Cat extends Animal {
        boolean likesCream;
        public int lives;
    }
}
```

当我们进行序列化时:

```java
@Test
public void whenSerializingPolymorphic_thenCorrect()
  throws JsonProcessingException {
    Zoo.Dog dog = new Zoo.Dog("lacy");
    Zoo zoo = new Zoo(dog);

    String result = new ObjectMapper()
      .writeValueAsString(zoo);

    assertThat(result, containsString("type"));
    assertThat(result, containsString("dog"));
}
```

下面是用`Dog`序列化`Zoo`实例的结果:

```java
{
    "animal": {
        "type": "dog",
        "name": "lacy",
        "barkVolume": 0
    }
}
```

现在来解序列化。让我们从下面的 JSON 输入开始:

```java
{
    "animal":{
        "name":"lacy",
        "type":"cat"
    }
}
```

然后让我们看看如何将它解组到一个`Zoo`实例:

```java
@Test
public void whenDeserializingPolymorphic_thenCorrect()
throws IOException {
    String json = "{\"animal\":{\"name\":\"lacy\",\"type\":\"cat\"}}";

    Zoo zoo = new ObjectMapper()
      .readerFor(Zoo.class)
      .readValue(json);

    assertEquals("lacy", zoo.animal.name);
    assertEquals(Zoo.Cat.class, zoo.animal.getClass());
}
```

## 6。杰克逊将军注解

接下来让我们讨论一些杰克逊的更一般的注释。

### 6.1。`@JsonProperty`

我们可以添加**注释来指示 JSON** 中的属性名。

当我们处理非标准的 getters 和 setters 时，让我们使用`@JsonProperty`来序列化/反序列化属性`name`:

```java
public class MyBean {
    public int id;
    private String name;

    @JsonProperty("name")
    public void setTheName(String name) {
        this.name = name;
    }

    @JsonProperty("name")
    public String getTheName() {
        return name;
    }
}
```

接下来是我们的测试:

```java
@Test
public void whenUsingJsonProperty_thenCorrect()
  throws IOException {
    MyBean bean = new MyBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));

    MyBean resultBean = new ObjectMapper()
      .readerFor(MyBean.class)
      .readValue(result);
    assertEquals("My bean", resultBean.getTheName());
}
```

### 6.2。`@JsonFormat`

**`@JsonFormat`注释指定了序列化日期/时间值时的格式。**

在下面的例子中，我们使用`@JsonFormat`来控制属性`eventDate`的格式:

```java
public class EventWithFormat {
    public String name;

    @JsonFormat(
      shape = JsonFormat.Shape.STRING,
      pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;
}
```

接下来是测试:

```java
@Test
public void whenSerializingUsingJsonFormat_thenCorrect()
  throws JsonProcessingException, ParseException {
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));

    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    EventWithFormat event = new EventWithFormat("party", date);

    String result = new ObjectMapper().writeValueAsString(event);

    assertThat(result, containsString(toParse));
}
```

### 6.3。`@JsonUnwrapped`

**`@JsonUnwrapped`定义序列化/反序列化时应该展开/展平的值。**

让我们看看这到底是如何工作的；我们将使用注释来展开属性`name`:

```java
public class UnwrappedUser {
    public int id;

    @JsonUnwrapped
    public Name name;

    public static class Name {
        public String firstName;
        public String lastName;
    }
}
```

现在让我们序列化这个类的一个实例:

```java
@Test
public void whenSerializingUsingJsonUnwrapped_thenCorrect()
  throws JsonProcessingException, ParseException {
    UnwrappedUser.Name name = new UnwrappedUser.Name("John", "Doe");
    UnwrappedUser user = new UnwrappedUser(1, name);

    String result = new ObjectMapper().writeValueAsString(user);

    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("name")));
}
```

最后，下面是输出的样子——静态嵌套类的字段与其他字段一起展开:

```java
{
    "id":1,
    "firstName":"John",
    "lastName":"Doe"
}
```

### 6.4。`@JsonView`

**`@JsonView`表示包含属性进行序列化/反序列化的视图。**

例如，我们将使用`@JsonView`来序列化`Item`实体的一个实例。

首先，让我们从视图开始:

```java
public class Views {
    public static class Public {}
    public static class Internal extends Public {}
}
```

接下来是使用视图的`Item`实体:

```java
public class Item {
    @JsonView(Views.Public.class)
    public int id;

    @JsonView(Views.Public.class)
    public String itemName;

    @JsonView(Views.Internal.class)
    public String ownerName;
}
```

最后，全面测试:

```java
@Test
public void whenSerializingUsingJsonView_thenCorrect()
  throws JsonProcessingException {
    Item item = new Item(2, "book", "John");

    String result = new ObjectMapper()
      .writerWithView(Views.Public.class)
      .writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("2"));
    assertThat(result, not(containsString("John")));
}
```

### 6.5。`@JsonManagedReference, @JsonBackReference`

**`@JsonManagedReference`和`@JsonBackReference`注释可以处理父/子关系**并解决循环问题。

在下面的例子中，我们使用`@JsonManagedReference`和`@JsonBackReference`来序列化我们的`ItemWithRef`实体:

```java
public class ItemWithRef {
    public int id;
    public String itemName;

    @JsonManagedReference
    public UserWithRef owner;
}
```

我们的`UserWithRef`实体:

```java
public class UserWithRef {
    public int id;
    public String name;

    @JsonBackReference
    public List<ItemWithRef> userItems;
}
```

然后是测试:

```java
@Test
public void whenSerializingUsingJacksonReferenceAnnotation_thenCorrect()
  throws JsonProcessingException {
    UserWithRef user = new UserWithRef(1, "John");
    ItemWithRef item = new ItemWithRef(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("userItems")));
}
```

### 6.6。`@JsonIdentityInfo`

`@JsonIdentityInfo` 表示在序列化/反序列化值时应该使用对象标识，例如在处理无限递归类型的问题时。

在下面的例子中，我们有一个与`UserWithIdentity`实体有双向关系的`ItemWithIdentity`实体:

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class,
  property = "id")
public class ItemWithIdentity {
    public int id;
    public String itemName;
    public UserWithIdentity owner;
}
```

`UserWithIdentity` 实体:

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class,
  property = "id")
public class UserWithIdentity {
    public int id;
    public String name;
    public List<ItemWithIdentity> userItems;
}
```

现在**让我们看看无限递归问题是如何处理的**:

```java
@Test
public void whenSerializingUsingJsonIdentityInfo_thenCorrect()
  throws JsonProcessingException {
    UserWithIdentity user = new UserWithIdentity(1, "John");
    ItemWithIdentity item = new ItemWithIdentity(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, containsString("userItems"));
}
```

以下是序列化项目和用户的完整输出:

```java
{
    "id": 2,
    "itemName": "book",
    "owner": {
        "id": 1,
        "name": "John",
        "userItems": [
            2
        ]
    }
}
```

### 6.7。`@JsonFilter`

**`@JsonFilter`注释指定了在序列化过程中使用的过滤器。**

首先，我们定义实体并指向过滤器:

```java
@JsonFilter("myFilter")
public class BeanWithFilter {
    public int id;
    public String name;
}
```

现在，在完整的测试中，我们定义了过滤器，它从序列化中排除了除`name`之外的所有其他属性:

```java
@Test
public void whenSerializingUsingJsonFilter_thenCorrect()
  throws JsonProcessingException {
    BeanWithFilter bean = new BeanWithFilter(1, "My bean");

    FilterProvider filters 
      = new SimpleFilterProvider().addFilter(
        "myFilter", 
        SimpleBeanPropertyFilter.filterOutAllExcept("name"));

    String result = new ObjectMapper()
      .writer(filters)
      .writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

## 7。自定义杰克逊注释

接下来让我们看看如何创建一个定制的 Jackson 注释。**我们可以利用`@JacksonAnnotationsInside`注解:**

```java
@Retention(RetentionPolicy.RUNTIME)
    @JacksonAnnotationsInside
    @JsonInclude(Include.NON_NULL)
    @JsonPropertyOrder({ "name", "id", "dateCreated" })
    public @interface CustomAnnotation {}
```

现在，如果我们在实体上使用新的注释:

```java
@CustomAnnotation
public class BeanWithCustomAnnotation {
    public int id;
    public String name;
    public Date dateCreated;
}
```

我们可以看到它是如何将现有的注释合并成一个简单的自定义注释，我们可以用它来简化:

```java
@Test
public void whenSerializingUsingCustomAnnotation_thenCorrect()
  throws JsonProcessingException {
    BeanWithCustomAnnotation bean 
      = new BeanWithCustomAnnotation(1, "My bean", null);

    String result = new ObjectMapper().writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("dateCreated")));
}
```

序列化过程的输出:

```java
{
    "name":"My bean",
    "id":1
}
```

## 8。杰克逊 MixIn 注解

接下来让我们看看如何使用 Jackson MixIn 注释。

例如，让我们使用 MixIn 注释来忽略类型`User`的属性:

```java
public class Item {
    public int id;
    public String itemName;
    public User owner;
}
```

```java
@JsonIgnoreType
public class MyMixInForIgnoreType {}
```

那么让我们来看看实际情况:

```java
@Test
public void whenSerializingUsingMixInAnnotation_thenCorrect() 
  throws JsonProcessingException {
    Item item = new Item(1, "book", null);

    String result = new ObjectMapper().writeValueAsString(item);
    assertThat(result, containsString("owner"));

    ObjectMapper mapper = new ObjectMapper();
    mapper.addMixIn(User.class, MyMixInForIgnoreType.class);

    result = mapper.writeValueAsString(item);
    assertThat(result, not(containsString("owner")));
}
```

## 9。禁用杰克逊注释

最后，让我们看看如何**禁用所有杰克森注释**。我们可以通过禁用`MapperFeature.` USE_ANNOTATIONS 来做到这一点，如下例所示:

```java
@JsonInclude(Include.NON_NULL)
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```

现在，禁用注释后，这些注释应该没有效果，并且应该应用库的默认值:

```java
@Test
public void whenDisablingAllAnnotations_thenAllDisabled()
  throws IOException {
    MyBean bean = new MyBean(1, null);

    ObjectMapper mapper = new ObjectMapper();
    mapper.disable(MapperFeature.USE_ANNOTATIONS);
    String result = mapper.writeValueAsString(bean);

    assertThat(result, containsString("1"));
    assertThat(result, containsString("name"));
}
```

禁用批注前序列化的结果:

```java
{"id":1}
```

禁用批注后序列化的结果:

```java
{
    "id":1,
    "name":null
}
```

## 10。结论

在本文中，我们研究了 Jackson 注释，只是触及了通过正确使用它们所能获得的灵活性的皮毛。

所有这些例子和代码片段的实现都可以在 GitHub 的[中找到。](https://web.archive.org/web/20221230230642/https://github.com/eugenp/tutorials/tree/master/jackson-simple)