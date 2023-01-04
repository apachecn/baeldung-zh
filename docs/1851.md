# 从 JSON 生成一个 Java 类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generate-class-from-json>

## 1.概观

在某些情况下，我们需要使用 JSON 文件创建 Java 类，也称为[POJO](/web/20220628122907/https://www.baeldung.com/java-pojo-class)。这是可能的，不用使用一个方便的 **`[jsonschema2pojo](https://web.archive.org/web/20220628122907/https://github.com/joelittlejohn/jsonschema2pojo)`** 库从头开始编写整个类。

在本教程中，我们将看到如何使用这个库从 JSON 对象创建一个 Java 类。

## 2.设置

**我们可以使用`[jsonschema2pojo-core](https://web.archive.org/web/20220628122907/https://search.maven.org/search?q=g:%20org.jsonschema2pojo%20a:%20jsonschema2pojo-core)`依赖关系:**将 JSON 对象转换成 Java 类

```
<dependency>
    <groupId>org.jsonschema2pojo</groupId>
    <artifactId>jsonschema2pojo-core</artifactId>
    <version>1.1.1</version>
</dependency>
```

## 3.JSON 到 Java 类的转换

让我们看看如何使用`jsonschema2pojo`库编写一个程序，它将把一个 JSON 文件转换成一个 Java 类。

首先，我们将创建一个方法`convertJsonToJavaClass`，它将 JSON 文件转换成 POJO 类，并接受四个参数:

*   一个`inputJson`文件的 URL
*   POJO 将在哪里生成
*   `packageName`POJO 类所属的
*   一个输出 POJO `className`。

然后，我们将定义该方法中的步骤:

*   我们将从创建一个`JCodeModel` 类的对象开始，这将生成 Java 类
*   然后，我们将定义`jsonschema2pojo`的配置，它让程序识别输入源文件是 JSON(`getSourceType`方法)
*   此外，我们将把这个配置传递给一个 [`RuleFactory`](https://web.archive.org/web/20220628122907/https://github.com/joelittlejohn/jsonschema2pojo/blob/888884421d8357cb8d5537b3d9ffb27cca278edc/jsonschema2pojo-core/src/main/java/org/jsonschema2pojo/rules/RuleFactory.java) ，它将用于为这个映射创建类型生成规则
*   我们将使用这个工厂和`SchemaGenerator`对象创建一个 [`SchemaMapper`](https://web.archive.org/web/20220628122907/https://github.com/joelittlejohn/jsonschema2pojo/blob/master/jsonschema2pojo-core/src/main/java/org/jsonschema2pojo/SchemaMapper.java) ，它从提供的 JSON 生成 Java 类型
*   最后，我们将调用`JCodeModel`的`build`方法来创建输出类

让我们来看看实现:

```
public void convertJsonToJavaClass(URL inputJsonUrl, File outputJavaClassDirectory, String packageName, String javaClassName) 
  throws IOException {
    JCodeModel jcodeModel = new JCodeModel();

    GenerationConfig config = new DefaultGenerationConfig() {
        @Override
        public boolean isGenerateBuilders() {
            return true;
        }

        @Override
        public SourceType getSourceType() {
            return SourceType.JSON;
        }
    };

    SchemaMapper mapper = new SchemaMapper(new RuleFactory(config, new Jackson2Annotator(config), new SchemaStore()), new SchemaGenerator());
    mapper.generate(jcodeModel, javaClassName, packageName, inputJsonUrl);

    jcodeModel.build(outputJavaClassDirectory);
}
```

## 4.输入和输出

让我们使用这个样本 JSON 来执行程序:

```
{
  "name": "Baeldung",
  "area": "tech blogs",
  "author": "Eugen",
  "id": 32134,
  "topics": [
    "java",
    "kotlin",
    "cs",
    "linux"
  ],
  "address": {
    "city": "Bucharest",
    "country": "Romania"
  }
}
```

一旦我们执行我们的程序，它就在给定的目录中创建下面的 Java 类:

```
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder({"name", "area", "author", "id", "topics", "address"})
@Generated("jsonschema2pojo")
public class Input {

    @JsonProperty("name")
    private String name;
    @JsonProperty("area")
    private String area;
    @JsonProperty("author")
    private String author;
    @JsonProperty("id")
    private Integer id;
    @JsonProperty("topics")
    private List<String> topics = new ArrayList<String>();
    @JsonProperty("address")
    private Address address;
    @JsonIgnore
    private Map<String, Object> additionalProperties = new HashMap<String, Object>();

    // getters & setters
    // hashCode & equals
    // toString
}
```

**注意，它已经为嵌套的 JSON 对象创建了一个新的`Address`类**:

```
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder({"city", "country"})
@Generated("jsonschema2pojo")
public class Address {

    @JsonProperty("city")
    private String city;
    @JsonProperty("country")
    private String country;
    @JsonIgnore
    private Map<String, Object> additionalProperties = new HashMap<String, Object>();

    // getters & setters
    // hashCode & equals
    // toString
}
```

我们也可以通过参观 jsonschema2pojo.org T4 来实现这一切。`jsonschema2pojo`工具接受一个 JSON(或 YAML)模式文档并生成 DTO 风格的 Java 类。它提供了许多选项，您可以选择包含在 Java 类中，包括构造函数以及`hashCode, equals,` 和`toString`方法。

## 5.结论

在本教程中，我们通过使用`jsonschema2pojo`库的例子讲述了如何从 JSON 创建一个 Java 类。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220628122907/https://github.com/eugenp/tutorials/tree/master/json-modules/json-2)