# Spring 应用程序上下文

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-application-context>

## 1.概观

在本教程中，我们将详细探索 Spring `ApplicationContext`接口。

## 延伸阅读:

## [春季数据 JPA @Query](/web/20221001115718/https://www.baeldung.com/spring-data-jpa-query)

Learn how to use the @Query annotation in Spring Data JPA to define custom queries using JPQL and native SQL.[Read more](/web/20221001115718/https://www.baeldung.com/spring-data-jpa-query) →

## [Spring Boot 错误应用上下文异常](/web/20221001115718/https://www.baeldung.com/spring-boot-application-context-exception)

Learn how to solve the ApplicationContextException in Spring Boot.[Read more](/web/20221001115718/https://www.baeldung.com/spring-boot-application-context-exception) →

## [加载 Spring 控制器的 JUnit 测试应用上下文失败](/web/20221001115718/https://www.baeldung.com/spring-junit-failed-to-load-applicationcontext)

Learn about the "Failed to Load ApplicationContext" error message when running Junit tests with the Spring Controller, and how to fix it.[Read more](/web/20221001115718/https://www.baeldung.com/spring-junit-failed-to-load-applicationcontext) →

## 2.`ApplicationContext`界面

Spring 框架的主要特性之一是 IoC(控制反转)容器。Spring IoC 容器负责管理应用程序的对象。它使用依赖注入来实现控制反转。

接口`[BeanFactory](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)` 和`[ApplicationContext](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)` **代表 Spring IoC 容器**。这里，`BeanFactory`是访问 Spring 容器的根接口。它提供了管理 beans 的基本功能。

另一方面，`ApplicationContext`是`BeanFactory`的子接口。因此，它提供了`BeanFactory.`的所有功能

此外，它**提供了** **更多的企业特定功能**。`ApplicationContext`的重要特性是**解析消息、支持国际化、发布事件和应用层特定上下文**。这就是为什么我们把它作为默认的 Spring 容器。

## 3.什么是春豆？

在我们深入研究`ApplicationContext`容器之前，了解一下 Spring beans 是很重要的。在 Spring 中，一个 [bean](/web/20221001115718/https://www.baeldung.com/spring-bean) 就是**一个 Spring 容器实例化、组装和管理**的对象。

那么，我们应该将应用程序的所有对象配置为 Spring beans 吗？作为最佳实践，我们不应该这样做。

按照 Spring 文档的说法，一般来说，我们应该为服务层对象、数据访问对象(Dao)、表示对象、基础设施对象(比如 Hibernate `SessionFactories,` JMS 队列)等等定义 beans。

此外，通常，我们不应该在容器中配置细粒度的域对象。创建和加载域对象通常是 Dao 和业务逻辑的责任。

现在让我们定义一个简单的 Java 类，我们将在本教程中将其用作 Spring bean:

```
public class AccountService {

  @Autowired
  private AccountRepository accountRepository;

  // getters and setters
}
```

## 4.在容器中配置 Beans

我们知道，`ApplicationContext`的主要工作是管理 beans。

因此，应用程序必须向`ApplicationContext`容器提供 bean 配置。Spring bean 配置由一个或多个 bean 定义组成。此外，Spring 支持不同的配置 beans 的方式。

### 4.1.基于 Java 的配置

首先，我们将从基于 Java 的配置开始，因为它是 bean 配置的最新和最受欢迎的方式。从 Spring 3.0 开始就可以使用了。

Java 配置通常在一个`@Configuration`类中使用 **`@Bean`注释的方法。方法上的`@Bean`注释表明该方法创建了一个 Spring bean。此外，用`@Configuration`标注的类表明它包含 Spring bean 配置。**

现在让我们创建一个配置类，将我们的`AccountService`类定义为一个 Spring bean:

```
@Configuration
public class AccountConfig {

  @Bean
  public AccountService accountService() {
    return new AccountService(accountRepository());
  }

  @Bean
  public AccountRepository accountRepository() {
    return new AccountRepository();
  }
}
```

### 4.2.基于注释的配置

Spring 2.5 引入了基于注释的配置，作为在 Java 中启用 bean 配置的第一步。

在这种方法中，我们首先通过 XML 配置启用基于注释的配置。然后，我们在 Java 类、方法、构造函数或字段上使用一组注释来配置 beans。这些注释的一些例子是`@Component`、`@Controller`、`@Service`、`@Repository`、`@Autowired`和`@Qualifier`。

值得注意的是，我们也在基于 Java 的配置中使用这些注释。同样值得一提的是，Spring 在每个版本中都不断为这些注释添加更多的功能。

现在让我们来看一个简单的配置示例。

首先，我们将创建 XML 配置`user-bean-config.xml`，以启用注释:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

  <context:annotation-config/>
  <context:component-scan base-package="com.baeldung.applicationcontext"/>

</beans>
```

在这里，**`annotation-config`标签支持基于注释的映射**。`component-scan`标签还告诉 Spring 在哪里寻找带注释的类。

其次，我们将创建`UserService`类，并使用`@Component`注释将其定义为一个 Spring bean:

```
@Component
public class UserService {
  // user service code
}
```

然后我们将编写一个简单的测试用例来测试这个配置:

```
ApplicationContext context = new ClassPathXmlApplicationContext("applicationcontext/user-bean-config.xml");
UserService userService = context.getBean(UserService.class);
assertNotNull(userService);
```

### 4.3.基于 XML 的配置

最后，让我们看看基于 XML 的配置。这是 Spring 配置 beans 的传统方式。

显然，在这种方法中，我们在 XML 配置文件中完成所有的 **bean 映射。**

因此，让我们创建一个 XML 配置文件`account-bean-config.xml`，并为我们的`AccountService`类定义 beans:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="
    http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="accountService" class="com.baeldung.applicationcontext.AccountService">
    <constructor-arg name="accountRepository" ref="accountRepository" />
  </bean>

  <bean id="accountRepository" class="com.baeldung.applicationcontext.AccountRepository" />
</beans>
```

## 5.`ApplicationContext`的类型

Spring 提供了适合不同需求的不同类型的`ApplicationContext`容器。这些是`ApplicationContext`接口的实现。那么让我们来看看一些常见的`ApplicationContext`类型。

### 5.1.`AnnotationConfigApplicationContext`

首先，我们来看看 Spring 3.0 中引入的 [`AnnotationConfigApplicationContext`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html) 类。它可以将注释有`@Configuration`、、`**@Component**,`和 JSR-330 元数据的**类作为输入。**

因此，让我们看一个简单的例子，将`AnnotationConfigApplicationContext`容器用于我们基于 Java 的配置:

```
ApplicationContext context = new AnnotationConfigApplicationContext(AccountConfig.class);
AccountService accountService = context.getBean(AccountService.class);
```

### 5.2.`AnnotationConfigWebApplicationContext`

[`**AnnotationConfigWebApplicationContext**`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/support/AnnotationConfigWebApplicationContext.html) **是`AnnotationConfigApplicationContext`的网络变种**。

当我们在一个`web.xml`文件中配置 Spring 的`ContextLoaderListener` servlet 监听器或 Spring MVC `DispatcherServlet`时，我们可能会用到这个类。

此外，从 Spring 3.0 开始，我们还可以通过编程来配置这个应用程序上下文容器。我们需要做的就是实现 [`WebApplicationInitializer`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/WebApplicationInitializer.html) 接口:

```
public class MyWebApplicationInitializer implements WebApplicationInitializer {

  public void onStartup(ServletContext container) throws ServletException {
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
    context.register(AccountConfig.class);
    context.setServletContext(container);

    // servlet configuration
  }
}
```

### 5.3.`XmlWebApplicationContext`

如果我们在 web 应用程序中使用基于 **XML 的配置，我们可以使用 [`XmlWebApplicationContext`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/context/support/XmlWebApplicationContext.html) 类。**

事实上，配置这个容器就像配置`AnnotationConfigWebApplicationContext`类一样，这意味着我们可以在`web.xml,`中配置它或者实现`WebApplicationInitializer` 接口:

```
public class MyXmlWebApplicationInitializer implements WebApplicationInitializer {

  public void onStartup(ServletContext container) throws ServletException {
    XmlWebApplicationContext context = new XmlWebApplicationContext();
    context.setConfigLocation("/WEB-INF/spring/applicationContext.xml");
    context.setServletContext(container);

    // Servlet configuration
  }
}
```

### 5.4.`FileSystemXMLApplicationContext`

我们使用 [`FileSystemXMLApplicationContext`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html) 类**从文件系统**或从 URL 加载一个基于 XML 的 Spring 配置文件。当我们需要以编程方式加载`ApplicationContext`时，这个类很有用。一般来说，测试工具和独立的应用程序是一些可能的用例。

例如，让我们看看如何创建这个 Spring 容器，并为基于 XML 的配置加载 beans:

```
String path = "C:/myProject/src/main/resources/applicationcontext/account-bean-config.xml";

ApplicationContext context = new FileSystemXmlApplicationContext(path);
AccountService accountService = context.getBean("accountService", AccountService.class);
```

### 5.5.`ClassPathXmlApplicationContext`

如果我们想从类路径中**加载一个 XML 配置文件，我们可以使用 [`ClassPathXmlApplicationContext`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) 类。类似于`FileSystemXMLApplicationContext,`,它对于测试工具以及嵌入在 jar 中的应用程序上下文都很有用。**

让我们看一个使用这个类的例子:

```
ApplicationContext context = new ClassPathXmlApplicationContext("applicationcontext/account-bean-config.xml");
AccountService accountService = context.getBean("accountService", AccountService.class);
```

## 6.`ApplicationContext`的附加功能

### 6.1.消息解析

`ApplicationContext`接口**通过扩展`MessageSource`接口**支持消息解析和国际化**。此外，Spring 提供了两个`MessageSource`实现， [`ResourceBundleMessageSource`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/ResourceBundleMessageSource.html) 和 [`StaticMessageSource`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/StaticMessageSource.html) 。**

我们可以使用`StaticMessageSource`以编程方式向源添加消息；但是，它支持基本的国际化，更适合测试而不是生产使用。

另一方面， **`ResourceBundleMessageSource`是`MessageSource`** 最常见的实现。它依靠底层 JDK 的 [`ResouceBundle`和](https://web.archive.org/web/20221001115718/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ResourceBundle.html)实现。它还使用了由 [`MessageFormat`](https://web.archive.org/web/20221001115718/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/MessageFormat.html) 提供的 JDK 标准消息解析。

现在让我们看看如何使用`MessageSource`从属性文件中读取消息。

首先，我们将在类路径上创建`messages.properties`文件:

```
account.name=TestAccount
```

其次，我们将在我们的`AccountConfig`类中添加一个 bean 定义:

```
@Bean
public MessageSource messageSource() {
  ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
  messageSource.setBasename("config/messages");
  return messageSource;
}
```

第三，我们将在`AccountService`中注入`MessageSource`:

```
@Autowired
private MessageSource messageSource;
```

最后，我们可以在`AccountService`中的任何地方使用`getMessage`方法来读取消息:

```
messageSource.getMessage("account.name", null, Locale.ENGLISH);
```

Spring 还提供了 [`ReloadableResourceBundleMessageSource`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/ReloadableResourceBundleMessageSource.html) 类，允许从任何 Spring 资源位置读取文件，并支持 bundle 属性文件的热重载。

### 6.2.事件处理

`ApplicationContext`借助`ApplicationEvent`类和`ApplicationListener`接口支持事件处理**。支持`ContextStartedEvent`、`ContextStoppedEvent`、`ContextClosedEvent`、`RequestHandledEvent`等[内置事件](/web/20221001115718/https://www.baeldung.com/spring-context-events)。此外，它还支持业务用例的[自定义事件](/web/20221001115718/https://www.baeldung.com/spring-events)。**

## 7.结论

在本文中，我们讨论了 Spring 中的`ApplicationContext`容器的各个方面。我们还探索了如何在`AppicationContext`中配置 Spring beans 的不同例子。最后，我们学习了如何创建和使用不同类型的`ApplicationContext`。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221001115718/https://github.com/eugenp/tutorials/tree/master/spring-core-4)