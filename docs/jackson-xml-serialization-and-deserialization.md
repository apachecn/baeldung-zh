# 用 Jackson 实现 XML 序列化和反序列化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-xml-serialization-and-deserialization>

## 1。概述

在本教程中，我们将学习**如何使用 Jackson 2.x 将 Java 对象序列化为 XML 数据，并将它们反序列化回 POJO** 。

我们将关注不需要太多复杂性或定制的基本操作。

## 2。`XmlMapper`对象

`XmlMapper` 是 Jackson 2.x 中帮助我们序列化的主类，所以我们需要创建它的一个实例:

```
XmlMapper mapper = new XmlMapper();
```

这个 `mapper` 在`jackson-dataformat-xml` jar 中是可用的，所以我们必须将它作为一个依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.11.1</version>
</dependency>
```

请在 Maven 仓库中检查最新版本的 jackson-dataformat-xml 依赖关系。

## 3。将 Java 序列化为 XML

`XmlMapper`是`ObjectMapper,`的子类，用于 JSON 序列化；但是，它向父类添加了一些特定于 XML 的调整。

让我们看看如何使用它来进行实际的序列化。让我们首先创建一个 Java 类:

```
class SimpleBean {
    private int x = 1;
    private int y = 2;

    //standard setters and getters
}
```

### 3.1。`String`序列化为 XML

我们可以将 Java 对象序列化成 XML `String`:

```
@Test
public void whenJavaSerializedToXmlStr_thenCorrect() throws JsonProcessingException {
    XmlMapper xmlMapper = new XmlMapper();
    String xml = xmlMapper.writeValueAsString(new SimpleBean());
    assertNotNull(xml);
}
```

结果，我们会得到:

```
<SimpleBean>
    <x>1</x>
    <y>2</y>
</SimpleBean>
```

### 3.2。序列化为 XML 文件

我们还可以将 Java 对象序列化为 XML 文件:

```
@Test
public void whenJavaSerializedToXmlFile_thenCorrect() throws IOException {
    XmlMapper xmlMapper = new XmlMapper();
    xmlMapper.writeValue(new File("simple_bean.xml"), new SimpleBean());
    File file = new File("simple_bean.xml");
    assertNotNull(file);
}
```

下面我们可以看到名为`simple_bean.xml`的结果文件的内容:

```
<SimpleBean>
    <x>1</x>
    <y>2</y>
</SimpleBean>
```

## 4。将 XML 反序列化为 Java

在这一节中，我们将研究如何从 XML 中获取 Java 对象。

### 4.1。从 XML 字符串反序列化

与序列化一样，我们也可以将 XML 字符串反序列化为 Java 对象:

```
@Test
public void whenJavaGotFromXmlStr_thenCorrect() throws IOException {
    XmlMapper xmlMapper = new XmlMapper();
    SimpleBean value
      = xmlMapper.readValue("<SimpleBean><x>1</x><y>2</y></SimpleBean>", SimpleBean.class);
    assertTrue(value.getX() == 1 && value.getY() == 2);
}
```

### 4.2。从 XML 文件反序列化

同样，如果我们有一个 XML 文件，我们可以将其转换回 Java 对象。

```
@Test
public void whenJavaGotFromXmlFile_thenCorrect() throws IOException {
    File file = new File("simple_bean.xml");
    XmlMapper xmlMapper = new XmlMapper();
    SimpleBean value = xmlMapper.readValue(file, SimpleBean.class);
    assertTrue(value.getX() == 1 && value.getY() == 2);
}
```

## 5。处理资本化元素

在这一节中，我们将讨论如何处理这样的场景:要么我们有要反序列化的带有大写元素的 XML，要么我们需要将 Java 对象序列化为带有一个或多个大写元素的 XML。

### 5.1。从 XML 中反序列化`String`

假设我们有一个字段大写的 XML:

```
<SimpleBeanForCapitalizedFields>
    <X>1</X>
    <y>2</y>
</SimpleBeanForCapitalizedFields>
```

为了正确处理大写的元素，我们需要用`@JsonProperty`注释来注释“x”字段:

```
class SimpleBeanForCapitalizedFields {
    @JsonProperty("X")
    private int x = 1;
    private int y = 2;

    // standard getters, setters
}
```

我们现在可以正确地将 XML `String`反序列化回 Java 对象:

```
@Test
public void whenJavaGotFromXmlStrWithCapitalElem_thenCorrect() throws IOException {
    XmlMapper xmlMapper = new XmlMapper();
    SimpleBeanForCapitalizedFields value
      = xmlMapper.readValue(
      "<SimpleBeanForCapitalizedFields><X>1</X><y>2</y></SimpleBeanForCapitalizedFields>",
      SimpleBeanForCapitalizedFields.class);
    assertTrue(value.getX() == 1 && value.getY() == 2);
}
```

### 5.2。序列化为 XML 字符串

通过用`@JsonProperty,`注释必需的字段，我们可以正确地将一个 Java 对象序列化为一个 XML `String`,其中包含一个或多个大写的元素:

```
@Test
public void whenJavaSerializedToXmlFileWithCapitalizedField_thenCorrect()
  throws IOException {
    XmlMapper xmlMapper = new XmlMapper();
    xmlMapper.writeValue(new File("target/simple_bean_capitalized.xml"),
      new SimpleBeanForCapitalizedFields());
    File file = new File("target/simple_bean_capitalized.xml");
    assertNotNull(file);
}
```

## 6.**将`List`序列化为 XML**

`XmlMapper`能够将整个 Java bean 序列化为一个文档。为了将 Java 对象转换成 XML，我们举一个简单的例子，它包含一个嵌套的对象和数组。

我们的目的是将一个`Person`对象及其组合的`Address`对象序列化为 XML。

我们最终的 XML 看起来会像这样:

```
<Person>
    <firstName>Rohan</firstName>
    <lastName>Daye</lastName>
    <phoneNumbers>
        <phoneNumbers>9911034731</phoneNumbers>
        <phoneNumbers>9911033478</phoneNumbers>
    </phoneNumbers>
    <address>
        <streetName>Name1</streetName>
        <city>City1</city>
    </address>
    <address>
        <streetName>Name2</streetName>
        <city>City2</city>
    </address>
</Person>
```

**注意，我们的电话号码被封装在一个`phoneNumbers`包装器中，而我们的地址没有。**

我们可以通过`Person`类中的`@JacksonXMLElementWrapper`注释来表达这种细微差别:

```
public final class Person {
    private String firstName;
    private String lastName;
    private List<String> phoneNumbers = new ArrayList<>();
    @JacksonXmlElementWrapper(useWrapping = false)
    private List<Address> address = new ArrayList<>();

    //standard setters and getters
}
```

事实上，我们可以用`@JacksonXmlElementWrapper(localName = ‘phoneNumbers').` 改变包装元素的名称，或者，如果我们不想包装我们的元素，我们可以用`@JacksonXmlElementWrapper(useWrapping = false)`禁用映射。

然后我们将定义我们的`Address` 类型:

```
public class Address {
    String streetName;
    String city;
    //standard setters and getters
}
```

**杰克逊为我们处理剩下的事情。**像以前一样，我们可以简单地再叫`writeValue`:

```
private static final String XML = "<Person>...</Person>";

@Test
public void whenJavaSerializedToXmlFile_thenSuccess() throws IOException {
    XmlMapper xmlMapper = new XmlMapper();
    Person person = testPerson(); // test data
    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
    xmlMapper.writeValue(byteArrayOutputStream, person); 
    assertEquals(XML, byteArrayOutputStream.toString()); 
}
```

## 7。将 XML 反序列化到`List`

Jackson 也可以读取包含对象列表的 XML。

如果我们像以前一样使用相同的 XML，`readValue` 方法就很好:

```
@Test
public void whenJavaDeserializedFromXmlFile_thenCorrect() throws IOException {
    XmlMapper xmlMapper = new XmlMapper();
    Person value = xmlMapper.readValue(XML, Person.class);
    assertEquals("City1", value.getAddress().get(0).getCity());
    assertEquals("City2", value.getAddress().get(1).getCity());
}
```

## 8。结论

这篇简短的文章演示了如何将简单的 POJO 序列化为 XML，并从基本的 XML 数据中获取 POJO。

我们还探索了如何序列化和反序列化包含集合的复杂 beans。

本文附带的源代码可以在 [GitHub](https://web.archive.org/web/20220727020632/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions) 上获得。