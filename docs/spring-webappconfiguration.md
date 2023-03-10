# Spring 测试中的 WebAppConfiguration

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webappconfiguration>

## 1。概述

在本文中，我们将探索 Spring 中的`@WebAppConfiguration`注释，为什么我们在集成测试中需要它，以及我们如何配置它，以便这些测试实际上引导一个`WebApplicationContext`。

## 2。`@WebAppConfiguration`

简单地说，这是一个类级注释，用于在 Spring 框架中创建应用程序上下文的 web 版本。

它用来表示为测试而引导的`ApplicationContext`应该是`WebApplicationContext`的一个实例。

关于用法的一个简短说明——我们通常会在集成测试中找到这个注释，因为`WebApplicationContext`用于构建一个`MockMvc`对象。你可以在这里找到更多关于 Spring [集成测试的信息。](/web/20220120163324/https://www.baeldung.com/integration-testing-in-spring)

## 3。`WebApplicationContext`载入一个

从 Spring 3.2 开始，现在支持在集成测试 : 中加载一个`WebApplicationContext`

```java
@WebAppConfiguration
@ContextConfiguration(classes = WebConfig.class)
public class EmployeeControllerTest {
    ...
} 
```

这指示`TestContext`框架应该为测试加载一个`WebApplicationContext`。

并且，在后台，`TestContext` 框架会创建一个`MockServletContext`并提供给我们测试的`WebApplicationContext` 。

### 3.1。配置选项

默认情况下，`WebApplicationContext`的基本资源路径将被设置为`“file:src/main/webapp”,` ，这是 Maven 项目中 WAR 根的默认位置。

然而，我们可以通过简单地提供一个到`@WebAppConfiguration`注释的替代路径来覆盖它:

```java
@WebAppConfiguration("src/test/webapp")
```

我们还可以从类路径而不是文件系统引用基本资源路径:

```java
@WebAppConfiguration("classpath:test-web-resources")
```

### 3.2。缓存

一旦`WebApplicationContext`被加载，它将被缓存，并在相同的测试套件中声明相同的惟一上下文配置的所有后续测试中重用。

关于缓存的更多细节，您可以参考参考手册的[上下文缓存](https://web.archive.org/web/20220120163324/https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-ctx-management-caching)部分。

## 4。在测试中使用`@WebAppConfiguration`

既然我们理解了为什么我们需要在我们的测试类中添加`@WebAppConfiguration` 注释，让我们看看**如果我们在使用`WebApplicationContext`的时候错过了添加**会发生什么。

```java
@RunWith(SpringJUnit4ClassRunner.class)
// @WebAppConfiguration omitted on purpose
@ContextConfiguration(classes = WebConfig.class)
public class EmployeeTest {

    @Autowired
    private WebApplicationContext webAppContext;
    private MockMvc mockMvc;

    @Before
    public void setup() {
        MockitoAnnotations.initMocks(this);
        mockMvc = MockMvcBuilders.webAppContextSetup(webAppContext).build();
    }

    ...
}
```

请注意，我们注释掉了注释，以模拟我们忘记添加注释的场景。这里很容易看出为什么我们运行 JUnit 测试时测试会失败:**我们试图在一个没有设置`.`** 的类中自动连接`WebApplicationContext`

然而，一个更典型的例子是使用支持 web 的 Spring 配置的测试；这实际上足以使测试中断。

让我们来看看:

```java
@RunWith(SpringJUnit4ClassRunner.class)
// @WebAppConfiguration omitted on purpose
@ContextConfiguration(classes = WebConfig.class)
public class EmployeeTestWithoutMockMvc {

    @Autowired
    private EmployeeController employeeController;

    ...
}
```

即使上面的例子没有自动连接一个`WebApplicationContext` ，它仍然会失败，因为它试图使用一个支持 web 的配置—`WebConfig`:

```java
@Configuration
@EnableWebMvc
@ComponentScan("com.baeldung.web")
public class WebConfig implements WebMvcConfigurer {
    ...
}
```

注释`@EnableWebMvc`是这里的罪魁祸首——这基本上需要一个支持 web 的 Spring 上下文，没有它——我们会看到测试失败:

```java
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
  No qualifying bean of type [javax.servlet.ServletContext] found for dependency: 
    expected at least 1 bean which qualifies as autowire candidate for this dependency. 

Dependency annotations: 
  {@org.springframework.beans.factory.annotation.Autowired(required=true)}
    at o.s.b.f.s.DefaultListableBeanFactory
      .raiseNoSuchBeanDefinitionException(DefaultListableBeanFactory.java:1373)
    at o.s.b.f.s.DefaultListableBeanFactory
      .doResolveDependency(DefaultListableBeanFactory.java:1119)
    at o.s.b.f.s.DefaultListableBeanFactory
      .resolveDependency(DefaultListableBeanFactory.java:1014)
    at o.s.b.f.a.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement
      .inject(AutowiredAnnotationBeanPostProcessor.java:545)
    ... 43 more
```

这就是我们通过在测试中添加`@WebAppConfiguration`注释可以轻松解决的问题。

## 5。结论

在本文中，我们展示了如何让`TestContext`框架通过添加注释将`WebApplicationContext`加载到我们的集成测试中。

最后，我们看了一些例子，即使我们将@ `ContextConfiguration`添加到测试中，除非我们添加了`@WebAppConfiguration`注释，否则这将无法工作。

本文中示例的实现可以在我们的资源库 [GitHub](https://web.archive.org/web/20220120163324/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java/src/test/java/com/baeldung/web/controller) 中找到。