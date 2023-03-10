# 放心的标题、Cookies 和参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-assured-header-cookie-parameter>

## 1。概述

在这个快速教程中，我们将探索一些放心的高级场景。我们之前在教程[放心指南](/web/20220721122549/https://www.baeldung.com/rest-assured-tutorial)中探讨过放心。

接下来，**我们将介绍一些例子，展示如何为我们的请求设置头、cookie 和参数。**

设置与前一篇文章相同，所以让我们深入研究我们的示例。

## 2。设置参数

现在，让我们讨论如何为我们的请求指定不同的参数——从路径参数开始。

### 2.1。路径参数

我们可以使用`pathParam(parameter-name, value)`来指定一个路径参数:

```java
@Test
public void whenUsePathParam_thenOK() {
    given().pathParam("user", "eugenp")
      .when().get("/users/{user}/repos")
      .then().statusCode(200);
}
```

要添加多个路径参数，我们将使用`pathParams()`方法:

```java
@Test
public void whenUseMultiplePathParam_thenOK() {
    given().pathParams("owner", "eugenp", "repo", "tutorials")
      .when().get("/repos/{owner}/{repo}")
      .then().statusCode(200);

    given().pathParams("owner", "eugenp")
      .when().get("/repos/{owner}/{repo}","tutorials")
      .then().statusCode(200);
}
```

在本例中，我们使用了命名的路径参数，但是我们也可以添加未命名的参数，甚至将两者结合起来:

```java
given().pathParams("owner", "eugenp")
  .when().get("/repos/{owner}/{repo}", "tutorials")
  .then().statusCode(200);
```

在这种情况下，产生的 URL 是`https://api.github.com/repos/eugenp/tutorials.`

请注意，未命名的参数是基于索引的。

### 2.2。查询参数

接下来，让我们看看如何使用`queryParam():`指定查询参数

```java
@Test
public void whenUseQueryParam_thenOK() {
    given().queryParam("q", "john").when().get("/search/users")
      .then().statusCode(200);

    given().param("q", "john").when().get("/search/users")
      .then().statusCode(200);
}
```

**`param()`方法将像`queryParam()`一样处理 GET 请求。**

为了添加多个查询参数，我们可以将几个`queryParam()`方法链接起来，或者将参数添加到一个`queryParams()`方法中:

```java
@Test
public void whenUseMultipleQueryParam_thenOK() {

    int perPage = 20;
    given().queryParam("q", "john").queryParam("per_page",perPage)
      .when().get("/search/users")
      .then().body("items.size()", is(perPage));   

    given().queryParams("q", "john","per_page",perPage)
      .when().get("/search/users")
      .then().body("items.size()", is(perPage));
}
```

### 2.3。表单参数

最后，我们可以使用`formParam():`指定表单参数

```java
@Test
public void whenUseFormParam_thenSuccess() {

    given().formParams("username", "john","password","1234").post("/");

    given().params("username", "john","password","1234").post("/");
}
```

**`param()`方法将为 POST 请求执行生命周期`formParam()`。**

还要注意的是，`formParam()`添加了一个值为“`application/x-www-form-urlencoded`”的`Content-Type`报头。

## 3。设置标题

接下来，**我们可以使用`header():`** 定制我们的请求头

```java
@Test
public void whenUseCustomHeader_thenOK() {

    given().header("User-Agent", "MyAppName").when().get("/users/eugenp")
      .then().statusCode(200);
}
```

在这个例子中，我们使用了`header()`来设置`User-Agent`头。

我们还可以使用相同的方法添加具有多个值的标题:

```java
@Test
public void whenUseMultipleHeaderValues_thenOK() {

    given().header("My-Header", "val1", "val2")
      .when().get("/users/eugenp")
      .then().statusCode(200);
}
```

在这个例子中，我们有一个带有两个头的请求:`My-Header:val1`和`My-Header:val2.`

**对于添加多个表头，我们将使用`headers()`方法:**

```java
@Test
public void whenUseMultipleHeaders_thenOK() {

    given().headers("User-Agent", "MyAppName", "Accept-Charset", "utf-8")
      .when().get("/users/eugenp")
      .then().statusCode(200);
}
```

## 4。添加 cookie

我们还可以使用`cookie()`为我们的请求指定定制 cookie:

```java
@Test
public void whenUseCookie_thenOK() {

    given().cookie("session_id", "1234").when().get("/users/eugenp")
      .then().statusCode(200);
}
```

我们还可以使用 cookie `Builder`定制我们的 cookie:

```java
@Test
public void whenUseCookieBuilder_thenOK() {
    Cookie myCookie = new Cookie.Builder("session_id", "1234")
      .setSecured(true)
      .setComment("session id cookie")
      .build();

    given().cookie(myCookie)
      .when().get("/users/eugenp")
      .then().statusCode(200);
}
```

## 5。结论

在本文中，我们展示了如何在使用放心时指定请求参数、头和 cookies。

和往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220721122549/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-assured)