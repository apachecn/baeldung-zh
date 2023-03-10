# 用 Spring 和 Spock 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-spock-testing>

## 1.介绍

在这个简短的教程中，我们将展示结合 [Spring Boot](/web/20220628064519/https://www.baeldung.com/spring-boot-start) 的[测试框架](/web/20220628064519/https://www.baeldung.com/spring-boot-testing)的支持能力和[斯波克框架](/web/20220628064519/https://www.baeldung.com/groovy-spock)的表达能力的好处，无论是针对单元测试还是[集成测试](/web/20220628064519/https://www.baeldung.com/integration-testing-in-spring)。

## 2.项目设置

让我们从一个简单的 web 应用程序开始。它可以通过简单的 REST 调用来问候、更改问候并将其重置回默认值。除了主类，我们使用一个简单的`RestController`来提供功能:

```java
@RestController
@RequestMapping("/hello")
public class WebController {

    @GetMapping
    public String salutation() {
        return "Hello world!";
    }
}
```

所以控制器打招呼说“你好，世界！”。 [`@RestController`](/web/20220628064519/https://www.baeldung.com/spring-controller-vs-restcontroller) 注释和[快捷注释](/web/20220628064519/https://www.baeldung.com/spring-new-requestmapping-shortcuts)保证了剩余端点的注册。

## 3.斯波克和 Spring Boot 测试的相关性

我们从添加 Maven 依赖项和必要的 Maven 插件配置开始。

### 3.1.使用 Spring 支持添加 Spock 框架依赖项

对于斯波克本身和 T2 弹簧支架，我们需要两个依赖项:

```java
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-core</artifactId>
    <version>1.2-groovy-2.4</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-spring</artifactId>
    <version>1.2-groovy-2.4</version>
    <scope>test</scope>
</dependency> 
```

注意，用指定的版本是对所用 groovy 版本的引用。

### 3.2.添加 Spring Boot 测试

为了使用 [Spring Boot 测试](https://web.archive.org/web/20220628064519/https://search.maven.org/classic/#search%7Cga%7C1%7C%20(g%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-test%22))的测试工具，我们需要以下依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.5.0</version>
    <scope>test</scope>
</dependency>
```

### 3.3.设置 Groovy

由于 Spock 是基于 [Groovy](/web/20220628064519/https://www.baeldung.com/groovy-language) ，**的，我们必须添加和配置`gmavenplus`-插件以及**，以便能够在我们的测试中使用这种语言:

```java
<plugin>
    <groupId>org.codehaus.gmavenplus</groupId>
    <artifactId>gmavenplus-plugin</artifactId>
    <version>1.6</version>
    <executions>
        <execution>
            <goals>
                <goal>compileTests</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

请注意，因为我们只需要 Groovy 进行测试，所以我们将插件目标限制在`compileTest`。

## 4.在 Spock 测试中加载`ApplicationContext`

一个简单的测试是**检查 Spring 应用程序上下文中的所有 Beans 是否都已创建**:

```java
@SpringBootTest
class LoadContextTest extends Specification {

    @Autowired (required = false)
    private WebController webController

    def "when context is loaded then all expected beans are created"() {
        expect: "the WebController is created"
        webController
    }
}
```

对于这个集成测试，我们需要启动`ApplicationContext`，这就是`@SpringBootTest` 为我们做的。Spock 在我们的测试中提供了像“`when”`、“`then”`或“`expect”`”这样的关键字。

此外，我们可以利用[Groovy true](/web/20220628064519/https://www.baeldung.com/groovy-language)来检查 bean 是否为空，这是我们测试的最后一行。

## 5.在 Spock 测试中使用`WebMvcTest`

同样，我们可以测试`WebController`的行为:

```java
@AutoConfigureMockMvc
@WebMvcTest
class WebControllerTest extends Specification {

    @Autowired
    private MockMvc mvc

    def "when get is performed then the response has status 200 and content is 'Hello world!'"() {
        expect: "Status is 200 and the response is 'Hello world!'"
        mvc.perform(get("/hello"))
          .andExpect(status().isOk())
          .andReturn()
          .response
          .contentAsString == "Hello world!"
    }
}
```

值得注意的是，在我们的 Spock 测试中(或者更确切地说是`Specifications) ` **)，我们可以使用来自我们习惯的 [Spring Boot 测试框架](/web/20220628064519/https://www.baeldung.com/spring-boot-testing)的所有熟悉的注释。**

## 6。结论

在本文中，我们已经解释了如何建立一个 Maven 项目来结合使用 Spock 和 Spring Boot 测试框架。此外，我们已经看到了两个框架是如何完美地互补的。

为了更深入的了解，请看一下我们的教程，关于用 Spring Boot 测试的教程，关于 T2 的 Spock 框架的教程和关于 T4 的 Groovy 语言的教程。

最后，带有附加示例的源代码可以在我们的 [GitHub 库](https://web.archive.org/web/20220628064519/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)中找到。