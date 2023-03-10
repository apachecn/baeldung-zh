# 找不到带有@DataJpaTest 的@SpringBootConfiguration

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-unable-to-find-springbootconfiguration-with-datajpatest>

## 1。简介

在我们在 Spring Boot 的[测试教程中，我们看到了如何使用`@DataJpaTest`注释。](/web/20221206033108/https://www.baeldung.com/spring-boot-testing)

在接下来的教程中，我们将看到**如何解决“找不到`@SpringBootConfiguration`”****的错误。**

## 2。原因

`@DataJpaTest`注释帮助我们建立一个 JPA 测试。为此，它初始化应用程序，忽略不相关的部分。例如，它会忽略 MVC 控制器。

然而，要初始化应用程序，它需要配置。

为此，它在当前包中搜索，并在包层次结构中向上搜索，直到找到一个配置。

例如，让我们在`com.baeldung.data.jpa` 包`.`中添加一个`@DataJpaTest`，然后，它将在:

*   `com.baeldung.data.jpa`
*   `com.baeldung.data`
*   等等

但是，如果找不到任何配置，应用程序将报告一个错误:

```java
Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=...)
  with your test java.lang.IllegalStateException
```

例如，这可能是因为配置类位于更具体的包中，比如`com.baeldung.data.jpa.application`。

**让我们把配置类移到`com.baeldung.data.jpa.` 这样，Spring 现在就能找到它了。**

另一方面，我们可以拥有一个没有任何`@SpringBootConfiguration`的模块。在下一节中，我们将研究这个场景。

## 3。`@SpringBootConfiguration` 失踪

如果我们的模块不包含任何`@SpringBootConfiguration?` 会怎样？这可能有多种原因。对于本教程，让我们假设我们有一个只包含模型类的模块。

因此，解决方案很简单。让我们给测试代码添加一个`@SpringBootApplication`:

```java
@SpringBootApplication
public class TestApplication {}
```

既然我们有了一个带注释的类，Spring 就能够引导我们的测试了。

为了验证我们的设置，让我们注入一个`TestEntityManager`并验证它已经设置好了:

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class DataJpaUnitTest {

    @Autowired
    TestEntityManager entityManager;

    @Test
    public void givenACorrectSetup_thenAnEntityManagerWillBeAvailable() {
        assertNotNull(entityManager);
    }
}
```

当 **Spring 可以在它自己的包或者它的父包**中找到`@SpringBootConfiguration`时，这个测试就成功了。

## 4。结论

在这个简短的教程中，我们研究了错误的两个不同原因:“找不到`@SpringBootConfiguration`”。

首先，我们看了一个找不到配置类的情况。这是因为它的位置。我们通过将配置类移动到另一个位置解决了这个问题。

其次，我们看了一个没有可用配置类的场景。我们通过在测试代码库中添加一个`@SpringBootApplication`来解决这个问题。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221206033108/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-config-jpa-error)