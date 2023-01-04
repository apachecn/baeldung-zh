# 用 JBehave 测试 REST API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jbehave-rest-testing>

## 1。简介

在本文中，我们将快速浏览一下 [JBehave](https://web.archive.org/web/20221128105745/http://jbehave.org/) ，然后着重从 BDD 的角度测试 REST API。

## 2.JBehave 和 BDD

JBehave 是一个行为驱动的开发框架。它旨在为自动化验收测试提供一种直观和可访问的方式。

如果你不熟悉 BDD，从这篇文章开始是一个好主意，它涵盖了另一个 BDD 测试框架——Cucumber,其中我们介绍了一般的 BDD 结构和特性。

与其他 BDD 框架类似，JBehave 采用了以下概念:

*   story——代表业务功能的可自动执行的增量，由一个或多个场景组成
*   场景——代表系统行为的具体例子
*   步骤——使用经典的 BDD 关键字来表示实际行为:`Given`、`When`和`Then`

典型的情况是:

```
Given a precondition
When an event occurs
Then the outcome should be captured
```

场景中的每一步都对应于 JBehave 中的一个注释:

*   `@Given`:启动上下文
*   `@When`:做动作
*   `@Then`:测试预期结果

## 3.Maven 依赖性

为了在我们的 maven 项目中使用 JBehave， [jbehave-core](https://web.archive.org/web/20221128105745/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jbehave%22%20AND%20a%3A%22jbehave-core%22) 依赖项应该包含在`pom`中:

```
<dependency>
    <groupId>org.jbehave</groupId>
    <artifactId>jbehave-core</artifactId>
    <version>4.1</version>
    <scope>test</scope>
</dependency>
```

## 4.一个简单的例子

要使用 JBehave，我们需要遵循以下步骤:

1.  写一个用户故事
2.  将步骤从用户故事映射到 Java 代码
3.  配置用户情景
4.  运行 JBehave 测试
5.  查看结果

### 4.1。故事

先说下面这个简单的故事:“作为用户，我想增加一个计数器，这样我就可以让计数器的值增加 1”。

我们可以在一个`.story`文件中定义这个故事:

```
Scenario: when a user increases a counter, its value is increased by 1

Given a counter
And the counter has any integral value
When the user increases the counter
Then the value of the counter must be 1 greater than previous value
```

### 4.2。映射步骤

给定步骤，让我们用 Java 实现它:

```
public class IncreaseSteps {
    private int counter;
    private int previousValue;

    @Given("a counter")
    public void aCounter() {
    }

    @Given("the counter has any integral value")
    public void counterHasAnyIntegralValue() {
        counter = new Random().nextInt();
        previousValue = counter;
    }

    @When("the user increases the counter")
    public void increasesTheCounter() {
        counter++;
    }

    @Then("the value of the counter must be 1 greater than previous value")
    public void theValueOfTheCounterMustBe1Greater() {
        assertTrue(1 == counter - previousValue);
    }
}
```

记住**注释中的值必须精确匹配描述**。

### 4.3。配置我们的故事

要执行这些步骤，我们需要为我们的故事搭建舞台:

```
public class IncreaseStoryLiveTest extends JUnitStories {

    @Override
    public Configuration configuration() {
        return new MostUsefulConfiguration()
          .useStoryLoader(new LoadFromClasspath(this.getClass()))
          .useStoryReporterBuilder(new StoryReporterBuilder()
            .withCodeLocation(codeLocationFromClass(this.getClass()))
            .withFormats(CONSOLE));
    }

    @Override
    public InjectableStepsFactory stepsFactory() {
        return new InstanceStepsFactory(configuration(), new IncreaseSteps());
    }

    @Override
    protected List<String> storyPaths() {
        return Arrays.asList("increase.story");
    }

}
```

在`storyPaths()`中，我们提供了要由 JBehave 解析的`.story`文件路径。实际实施步骤在`stepsFactory()`中提供。然后在`configuration()`中，故事加载器和故事报告被正确配置。

现在我们已经准备好了一切，我们可以开始我们的故事了。

### 4.4。查看测试结果

我们可以在控制台中看到我们的测试结果。随着我们的测试成功通过，输出将与我们的故事相同:

```
Scenario: when a user increases a counter, its value is increased by 1
Given a counter
And the counter has any integral value
When the user increases the counter
Then the value of the counter must be 1 greater than previous value
```

如果我们忘记实现场景中的任何步骤，报告会让我们知道。假设我们没有实现`@When`步骤:

```
Scenario: when a user increases a counter, its value is increased by 1
Given a counter
And the counter has any integral value
When the user increases the counter (PENDING)
Then the value of the counter must be 1 greater than previous value (NOT PERFORMED)
```

```
@When("the user increases the counter")
@Pending
public void whenTheUserIncreasesTheCounter() {
    // PENDING
}
```

报告会说`@When` a 步骤是待定的，因此`@Then`步骤不会被执行。

如果我们的@Then 步骤失败了怎么办？我们可以马上从报告中发现错误:

```
Scenario: when a user increases a counter, its value is increased by 1
Given a counter
And the counter has any integral value
When the user increases the counter
Then the value of the counter must be 1 greater than previous value (FAILED)
(java.lang.AssertionError)
```

## 5。测试 REST API

现在我们已经掌握了`JBhave`的基本；我们将看到如何用它来测试 REST API。我们的测试将基于我们之前讨论如何用 Java 测试 REST API 的文章。

在那篇文章中，我们测试了 [GitHub REST API](https://web.archive.org/web/20221128105745/https://docs.github.com/en/rest) ，主要关注 HTTP 响应代码、头部和有效负载。为了简单起见，我们可以把它们分别写成三个独立的故事。

### 5.1。测试状态代码

故事:

```
Scenario: when a user checks a non-existent user on github, github would respond 'not found'

Given github user profile api
And a random non-existent username
When I look for the random user via the api
Then github respond: 404 not found

When I look for eugenp1 via the api
Then github respond: 404 not found

When I look for eugenp2 via the api
Then github respond: 404 not found
```

步骤:

```
public class GithubUserNotFoundSteps {

    private String api;
    private String nonExistentUser;
    private int githubResponseCode;

    @Given("github user profile api")
    public void givenGithubUserProfileApi() {
        api = "https://api.github.com/users/%s";
    }

    @Given("a random non-existent username")
    public void givenANonexistentUsername() {
        nonExistentUser = randomAlphabetic(8);
    }

    @When("I look for the random user via the api")
    public void whenILookForTheUserViaTheApi() throws IOException {
        githubResponseCode = getGithubUserProfile(api, nonExistentUser)
          .getStatusLine()
          .getStatusCode();
    }

    @When("I look for $user via the api")
    public void whenILookForSomeNonExistentUserViaTheApi(
      String user) throws IOException {
        githubResponseCode = getGithubUserProfile(api, user)
          .getStatusLine()
          .getStatusCode();
    }

    @Then("github respond: 404 not found")
    public void thenGithubRespond404NotFound() {
        assertTrue(SC_NOT_FOUND == githubResponseCode);
    }

    //...
}
```

注意，在步骤实现中，**我们如何使用参数注入特性**。从候选步骤中提取的参数只是按照自然顺序与带注释的 Java 方法中的参数进行匹配。

此外，还支持带注释的命名参数:

```
@When("I look for $username via the api")
public void whenILookForSomeNonExistentUserViaTheApi(
  @Named("username") String user) throws IOException
```

### 5.2。测试媒体类型

这里有一个简单的 MIME 类型测试故事:

```
Scenario: when a user checks a valid user's profile on github, github would respond json data

Given github user profile api
And a valid username
When I look for the user via the api
Then github respond data of type json
```

以下是步骤:

```
public class GithubUserResponseMediaTypeSteps {

    private String api;
    private String validUser;
    private String mediaType;

    @Given("github user profile api")
    public void givenGithubUserProfileApi() {
        api = "https://api.github.com/users/%s";
    }

    @Given("a valid username")
    public void givenAValidUsername() {
        validUser = "eugenp";
    }

    @When("I look for the user via the api")
    public void whenILookForTheUserViaTheApi() throws IOException {
        mediaType = ContentType
          .getOrDefault(getGithubUserProfile(api, validUser).getEntity())
          .getMimeType();
    }

    @Then("github respond data of type json")
    public void thenGithubRespondDataOfTypeJson() {
        assertEquals("application/json", mediaType);
    }
}
```

### 5.3。测试 JSON 负载

然后是最后一个故事:

```
Scenario: when a user checks a valid user's profile on github, github's response json should include a login payload with the same username

Given github user profile api
When I look for eugenp via the api
Then github's response contains a 'login' payload same as eugenp
```

以及简单的直线步骤实现:

```
public class GithubUserResponsePayloadSteps {

    private String api;
    private GitHubUser resource;

    @Given("github user profile api")
    public void givenGithubUserProfileApi() {
        api = "https://api.github.com/users/%s";
    }

    @When("I look for $user via the api")
    public void whenILookForEugenpViaTheApi(String user) throws IOException {
        HttpResponse httpResponse = getGithubUserProfile(api, user);
        resource = RetrieveUtil.retrieveResourceFromResponse(httpResponse, GitHubUser.class);
    }

    @Then("github's response contains a 'login' payload same as $username")
    public void thenGithubsResponseContainsAloginPayloadSameAsEugenp(String username) {
        assertThat(username, Matchers.is(resource.getLogin()));
    }
}
```

## 6。总结

在本文中，我们简要介绍了 JBehave 并实现了 BDD 风格的 REST API 测试。

与我们普通的 Java 测试代码相比，用 JBehave 实现的代码看起来更加清晰直观，测试结果报告看起来更加优雅。

与往常一样，示例代码可以在 Github 项目中找到。