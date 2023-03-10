# 使用 Spring 和 RestTemplate 的 StackExchange REST 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tweeting-stackexchange-with-spring-social-part-1>

本文将涵盖一个快速的附带项目——一个自动从各种 Q & A 中发布热门问题的机器人；栈交换站点，如[栈溢出](https://web.archive.org/web/20221208143845/https://stackoverflow.com/ "Stackoverflow")、[服务器故障](https://web.archive.org/web/20221208143845/https://serverfault.com/ "Serverfault")、[超级用户](https://web.archive.org/web/20221208143845/https://superuser.com/ "Superuser")等。我们将为 [StackExchange API](https://web.archive.org/web/20221208143845/https://api.stackexchange.com/docs "Stackexchange API") 构建一个简单的客户端，然后我们将使用 [Spring Social](https://web.archive.org/web/20221208143845/https://spring.io/projects/spring-social "Spring Social") 设置与 Twitter API 的交互——这第一部分将只关注 StackExchange 客户端。

该实现的最初目的是**不成为整个 StackExchange API 的完全成熟的客户端**——这超出了本项目的范围。客户端存在的唯一原因是我找不到一个可以对抗官方 API 版本的客户端。

## 1。美芬依赖

要使用 StackExchange REST API，我们只需要很少的依赖项——基本上只需要一个 HTTP 客户端—`Apache HttpClient`就可以满足这个目的:

```java
<dependency>
   <groupId>org.apache.httpcomponents</groupId>
   <artifactId>httpclient</artifactId>
   <version>4.3.3</version>
</dependency>
```

[Spring `RestTemplate`](/web/20221208143845/https://www.baeldung.com/how-to-use-resttemplate-with-basic-authentication-in-spring#resttemplate "RestTemplate Tutorial") 也可以用来与 HTTP API 进行交互，但是那会在项目中引入很多其他 Spring 相关的依赖项，这些依赖项并不是绝对必要的，所以 HttpClient 会让事情变得简单明了。

## 2。提问客户端

这个客户端的目标是使用 StackExchange [发布的](https://web.archive.org/web/20221208143845/https://api.stackexchange.com/docs/types/question "Questions StackExchange API")REST 服务，而不是为整个 StackExchange APIs 提供一个通用的客户端——因此，出于本文的目的，我们将只研究它。
使用`HTTPClient`的实际 HTTP 通信相对简单:

```java
public String questions(int min, String questionsUri) {
   HttpGet request = null;
   try {
      request = new HttpGet(questionsUri);
      HttpResponse httpResponse = client.execute(request);
      InputStream entityContentStream = httpResponse.getEntity().getContent();
      return IOUtils.toString(entityContentStream, Charset.forName("utf-8"));
   } catch (IOException ex) {
      throw new IllegalStateException(ex);
   } finally {
      if (request != null) {
         request.releaseConnection();
      }
   }
}
```

这个简单的交互对于获取 API 发布的问题原始 JSON 来说已经足够了——下一步将是处理那个 JSON。
这里有一个相关的细节——那就是 **`questionsUri`方法参数**——有多个 StackExchange APIs 可以发布问题(正如[官方文档](https://web.archive.org/web/20221208143845/https://api.stackexchange.com/docs/types/question "StackExchange Questions API docs")所建议的那样)，并且这个方法需要足够灵活来使用所有这些 API。例如，它可以使用最简单的 API，通过将`questionUri`设置为`https://api.stackexchange.com/2.1/questions?site=stackoverflow` 来返回问题，或者它可以使用基于标签的`https://api.stackexchange.com/2.1/tags/{tags}/faq?site=stackoverflow` API，这取决于客户端的需求。

对 StackExchange API 的请求完全配置了查询参数，即使对于更复杂的高级搜索查询也是如此——没有发送正文。为了构建`questionsUri`，我们将构建一个基本的 fluent `RequestBuilder`类，该类将**利用 HttpClient 库中的`URIBuilder`** 。这将负责正确编码 URI，并通常确保最终结果是有效的:

```java
public class RequestBuilder {
   private Map<String, Object> parameters = new HashMap<>();

   public RequestBuilder add(String paramName, Object paramValue) {
       this.parameters.put(paramName, paramValue);
      return this;
   }
   public String build() {
      URIBuilder uriBuilder = new URIBuilder();
      for (Entry<String, Object> param : this.parameters.entrySet()) {
         uriBuilder.addParameter(param.getKey(), param.getValue().toString());
      }

      return uriBuilder.toString();
   }
}
```

现在，为 StackExchange API 构造一个有效的 URI:

```java
String params = new RequestBuilder().
   add("order", "desc").add("sort", "votes").add("min", min).add("site", site).build();
return "https://api.stackexchange.com/2.1/questions" + params;
```

## 3。测试客户端

客户端将输出原始的 JSON，但是为了进行测试，我们需要一个 JSON 处理库，特别是 [`Jackson 2`](https://web.archive.org/web/20221208143845/https://github.com/FasterXML/jackson "Jackson 2") :

```java
<dependency>
   <groupId>com.fasterxml.jackson.core</groupId>
   <artifactId>jackson-databind</artifactId>
   <version>2.3.3</version>
   <scope>test</scope>
</dependency>
```

我们将研究的测试将与实际的 StackExchange API 进行交互:

```java
@Test
public void whenRequestIsPerformed_thenSuccess() 
      throws ClientProtocolException, IOException {
   HttpResponse response = questionsApi.questionsAsResponse(50, Site.serverfault);
   assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
@Test
public void whenRequestIsPerformed_thenOutputIsJson() 
      throws ClientProtocolException, IOException {
   HttpResponse response = questionsApi.questionsAsResponse(50, Site.serverfault);
   String contentType = httpResponse.getHeaders(HttpHeaders.CONTENT_TYPE)[0].getValue();
   assertThat(contentType, containsString("application/json"));
}
@Test
public void whenParsingOutputFromQuestionsApi_thenOutputContainsSomeQuestions() 
     throws ClientProtocolException, IOException {
   String questionsAsJson = questionsApi.questions(50, Site.serverfault);

   JsonNode rootNode = new ObjectMapper().readTree(questionsAsJson);
   ArrayNode questionsArray = (ArrayNode) rootNode.get("items");
   assertThat(questionsArray.size(), greaterThan(20));
}
```

第一个测试已经验证了 API 提供的响应确实是 200 OK，所以检索问题的 GET 请求实际上是成功的。在确保了这个基本条件之后，我们继续讨论表示——由`Content-Type` HTTP 头指定——它需要是 JSON。接下来，我们实际解析 JSON，并验证输出中确实有问题——解析逻辑本身是低级和简单的，这足以满足测试的目的。

请注意，这些请求会计入 API 指定的[速率限制](https://web.archive.org/web/20221208143845/https://api.stackexchange.com/docs/throttle "StackExchange Rate Limits")——因此，现场测试被排除在标准 Maven 构建之外:

```java
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-surefire-plugin</artifactId>
   <version>2.17</version>
   <configuration>
      <excludes>
         <exclude>**/*LiveTest.java</exclude>
      </excludes>
   </configuration>
</plugin>
```

## 4。下一步

当前客户端只关注 StackExchange APIs 发布的众多可用类型中的一个**单一类型**的资源。这是因为它的最初目的是有限的——它只需要让用户能够从 StackExchange 组合中的各个站点消费`Questions`。因此，客户端可以被改进到超出这个初始用例的范围，从而能够使用[其他类型的 API](https://web.archive.org/web/20221208143845/https://api.stackexchange.com/docs?tab=type#docs "Resource Types of the StackExchange API") 。
实现也非常简单**原始**——在使用 Questions REST 服务后，它简单地将 JSON 输出作为字符串返回——而不是输出任何类型的问题模型。因此，潜在的下一步是将这个 JSON 解组到一个适当的域 DTO 中，并返回它而不是原始的 JSON。

## 5。结论

本文的目的是展示如何开始构建与 StackExchange API 的集成，或者实际上是一个基于 HTTP 的 API。它涵盖了如何编写针对实时 API 的集成测试，并确保端到端的交互能够正常工作。

本文的第二部分将展示如何通过使用 Spring Social library 与 Twitter API 进行交互，以及如何使用我们在这里构建的 StackExchange 客户端在新的 Twitter 帐户上发布问题。

我已经建立了几个 twitter 账户，现在每天在 Twitter 上发布不同学科的两个热门问题:

*   每天 StackOverflow 的两个最好的春季问题
*   [Java topso](https://web.archive.org/web/20221208143845/https://twitter.com/JavaTopSO "JavaTopSO twitter account")——stack overflow 每天的两个最佳 Java 问题
*   AskUbuntuBest——AskUbuntu 每天提出的两个最好的问题
*   来自所有 StackExchange 网站的两个最佳 Bash 问题
*   [server fault best](https://web.archive.org/web/20221208143845/https://twitter.com/ServerFaultBest "ServerFaultBest twitter account")–server fault 每天的两个最佳问题

这个 StackExchange 客户端的完整实现是在 github 上的[。](https://web.archive.org/web/20221208143845/https://github.com/eugenp/java-stackexchange#readme "StackExchange Client on Github")