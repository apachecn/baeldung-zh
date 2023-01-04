# 使用 OPA 的 Spring 安全授权

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-authorization-opa>

 ![announcement - icon](img/1c542d6dc3511393973106275abefa67.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220525034250/https://www.baeldung.com/lightrun-n-security)

## 1.介绍

在本教程中，我们将展示如何将 Spring Security 的授权决策外化到 OPA——[开放策略代理](https://web.archive.org/web/20220525034250/https://www.openpolicyagent.org/)。

## 2.前言:外部化授权的案例

**跨应用程序的一个常见需求是能够基于策略做出某些决策**。当这个策略足够简单并且不太可能改变时，我们可以直接在代码中实现这个策略，这是最常见的场景。

然而，在其他情况下，我们需要更多的灵活性。访问控制决策是典型的:随着应用程序复杂性的增加，授予对给定功能的访问权限可能不仅取决于您是谁，还取决于请求的其他上下文方面。这些方面可能包括 IP 地址、时间和登录身份验证方法(例如:“记住我”，OTP)等等。

此外，将上下文信息与用户身份结合起来的规则应该易于更改，最好不要让应用程序停机。这一需求自然导致了一种由专用服务处理策略评估请求的架构。

[![](img/8085e65b57c99eecca8497ca5012d147.png)](/web/20220525034250/https://www.baeldung.com/wp-content/uploads/2022/05/bael-5584-spring-opa-Page-1.png)

这里，这种灵活性的代价是调用外部服务会增加复杂性和性能损失。另一方面，我们可以发展甚至完全取代授权服务，而不会影响应用程序。此外，我们可以与多个应用程序共享该服务，从而在它们之间实现一致的授权模型。

## 3.OPA 是什么？

**开放策略代理，简称 OPA，是在 Go** 中实现的开源策略评估引擎。它最初是由 [Styra](https://web.archive.org/web/20220525034250/https://www.styra.com/) 开发的，现在是一个 CNCF 毕业的项目。以下是该工具的一些典型用法:

*   特使授权过滤器
*   立方输入控制器
*   地形规划评估

安装 OPA 非常简单:只需下载我们平台的二进制文件，把它放在操作系统路径下的一个文件夹中，我们就可以开始了。我们可以用一个简单的命令来验证它是否安装正确:

```java
$ opa version
Version: 0.39.0
Build Commit: cc965f6
Build Timestamp: 2022-03-31T12:34:56Z
Build Hostname: 5aba1d393f31
Go Version: go1.18
Platform: windows/amd64
WebAssembly: available
```

OPA 评估用 [REGO](https://web.archive.org/web/20220525034250/https://www.openpolicyagent.org/docs/latest/policy-language/) 编写的策略，REGO 是一种声明式语言，经过优化可以在复杂的对象结构上运行查询。然后，客户端应用程序根据特定的用例使用这些查询的结果。在我们的例子中，对象结构是一个授权请求，我们将使用策略来查询结果，以授予对给定功能的访问权限。

**需要注意的是，OPA 的政策是通用的，不以任何方式与表达授权决策相关联**。事实上，我们可以在传统上由 Drools 等规则引擎主导的其他场景中使用它。

## 4.撰写政策

用 REGO 编写的简单授权策略如下所示:

```java
package baeldung.auth.account

# Not authorized by default
default authorized = false

authorized = true {
    count(deny) == 0
    count(allow) > 0
}

# Allow access to /public
allow["public"] {
    regex.match("^/public/.*",input.uri)
}

# Account API requires authenticated user
deny["account_api_authenticated"] {
    regex.match("^/account/.*",input.uri)
    regex.match("ANONYMOUS",input.principal)
}

# Authorize access to account
allow["account_api_authorized"] {
    regex.match("^/account/.+",input.uri)
    parts := split(input.uri,"/")
    account := parts[2]
    role := concat(":",[ "ROLE_account", "read", account] )
    role == input.authorities[i]
} 
```

首先要注意的是包语句。OPA 策略使用包来组织规则，它们在评估传入请求时也起着关键作用，我们将在后面展示。我们可以跨多个目录组织策略文件。

接下来，我们定义实际的策略规则:

*   一个`default`规则，确保我们总是以一个`authorized`变量的值结束
*   当没有拒绝访问的规则而至少有一个允许访问的规则时，我们可以将主聚合器规则读作“`authorized`是`true`
*   允许和拒绝规则，每个规则表示一个条件，如果匹配，将分别向`allow`或`deny`数组添加一个条目

对 OPA 策略语言的完整描述超出了本文的范围，但是规则本身并不难理解。查看它们时，有几件事需要记住:

*   形式为`a := b`或`a=b`的语句是简单的赋值语句([虽然](https://web.archive.org/web/20220525034250/https://www.openpolicyagent.org/docs/latest/faq/#which-equality-operator-should-i-use)它们并不相同)
*   形式为`a = b { … conditions }`或`a { …conditions }`的语句表示“如果`conditions`为真，则将`b`分配给`a`
*   订单在策略文档中的出现是不相关的

除此之外，OPA 还提供了一个丰富的内置函数库，该函数库针对查询深度嵌套的数据结构进行了优化，此外还有一些更熟悉的特性，比如字符串操作、集合等等。

## 5.评估政策

让我们使用上一节中定义的策略来评估授权请求。在我们的例子中，我们将使用一个 JSON 结构构建这个授权请求，该结构包含来自传入请求的一些片段:

```java
{
    "input": {
        "principal": "user1",
        "authorities": ["ROLE_account:read:0001"],
        "uri": "/account/0001",
        "headers": {
            "WebTestClient-Request-Id": "1",
            "Accept": "application/json"
        }
    }
} 
```

注意，我们已经将请求属性包装在一个单独的`input`对象中。在策略评估期间，这个对象变成了`input`变量，我们可以使用类似 JavaScript 的语法访问它的属性。

为了测试我们的策略是否按预期工作，让我们在服务器模式下本地运行 OPA，并手动提交一些测试请求:

```java
$ opa run  -w -s src/test/rego
```

选项`-s`支持在服务器模式下运行，而`-w`支持自动重新加载规则文件。`src/test/rego`是包含样本代码中策略文件的文件夹。一旦运行，OPA 将在本地端口 8181 上监听 API 请求。如果需要，我们可以使用`-a`选项更改默认端口。

现在，我们可以使用`curl`或其他工具来发送请求:

```java
$ curl --location --request POST 'http://localhost:8181/v1/data/baeldung/auth/account' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "principal": "user1",
        "authorities": [],
        "uri": "/account/0001",
        "headers": {
            "WebTestClient-Request-Id": "1",
            "Accept": "application/json"
        }
    }
}'
```

**注意/v1/data 前缀之后的路径部分:它对应于策略的包名，点由正斜杠**代替。

响应将是一个 JSON 对象，包含根据输入数据评估策略产生的所有结果:

```java
{
  "result": {
    "allow": [],
    "authorized": false,
    "deny": []
  }
} 
```

`result`属性是包含由策略引擎产生的结果的对象。我们可以看到，在这种情况下，`authorized`属性是`false`。我们还可以看到`allow`和`deny`是空数组。**这意味着没有特定的规则与输入匹配。因此，主授权规则也不匹配。**

## 6.Spring 授权管理器集成

现在我们已经看到了 OPA 的工作方式，我们可以继续前进，将其集成到 Spring 授权框架中。**在这里，我们将关注它的反应式 web 变体，但是一般的想法也适用于常规的基于 MVC 的应用程序**。

首先，我们需要实现使用 OPA 作为后端的`ReactiveAuthorizationManager` bean:

```java
@Bean
public ReactiveAuthorizationManager<AuthorizationContext> opaAuthManager(WebClient opaWebClient) {

    return (auth, context) -> {
        return opaWebClient.post()
          .accept(MediaType.APPLICATION_JSON)
          .contentType(MediaType.APPLICATION_JSON)
          .body(toAuthorizationPayload(auth,context), Map.class)
          .exchangeToMono(this::toDecision);
    };
} 
```

这里，注入的`WebClient`来自另一个 bean，我们从一个`@ConfigurationPropreties`类预初始化它的属性。

处理管道将从当前的`Authentication`和`AuthorizationContext`收集信息的任务委托给`toAuthorizationRequest`方法，然后构建授权请求有效负载。类似地，`toAuthorizationDecision`获取授权响应并将其映射到一个`AuthorizationDecision.`

现在，我们使用这个 bean 来构建一个`SecurityWebFilterChain:`

```java
@Bean
public SecurityWebFilterChain accountAuthorization(ServerHttpSecurity http, @Qualifier("opaWebClient") WebClient opaWebClient) {
    return http
      .httpBasic()
      .and()
      .authorizeExchange(exchanges -> {
          exchanges
            .pathMatchers("/account/*")
            .access(opaAuthManager(opaWebClient));
      })
      .build();
} 
```

我们只将自定义的`AuthorizationManager`应用于`/account` API。这种方法背后的原因是，我们可以很容易地扩展这个逻辑来支持多个策略文档，从而使它们更容易维护。例如，我们可以有一个配置，它使用请求 URI 来选择一个适当的规则包，并使用这个信息来构建授权请求。

在我们的例子中，`/account` API 本身只是一个简单的控制器/服务对，它返回一个用假余额填充的`Account`对象。

## 7.测试

最后但同样重要的是，让我们构建一个集成测试来将所有东西放在一起。首先，让我们确保“快乐之路”行得通。这意味着给定一个经过身份验证的用户，他们应该能够访问自己的帐户:

```java
@Test
@WithMockUser(username = "user1", roles = { "account:read:0001"} )
void testGivenValidUser_thenSuccess() {
    rest.get()
     .uri("/account/0001")
      .accept(MediaType.APPLICATION_JSON)
      .exchange()
      .expectStatus()
      .is2xxSuccessful();
} 
```

其次，我们还必须验证经过身份验证的用户应该只能访问他们自己的帐户:

```java
@Test
@WithMockUser(username = "user1", roles = { "account:read:0002"} )
void testGivenValidUser_thenUnauthorized() {
    rest.get()
     .uri("/account/0001")
      .accept(MediaType.APPLICATION_JSON)
      .exchange()
      .expectStatus()
      .isForbidden();
} 
```

最后，让我们测试一下经过身份验证的用户没有权限的情况:

```java
@Test
@WithMockUser(username = "user1", roles = {} )
void testGivenNoAuthorities_thenForbidden() {
    rest.get()
      .uri("/account/0001")
      .accept(MediaType.APPLICATION_JSON)
      .exchange()
      .expectStatus()
      .isForbidden();
} 
```

我们可以从 IDE 或命令行运行这些测试。**请注意，无论哪种情况，我们都必须首先启动 OPA 服务器，指向包含授权策略文件的文件夹。**

## 8.结论

在本文中，我们展示了如何使用 OPA 将基于 Spring 安全性的应用程序的授权决策具体化。像往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220525034250/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-opa)