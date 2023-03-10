# 更多杰克逊注解

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-advanced-annotations>

## 1。概述

本文涵盖了前一篇文章[Jackson 注解指南](/web/20221019195805/https://www.baeldung.com/jackson-annotations)中没有涉及的一些附加注解——我们将讨论其中的七个。

## 2。`@JsonIdentityReference`

`@JsonIdentityReference`用于定制将被序列化为对象标识而非完整 POJOs 的对象引用。它与`@JsonIdentityInfo`协同工作，在每次序列化中强制使用对象标识，这与`@JsonIdentityReference`不在时的情况不同。当处理对象之间的循环依赖时，这两个注释非常有用。请参考[Jackson-双向关系](/web/20221019195805/https://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion)一文的第 4 节了解更多信息。

为了演示使用`@JsonIdentityReference`，我们将定义两个不同的 bean 类，不使用和使用这个注释。

没有`@JsonIdentityReference`的豆子:

```java
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
public class BeanWithoutIdentityReference {
    private int id;
    private String name;

    // constructor, getters and setters
}
```

对于使用`@JsonIdentityReference`的 bean，我们选择`id`属性作为对象标识:

```java
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
@JsonIdentityReference(alwaysAsId = true)
public class BeanWithIdentityReference {
    private int id;
    private String name;

    // constructor, getters and setters
}
```

在第一种情况下，当`@JsonIdentityReference`不存在时，该 bean 被序列化，并包含其属性的全部细节:

```java
BeanWithoutIdentityReference bean 
  = new BeanWithoutIdentityReference(1, "Bean Without Identity Reference Annotation");
String jsonString = mapper.writeValueAsString(bean);
```

上面序列化的输出:

```java
{
    "id": 1,
    "name": "Bean Without Identity Reference Annotation"
}
```

当使用`@JsonIdentityReference`时，bean 被序列化为一个简单的标识:

```java
BeanWithIdentityReference bean 
  = new BeanWithIdentityReference(1, "Bean With Identity Reference Annotation");
String jsonString = mapper.writeValueAsString(bean);
assertEquals("1", jsonString);
```

## 3。`@JsonAppend`

当对象被序列化时, `@JsonAppend`注释用于向对象添加除常规属性之外的虚拟属性。当我们想要将补充信息直接添加到 JSON 字符串中，而不是更改类定义时，这是必要的。例如，将 bean 的`version`元数据插入到相应的 JSON 文档中可能比为它提供一个额外的属性更方便。

假设我们有一个没有`@JsonAppend`的 bean，如下所示:

```java
public class BeanWithoutAppend {
    private int id;
    private String name;

    // constructor, getters and setters
}
```

一个测试将确认在没有`@JsonAppend`注释的情况下，序列化输出不包含关于补充`version` 属性的信息，尽管我们试图添加到`ObjectWriter`对象:

```java
BeanWithoutAppend bean = new BeanWithoutAppend(2, "Bean Without Append Annotation");
ObjectWriter writer 
  = mapper.writerFor(BeanWithoutAppend.class).withAttribute("version", "1.0");
String jsonString = writer.writeValueAsString(bean);
```

序列化输出:

```java
{
    "id": 2,
    "name": "Bean Without Append Annotation"
}
```

现在，假设我们有一个用`@JsonAppend`注释的 bean:

```java
@JsonAppend(attrs = { 
  @JsonAppend.Attr(value = "version") 
})
public class BeanWithAppend {
    private int id;
    private String name;

    // constructor, getters and setters
}
```

与前一个类似的测试将验证当应用`@JsonAppend`注释时，补充属性在序列化后被包含:

```java
BeanWithAppend bean = new BeanWithAppend(2, "Bean With Append Annotation");
ObjectWriter writer 
  = mapper.writerFor(BeanWithAppend.class).withAttribute("version", "1.0");
String jsonString = writer.writeValueAsString(bean);
```

该序列化的输出显示已经添加了`version`属性:

```java
{
    "id": 2,
    "name": "Bean With Append Annotation",
    "version": "1.0"
}
```

## 4。`@JsonNaming`

`@JsonNaming`注释用于选择序列化中属性的命名策略，覆盖默认值。使用`value`元素，我们可以指定任何策略，包括定制策略。

除了默认的`LOWER_CAMEL_CASE`(例如`lowerCamelCase`)之外，为了方便起见，Jackson library 还为我们提供了另外四种内置的属性命名策略:

*   `KEBAB_CASE`:名称元素用连字符分隔，如`kebab-case`。
*   `LOWER_CASE`:所有字母小写，无分隔符，如`lowercase`。
*   `SNAKE_CASE`:所有字母小写，名称元素之间用下划线作为分隔符，如`snake_case`。
*   `UPPER_CAMEL_CASE`:所有名称元素，包括第一个，都以大写字母开头，后面跟着小写字母，没有分隔符，例如`UpperCamelCase`。

这个例子将说明使用 snake case 名称序列化属性的方法，其中名为`beanName`的属性被序列化为`bean_name.`

给定一个 bean 定义:

```java
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class NamingBean {
    private int id;
    private String beanName;

    // constructor, getters and setters
}
```

下面的测试演示了指定的命名规则按要求工作:

```java
NamingBean bean = new NamingBean(3, "Naming Bean");
String jsonString = mapper.writeValueAsString(bean);        
assertThat(jsonString, containsString("bean_name"));
```

`jsonString`变量包含以下数据:

```java
{
    "id": 3,
    "bean_name": "Naming Bean"
}
```

## 5。`@JsonPropertyDescription`

Jackson 库能够在名为 [JSON Schema](https://web.archive.org/web/20221019195805/https://github.com/FasterXML/jackson-module-jsonSchema) 的独立模块的帮助下为 Java 类型创建 JSON 模式。当我们希望在序列化 Java 对象时指定预期的输出，或者在反序列化之前验证 JSON 文档时，该模式非常有用。

通过提供`description`字段，`@JsonPropertyDescription`注释允许向创建的 JSON 模式添加人类可读的描述。

本节使用下面声明的 bean 来演示`@JsonPropertyDescription`的功能:

```java
public class PropertyDescriptionBean {
    private int id;
    @JsonPropertyDescription("This is a description of the name property")
    private String name;

    // getters and setters
}
```

添加了`description`字段的 JSON 模式的生成方法如下所示:

```java
SchemaFactoryWrapper wrapper = new SchemaFactoryWrapper();
mapper.acceptJsonFormatVisitor(PropertyDescriptionBean.class, wrapper);
JsonSchema jsonSchema = wrapper.finalSchema();
String jsonString = mapper.writeValueAsString(jsonSchema);
assertThat(jsonString, containsString("This is a description of the name property"));
```

正如我们所看到的，JSON 模式的生成是成功的:

```java
{
    "type": "object",
    "id": "urn:jsonschema:com:baeldung:jackson:annotation:extra:PropertyDescriptionBean",
    "properties": 
    {
        "name": 
        {
            "type": "string",
            "description": "This is a description of the name property"
        },

        "id": 
        {
            "type": "integer"
        }
    }
}
```

## 6。`@JsonPOJOBuilder`

`@JsonPOJOBuilder`注释用于配置一个构建器类，以定制 JSON 文档的反序列化，从而在命名约定不同于默认约定时恢复 POJOs。

假设我们需要反序列化以下 JSON 字符串:

```java
{
    "id": 5,
    "name": "POJO Builder Bean"
}
```

该 JSON 源代码将用于创建`POJOBuilderBean`的实例:

```java
@JsonDeserialize(builder = BeanBuilder.class)
public class POJOBuilderBean {
    private int identity;
    private String beanName;

    // constructor, getters and setters
}
```

bean 属性的名称不同于 JSON string 中的字段名称。这就是`@JsonPOJOBuilder`来救援的地方。

`@JsonPOJOBuilder`注释伴随着两个属性:

*   `buildMethodName`:在将 JSON 字段绑定到 bean 的属性之后，用于实例化预期 bean 的无参数方法的名称。默认名称是`build`。
*   `withPrefix`:自动检测 JSON 和 bean 属性匹配的名称前缀。默认前缀是`with`。

这个例子使用了下面的`BeanBuilder`类，它用在`POJOBuilderBean`上:

```java
@JsonPOJOBuilder(buildMethodName = "createBean", withPrefix = "construct")
public class BeanBuilder {
    private int idValue;
    private String nameValue;

    public BeanBuilder constructId(int id) {
        idValue = id;
        return this;
    }

    public BeanBuilder constructName(String name) {
        nameValue = name;
        return this;
    }

    public POJOBuilderBean createBean() {
        return new POJOBuilderBean(idValue, nameValue);
    }
}
```

在上面的代码中，我们配置了`@JsonPOJOBuilder`来使用名为`createBean`的构建方法和用于匹配属性的`construct`前缀。

对 bean 应用`@JsonPOJOBuilder`的描述和测试如下:

```java
String jsonString = "{\"id\":5,\"name\":\"POJO Builder Bean\"}";
POJOBuilderBean bean = mapper.readValue(jsonString, POJOBuilderBean.class);

assertEquals(5, bean.getIdentity());
assertEquals("POJO Builder Bean", bean.getBeanName());
```

结果显示，尽管属性名不匹配，但已经从 JSON 源成功地重新创建了一个新的数据对象。

## 7。`@JsonTypeId`

`@JsonTypeId`注释用于指示当包含多态类型信息时，带注释的属性应该序列化为类型 id，而不是常规属性。在反序列化过程中使用多态元数据来重新创建与序列化前相同子类型的对象，而不是已声明超类型的对象。

有关 Jackson 处理继承的更多信息，请参见 Jackson 中[继承的第 2 节。](/web/20221019195805/https://www.baeldung.com/jackson-inheritance)

假设我们有一个 bean 类定义如下:

```java
public class TypeIdBean {
    private int id;
    @JsonTypeId
    private String name;

    // constructor, getters and setters
}
```

下面的测试验证了`@JsonTypeId`如预期的那样工作:

```java
mapper.enableDefaultTyping(DefaultTyping.NON_FINAL);
TypeIdBean bean = new TypeIdBean(6, "Type Id Bean");
String jsonString = mapper.writeValueAsString(bean);

assertThat(jsonString, containsString("Type Id Bean"));
```

序列化过程的输出:

```java
[
    "Type Id Bean",
    {
        "id": 6
    }
]
```

## 8。`@JsonTypeIdResolver`

`@JsonTypeIdResolver`注释用于表示序列化和反序列化中的自定义类型标识处理程序。该处理程序负责 JSON 文档中包含的 Java 类型和类型 id 之间的转换。

假设在处理下面的类层次结构时，我们希望在 JSON 字符串中嵌入类型信息。

`AbstractBean`超类:

```java
@JsonTypeInfo(
  use = JsonTypeInfo.Id.NAME, 
  include = JsonTypeInfo.As.PROPERTY, 
  property = "@type"
)
@JsonTypeIdResolver(BeanIdResolver.class)
public class AbstractBean {
    private int id;

    protected AbstractBean(int id) {
        this.id = id;
    }

    // no-arg constructor, getter and setter
}
```

`FirstBean`子类:

```java
public class FirstBean extends AbstractBean {
    String firstName;

    public FirstBean(int id, String name) {
        super(id);
        setFirstName(name);
    }

    // no-arg constructor, getter and setter
}
```

`LastBean`子类:

```java
public class LastBean extends AbstractBean {
    String lastName;

    public LastBean(int id, String name) {
        super(id);
        setLastName(name);
    }

    // no-arg constructor, getter and setter
}
```

这些类的实例用于填充一个`BeanContainer`对象:

```java
public class BeanContainer {
    private List<AbstractBean> beans;

    // getter and setter
}
```

我们可以看到,`AbstractBean`类用`@JsonTypeIdResolver`进行了注释，表明它使用自定义的`TypeIdResolver`来决定如何在序列化中包含子类型信息，以及如何反过来利用元数据。

下面是处理类型信息包含的解析器类:

```java
public class BeanIdResolver extends TypeIdResolverBase {

    private JavaType superType;

    @Override
    public void init(JavaType baseType) {
        superType = baseType;
    }

    @Override
    public Id getMechanism() {
        return Id.NAME;
    }

    @Override
    public String idFromValue(Object obj) {
        return idFromValueAndType(obj, obj.getClass());
    }

    @Override
    public String idFromValueAndType(Object obj, Class<?> subType) {
        String typeId = null;
        switch (subType.getSimpleName()) {
        case "FirstBean":
            typeId = "bean1";
            break;
        case "LastBean":
            typeId = "bean2";
        }
        return typeId;
    }

    @Override
    public JavaType typeFromId(DatabindContext context, String id) {
        Class<?> subType = null;
        switch (id) {
        case "bean1":
            subType = FirstBean.class;
            break;
        case "bean2":
            subType = LastBean.class;
        }
        return context.constructSpecializedType(superType, subType);
    }
}
```

两个最著名的方法是`idFromValueAndType`和`typeFromId`，前者告诉在序列化 POJOs 时包含类型信息的方法，后者使用元数据确定重新创建的对象的子类型。

为了确保序列化和反序列化都能正常工作，让我们编写一个测试来验证整个过程。

首先，我们需要实例化一个 bean 容器和 bean 类，然后用 bean 实例填充该容器:

```java
FirstBean bean1 = new FirstBean(1, "Bean 1");
LastBean bean2 = new LastBean(2, "Bean 2");

List<AbstractBean> beans = new ArrayList<>();
beans.add(bean1);
beans.add(bean2);

BeanContainer serializedContainer = new BeanContainer();
serializedContainer.setBeans(beans);
```

接下来，`BeanContainer`对象被序列化，我们确认结果字符串包含类型信息:

```java
String jsonString = mapper.writeValueAsString(serializedContainer);
assertThat(jsonString, containsString("bean1"));
assertThat(jsonString, containsString("bean2"));
```

序列化的输出如下所示:

```java
{
    "beans": 
    [
        {
            "@type": "bean1",
            "id": 1,
            "firstName": "Bean 1"
        },

        {
            "@type": "bean2",
            "id": 2,
            "lastName": "Bean 2"
        }
    ]
}
```

该 JSON 结构将用于重新创建与序列化前相同子类型的对象。以下是反序列化的实现步骤:

```java
BeanContainer deserializedContainer = mapper.readValue(jsonString, BeanContainer.class);
List<AbstractBean> beanList = deserializedContainer.getBeans();
assertThat(beanList.get(0), instanceOf(FirstBean.class));
assertThat(beanList.get(1), instanceOf(LastBean.class));
```

## 9。结论

本教程详细解释了几种不常用的 Jackson 注释。这些例子和代码片段的实现可以在 GitHub 项目中找到。