# 用 Spring 介绍 FaunaDB

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/faunadb-spring>

## 1。简介

**在本文中，我们将探索[动物群分布数据库](https://web.archive.org/web/20220630132911/https://fauna.com/)。**我们将了解它给我们的应用带来了哪些特性，我们可以用它做什么，以及如何与它交互。

## 2。什么是动物群？

**fana 是一种多协议、多模式、多租户、分布式、事务型数据库即服务(DBaaS)产品。**这听起来很复杂，我们来分解一下。

### 2.1。数据库即服务

**“数据库即服务”意味着数据库由云提供商托管，云提供商负责所有的基础设施和维护，因此我们只需处理特定领域的细节** —集合、索引、查询等。这有助于消除管理这样一个系统的复杂性，同时仍然受益于它的特性。

### 2.2。分布式事务数据库

**分布式意味着数据库跨多个服务器运行。**这有助于同时提高效率和容错能力。如果一台服务器出现故障，那么整个数据库仍然能够继续正常工作。

事务性意味着数据库对数据的有效性提供了强有力的保证。在单个事务中执行的数据更新作为一个整体要么成功，要么失败，没有将数据留在部分状态的风险。

作为进一步的措施，Fauna 提供了隔离级别，这将确保跨多个分布式节点播放多个事务的结果总是正确的。对于分布式数据库来说，这是一个重要的考虑事项——否则，不同的事务可能会在不同的节点上以不同的方式运行，并以不同的结果结束。

例如，让我们考虑应用于同一记录的以下事务:

1.  将该值设置为“15”
2.  将该值增加“3”

如果按照显示的顺序玩，最终结果将是“18”。然而，如果它们以相反的顺序播放，最终结果将是“15”。如果在同一个系统的不同节点上的结果不同，这就更加令人困惑，因为这意味着我们的数据在不同的节点上是不一致的。

### 2.3。多模型数据库

**多模型数据库意味着它允许我们以不同的方式对不同类型的数据建模**，所有这些都在同一个数据库引擎中，并且可以通过相同的连接进行访问。

在内部，动物群是一个文档数据库。这意味着它将每个记录存储为一个结构化文档，用 JSON 表示任意形状。这使得 Fauna 可以充当键值存储(文档只有一个字段`value`)或表格存储(文档有任意多个字段，但它们都是平面的)。但是，我们也可以存储更复杂的文档，包括嵌套的字段、数组等等:

```java
// Key-Value document
{
  "value": "Baeldung"
}

// Tabular document
{
  "name": "Baeldung",
  "url": "https://www.baeldung.com/"
}

// Structured document
{
  "name": "Baeldung",
  "sites": [
    {
      "id": "cs",
      "name": "Computer Science",
      "url": "https://www.baeldung.com/cs"
    },
    {
      "id": "linux",
      "name": "Linux",
      "url": "https://www.baeldung.com/linux"
    },
    {
      "id": "scala",
      "name": "Scala",
      "url": "https://www.baeldung.com/scala"
    },
    {
      "id": "kotlin",
      "name": "Kotlin",
      "url": "https://www.baeldung.com/kotlin"
    },
  ]
}
```

除此之外，我们还可以访问关系数据库中常见的一些特性。具体来说，我们可以在文档上创建索引以提高查询效率，跨多个集合应用约束以确保数据保持一致，并一次性执行跨多个集合的查询。

Fauna 的查询引擎还支持图形查询，允许我们构建跨越多个集合的复杂数据结构，并像访问单个数据图形一样访问它们。

最后，动物群有时间建模工具，可以让我们在其生命的任何时间点与我们的数据库进行交互。这意味着，我们不仅可以看到记录随时间发生的所有变化，还可以直接访问给定时间点的数据。

### 2.4。多租户数据库

多租户数据库服务器意味着它支持不同用户使用的多个不同的数据库。这在用于云托管的数据库引擎中很常见，因为这意味着一台服务器可以支持许多不同的客户。

动物群从一个稍微不同的方向来看待这个问题。在一个安装的数据库引擎中，不同的租户代表不同的客户，而不是使用租户来代表单个客户的不同数据子集。

可以创建本身是其他数据库的子数据库。然后我们可以创建访问这些子数据库的凭证。然而，Fauna 的不同之处在于，我们可以对我们所连接的子数据库中的数据执行只读查询。但是，不可能访问父数据库或同级数据库中的数据。

这允许我们在同一个父数据库中为不同的服务创建子数据库，然后让管理员用户一次性查询所有数据——这对于分析目的来说非常方便。

### 2.5。多协议数据库

这意味着我们有多种不同的方式来访问相同的数据。

访问我们数据的标准方式是通过提供的驱动程序之一使用动物查询语言(FQL)。这使我们能够使用数据库引擎的全部功能，允许我们以任何需要的方式访问所有数据。

或者，Fauna 也公开了一个我们可以使用的 GraphQL 端点。这样做的好处是，我们可以在任何应用程序中使用它，而不考虑编程语言，而不是依赖于我们语言的专用驱动程序。但是，并非所有功能都可以通过该界面使用。特别是，我们需要提前创建一个描述数据形状的 GraphQL 模式，这意味着我们不能在同一个集合中拥有不同形状的不同记录。

## 3。创建动物数据库

现在我们知道了动物群能为我们做什么，让我们实际上创建一个数据库供我们使用。

如果我们还没有账户，我们需要[创建一个](/web/20220630132911/https://www.baeldung.com/fauna-register)。

登录后，我们只需在仪表板上单击“创建数据库”链接:

[![](img/a72fb3f25a955954756eae15b9fb08b3.png)](/web/20220630132911/https://www.baeldung.com/wp-content/uploads/2022/01/fauna-create-db.png)

这将打开一个窗格，显示数据库的名称和区域。我们还可以选择用一些示例数据预先填充数据库，看看它是如何工作的，以帮助我们习惯这个系统:

[![](img/84f467681b3cbd1a95c859af739d0925.png)](/web/20220630132911/https://www.baeldung.com/wp-content/uploads/2022/01/fauna-db-region.png)

在这个屏幕上，选择“Region Group”很重要，这不仅是因为我们必须为超出免费限额的任何东西支付费用，也是因为我们需要用来从外部连接到数据库的端点。

一旦我们做到了这一点，我们就有了一个完整的数据库，可以根据需要使用。如果我们选择了演示数据，那么它将包含一些填充的集合、索引、定制函数和一个 GraphQL 模式。如果没有，那么数据库是完全空的，我们可以创建我们想要的结构了:

[![](img/e3d92407308b881f3f0ac562020058c9.png)](/web/20220630132911/https://www.baeldung.com/wp-content/uploads/2022/01/fauna-db-structure.png)

最后，为了从外部连接到数据库，我们需要一个认证密钥。我们可以从侧边栏的 Security 选项卡中创建一个:

[![](img/45753d63eedb3eeb3344a6859851468c.png)](/web/20220630132911/https://www.baeldung.com/wp-content/uploads/2022/01/fauna-auth-key.png)

当创建一个新的密钥时，请确保将其复制下来，因为出于安全原因，在离开屏幕后没有办法再找回它。

## 4。与动物互动

现在我们有了一个数据库，我们可以开始使用它了。

动物群提供了两种不同的从外部读取和写入数据库数据的方法:FQL 驱动程序和 GraphQL API。我们还可以访问 Fauna Shell，这允许我们在 web UI 中执行任意命令。

### 4.1。动物群外壳

动物群 Shell 允许我们在 web UI 中执行任何命令。我们可以使用我们配置的任何键来实现这一点，就像我们使用该键从外部连接一样，或者作为某些特殊的管理连接:

[![](img/939207b66bc00b65913762ed17733b4c.png)](/web/20220630132911/https://www.baeldung.com/wp-content/uploads/2022/01/fauna-shell.png)

这允许我们探索我们的数据，并以一种非常低摩擦的方式测试我们希望在应用程序中使用的查询。

### 4.2。连接 FQL

**如果我们想将我们的应用程序连接到动物群并使用 FQL，我们需要使用[提供的驱动程序](https://web.archive.org/web/20220630132911/https://docs.fauna.com/fauna/current/drivers/)**——包括 Java 和 Scala 的驱动程序。

Java 驱动程序要求我们在 Java 11 或更高版本上运行。

我们需要做的第一件事是添加依赖性。如果我们使用 Maven，我们只需将它添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.faunadb</groupId>
    <artifactId>faunadb-java</artifactId>
    <version>4.2.0</version>
    <scope>compile</scope>
</dependency>
```

然后，我们需要创建一个客户机连接，我们可以用它来与数据库通信:

```java
FaunaClient client = FaunaClient.builder()
    .withEndpoint("https://db.us.fauna.com/")
    .withSecret("put-your-authorization-key-here")
    .build();
```

注意，我们需要为数据库端点提供正确的值——根据创建数据库时选择的区域组而有所不同——以及我们之前创建的密钥。

这个客户端将充当一个连接池，根据不同查询的需要打开到数据库的新连接。这意味着我们可以在应用程序开始时创建一次，然后根据需要重用它。

如果我们需要连接不同的秘密，这将需要不同的客户。例如，如果我们想要与同一个父数据库中的多个不同的子数据库进行交互。

现在我们有了一个客户机，我们可以用它向数据库发送查询:

```java
client.query(
    language.Get(language.Ref(language.Collection("customers"), 101))
).get();
```

### 4.3。与 GraphQL 连接

Fauna 提供了一个完整的 GraphQL API 来与我们的数据库交互。这可以让我们在没有任何特殊驱动程序的情况下使用数据库，只需要一个 HTTP 客户端。

**为了使用 GraphQL 支持，我们需要首先创建一个 GraphQL 模式。**这将定义模式本身，以及它如何映射到我们预先存在的动物群数据库结构上——比如集合、索引和函数。**一旦完成，任何支持 GraphQL 的客户端——甚至只是一个 HTTP 客户端，比如`RestTemplate` —都可以用来调用我们的数据库。**

请注意，这将只允许我们与数据库中的数据进行交互。如果我们希望使用任何管理命令——比如创建新的集合或索引——那么这需要 FQL 命令或 web 管理 UI。

通过 GraphQL 连接到动物群需要我们使用正确的 URL——美国地区的 https://graphql.us.fauna.com/graphql——并在`Authorization`头中提供我们的认证密钥作为不记名令牌。此时，我们可以将它用作任何普通的 GraphQL 端点，方法是向 URL 发出 POST 请求，并在正文中提供查询或变异，还可以选择使用任何变量。

## 5。利用春天的动物群

现在我们已经了解了什么是动物群以及如何使用它，我们可以看看如何将它集成到我们的 Spring 应用程序中。

动物群没有任何本地的 Spring 驱动程序。相反，我们将把普通的 Java 驱动程序配置为 Spring beans，以便在我们的应用程序中使用。

### 5.1.动物群结构

在我们能够利用动物群之前，我们需要一些配置。具体来说，我们需要知道我们的动物数据库所在的区域——然后我们可以从中获得适当的 URL——并且我们需要知道一个可以用来连接数据库的秘密。

为此，我们将把`fauna.region`和`fauna.secret`的属性添加到我们的`application.properties`文件中——或者任何其他支持的[弹簧配置方法](/web/20220630132911/https://www.baeldung.com/properties-with-spring):

```java
fauna.region=us
fauna.secret=FaunaSecretHere
```

请注意，我们在这里定义的是动物群区域，而不是 URL。这允许我们从相同的设置中正确地导出 FQL 和 GraphQL 的 URL。这避免了我们可能不同地配置两个 URL 的风险。

### 5.2。FQL 客户端

**如果我们计划在应用程序中使用 FQL，我们可以在 Spring 上下文中添加一个`FaunaClient` bean。**这将涉及创建一个 Spring 配置对象来使用适当的属性并构造`FaunaClient`对象:

```java
@Configuration
class FaunaClientConfiguration {
    @Value("https://db.${fauna.region}.fauna.com/")
    private String faunaUrl;

    @Value("${fauna.secret}")
    private String faunaSecret;

    @Bean
    FaunaClient getFaunaClient() throws MalformedURLException {
        return FaunaClient.builder()
            .withEndpoint(faunaUrl)
            .withSecret(faunaSecret)
            .build();
    }
} 
```

这让我们可以在应用程序的任何地方直接使用`FaunaClient`，就像我们使用`JdbcTemplate`访问 JDBC 数据库一样。如果我们愿意的话，我们也有机会将它包装在一个更高级别的对象中，以特定于领域的方式工作。

### 5.3。GraphQL 客户端

如果我们计划使用 GraphQL 来访问动物群，需要做更多的工作。没有用于调用 GraphQL APIs 的标准客户端。相反，**我们将使用 [Spring RestTemplate](/web/20220630132911/https://www.baeldung.com/rest-template) 向 GraphQL 端点发出标准的 HTTP 请求。如果我们正在构建一个基于 WebFlux 的应用程序，新的 [WebClient](/web/20220630132911/https://www.baeldung.com/spring-5-webclient) 也会工作得很好。**

为了实现这一点，我们将编写一个类来包装`RestTemplate`，并可以对动物群进行适当的 HTTP 调用:

```java
@Component
public class GraphqlClient {
    @Value("https://graphql.${fauna.region}.fauna.com/graphql")
    private String faunaUrl;

    @Value("${fauna.secret}")
    private String faunaSecret;

    private RestTemplate restTemplate = new RestTemplate();

    public <T> T query(String query, Class<T> cls) {
        return query(query, Collections.emptyMap(), cls);
    }

    public <T, V> T query(String query, V variables, Class<T> cls) {
        var body = Map.of("query", query, "variables", variables);

        var request = RequestEntity.post(faunaUrl)
            .header("Authorization", "Bearer " + faunaSecret)
            .body(body);
        var response = restTemplate.exchange(request, cls);

        return response.getBody();
    }
}
```

这个客户端允许我们从应用程序的其他组件对 Fauna 进行 GraphQL 调用。我们有两个方法，一个只接受 GraphQL 查询字符串，另一个额外接受一些变量。

它们也都接受将查询结果反序列化到的类型。使用它将处理与动物对话的所有细节，使我们能够专注于我们的应用程序需求。

## 6。总结

在本文中，我们简要介绍了动物群数据库，看到了它提供的一些功能，这些功能使它成为我们下一个项目的一个非常有吸引力的选择，也看到了我们如何从我们的应用程序中与它进行交互。

为什么不在你的下一个项目中探索我们在这里提到的一些特性呢？