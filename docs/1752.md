# 黄瓜的 REST API 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cucumber-rest-api-testing>

## 1。概述

本教程介绍了用户验收测试的常用工具[cumber](https://web.archive.org/web/20220803030100/https://cucumber.io/)，以及如何在 REST API 测试中使用它。

此外，为了使文章独立于任何外部 REST 服务，我们将使用 WireMock，一个 stubbing 和 mocking web 服务库。如果你想进一步了解这个库，请参考 WireMock 的[介绍。](/web/20220803030100/https://www.baeldung.com/introduction-to-wiremock)

## 2。小黄瓜——黄瓜的语言

Cucumber 是一个支持[行为驱动开发(BDD)](/web/20220803030100/https://www.baeldung.com/cs/bdd-guide) 的测试框架，允许用户以纯文本方式定义应用操作。它基于[小黄瓜领域专用语言](https://web.archive.org/web/20220803030100/https://github.com/cucumber/cucumber/wiki/Gherkin) (DSL)工作。这种简单但功能强大的 Gherkin 语法允许开发人员和测试人员编写复杂的测试，同时让非技术用户也能理解。

### 2.1。小黄瓜介绍

Gherkin 是一种面向行的语言，使用行尾、缩进和关键字来定义文档。每个非空行通常以一个 Gherkin 关键字开始，后面是一个任意文本，通常是对关键字的描述。

整个结构必须被写入一个扩展名为`feature`的文件才能被 Cucumber 识别。

下面是一个简单的小黄瓜文档示例:

```
Feature: A short description of the desired functionality

  Scenario: A business situation
    Given a precondition
    And another precondition
    When an event happens
    And another event happens too
    Then a testable outcome is achieved
    And something else is also completed
```

在接下来的章节中，我们将描述一个小黄瓜结构中几个最重要的元素。

### 2.2。特色

我们使用一个小黄瓜文件来描述一个需要测试的应用程序特性。该文件在最开始包含`Feature`关键字，后面是同一行上的特性名称和下面可能跨越多行的可选描述。

Cucumber parser 跳过除了关键字`Feature`之外的所有文本，只是出于文档的目的才包含它。

### 2.3。场景和步骤

一个小黄瓜结构可能由一个或多个场景组成，由关键字`Scenario`识别。一个场景基本上是一个允许用户验证应用程序功能的测试。它应该描述初始环境、可能发生的事件以及这些事件产生的预期结果。

这些事情是使用步骤来完成的，这些步骤由五个关键字中的一个来标识:`Given`、`When`、`Then`、`And`和`But`。

*   `Given`:这一步是在用户开始与应用程序交互之前，将系统置于一个明确定义的状态。一个`Given`子句可以被认为是用例的先决条件。
*   `When`:`When`步骤用于描述应用程序发生的事件。这可以是用户采取的操作，也可以是另一个系统触发的事件。
*   这一步是规定测试的预期结果。结果应该与被测试特性的商业价值相关。
*   `And`和`But`:当有多个相同类型的步骤时，这些关键字可以用来替换上述步骤关键字。

Cucumber 实际上并没有区分这些关键字，但是它们仍然存在，使得特性更具可读性，并且与 BDD 结构保持一致。

### 3。Cucumber-JVM 实现

Cucumber 最初是用 Ruby 编写的，已经通过 Cucumber-JVM 实现移植到 Java 中，这是本节的主题。

### 3.1。Maven 依赖关系

为了在 Maven 项目中使用 Cucumber-JVM，POM 中需要包含以下依赖项:

```
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>6.8.0</version>
    <scope>test</scope>
</dependency>
```

为了方便用 Cucumber 进行 JUnit 测试，我们还需要一个依赖项:

```
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>6.8.0</version>
</dependency>
```

或者，我们可以使用另一个工件来利用 Java 8 中的 lambda 表达式，这不会在本教程中讨论。

### 3.2。步骤定义

如果不将小黄瓜场景转化为行动，它们将毫无用处，这就是步骤定义发挥作用的地方。基本上，步骤定义是一个带注释的 Java 方法，附带一个模式，其工作是将纯文本格式的小黄瓜步骤转换为可执行代码。在解析一个特征文档之后，Cucumber 将搜索与预定义的小黄瓜步骤相匹配的步骤定义来执行。

为了使它更清楚，让我们来看看下面的步骤:

```
Given I have registered a course in Baeldung
```

和步骤定义:

```
@Given("I have registered a course in Baeldung")
public void verifyAccount() {
    // method implementation
}
```

当 Cucumber 读取给定的步骤时，它将寻找注释模式与 Gherkin 文本匹配的步骤定义。

## 4。创建和运行测试

### 4.1。编写特征文件

让我们从在一个文件中声明场景和步骤开始，该文件的名称以扩展名`.feature`结尾:

```
Feature: Testing a REST API
  Users should be able to submit GET and POST requests to a web service, 
  represented by WireMock

  Scenario: Data Upload to a web service
    When users upload data on a project
    Then the server should handle it and return a success status

  Scenario: Data retrieval from a web service
    When users want to get information on the 'Cucumber' project
    Then the requested data is returned
```

我们现在将该文件保存在一个名为`Feature`的目录中，条件是该目录将在运行时加载到类路径中，例如`src/main/resources`。

### 4.2。配置 JUnit 与 Cucumber 一起工作

为了让 JUnit 知道 Cucumber 并在运行时读取特性文件，必须将`Cucumber`类声明为`Runner`。我们还需要告诉 JUnit 搜索特性文件和步骤定义的位置。

```
@RunWith(Cucumber.class)
@CucumberOptions(features = "classpath:Feature")
public class CucumberIntegrationTest {

}
```

如您所见，`CucumberOption`的`features`元素定位之前创建的特征文件。另一个重要的元素叫做`glue`，它提供了步骤定义的路径。然而，如果测试用例以及步骤定义在本教程中的同一个包中，那么这个元素可能会被删除。

### 4.3。编写步骤定义

当 Cucumber 解析步骤时，它将搜索用 Gherkin 关键字注释的方法来定位匹配的步骤定义。

步骤定义的表达式可以是正则表达式，也可以是黄瓜表达式。在本教程中，我们将使用黄瓜表达式。

下面是一个完全匹配一个小黄瓜步骤的方法。该方法将用于向 REST web 服务发送数据:

```
@When("users upload data on a project")
public void usersUploadDataOnAProject() throws IOException {

}
```

下面是一个匹配 Gherkin 步骤的方法，它从文本中获取一个参数，该参数将用于从 REST web 服务中获取信息:

```
@When("users want to get information on the {string} project")
public void usersGetInformationOnAProject(String projectName) throws IOException {

}
```

如您所见，`usersGetInformationOnAProject`方法带有一个`String`参数，即项目名称。该参数由注释中的`{string}`声明，在这里它对应于步骤文本中的`Cucumber`。

或者，我们可以使用正则表达式:

```
@When("^users want to get information on the '(.+)' project$")
public void usersGetInformationOnAProject(String projectName) throws IOException {

}
```

注意，`‘^'`和`‘$'`分别表示正则表达式的开始和结束。而`‘(.+)'`对应于`String`参数。

我们将在下一节提供上述两种方法的工作代码。

### 4.4。创建和运行测试

首先，我们将从一个 JSON 结构开始，说明通过 POST 请求上传到服务器的数据，以及使用 GET 下载到客户机的数据。该结构保存在`jsonString`字段，如下所示:

```
{
    "testing-framework": "cucumber",
    "supported-language": 
    [
        "Ruby",
        "Java",
        "Javascript",
        "PHP",
        "Python",
        "C++"
    ],

    "website": "cucumber.io"
}
```

为了演示 REST API，我们使用一个 WireMock 服务器:

```
WireMockServer wireMockServer = new WireMockServer(options().dynamicPort());
```

此外，我们将使用 Apache HttpClient API 来表示用于连接到服务器的客户机:

```
CloseableHttpClient httpClient = HttpClients.createDefault();
```

现在，让我们继续在步骤定义中编写测试代码。我们将首先为`usersUploadDataOnAProject`方法这样做。

在客户端连接到服务器之前，服务器应该正在运行:

```
wireMockServer.start();
```

使用 WireMock API 来存根化 REST 服务:

```
configureFor("localhost", wireMockServer.port());
stubFor(post(urlEqualTo("/create"))
  .withHeader("content-type", equalTo("application/json"))
  .withRequestBody(containing("testing-framework"))
  .willReturn(aResponse().withStatus(200)));
```

现在，向服务器发送一个 POST 请求，内容取自上面声明的`jsonString`字段:

```
HttpPost request = new HttpPost("http://localhost:" + wireMockServer.port() + "/create");
StringEntity entity = new StringEntity(jsonString);
request.addHeader("content-type", "application/json");
request.setEntity(entity);
HttpResponse response = httpClient.execute(request);
```

以下代码断言 POST 请求已被成功接收和处理:

```
assertEquals(200, response.getStatusLine().getStatusCode());
verify(postRequestedFor(urlEqualTo("/create"))
  .withHeader("content-type", equalTo("application/json")));
```

服务器在使用后应停止运行:

```
wireMockServer.stop();
```

我们将在这里实现的第二个方法是`usersGetInformationOnAProject(String projectName)`。与第一个测试类似，我们需要启动服务器，然后存根化 REST 服务:

```
wireMockServer.start();

configureFor("localhost", wireMockServer.port());
stubFor(get(urlEqualTo("/projects/cucumber"))
  .withHeader("accept", equalTo("application/json"))
  .willReturn(aResponse().withBody(jsonString)));
```

提交 GET 请求并接收响应:

```
HttpGet request = new HttpGet("http://localhost:" + wireMockServer.port() + "/projects/" + projectName.toLowerCase());
request.addHeader("accept", "application/json");
HttpResponse httpResponse = httpClient.execute(request);
```

我们将使用一个助手方法将`httpResponse`变量转换为`String`:

```
String responseString = convertResponseToString(httpResponse);
```

下面是转换助手方法的实现:

```
private String convertResponseToString(HttpResponse response) throws IOException {
    InputStream responseStream = response.getEntity().getContent();
    Scanner scanner = new Scanner(responseStream, "UTF-8");
    String responseString = scanner.useDelimiter("\\Z").next();
    scanner.close();
    return responseString;
}
```

下面验证整个过程:

```
assertThat(responseString, containsString("\"testing-framework\": \"cucumber\""));
assertThat(responseString, containsString("\"website\": \"cucumber.io\""));
verify(getRequestedFor(urlEqualTo("/projects/cucumber"))
  .withHeader("accept", equalTo("application/json")));
```

最后，如前所述停止服务器。

## 5。并行运行特性

Cucumber-JVM 本身支持多线程的并行测试执行。我们将使用 JUnit 和 Maven Failsafe 插件来执行运行程序。或者，我们可以使用 Maven Surefire。

JUnit 并行运行特性文件而不是场景，这意味着特性文件中的所有场景将由同一个线程执行。

现在让我们添加插件配置:

```
<plugin>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${maven-failsafe-plugin.version}</version>
    <configuration>
        <includes>
            <include>CucumberIntegrationTest.java</include>
        </includes>
        <parallel>methods</parallel>
        <threadCount>2</threadCount>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

请注意:

*   `parallel:`可以是`classes, methods`，或者两者都是——在我们的例子中，`classes`将使每个测试类在一个单独的线程中运行
*   `threadCount:`表示应该为此执行分配多少线程

这就是我们并行运行 Cucumber 特性所需要做的全部工作。

## 6。结论

在本教程中，我们介绍了 Cucumber 的基础知识，以及这个框架如何使用 Gherkin 领域特定语言来测试 REST API。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220803030100/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-testing)