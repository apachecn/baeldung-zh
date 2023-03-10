# 在 Spring Boot 测试中排除自动配置类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-exclude-auto-configuration-test>

## 1。概述

在这个快速教程中，我们将讨论**如何从 Spring Boot 测试**中排除自动配置类。

Spring Boot 的自动配置功能非常方便，因为它为我们处理了很多设置工作。然而，如果我们不希望某个自动配置干扰我们对一个模块的测试，这也可能是测试期间的一个问题。

一个常见的例子是安全性自动配置，我们也将在我们的例子中使用它。

## 2。测试示例

首先，我们来看看我们的测试示例。

我们将拥有一个带有简单主页的安全 Spring Boot 应用程序。

当我们试图在没有身份验证的情况下访问主页时，得到的响应是“401 未授权”。

让我们在一个使用[放心](/web/20220630005345/https://www.baeldung.com/rest-assured-tutorial)进行呼叫的测试中看到这一点:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class, webEnvironment = WebEnvironment.DEFINED_PORT)
public class AutoConfigIntegrationTest {

    @Test
    public void givenNoAuthentication_whenAccessHome_thenUnauthorized() {
        int statusCode = RestAssured.get("http://localhost:8080/").statusCode();

        assertEquals(HttpStatus.UNAUTHORIZED.value(), statusCode);
    }

}
```

另一方面，我们可以通过身份验证成功访问主页:

```java
@Test
public void givenAuthentication_whenAccessHome_thenOK() {
    int statusCode = RestAssured.given().auth().basic("john", "123")
      .get("http://localhost:8080/")
      .statusCode();

    assertEquals(HttpStatus.OK.value(), statusCode);
}
```

在下面的章节中，**我们将尝试不同的方法从我们的**配置中排除`SecurityAutoConfiguration`类。

## 3。使用`@EnableAutoConfiguration`

从测试配置中排除特定的自动配置类有多种方法。

首先，**让我们看看如何使用`@EnableAutoConfiguration(exclude={CLASS_NAME})` 注释**:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class, webEnvironment = WebEnvironment.DEFINED_PORT)
@EnableAutoConfiguration(exclude=SecurityAutoConfiguration.class)
public class ExcludeAutoConfigIntegrationTest {

    @Test
    public void givenSecurityConfigExcluded_whenAccessHome_thenNoAuthenticationRequired() {
        int statusCode = RestAssured.get("http://localhost:8080/").statusCode();

        assertEquals(HttpStatus.OK.value(), statusCode);
    }
}
```

在这个例子中，我们使用`exclude`属性排除了`SecurityAutoConfiguration `类，但是我们可以对任何一个[自动配置类](https://web.archive.org/web/20220630005345/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-auto-configuration-classes.html)做同样的事情。

现在，我们可以运行我们的测试，在没有身份验证的情况下访问主页，它将不再失败。

## 4。使用`@TestPropertySource`

接下来，**我们可以用`@TestPropertySource`来注入属性`spring.autoconfigure.exclude`**:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class, webEnvironment = WebEnvironment.DEFINED_PORT)
@TestPropertySource(properties = 
 "spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration")
public class ExcludeAutoConfigIntegrationTest {
    // ...
}
```

注意，我们需要为属性指定完整的类名(包名+简单名)。

## 5。使用配置文件

**我们还可以使用概要文件:**为我们的测试设置属性“`spring.autoconfigure.exclude`

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class, webEnvironment = WebEnvironment.DEFINED_PORT)
@ActiveProfiles("test")
public class ExcludeAutoConfigIntegrationTest {
    // ...
}
```

并在`application-test.properties`中包含所有`test`配置文件的特定属性:

```java
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
```

## 6。使用自定义测试配置

最后，**我们可以为我们的测试使用单独的配置应用程序**:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = TestApplication.class, webEnvironment = WebEnvironment.DEFINED_PORT)
public class ExcludeAutoConfigIntegrationTest {
    // ...
}
```

并从`@SpringBootApplication(exclude={CLASS_NAME})`中排除自动配置类:

```java
@SpringBootApplication(exclude=SecurityAutoConfiguration.class)
public class TestApplication {

    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}
```

## 7。结论

在本文中，我们探索了从 Spring Boot 测试中排除自动配置类的不同方法。

完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630005345/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)