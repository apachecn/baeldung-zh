# 放心认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-assured-authentication>

## 1.概观

在本教程中，我们将分析如何使用[放心](/web/20220630012905/https://www.baeldung.com/rest-assured-tutorial)进行认证，以正确测试和验证安全的 API。

**该工具支持多种认证方案**:

*   基本认证
*   摘要认证
*   表单认证
*   OAuth 1 和 OAuth 2

我们会看到每个例子。

## 2.使用基本身份验证

[基本认证方案](https://web.archive.org/web/20220630012905/https://tools.ietf.org/html/rfc7617)要求消费者发送在`Base64`中编码的用户 id 和密码。

放心提供了一种简单的方法来配置请求所需的凭证:

```
given().auth()
  .basic("user1", "user1Pass")
  .when()
  .get("http://localhost:8080/spring-security-rest-basic-auth/api/foos/1")
  .then()
  .assertThat()
  .statusCode(HttpStatus.OK.value());
```

### 2.1.抢先认证

正如我们在[上看到的关于 Spring 安全认证](/web/20220630012905/https://www.baeldung.com/spring-security-basic-authentication#usage)的前一篇文章，服务器可能会使用[一种挑战-响应机制](https://web.archive.org/web/20220630012905/https://tools.ietf.org/html/rfc2617#section-1.2)来明确指示消费者何时需要认证才能访问资源。

**默认情况下，放心会在发送凭证之前等待服务器质询。**

这在某些情况下可能会很麻烦，例如，服务器被配置为检索登录表单而不是质询响应。

出于这个原因，库提供了我们可以使用的`preemptive `指令:

```
given().auth()
  .preemptive()
  .basic("user1", "user1Pass")
  .when()
  // ...
```

准备就绪后，放心将发送凭证，而无需等待`Unauthorized`响应。

我们几乎从未对测试服务器的挑战能力感兴趣。因此，我们通常可以添加这个命令，以避免额外请求的复杂性和开销。

## 3.使用摘要式身份验证

尽管这也被认为是一种[“弱”认证方法](https://web.archive.org/web/20220630012905/https://tools.ietf.org/html/rfc2617#section-4.4)，但使用[摘要认证](https://web.archive.org/web/20220630012905/https://tools.ietf.org/html/rfc7616)比基本协议更有优势。

这是因为该方案避免了以明文形式发送密码。

**尽管存在这种差异，但放心地实现这种形式的身份验证与我们在上一节中遵循的非常相似:**

```
given().auth()
  .digest("user1", "user1Pass")
  .when()
  // ...
```

注意，目前这个库只支持这个方案的挑战认证，**,所以我们不能像前面那样使用`preemptive()` 。**

## 4.使用表单身份验证

许多服务为用户提供了一个 HTML 表单，用户可以通过填写域中的凭证来进行身份验证。

当用户提交表单时，浏览器执行带有信息的 POST 请求。

通常，表单用它的`action`属性指示它将调用的端点，每个`input` 字段对应于请求中发送的一个表单参数。

如果登录表单足够简单并遵循这些规则，那么我们可以放心地为我们计算出这些值:

```
given().auth()
  .form("user1", "user1Pass")
  .when()
  // ...
```

无论如何，这不是一个最佳的方法，因为 REST Assured 需要执行一个额外的请求并解析 HTML 响应来找到字段。

我们还必须记住，这个过程仍然可能失败，例如，如果网页很复杂，或者如果服务配置了不包含在`action`属性中的上下文路径。

**因此，更好的解决方案是我们自己提供配置，明确指出三个必填字段:**

```
given().auth()
  .form(
    "user1",
    "user1Pass",
    new FormAuthConfig("/perform_login", "username", "password"))
  // ... 
```

除了这些基本配置之外，还附带了以下功能:

*   检测或指示网页中的 CSRF 令牌字段
*   在请求中使用附加的表单字段
*   记录有关身份验证过程的信息

## 5.OAuth 支持

OAuth 在技术上是一个`authorization`框架，它没有定义任何认证用户的机制。

尽管如此，它仍然可以用作构建认证和身份协议的基础，就像 [OpenID Connect](/web/20220630012905/https://www.baeldung.com/spring-security-openid-connect) 的情况一样。

### 5.1.OAuth 2.0

**放心允许配置 OAuth 2.0 访问令牌来请求安全资源:**

```
given().auth()
  .oauth2(accessToken)
  .when()
  .// ...
```

这个库在获取访问令牌方面没有提供任何帮助，所以我们必须自己解决这个问题。

对于客户端凭证和密码流来说，这是一个简单的任务，因为只需出示相应的凭证就可以获得令牌。

另一方面，自动化授权代码流可能不那么容易，我们可能还需要其他工具的帮助。

为了正确理解这个流程以及如何获得一个访问令牌，我们可以看看这个关于这个主题的帖子。

### 5.2.OAuth 1.0a

在 OAuth 1.0a 的情况下，**放心提供了一种接收消费者密钥、秘密、访问令牌和令牌秘密**以访问安全资源的方法:

```
given().accept(ContentType.JSON)
  .auth()
  .oauth(consumerKey, consumerSecret, accessToken, tokenSecret)
  // ...
```

这个协议需要用户输入，因此获取最后两个字段不是一件简单的事情。

注意，如果我们在 2.5.0 之前的版本中使用 OAuth 2.0 特性，或者如果我们使用 OAuth 1.0a 功能，我们将需要在我们的项目中添加`scribejava-apis`依赖项。

## 6.结论

在本教程中，我们学习了如何使用放心认证来访问安全的 API。

该库简化了我们实现的几乎所有方案的认证过程。

和往常一样，我们可以在 Github repo 上找到带有说明的工作示例。