# 零代码简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/zerocode-intro>

## 1.概观

在本文中，我们将介绍 [ZeroCode](https://web.archive.org/web/20221128051938/https://github.com/authorjapps/zerocode) 自动化测试框架。我们将通过一个 REST API 测试的例子来学习基础知识。

## 2.方法

ZeroCode 框架采用以下方法:

*   多方面的测试支持
*   测试的声明式风格

两个都讨论一下吧。

### 2.1.多方面的测试支持

该框架旨在支持我们的应用程序的多个方面的自动化测试。除其他外，它让我们能够测试:

*   休息
*   肥皂
*   安全性
*   负载/压力
*   数据库ˌ资料库
*   阿帕奇卡夫卡
*   GraphQL
*   开放 API 规范

测试是通过框架的 DSL 来完成的，我们将很快讨论它。

### 2.2.声明式风格

ZeroCode 使用声明式测试，这意味着我们不必编写实际的测试代码。我们在 JSON/YAML 文件中声明场景，框架会在幕后将它们“翻译”成测试代码。这有助于我们**专注于我们想要测试的东西，而不是如何测试它**。

## 3.设置

让我们在`pom.xml`文件中添加 Maven 依赖项:

```java
 <dependency>
      <groupId>org.jsmart</groupId>
      <artifactId>zerocode-tdd</artifactId>
      <version>1.3.27</version>
      <scope>test</scope>
 </dependency>
```

最新版本可在 Maven Central 上[获得。我们也可以用格雷尔。如果我们使用 IntelliJ，我们可以从](https://web.archive.org/web/20221128051938/https://search.maven.org/artifact/org.jsmart/zerocode-tdd) [Jetbrains Marketplace](https://web.archive.org/web/20221128051938/https://plugins.jetbrains.com/marketplace) 下载 ZeroCode 插件。

## 4.REST API 测试

正如我们上面所说的，ZeroCode 可以支持测试我们应用程序的多个部分。在本文中，我们将关注 REST API 测试。因此，我们将创建一个小型的 Spring Boot web 应用程序，并公开一个端点:

```java
@PostMapping
public ResponseEntity create(@RequestBody User user) {
    if (!StringUtils.hasText(user.getFirstName())) {
        return new ResponseEntity("firstName can't be empty!", HttpStatus.BAD_REQUEST);
    }
    if (!StringUtils.hasText(user.getLastName())) {
        return new ResponseEntity("lastName can't be empty!", HttpStatus.BAD_REQUEST);
    }
    user.setId(UUID.randomUUID().toString());
    users.add(user);
    return new ResponseEntity(user, HttpStatus.CREATED);
} 
```

让我们看看控制器中引用的`User`类:

```java
public class User {
    private String id;
    private String firstName;
    private String lastName;

    // standard getters and setters
}
```

当我们创建一个用户时，我们设置一个惟一的 id 并将整个`User`对象返回给客户端。在`/api/users`路径上可以到达端点。我们将在内存中保存用户以保持简单。

## 5.编写场景

这个场景在 ZeroCode 中起着核心作用。它由一个或多个步骤组成，这些步骤是我们想要测试的实际内容。让我们用一个步骤编写一个场景，测试用户创建的成功路径:

```java
{
  "scenarioName": "test user creation endpoint",
  "steps": [
    {
      "name": "test_successful_creation",
      "url": "/api/users",
      "method": "POST",
      "request": {
        "body": {
          "firstName": "John",
          "lastName": "Doe"
        }
      },
      "verify": {
        "status": 201,
        "body": {
          "id": "$NOT.NULL",
          "firstName": "John",
          "lastName": "Doe"
        }
      }
    }
  ]
}
```

让我们解释一下每个属性代表什么:

*   `scenarioName`–这是场景的名称；我们可以用任何我们想要的名字
*   一组 JSON 对象，在这里我们描述了我们想要测试的内容
    *   `name`–我们给步骤起的名字
    *   `url`–端点的相对 URL 我们也可以放一个绝对的 URL，但是一般来说，这不是一个好主意
    *   `method`–HTTP 方法
    *   `request`–HTTP 请求
        *   `body`–HTTP 请求正文
    *   `verify`–在这里，我们验证/断言服务器返回的响应
        *   `status`–HTTP 响应状态代码
        *   `body` (在验证属性内)–HTTP 响应体

在这一步中，我们检查用户创建是否成功。我们检查服务器返回的`firstName`和`lastName`值。同样，我们用 ZeroCode 的断言标记验证 id 不是`null`。

通常，我们在场景中有不止一个步骤。让我们在场景的`steps`数组中添加另一个步骤:

```java
{
  "name": "test_firstname_validation",
  "url": "/api/users",
  "method": "POST",
  "request": {
    "body": {
      "firstName": "",
      "lastName": "Doe"
    }
  },
  "verify": {
    "status": 400,
    "rawBody": "firstName can't be empty!"
  }
}
```

在这一步中，我们提供了一个空的名字，这将导致一个错误的请求。这里，响应体不是 JSON 格式的，所以我们使用`rawbody` 属性将其作为普通字符串。

ZeroCode 不能直接运行场景——为此，我们需要一个相应的测试用例。

## 6.编写测试用例

为了执行我们的场景，让我们编写一个相应的测试用例:

```java
@RunWith(ZeroCodeUnitRunner.class)
@TargetEnv("rest_api.properties")
public class UserEndpointIT {

    @Test
    @Scenario("rest/user_create_test.json")
    public void test_user_creation_endpoint() {
    }
}
```

这里，我们声明了一个方法，并使用 JUnit 4 的`@Test`注释将其标记为测试。我们可以使用 JUnit 5 和 ZeroCode 进行负载测试。

我们还使用来自 ZeroCode 框架的`@Scenario `注释来指定场景的位置。方法体为空。正如我们所说的，我们不写实际的测试代码。在我们的场景中描述了我们想要测试的内容。我们只是在测试用例方法中引用这个场景。我们的`UserEndpointIT`类有两个注释:

*   这里，我们指定哪个零代码类负责运行我们的场景
*   `@TargetEnv`–这指向我们的场景运行时将使用的属性文件

当我们在上面声明`url` 属性时，我们指定了相对路径。显然，ZeroCode 框架需要一个绝对路径，所以我们创建了一个`rest_api.properties`文件，其中包含一些定义测试应该如何运行的属性:

*   `web.application.endpoint.host`–REST API 的宿主；在我们的例子中，它是`http://localhost`
*   `web.application.endpoint.port`–应用服务器的端口，我们的 REST API 暴露在这里；在我们的例子中，它是 8080
*   `web.application.endpoint.context`–API 的上下文；在我们的例子中，它是空的

我们在属性文件中声明的属性取决于我们正在进行的测试类型。例如，如果我们想要测试一个 [Kafka](https://web.archive.org/web/20221128051938/https://gitlab.com/zerocodeio/samurai-user-manual/-/wikis/home) 生产者/消费者，我们将拥有不同的属性。

## 7.执行测试

我们已经创建了一个场景、属性文件和测试用例。现在，我们准备运行我们的测试。由于 ZeroCode 是一个集成测试工具，我们可以利用 Maven 的`failsafe` 插件:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>3.0.0-M5</version>
    <dependencies>
        <dependency>
            <groupId>org.apache.maven.surefire</groupId>
            <artifactId>surefire-junit47</artifactId>
            <version>3.0.0-M5</version>
        </dependency>
    </dependencies>
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

要运行测试，我们可以使用以下命令:

```java
mvn verify -Dskip.it=false
```

 ZeroCode 创建了多种类型的日志，我们可以在`${project.basedir}/target`文件夹中查看。

## 8.结论

在本文中，我们看了一下零代码自动化测试框架。我们用 REST API 测试的例子展示了该框架是如何工作的。我们还了解到，ZeroCode DSL 消除了编写实际测试代码的需要。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221128051938/https://github.com/eugenp/tutorials/tree/master/testing-modules/zerocode)