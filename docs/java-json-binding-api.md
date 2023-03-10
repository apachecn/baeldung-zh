# Java 中的 JSON 绑定 API (JSR 367)简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-json-binding-api>

## 1。概述

很长一段时间，Java 中没有 JSON 处理的标准。JSON 处理最常用的库是 Jackson 和 Gson。

最近，Java EE7 附带了一个用于解析和生成 JSON 的 API([JSR 353:用于 JSON 处理的 Java API](https://web.archive.org/web/20221208143832/https://www.jcp.org/en/jsr/detail?id=353))。

最后，随着 JEE 8 的发布，有了一个标准化的 API ( [JSR 367:用于 JSON 绑定的 Java API(JSON-B)](https://web.archive.org/web/20221208143832/https://jcp.org/en/jsr/detail?id=367))。

目前，它的主要实现是 [Eclipse Yasson (RI)](https://web.archive.org/web/20221208143832/https://github.com/eclipse/yasson) 和 [Apache Johnzon](https://web.archive.org/web/20221208143832/https://johnzon.apache.org/) 。

## 2。JSON-B API

### 2.1。Maven 依赖关系

让我们从添加必要的依赖项开始。

请记住，在许多情况下，包含所选实现的依赖关系就足够了，并且`javax.json.bind-api`将被过渡性地包含:

```java
<dependency>
    <groupId>javax.json.bind</groupId>
    <artifactId>javax.json.bind-api</artifactId>
    <version>1.0</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Cjavax.json.bind-api) 找到。

## 3。使用 Eclipse Yasson

**Eclipse Yasson 是 JSON 绑定 API 的官方参考实现**([JSR-367](https://web.archive.org/web/20221208143832/https://jcp.org/en/jsr/detail?id=367))。

### 3.1。Maven 依赖关系

要使用它，我们需要在我们的 Maven 项目中包含以下依赖项:

```java
<dependency>
    <groupId>org.eclipse</groupId>
    <artifactId>yasson</artifactId>
    <version>1.0.1</version>
</dependency>
<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>javax.json</artifactId>
    <version>1.1.2</version>
</dependency>
```

最新版本可以在 [Maven Central 找到。](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22yasson%22)

## 4。使用 Apache Johnzon

我们可以使用的另一个实现是 Apache Johnzon，它符合 JSON-P (JSR-353)和 JSON-B(JSR-367)API。

### 4.1。Maven 依赖关系

要使用它，我们需要在我们的 Maven 项目中包含以下依赖项:

```java
<dependency>
    <groupId>org.apache.geronimo.specs</groupId>
    <artifactId>geronimo-json_1.1_spec</artifactId>
    <version>1.0</version>
</dependency>
<dependency>
    <groupId>org.apache.johnzon</groupId>
    <artifactId>johnzon-jsonb</artifactId>
    <version>1.1.4</version>
</dependency>
```

最新版本可以在 [Maven Central 找到。](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22johnzon-jsonb%22)

## 5。API 特性

API 为定制序列化/反序列化提供了注释。

让我们创建一个简单的类，看看示例配置是什么样子的:

```java
public class Person {

    private int id;

    @JsonbProperty("person-name")
    private String name;

    @JsonbProperty(nillable = true)
    private String email;

    @JsonbTransient
    private int age;

    @JsonbDateFormat("dd-MM-yyyy")
    private LocalDate registeredDate;

    private BigDecimal salary;

    @JsonbNumberFormat(locale = "en_US", value = "#0.0")
    public BigDecimal getSalary() {
        return salary;
    }

    // standard getters and setters
}
```

序列化后，该类的对象将类似于:

```java
{
   "email":"[[email protected]](/web/20221208143832/https://www.baeldung.com/cdn-cgi/l/email-protection)",
   "id":1,
   "person-name":"Jhon",
   "registeredDate":"07-09-2019",
   "salary":"1000.0"
}
```

这里使用的注释是:

*   `@JsonbProperty`–用于指定自定义字段名称
*   `@JsonbTransient`–当我们想在反序列化/序列化过程中忽略该字段时
*   `@JsonbDateFormat`–当我们想要定义日期的显示格式时
*   `@JsonbNumberFormat`–用于指定数值的显示格式
*   `@JsonbNillable`–用于启用空值的序列化

### 5.1。序列化和反序列化

首先，要获得我们对象的 JSON 表示，我们需要使用`JsonbBuilder`类及其`toJson()`方法。

首先，让我们创建一个简单的`Person`对象，如下所示:

```java
Person person = new Person(
  1, 
  "Jhon", 
  "[[email protected]](/web/20221208143832/https://www.baeldung.com/cdn-cgi/l/email-protection)", 
  20, 
  LocalDate.of(2019, 9, 7), 
  BigDecimal.valueOf(1000));
```

并且，实例化`Jsonb`类:

```java
Jsonb jsonb = JsonbBuilder.create();
```

然后，我们使用`toJson`方法:

```java
String jsonPerson = jsonb.toJson(person);
```

要获得以下 JSON 表示:

```java
{
    "email":"[[email protected]](/web/20221208143832/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "id":1,
    "person-name":"Jhon",
    "registeredDate":"07-09-2019",
    "salary":"1000.0"
}
```

如果我们想以另一种方式进行转换，我们可以使用`fromJson`方法:

```java
Person person = jsonb.fromJson(jsonPerson, Person.class);
```

自然，我们也可以处理集合:

```java
List<Person> personList = Arrays.asList(...);
String jsonArrayPerson = jsonb.toJson(personList);
```

要获得以下 JSON 表示:

```java
[ 
    {
      "email":"[[email protected]](/web/20221208143832/https://www.baeldung.com/cdn-cgi/l/email-protection)",
      "id":1,
      "person-name":"Jhon", 
      "registeredDate":"09-09-2019",
      "salary":"1000.0"
    },
    {
      "email":"[[email protected]](/web/20221208143832/https://www.baeldung.com/cdn-cgi/l/email-protection)",
      "id":2,
      "person-name":"Jhon",
      "registeredDate":"09-09-2019",
      "salary":"1500.0"
    },
    ...
]
```

为了从 JSON 数组转换到`List`，我们将使用`fromJson` API:

```java
List<Person> personList = jsonb.fromJson(
  personJsonArray, 
  new ArrayList<Person>(){}.getClass().getGenericSuperclass()
);
```

### 5.2。用`JsonbConfig` 自定义贴图

`JsonbConfig`类允许我们为所有类定制映射过程。

例如，我们可以更改默认的命名策略或属性顺序。

现在，我们将使用`LOWER_CASE_WITH_UNDERSCORES`策略:

```java
JsonbConfig config = new JsonbConfig().withPropertyNamingStrategy(
  PropertyNamingStrategy.LOWER_CASE_WITH_UNDERSCORES);
Jsonb jsonb = JsonbBuilder.create(config);
String jsonPerson = jsonb.toJson(person);
```

要获得以下 JSON 表示:

```java
{
   "email":"[[email protected]](/web/20221208143832/https://www.baeldung.com/cdn-cgi/l/email-protection)",
   "id":1,
   "person-name":"Jhon",
   "registered_date":"07-09-2019",
   "salary":"1000.0"
}
```

现在，我们将使用`REVERSE`策略改变属性顺序。使用这种策略，属性的顺序与字典顺序相反。
这也可以在编译时用注释`@JsonbPropertyOrder.`进行配置，让我们来看看它的运行情况:

```java
JsonbConfig config 
  = new JsonbConfig().withPropertyOrderStrategy(PropertyOrderStrategy.REVERSE);
Jsonb jsonb = JsonbBuilder.create(config);
String jsonPerson = jsonb.toJson(person); 
```

要获得以下 JSON 表示:

```java
{
    "salary":"1000.0",
    "registeredDate":"07-09-2019",
    "person-name":"Jhon",
    "id":1,
    "email":"[[email protected]](/web/20221208143832/https://www.baeldung.com/cdn-cgi/l/email-protection)"
}
```

### 5.3。带适配器的自定义映射

当注释和`JsonbConfig`类对我们来说不够时，我们可以使用适配器。

要使用它们，我们需要实现`JsonbAdapter`接口，它定义了以下方法:

*   `adaptToJson`–通过这种方法，我们可以为序列化过程使用定制的转换逻辑。
*   `adaptFromJson`–该方法允许我们为反序列化过程使用自定义转换逻辑。

让我们创建一个`PersonAdapter`来处理`Person`类的`id`和`name`属性:

```java
public class PersonAdapter implements JsonbAdapter<Person, JsonObject> {

    @Override
    public JsonObject adaptToJson(Person p) throws Exception {
        return Json.createObjectBuilder()
          .add("id", p.getId())
          .add("name", p.getName())
          .build();
    }

    @Override
    public Person adaptFromJson(JsonObject adapted) throws Exception {
        Person person = new Person();
        person.setId(adapted.getInt("id"));
        person.setName(adapted.getString("name"));
        return person;
    }
}
```

此外，我们将把适配器分配给我们的`JsonbConfig`实例:

```java
JsonbConfig config = new JsonbConfig().withAdapters(new PersonAdapter());
Jsonb jsonb = JsonbBuilder.create(config);
```

我们会得到下面的 JSON 表示:

```java
{
    "id":1, 
    "name":"Jhon"
}
```

## 6。结论

在本教程中，我们看到了如何使用可用的实现将 JSON-B API 与 Java 应用程序集成的示例，以及在编译和运行时定制序列化和反序列化的示例。

完整的代码一如既往地在 Github 上[可用。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/json-modules/json)