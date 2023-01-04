# 放心指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-assured-tutorial>

## 1。简介

REST-assured 旨在简化 REST APIs 的测试和验证，并且深受 Ruby 和 Groovy 等动态语言中使用的测试技术的影响。

该库对 HTTP 有坚实的支持，当然从动词和标准 HTTP 操作开始，但也远远超出了这些基础。

在本指南中，我们将**探索放心**，并使用 Hamcrest 进行断言。如果你还不熟悉 Hamcrest，你应该先温习一下教程:[用 Hamcrest](/web/20220625174625/https://www.baeldung.com/java-junit-hamcrest-guide) 测试。

此外，要了解更高级的放心使用案例，请查看我们的其他文章:

*   [放心使用 Groovy](/web/20220625174625/https://www.baeldung.com/rest-assured-groovy)
*   [放心的 JSON 模式验证](/web/20220625174625/https://www.baeldung.com/rest-assured-json-schema)
*   [放心的参数、标题和 Cookies】](/web/20220625174625/https://www.baeldung.com/rest-assured-header-cookie-parameter)

现在让我们深入一个简单的例子。

## 2。简单示例测试

在我们开始之前，让我们确保我们的测试有以下静态导入:

```
io.restassured.RestAssured.*
io.restassured.matcher.RestAssuredMatchers.*
org.hamcrest.Matchers.*
```

我们需要它来保持测试的简单性，并且容易访问主要的 API。

现在，让我们从一个简单的例子开始——一个公开游戏数据的基本下注系统:

```
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

假设这是点击本地部署的 API 得到的 JSON 响应—`http://localhost:8080/events?id=390\.` :

现在让我们使用放心来验证 response JSON 的一些有趣特性:

```
@Test
public void givenUrl_whenSuccessOnGetsResponseAndJsonHasRequiredKV_thenCorrect() {
   get("/events?id=390").then().statusCode(200).assertThat()
      .body("data.leagueId", equalTo(35)); 
}
```

因此，我们在这里所做的是——我们验证了对端点`/events?id=390`的调用用包含一个`JSON String`的主体来响应，该主体的`data`对象的`leagueId` 是 35。

让我们看一个更有趣的例子。假设您想要验证`odds`数组中有价格为`1.30`和`5.25`的记录:

```
@Test
public void givenUrl_whenJsonResponseHasArrayWithGivenValuesUnderKey_thenCorrect() {
   get("/events?id=390").then().assertThat()
      .body("odds.price", hasItems("1.30", "5.25"));
}
```

## 3。放心设置

如果您最喜欢的依赖工具是 Maven，我们在`pom.xml`文件中添加以下依赖:

```
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>3.3.0</version>
    <scope>test</scope>
</dependency>
```

要获得最新版本，请点击此链接。
REST-assured 利用 Hamcrest matchers 的能力来执行其断言，因此我们也必须包括该依赖性:

```
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-all</artifactId>
    <version>2.1</version>
</dependency>
```

最新版本将始终可在[此链接](https://web.archive.org/web/20220625174625/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22hamcrest-all%22)获得。

## 4。匿名 JSON 根验证

考虑一个由基元而不是对象组成的数组:

```
[1, 2, 3]
```

这被称为匿名 JSON 根，意味着它没有键值对，但是它仍然是有效的 JSON 数据。

在这种情况下，我们可以使用``$`` 符号或空字符串( "" )作为路径来运行验证。假设我们通过`http://localhost:8080/json`公开上述服务，那么我们可以放心地像这样验证它:

```
when().get("/json").then().body("$", hasItems(1, 2, 3));
```

或者像这样:

```
when().get("/json").then().body("", hasItems(1, 2, 3));
```

## 5。浮点型和双精度型

当我们开始使用 REST-assured 来测试我们的 REST 服务时，我们需要理解 JSON 响应中的浮点数被映射到原始类型`float.`

与 java 中的许多场景不同，`float`类型的使用不能与`double` 互换。

恰当的例子是这种反应:

```
{
    "odd": {
        "price": "1.30",
        "ck": 12.2,
        "name": "1"
    }
}
```

假设我们正在对`ck`的值运行以下测试:

```
get("/odd").then().assertThat().body("odd.ck", equalTo(12.2));
```

即使我们测试的值等于响应中的值，这个测试也会失败。这是因为我们正在与一个`double`而不是一个`float`进行比较。

为了让它工作，我们必须显式地将`equalTo` matcher 方法的操作数指定为`float`，就像这样:

```
get("/odd").then().assertThat().body("odd.ck", equalTo(12.2f));
```

## 6。指定请求方法

通常，我们会通过调用与我们想要使用的请求方法相对应的方法(如`get(),`)来执行请求。

此外，**我们还可以使用`request()`方法**来指定 HTTP 动词:

```
@Test
public void whenRequestGet_thenOK(){
    when().request("GET", "/users/eugenp").then().statusCode(200);
}
```

上面的例子相当于直接用`get()`。

同样，我们可以发送`HEAD`、`CONNECT`和`OPTIONS`请求:

```
@Test
public void whenRequestHead_thenOK() {
    when().request("HEAD", "/users/eugenp").then().statusCode(200);
}
```

**`POST`请求也遵循类似的语法，我们可以通过使用`with() `和`body()`方法来指定`the body `。**

因此，通过发送一个`POST `请求来创建一个新的`Odd `:

```
@Test
public void whenRequestedPost_thenCreated() {
    with().body(new Odd(5.25f, 1, 13.1f, "X"))
      .when()
      .request("POST", "/odds/new")
      .then()
      .statusCode(201);
}
```

作为`body `发送的`Odd `对象将自动转换成 JSON。我们也可以将任何我们想发送的`String`作为我们的`POST` `body.`

## 7。默认值配置

我们可以为测试配置许多默认值:

```
@Before
public void setup() {
    RestAssured.baseURI = "https://api.github.com";
    RestAssured.port = 443;
}
```

这里，我们为我们的请求设置了一个基本的 URI 和端口。除此之外，我们还可以配置基本路径、根 pat 和身份验证。

注意:我们还可以使用以下命令重置为标准的放心默认值:

```
RestAssured.reset();
```

## 8。测量响应时间

让我们看看如何使用`Response`对象的`time()`和`timeIn()`方法**来测量响应时间:**

```
@Test
public void whenMeasureResponseTime_thenOK() {
    Response response = RestAssured.get("/users/eugenp");
    long timeInMS = response.time();
    long timeInS = response.timeIn(TimeUnit.SECONDS);

    assertEquals(timeInS, timeInMS/1000);
}
```

请注意:

*   `time()`用于获得以毫秒为单位的响应时间
*   `timeIn()`用于获取指定时间单位的响应时间

### 8.1。验证响应时间

我们还可以借助简单的`long` `Matcher:`来验证响应时间(毫秒)

```
@Test
public void whenValidateResponseTime_thenSuccess() {
    when().get("/users/eugenp").then().time(lessThan(5000L));
}
```

如果我们想用不同的时间单位来验证响应时间，那么我们将使用带有第二个`TimeUnit`参数的`time()`匹配器:

```
@Test
public void whenValidateResponseTimeInSeconds_thenSuccess(){
    when().get("/users/eugenp").then().time(lessThan(5L),TimeUnit.SECONDS);
}
```

## 9。XML 响应验证

它不仅可以验证 JSON 响应，还可以验证 XML。

假设我们向`http://localhost:8080/employees`发出一个请求，得到如下响应:

```
<employees>
    <employee category="skilled">
        <first-name>Jane</first-name>
        <last-name>Daisy</last-name>
        <sex>f</sex>
    </employee>
</employees>
```

我们可以这样验证`first-name`是`Jane` :

```
@Test
public void givenUrl_whenXmlResponseValueTestsEqual_thenCorrect() {
    post("/employees").then().assertThat()
      .body("employees.employee.first-name", equalTo("Jane"));
}
```

我们还可以通过如下方式将正文匹配器链接在一起，来验证所有值是否与预期值匹配:

```
@Test
public void givenUrl_whenMultipleXmlValuesTestEqual_thenCorrect() {
    post("/employees").then().assertThat()
      .body("employees.employee.first-name", equalTo("Jane"))
        .body("employees.employee.last-name", equalTo("Daisy"))
          .body("employees.employee.sex", equalTo("f"));
}
```

或者使用带有可变参数的简写版本:

```
@Test
public void givenUrl_whenMultipleXmlValuesTestEqualInShortHand_thenCorrect() {
    post("/employees")
      .then().assertThat().body("employees.employee.first-name", 
        equalTo("Jane"),"employees.employee.last-name", 
          equalTo("Daisy"), "employees.employee.sex", 
            equalTo("f"));
}
```

## 10。XML 的 XPath】

**我们还可以使用 XPath 来验证我们的响应。**考虑下面这个在`first-name`上执行匹配器的例子:

```
@Test
public void givenUrl_whenValidatesXmlUsingXpath_thenCorrect() {
    post("/employees").then().assertThat().
      body(hasXPath("/employees/employee/first-name", containsString("Ja")));
}
```

XPath 还接受另一种运行`equalTo`匹配器的方式:

```
@Test
public void givenUrl_whenValidatesXmlUsingXpath2_thenCorrect() {
    post("/employees").then().assertThat()
      .body(hasXPath("/employees/employee/first-name[text()='Jane']"));
}
```

## 11.记录测试详细信息

### 11.1。日志请求详细信息

首先，让我们看看如何使用`**log().all()**:`**记录整个请求细节**

```
@Test
public void whenLogRequest_thenOK() {
    given().log().all()
      .when().get("/users/eugenp")
      .then().statusCode(200);
}
```

这将记录类似这样的内容:

```
Request method:	GET
Request URI:	https://api.github.com:443/users/eugenp
Proxy:			<none>
Request params:	<none>
Query params:	<none>
Form params:	<none>
Path params:	<none>
Multiparts:		<none>
Headers:		Accept=*/*
Cookies:		<none>
Body:			<none>
```

为了只记录请求的特定部分，我们结合使用了`log()`方法和`params(), body(), headers(), cookies(), method(), path()`方法，例如`log.().params().`

**注意，使用的其他库或过滤器可能会改变实际发送到服务器的内容，所以这应该只用于记录初始请求规范。**

### 11.2。日志响应详细信息

类似地，我们可以记录响应细节。

在下面的示例中，我们只记录响应正文:

```
@Test
public void whenLogResponse_thenOK() {
    when().get("/repos/eugenp/tutorials")
      .then().log().body().statusCode(200);
}
```

样本输出:

```
{
    "id": 9754983,
    "name": "tutorials",
    "full_name": "eugenp/tutorials",
    "private": false,
    "html_url": "https://github.com/eugenp/tutorials",
    "description": "The \"REST With Spring\" Course: ",
    "fork": false,
    "size": 72371,
    "license": {
        "key": "mit",
        "name": "MIT License",
        "spdx_id": "MIT",
        "url": "https://api.github.com/licenses/mit"
    },
...
}
```

### 11.3。条件发生时记录响应

我们还可以选择仅在发生错误或状态代码与给定值匹配时记录响应:

```
@Test
public void whenLogResponseIfErrorOccurred_thenSuccess() {

    when().get("/users/eugenp")
      .then().log().ifError();
    when().get("/users/eugenp")
      .then().log().ifStatusCodeIsEqualTo(500);
    when().get("/users/eugenp")
      .then().log().ifStatusCodeMatches(greaterThan(200));
}
```

### 11.4。验证失败时记录日志

只有在验证失败时，我们才能记录请求和响应:

```
@Test
public void whenLogOnlyIfValidationFailed_thenSuccess() {
    when().get("/users/eugenp")
      .then().log().ifValidationFails().statusCode(200);

    given().log().ifValidationFails()
      .when().get("/users/eugenp")
      .then().statusCode(200);
}
```

在本例中，我们希望验证状态代码是 200。只有当失败时，请求和响应才会被记录。

## 12。结论

在本教程中，我们已经**探索了放心框架**并查看了它最重要的特性，我们可以用它来测试我们的 RESTful 服务并验证它们的响应。

所有这些例子和代码片段的完整实现可以在放心的 GitHub 项目中找到。