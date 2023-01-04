# Spring 5 中的 SpringJUnitConfig 和 SpringJUnitWebConfig 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-5-junit-config>

## 1。简介

在这篇简短的文章中，我们将看看 Spring 5 中新的`@SpringJUnitConfig`和`@SpringJUnitWebConfig`注释。

这些注释是 JUnit 5 和 Spring 5 注释的组合,使得测试创建更加容易和快速。

## 2。`@SpringJUnitConfig`

`@SpringJUnitConfig`结合了这两种注释:

*   从 JUnit 5 的**T0**运行与`SpringExtension`类的测试
*   **`@ContextConfiguration`从弹簧测试**加载弹簧上下文

让我们创建一个测试，并在实践中使用这个注释:

```
@SpringJUnitConfig(SpringJUnitConfigIntegrationTest.Config.class)
public class SpringJUnitConfigIntegrationTest {

    @Configuration
    static class Config {}
}
```

**注意，与`@ContextConfiguration`不同，配置类是使用`value`属性声明的。**然而，资源位置应该用`locations`属性来指定。

我们现在可以验证 Spring 上下文是否真的被加载了:

```
@Autowired
private ApplicationContext applicationContext;

@Test
void givenAppContext_WhenInjected_ThenItShouldNotBeNull() {
    assertNotNull(applicationContext);
}
```

最后，这里我们有`@SpringJUnitConfig(SpringJUnitConfigTest.Config.class):`的等价代码

```
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = SpringJUnitConfigTest.Config.class)
```

## 3。`@SpringJUnitWebConfig`

`@SpringJUnitWebConfig` **结合`@SpringJUnitConfig`的相同注释加上弹簧测试**的`@WebAppConfiguration`——加载`WebApplicationContext`。

让我们看看这个注释是如何工作的:

```
@SpringJUnitWebConfig(SpringJUnitWebConfigIntegrationTest.Config.class)
public class SpringJUnitWebConfigIntegrationTest {

    @Configuration
    static class Config {
    }
}
```

像`@SpringJUnitConfig`、**一样，配置类放在`value`属性**中，任何资源都使用`locations`属性指定。

另外，现在应该使用`resourcePath`属性来指定`@WebAppConfiguration`的`value`属性。**默认情况下，该属性设置为`“src/main/webapp”`。**

现在让我们验证一下`WebApplicationContext`是否真的加载了:

```
@Autowired
private WebApplicationContext webAppContext;

@Test
void givenWebAppContext_WhenInjected_ThenItShouldNotBeNull() {
    assertNotNull(webAppContext);
}
```

同样，这里我们有不使用`@SpringJUnitWebConfig`的等价代码:

```
@ExtendWith(SpringExtension.class)
@WebAppConfiguration
@ContextConfiguration(classes = SpringJUnitWebConfigIntegrationTest.Config.class)
```

## 4。结论

在这个简短的教程中，我们展示了如何在 Spring 5 中使用新引入的`@SpringJUnitConfig`和`@SpringJUnitWebConfig`注释。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220926191254/https://github.com/eugenp/tutorials/tree/master/spring-5)