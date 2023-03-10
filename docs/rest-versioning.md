# 对 REST API 进行版本控制

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-versioning>

## 1。问题

开发一个 REST API 是一个困难的问题——有很多选择。本文讨论了其中的一些选项。

## 延伸阅读:

## [Spring Boot 教程——引导一个简单的应用](/web/20220913175102/https://www.baeldung.com/spring-boot-start)

This is how you start understanding Spring Boot.[Read more](/web/20220913175102/https://www.baeldung.com/spring-boot-start) →

## [探索 Spring Boot TestRestTemplate](/web/20220913175102/https://www.baeldung.com/spring-boot-testresttemplate)

Learn how to use the new TestRestTemplate in Spring Boot to test a simple API.[Read more](/web/20220913175102/https://www.baeldung.com/spring-boot-testresttemplate) →

## [带运动衫和弹簧的 REST API](/web/20220913175102/https://www.baeldung.com/jersey-rest-api-with-spring)

Building Restful Web Services using Jersey 2 and Spring.[Read more](/web/20220913175102/https://www.baeldung.com/jersey-rest-api-with-spring) →

## 2。合同里有什么？

在做任何事情之前，我们需要回答一个简单的问题: **`What is the Contract between the API and the`客户端？**

#### 2.1。URIs 是合同的一部分吗？

让我们首先考虑 REST API 的 URI 结构——这是合同的一部分吗？客户应该书签，硬编码和一般依赖 API 的 URIs 吗？

如果是这种情况，那么客户端与 REST 服务的交互将不再由服务本身驱动，而是由[罗伊·菲尔丁所说的](https://web.archive.org/web/20220913175102/http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven "REST APIs must be hypertext-driven") `out-of-band`信息驱动:

> 进入 REST API 时，除了最初的 URI(书签)和一组适合目标受众的标准化媒体类型之外，没有任何先验知识……这里的失败意味着带外信息而不是超文本正在驱动交互。

所以很明显 URIs 不是合同的一部分！客户端应该只知道一个 URI——API 的入口点。在使用 API 时，应该发现所有其他 URIs。

#### 2.2。媒体类型是合同的一部分吗？

用于表示资源的媒体类型信息怎么样——这些是客户机和服务之间的契约的一部分吗？

**为了成功使用 API，客户必须事先了解这些媒体类型**。事实上，这些媒体类型的定义代表了整个合同。

因此，这是 REST 服务最应该关注的地方:

> REST API 应该将几乎所有的描述性工作用于定义用于表示资源和驱动应用程序状态的媒体类型，或者用于为现有的标准媒体类型定义扩展关系名称和/或支持超文本的标记。

所以**媒体类型定义是契约**的一部分，应该是使用 API 的客户的先验知识。这就是标准化的由来。

我们现在对什么是契约有了一个很好的想法，让我们继续讨论如何实际解决版本控制问题。

## 3。高级选项

现在让我们讨论版本化 REST API 的高级方法:

*   **URI 版本控制**–使用版本指示器对 URI 空间进行版本控制
*   **媒体类型版本化**–版本化资源的表示

当我们在 URI 空间中引入版本时，资源的表示被认为是不可变的。因此，当需要在 API 中引入变更时，需要创建一个新的 URI 空间。

例如，假设一个 API 发布了以下资源——用户和权限:

```java
http://host/v1/users
http://host/v1/privileges
```

现在，让我们考虑一下`users` API 中的一个突破性变化需要引入第二个版本:

```java
http://host/v2/users
http://host/v2/privileges
```

**当我们对媒体类型进行版本化并扩展语言时，[我们基于这个头进行](https://web.archive.org/web/20220913175102/https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-p3-payload-16#section-5.2 "Agent Driven content negotiation - the spec")内容协商。**REST API 将使用定制的[供应商 MIME 媒体类型](https://web.archive.org/web/20220913175102/https://datatracker.ietf.org/doc/html/rfc4288#section-3.2 "Media Type Specifications and Registration Procedures")，而不是通用的媒体类型，比如`application/json`。我们将对这些媒体类型进行版本控制，而不是 URIs。

例如:

```java
===>
GET /users/3 HTTP/1.1
Accept: application/vnd.myname.v1+json
<===
HTTP/1.1 200 OK
Content-Type: application/vnd.myname.v1+json
{
    "user": {
        "name": "John Smith"
    }
}
```

我们可以查看这篇[“Rest API 的自定义媒体类型”文章](/web/20220913175102/https://www.baeldung.com/spring-rest-custom-media-type)以获得关于这个主题的更多信息和示例。

这里需要理解的重要一点是，除了媒体类型中定义的内容之外，**客户端不对响应**的结构做任何假设。

这就是普通媒体类型不理想的原因。这些**没有提供足够的语义信息**，并迫使客户端使用要求额外的提示来处理资源的实际表示。

一个例外是使用一些其他方法来唯一地标识内容的语义——比如 XML 模式。

## 4。优缺点

现在，我们已经对客户端和服务之间的契约的一部分有了一个清晰的概念，并且对 API 版本化的选项有了一个高层次的概述，让我们来讨论每种方法的优缺点。

首先，在 URI 中引入版本标识符会导致非常大的 URI 足迹。这是因为任何已发布 API 中的任何突破性变化都将为整个 API 引入一个全新的表示树。随着时间的推移，这不仅成为维护的负担，也成为客户的一个问题——客户现在有了更多的选择。

URI 中的版本标识符也非常不灵活。没有办法简单地发展单个资源的 API，或者整个 API 的一个小的子集。

正如我们之前提到的，这是一个全有或全无的方法。如果 API 的一部分迁移到新版本，那么整个 API 必须随之迁移。这也使得客户端从 v1 升级到 v2 成为一项重大任务，这导致旧版本的升级速度更慢，日落期更长。

当涉及到版本控制时，HTTP 缓存也是一个主要问题。

从中间代理缓存的角度来看，每种方法都有优点和缺点。如果 URI 是版本化的，那么缓存将需要保存每个资源的多个副本-每个 API 版本一个副本。这增加了缓存的负载，并降低了缓存命中率，因为不同的客户端将使用不同的版本。

此外，一些缓存失效机制将不再工作。如果媒体类型是被版本化的，那么客户端和服务都需要支持 [Vary HTTP header](https://web.archive.org/web/20220913175102/https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.44 "HTTP1.1 - Vary Header") 来指示有多个版本被缓存。

然而，从客户端缓存的**角度来看，对媒体类型进行版本控制的解决方案比 URIs 包含版本标识符的解决方案涉及的工作稍微多一些。这是因为当某个东西的关键字是 URL 时，它比媒体类型更容易缓存。**

让我们以定义一些目标来结束这一部分(直接从 [API 进化](https://web.archive.org/web/20220913175102/https://www.mnot.net/blog/2012/12/04/api-evolution)):

*   保持名称的兼容性变化
*   避免新的主要版本
*   使更改向后兼容
*   想想向前兼容性

## 5。API 的可能变化

接下来，让我们考虑一下 REST API 的变化类型——这里介绍了这些变化:

*   表示格式变化
*   资源变化

### 5.1。添加到资源的表示中

媒体类型的格式文档在设计时应考虑向前兼容性。具体来说，客户端应该忽略它不理解的信息(JSON 比 XML 做得更好)。

现在，在资源的表示中添加信息将不会破坏现有的客户端，如果它们被正确实现的话。

继续我们之前的例子，在`user`的表示中添加`amount`并不是一个突破性的改变:

```java
{
    "user": {
        "name": "John Smith", 
        "amount": "300"
    }
}
```

### 5.2。移除或更改现有表示

对客户来说，删除、重命名或重新构建现有展示设计中的信息是一个突破性的变化。这是因为他们已经了解并依赖旧的格式。

这就是内容协商的用武之地。对于这样的变化，我们可以添加一个新的供应商 MIME 媒体类型。

让我们继续前面的例子。比方说我们要把`user`的`name`分解成`firstname`和`lastname`:

```java
===>
GET /users/3 HTTP/1.1
Accept: application/vnd.myname.v2+json
<===
HTTP/1.1 200 OK
Content-Type: application/vnd.myname.v2+json
{
    "user": {
        "firstname": "John", 
        "lastname": "Smith", 
        "amount": "300"
    }
}
```

因此，这确实代表了客户端不兼容的变更——客户端必须请求新的表示并理解新的语义。然而，URI 空间将保持稳定，不会受到影响。

### 5.3。主要语义变化

这些是资源的意义、它们之间的关系或后端映射的变化。这种改变可能需要新的媒体类型，或者可能需要在旧资源旁边发布新的兄弟资源，并利用链接指向它。

虽然这听起来像是在 URI 中再次使用版本标识符，但重要的区别是新资源**是独立于 API** 中的任何其他资源发布的，并且不会在根处派生整个 API。

REST API 应该遵守 HATEOAS 约束。据此，大部分 URIs 应该由客户端发现，而不是硬编码。改变这样的 URI 不应该被认为是不相容的改变。新的 URI 可以取代旧的，客户将能够重新发现 URI，仍然发挥作用。

然而，值得注意的是，尽管由于所有这些原因，在 URI 中使用版本标识符是有问题的，**它在任何方面都不是不 RESTful 的**。

## 6。结论

这篇文章试图提供一个关于发展 REST 服务的非常多样和困难的问题的概述。我们讨论了两种常见的解决方案，每种方案的优点和缺点，以及在 REST 环境中对这些方法进行推理的方法。

文章最后给出了第二个解决方案的案例—**对媒体类型进行版本控制**,同时研究了对 RESTful API 的可能更改。

本教程的完整实现可以在 [GitHub 项目](https://web.archive.org/web/20220913175102/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest "Versioning a REST API")中找到。

## 7 .**。延伸阅读**

通常，这些阅读资源贯穿整篇文章，但在这种情况下，有太多好的资源: