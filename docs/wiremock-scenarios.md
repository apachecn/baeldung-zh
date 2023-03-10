# 使用 WireMock 场景

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/wiremock-scenarios>

## 1.概观

这个快速教程将向**展示我们如何用 WireMock** 测试一个有状态的基于 HTTP 的 API。

要开始使用这个库，先看看我们的[wire mock 简介](/web/20220607162151/https://www.baeldung.com/introduction-to-wiremock)教程。

## 2.Maven 依赖性

为了能够利用 [WireMock 库](https://web.archive.org/web/20220607162151/https://search.maven.org/search?q=a:wiremock)，我们需要在 POM 中包含以下依赖项:

```java
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock</artifactId>
    <version>2.21.0</version>
    <scope>test</scope>
</dependency>
```

## 3.我们想要模仿的示例 API

Wiremock 中场景的概念是**帮助模拟 REST API** 的不同状态。这使我们能够创建测试，在这些测试中，我们使用的 API 根据其所处的状态而表现不同。

为了说明这一点，我们来看一个实际的例子:一个“Java 提示”服务，每当我们请求它的`/java-tip `端点时，它就给我们一个关于 Java 的不同提示。

如果我们要小费，我们会在`text/plain`得到一份:

```java
"use composition rather than inheritance" 
```

如果我们再打一次，我们会得到不同的小费。

## 4.创建场景状态

我们需要让 WireMock 为`“/java-tip”`端点创建存根。每一个存根都将返回一个特定的文本，该文本对应于模拟 API 的三种状态之一:

```java
public class WireMockScenarioExampleIntegrationTest {
    private static final String THIRD_STATE = "third";
    private static final String SECOND_STATE = "second";
    private static final String TIP_01 = "finally block is not called when System.exit()" 
      + " is called in the try block";
    private static final String TIP_02 = "keep your code clean";
    private static final String TIP_03 = "use composition rather than inheritance";
    private static final String TEXT_PLAIN = "text/plain";

    static int port = 9999;

    @Rule
    public WireMockRule wireMockRule = new WireMockRule(port);    

    @Test
    public void changeStateOnEachCallTest() throws IOException {
        createWireMockStub(Scenario.STARTED, SECOND_STATE, TIP_01);
        createWireMockStub(SECOND_STATE, THIRD_STATE, TIP_02);
        createWireMockStub(THIRD_STATE, Scenario.STARTED, TIP_03);

    }

    private void createWireMockStub(String currentState, String nextState, String responseBody) {
        stubFor(get(urlEqualTo("/java-tip"))
          .inScenario("java tips")
          .whenScenarioStateIs(currentState)
          .willSetStateTo(nextState)
          .willReturn(aResponse()
            .withStatus(200)
            .withHeader("Content-Type", TEXT_PLAIN)
            .withBody(responseBody)));
    }

}
```

在上面的类中，我们使用 WireMock 的 JUnit 规则类`WireMockRule`。这将在运行 JUnit 测试时设置 WireMock 服务器。

**然后，我们使用 WireMock 的`stubFor`方法来创建我们稍后将使用的存根。**

创建存根时使用的主要方法有:

*   `whenScenarioStateIs`:定义场景需要处于哪种状态，WireMock 才能使用这个存根
*   `willSetStateTo`:给出使用这个存根后 WireMock 将状态设置为的值

任何场景的初始状态都是`Scenario.STARTED`。因此，我们创建一个存根，在状态为`Scenario.STARTED.`时使用，这将状态转移到 SECOND_STATE。

我们还添加了存根，以便从 SECOND_STATE 移动到 THIRD_STATE，最后从 THIRD_STATE 返回到`Scenario.STARTED.`,因此如果我们继续调用 `/java-tip`端点，状态会发生如下变化:

**T2`Scenario.STARTED -> SECOND_STATE -> THIRD_STATE -> Scenario.STARTED`**

## 5.使用该场景

为了使用 WireMock 场景，我们简单地重复调用`/java-tip`端点。所以我们需要修改我们的测试类，如下所示:

```java
 @Test
    public void changeStateOnEachCallTest() throws IOException {
        createWireMockStub(Scenario.STARTED, SECOND_STATE, TIP_01);
        createWireMockStub(SECOND_STATE, THIRD_STATE, TIP_02);
        createWireMockStub(THIRD_STATE, Scenario.STARTED, TIP_03);

        assertEquals(TIP_01, nextTip());
        assertEquals(TIP_02, nextTip());
        assertEquals(TIP_03, nextTip());
        assertEquals(TIP_01, nextTip());        
    }

    private String nextTip() throws ClientProtocolException, IOException {
        CloseableHttpClient httpClient = HttpClients.createDefault();
        HttpGet request = new HttpGet(String.format("http://localhost:%s/java-tip", port));
        HttpResponse httpResponse = httpClient.execute(request);
        return firstLineOfResponse(httpResponse);
    }

    private static String firstLineOfResponse(HttpResponse httpResponse) throws IOException {
        try (BufferedReader reader = new BufferedReader(
          new InputStreamReader(httpResponse.getEntity().getContent()))) {
            return reader.readLine();
        }
    }
```

`nextTip()`方法调用`/java-tip`端点，然后将响应作为`String`返回。所以我们在每个 `assertEquals()`调用中使用它来检查调用是否确实让场景在不同的状态之间循环。

## 6.结论

在本文中，我们看到了如何使用 WireMock 场景来模拟一个 API，该 API 根据其所处的状态改变其响应。

和往常一样，本教程中使用的所有代码都可以在 GitHub 上获得[。](https://web.archive.org/web/20220607162151/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-testing)