# 放心的 JSON 模式验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-assured-json-schema>

## 1.概观

放心库提供了对测试 REST APIs 的支持，通常采用 JSON 格式。

有时，在不详细分析响应的情况下，可能需要首先知道 JSON 主体是否符合特定的 JSON 格式。

在这个快速教程中，我们将看看**如何基于预定义的 JSON 模式**来验证 JSON 响应。

## 2。设置

最初的放心设置与我们上一篇文章中的[相同。](/web/20221222124608/https://www.baeldung.com/rest-assured-tutorial)

此外，我们还需要在`pom.xml`文件中包含`json-schema-validator`模块:

```java
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>3.3.0</version>
    <scope>test</scope>
</dependency>
```

为确保您拥有最新版本，请点击[此链接](https://web.archive.org/web/20221222124608/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22json-schema-validator%22)。

## 3。JSON 模式验证

让我们看一个例子。

作为 JSON 模式，我们将使用保存在名为`event_0.json`的文件中的 JSON，该文件位于类路径中:

```java
{
    "id": "390",
    "data": {
        "leagueId": 35,
        "homeTeam": "Norway",
        "visitingTeam": "England",
    },
    "odds": [{
        "price": "1.30",
        "name": "1"
    },
    {
        "price": "5.25",
        "name": "X"
    }]
}
```

然后假设这是 REST API 返回的所有数据遵循的通用格式，我们可以检查 JSON 响应的一致性，如下所示:

```java
@Test
public void givenUrl_whenJsonResponseConformsToSchema_thenCorrect() {
    get("/events?id=390").then().assertThat()
      .body(matchesJsonSchemaInClasspath("event_0.json"));
}
```

注意，我们仍将从`io.restassured.module.jsv.JsonSchemaValidator.`静态导入`matchesJsonSchemaInClasspath`

## 4。JSON 模式验证 **设置**

### 4.1。验证响应

放心的`json-schema-validator`模块通过定义我们自己的定制配置规则，赋予我们执行细粒度验证的能力。

假设我们希望我们的验证总是使用 JSON 模式版本 4:

```java
@Test
public void givenUrl_whenValidatesResponseWithInstanceSettings_thenCorrect() {
    JsonSchemaFactory jsonSchemaFactory = JsonSchemaFactory.newBuilder()
      .setValidationConfiguration(
        ValidationConfiguration.newBuilder()
          .setDefaultVersion(SchemaVersion.DRAFTV4).freeze())
            .freeze();
    get("/events?id=390").then().assertThat()
      .body(matchesJsonSchemaInClasspath("event_0.json")
        .using(jsonSchemaFactory));
}
```

我们将通过使用`JsonSchemaFactory`并指定版本 4 `SchemaVersion`来实现这一点，并在发出请求时断言它正在使用该模式。

### 4.2。检查验证

默认情况下，`json-schema-validator`对 JSON 响应字符串运行检查验证。这意味着如果模式将`odds`定义为如下 JSON 所示的数组:

```java
{
    "odds": [{
        "price": "1.30",
        "name": "1"
    },
    {
        "price": "5.25",
        "name": "X"
    }]
}
```

那么验证器将总是期待一个数组作为`odds`的值，因此当`odds`是一个`String`时，响应将无法通过验证。因此，如果我们希望对我们的响应不那么严格，我们可以在验证期间添加一个自定义规则，首先进行以下静态导入:

```java
io.restassured.module.jsv.JsonSchemaValidatorSettings.settings;
```

然后在验证检查设置为`false`的情况下执行测试:

```java
@Test
public void givenUrl_whenValidatesResponseWithStaticSettings_thenCorrect() {
    get("/events?id=390").then().assertThat().body(matchesJsonSchemaInClasspath
      ("event_0.json").using(settings().with().checkedValidation(false)));
}
```

### 4.3。全局验证配置

这些定制是非常灵活的，但是对于大量的测试，我们必须为每个测试定义一个验证，这是很麻烦的，并且不太容易维护。

为了避免这种情况，**我们可以自由地只定义一次我们的配置，并让它应用于所有的测试**。

我们将把验证配置为未检查的，并且总是在 JSON 模式版本 3:

```java
JsonSchemaFactory factory = JsonSchemaFactory.newBuilder()
  .setValidationConfiguration(
   ValidationConfiguration.newBuilder()
    .setDefaultVersion(SchemaVersion.DRAFTV3)
      .freeze()).freeze();
JsonSchemaValidator.settings = settings()
  .with().jsonSchemaFactory(factory)
      .and().with().checkedValidation(false);
```

然后调用 reset 方法来删除此配置:

```java
JsonSchemaValidator.reset();
```

## 5。结论

在本文中，我们展示了如何在使用 REST-assured 时根据模式验证 JSON 响应。

与往常一样，该示例的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221222124608/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-assured)