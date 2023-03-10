# 用 JSON-LD 实现超媒体序列化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/json-linked-data>

## 1。概述

[JSON-LD](https://web.archive.org/web/20220627091154/https://json-ld.org/) 是基于 JSON 的 [RDF](https://web.archive.org/web/20220627091154/https://www.w3.org/TR/rdf11-concepts/) 格式，用于表示[关联数据](https://web.archive.org/web/20220627091154/https://www.w3.org/DesignIssues/LinkedData.html)。它能够用超媒体能力扩展现有的 JSON 对象；换句话说，以机器可读的方式包含链接的能力。

在本教程中，**我们将研究几个基于 Jackson 的选项，将 JSON-LD 格式直接序列化和反序列化为 POJO**。我们还将介绍 JSON-LD 的基本概念，这将使我们能够理解这些示例。

## 2。基本概念

第一次看到 JSON-LD 文档时，我们注意到一些成员名以字符`@`开头。这些是 JSON-LD 关键字，它们的值帮助我们理解文档的其余部分。

为了浏览 JSON-LD 的世界并理解本教程，我们需要注意四个关键词:

*   `@context` 是 JSON 对象的描述，它包含解释文档所需的所有内容的键值映射
*   `@vocab`是`@context`中的一个可能的键，它引入了一个默认词汇表，使`@context`对象变得更短
*   `@id` 是标识链接的关键字，或者作为资源属性表示到资源本身的直接链接，或者作为`@type`值将任何字段标记为链接
*   `@type`是在资源层或`@context`中标识资源类型的关键字；例如，定义嵌入资源的类型

## 3。Java 中的序列化

在我们继续之前，我们应该看看我们以前的教程，以刷新我们对[杰克逊`ObjectMapper`、](/web/20220627091154/https://www.baeldung.com/jackson-object-mapper-tutorial)杰克逊注释和[自定义杰克逊序列化器](/web/20220627091154/https://www.baeldung.com/jackson-custom-serialization)的记忆。

已经熟悉了 Jackson，我们可能会意识到，我们可以使用`@JsonProperty`注释轻松地将任何 POJO 中的两个定制字段序列化为`@id`和`@type`。然而，**手工编写`@context`可能工作量很大，也容易出错**。

因此，为了避免这种容易出错的方法，让我们更仔细地看看我们可以用于`@context`生成的两个库。不幸的是，它们都不能生成 JSON-LD 的所有特性，但是我们稍后也会看到它们的缺点。

## 4。与 Jackson-Jsonld 的序列化

**[Jackson-Jsonld](https://web.archive.org/web/20220627091154/https://github.com/io-informatics/jackson-jsonld) 是一个 Jackson 模块，它能够以一种方便的方式对 POJOs 进行注释，从而生成 JSON-LD 文档。**

### 4.1。Maven 依赖关系

首先，让我们将 [`jackson-jsonld`](https://web.archive.org/web/20220627091154/https://search.maven.org/artifact/com.io-informatics.oss/jackson-jsonld) 作为依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>com.io-informatics.oss</groupId>
    <artifactId>jackson-jsonld</artifactId>
    <version>0.1.1</version>
</dependency>
```

### 4.2。示例

然后，让我们创建我们的示例 POJO，并为`@context`生成进行注释:

```java
@JsonldResource
@JsonldNamespace(name = "s", uri = "http://schema.org/")
@JsonldType("s:Person")
@JsonldLink(rel = "s:knows", name = "knows", href = "http://example.com/person/2345")
public class Person {
    @JsonldId
    private String id;
    @JsonldProperty("s:name")
    private String name;

    // constructor, getters, setters
}
```

让我们解构一下这些步骤，以了解我们做了什么:

*   使用`@JsonldResource`,我们将 POJO 标记为 JSON-LD 资源进行处理
*   在`@JsonldNamespace`中，我们定义了我们想要使用的词汇的简写
*   我们在`@JsonldType`中指定的参数将成为资源的`@type`
*   我们使用了`@JsonldLink`注释来添加到资源的链接。处理后，`name`参数将用作字段名，并作为关键字添加到`@context.`中，`href`将是字段值，`rel`将是`@context`中的映射值
*   我们用`@JsonldId`标记的字段将成为资源的`@id`
*   我们在`@JsonldProperty`中指定的参数将成为映射到`@context`中字段名称的值

接下来，让我们生成 JSON-LD 文档。

首先，我们应该在`ObjectMapper`中注册`JsonldModule`。**这个模块包含了一个定制的`Serializer`，Jackson 将会把它用于标有`@JsonldResource`注释**的 POJOs。

然后，我们将继续使用`ObjectMapper`来生成 JSON-LD 文档:

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.registerModule(new JsonldModule());

Person person = new Person("http://example.com/person/1234", "Example Name");
String personJsonLd = objectMapper.writeValueAsString(person);
```

**因此，`personJsonLd`变量现在应该包含:**

```java
{
  "@type": "s:Person",
  "@context": {
    "s": "http://schema.org/",
    "name": "s:name",
    "knows": {
      "@id": "s:knows",
      "@type": "@id"
    }
  },
  "name": "Example Name",
  "@id": "http://example.com/person/1234",
  "knows": "http://example.com/person/2345"
}
```

### 4.3。注意事项

在我们为项目选择这个库之前，我们应该考虑以下几点:

*   使用`@vocab`关键字是不可能的，所以我们必须使用`@JsonldNamespace`来提供解析域名的简写，或者每次都写出完整的国际化资源标识符(IRI)
*   我们只能在编译时定义链接，所以为了添加链接运行时，我们需要使用反射来更改注释中的参数

## 5。用 Hydra-Jsonld 进行序列化

Hydra-Jsonld 是 [Hydra-Java](https://web.archive.org/web/20220627091154/https://github.com/dschulten/hydra-java/) 库的一个模块，主要用于为 Spring 应用程序创建方便的 JSON-LD 响应。它使用 [Hydra 词汇表](https://web.archive.org/web/20220627091154/http://www.hydra-cg.com/spec/latest/core/)使 JSON-LD 文档更具表现力。

**然而**，**Hydra-Jsonld 模块包含一个 Jackson `Serializer`和一些注释，我们可以用它们在 Spring 框架之外生成 JSON-LD 文档**。

### 5.1。Maven 依赖关系

首先，让我们将 [`hydra-jsonld`](https://web.archive.org/web/20220627091154/https://search.maven.org/artifact/de.escalon.hypermedia/hydra-jsonld) 的依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>de.escalon.hypermedia</groupId>
    <artifactId>hydra-jsonld</artifactId>
    <version>0.4.2</version>
</dependency>
```

### 5.2。示例

其次，我们来为`@context`代注释一下我们的 POJO。

Hydra-Jsonld 自动生成一个默认的`@context`，不需要注释。**如果我们对缺省值满意，我们只需要添加`@id`来获得一个有效的 JSON-LD 文档。**

默认词汇表将是 schema.org 的[词汇表、`@type`Java 的`class`名称，POJO 的公共属性都将包含在生成的 JSON-LD 文档中。](https://web.archive.org/web/20220627091154/https://schema.org/)

在本例中，**让我们用自定义值**覆盖这些默认值:

```java
@Vocab("http://example.com/vocab/")
@Expose("person")
public class Person {
    private String id;
    private String name;

    // constructor

    @JsonProperty("@id")
    public String getId() {
        return id;
    }

    @Expose("fullName")
    public String getName() {
        return name;
    }
}
```

同样，让我们仔细看看所涉及的步骤:

*   与 Jackson-Jsonld 的例子相比，由于 Spring 框架之外的 Hydra-Jsonld 的限制，我们在 POJO 中省略了`knows`字段
*   我们用`@Vocab`注释设置我们的首选词汇表
*   通过在类上使用`@Expose`注释，我们设置了一个不同的资源`@type`
*   我们在属性上使用了相同的`@Expose`注释，将其映射设置为`@context`中的自定义值
*   为了从属性生成`@id`,我们使用了 Jackson 的`@JsonProperty`注释

接下来，让我们配置一个 Jackson `Module`的实例，我们可以在`ObjectMapper`中注册它。**我们将添加`JacksonHydraSerializer`作为`BeanSerializerModifier`，这样它可以应用于所有正在序列化的 POJOs】:**

```java
SimpleModule getJacksonHydraSerializerModule() {
    return new SimpleModule() {
        @Override
        public void setupModule(SetupContext context) {
            super.setupModule(context);

            context.addBeanSerializerModifier(new BeanSerializerModifier() {
                @Override
                public JsonSerializer<?> modifySerializer(
                  SerializationConfig config, 
                  BeanDescription beanDesc, 
                  JsonSerializer<?> serializer) {
                    if (serializer instanceof BeanSerializerBase) {
                        return new JacksonHydraSerializer((BeanSerializerBase) serializer);
                    } else {
                        return serializer;
                    }
                }
            });
        }
    };
}
```

然后让我们在`ObjectMapper`中注册`Module`并使用它`.` **我们还应该设置`ObjectMapper`只包含非`null`值**以生成有效的 JSON-LD 文档:

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.registerModule(getJacksonHydraSerializerModule());
objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

Person person = new Person("http://example.com/person/1234", "Example Name");

String personJsonLd = objectMapper.writeValueAsString(person);
```

**现在，`personJsonLd`变量应该包含:**

```java
{
  "@context": {
    "@vocab": "http://example.com/vocab/",
    "name": "fullName"
  },
  "@type": "person",
  "name": "Example Name",
  "@id": "http://example.com/person/1234"
}
```

### 5.3。注意事项

尽管在 Spring 框架之外使用 Hydra-Jsonld 在技术上是可行的，但它最初是为与 [Spring-HATEOAS](/web/20220627091154/https://www.baeldung.com/spring-hateoas-tutorial) 一起使用而设计的。因此，没有办法像我们在 Jackson-Jsonld 中看到的那样生成带有注释的链接。另一方面，它们是为一些特定于 Spring 的类自动生成的。

在我们为项目选择这个库之前，我们应该考虑以下几点:

*   将它与 Spring 框架一起使用将会启用额外的特性
*   如果我们不使用 Spring 框架，就没有简单的方法来生成链接
*   我们不能禁用`@vocab`的用法，我们只能覆盖它

## 6。用 Jsonld-Java 和 Jackson 进行反序列化

[Jsonld-Java](https://web.archive.org/web/20220627091154/https://github.com/jsonld-java/jsonld-java) 是 JSON-LD 1.0 规范和 API 的 Java 实现，可惜不是最新版本。

对于 1.1 规范版本的实现，请看一下 [Titanium JSON-LD](https://web.archive.org/web/20220627091154/https://github.com/filip26/titanium-json-ld) 库。

**为了反序列化一个 JSON-LD 文档，让我们用一个叫做压缩的 JSON-LD API 特性将它转换成一种我们可以用`ObjectMapper`映射到 POJO 的格式。**

### 6.1。Maven 依赖关系

首先，让我们为 [`jsonld-java`](https://web.archive.org/web/20220627091154/https://search.maven.org/artifact/com.github.jsonld-java/jsonld-java) 添加依赖关系:

```java
<dependency>
    <groupId>com.github.jsonld-java</groupId>
    <artifactId>jsonld-java</artifactId>
    <version>0.13.0</version>
</dependency>
```

### 6.2。示例

让我们使用这个 JSON-LD 文档作为我们的输入:

```java
{
  "@context": {
    "@vocab": "http://schema.org/",
    "knows": {
      "@type": "@id"
    }
  },
  "@type": "Person",
  "@id": "http://example.com/person/1234",
  "name": "Example Name",
  "knows": "http://example.com/person/2345"
}
```

为了简单起见，让我们假设我们在一个名为`inputJsonLd`的`String`变量中有文档的内容。

首先，让我们将其压缩并转换回一个`String`:

```java
Object jsonObject = JsonUtils.fromString(inputJsonLd);
Object compact = JsonLdProcessor.compact(jsonObject, new HashMap<>(), new JsonLdOptions());
String compactContent = JsonUtils.toString(compact);
```

*   我们可以用来自 Jsonld-Java 库的`JsonUtils,`的方法解析和编写 JSON-LD 对象
*   当使用`compact`方法时，作为第二个参数，我们可以使用一个空的`Map`。这样，压缩算法将产生一个简单的 JSON 对象，其中的键被解析为它们的 IRI 形式

**`compactContent`变量应该包含:**

```java
{
  "@id": "http://example.com/person/1234",
  "@type": "http://schema.org/Person",
  "http://schema.org/knows": {
    "@id": "http://example.com/person/2345"
  },
  "http://schema.org/name": "Example Name"
}
```

其次，让我们用 Jackson 注释来定制我们的 POJO，以适应这样的文档结构:

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class Person {
    @JsonProperty("@id")
    private String id;
    @JsonProperty("http://schema.org/name")
    private String name;
    @JsonProperty("http://schema.org/knows")
    private Link knows;

    // constructors, getters, setters

    public static class Link {
        @JsonProperty("@id")
        private String id;

        // constructors, getters, setters
    }
}
```

最后，让我们将 JSON-LD 映射到 POJO:

```java
ObjectMapper objectMapper = new ObjectMapper();
Person person = objectMapper.readValue(compactContent, Person.class);
```

## 7。结论

在本文中，我们研究了两个用于将 POJO 序列化为 JSON-LD 文档的基于 Jackson 的库，以及一种将 JSON-LD 反序列化为 POJO 的方法。

正如我们所强调的，这两个序列化库都有缺点，我们应该在使用它们之前考虑一下。如果我们需要使用比这些库所能提供的更多的 JSON-LD 特性，我们可以通过具有 JSON-LD 输出格式的 RDF 库来创建我们的文档。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627091154/https://github.com/eugenp/tutorials/tree/master/json-modules/json-2)