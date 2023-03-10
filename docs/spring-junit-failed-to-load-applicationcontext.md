# 无法为 Spring 控制器的 JUnit 测试加载应用程序上下文

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-junit-failed-to-load-applicationcontext>

## 1.概观

Spring Boot 应用程序中 beans 的**混合定义包括基于[注释的](/web/20220525130539/https://www.baeldung.com/spring-core-annotations)和基于 [XML 的](/web/20220525130539/https://www.baeldung.com/spring-boot-xml-beans)配置**。在这个环境中，**我们可能想要在测试类**中使用基于 XML 的配置。然而，有时在这种情况下，我们可能会遇到应用程序上下文加载错误"`**Failed to load ApplicationContext**.”`。这个错误出现在测试类中，因为应用程序上下文没有加载到测试上下文中。

在本教程中，我们将讨论如何将 XML 应用程序上下文集成到 Spring Boot 应用程序的测试中。

## 延伸阅读:

## [在 Spring Boot 测试](/web/20220525130539/https://www.baeldung.com/spring-boot-testing)

Learn about how the Spring Boot supports testing, to write unit tests efficiently.[Read more](/web/20220525130539/https://www.baeldung.com/spring-boot-testing) →

## [春季集成测试](/web/20220525130539/https://www.baeldung.com/integration-testing-in-spring)

A quick guide to writing integration tests for a Spring Web application.[Read more](/web/20220525130539/https://www.baeldung.com/integration-testing-in-spring) →

## [Spring Boot 错误应用上下文异常](/web/20220525130539/https://www.baeldung.com/spring-boot-application-context-exception)

Learn how to solve the ApplicationContextException in Spring Boot.[Read more](/web/20220525130539/https://www.baeldung.com/spring-boot-application-context-exception) →

## `2\. “Failed to load ApplicationContext” Error`

让我们通过在 Spring Boot 应用程序中集成基于 XML 的应用程序上下文来重现这个错误。

首先，假设我们有一个带有服务 bean 定义的`application-context.xml`文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
    http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="employeeServiceImpl" class="com.baeldung.xmlapplicationcontext.service.EmployeeServiceImpl" />
</beans> 
```

现在我们可以在`webapp/WEB-INF/`位置添加`application-context.xml`文件:

[![](img/c98506a065288256f3409e75ba5aad29.png)](/web/20220525130539/https://www.baeldung.com/wp-content/uploads/2022/01/ApplicationContextDirecory.png)

我们还将创建一个服务接口和类:

```java
public interface EmployeeService {
    Employee getEmployee();
}

public class EmployeeServiceImpl implements EmployeeService {

    @Override
    public Employee getEmployee() {
        return new Employee("Baeldung", "Admin");
    }
}
```

最后，我们将创建一个从应用程序上下文中获取`EmployeeService` bean 的测试用例:

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(locations={"classpath:WEB-INF/application-context.xml"})
public class EmployeeServiceAppContextIntegrationTest {

    @Autowired
    private EmployeeService service;

    @Test
    public void whenContextLoads_thenServiceISNotNull() {
        assertThat(service).isNotNull();
    }

}
```

现在，如果我们尝试运行这个测试，我们将观察到错误:

```java
java.lang.IllegalStateException: Failed to load ApplicationContext
```

这个错误出现在测试类中，因为应用程序上下文没有加载到测试上下文中。此外，**根本原因是`WEB-INF`没有包含在类路径**中:

```java
@ContextConfiguration(locations={"classpath:WEB-INF/application-context.xml"})
```

## 3.在测试中使用基于 XML 的`ApplicationContext`

让我们看看如何在测试类中使用基于 XML 的`ApplicationContext`。**我们有两个选项可以在测试**中使用基于 XML 的`ApplicationContext`:[`@SpringBootTest`](/web/20220525130539/https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)和 [`@ContextConfiguration`](https://web.archive.org/web/20220525130539/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/ContextConfiguration.html) 注释。

### 3.1.使用`@SpringBootTest`和`@ImportResource`进行测试

Spring Boot 为**提供了`@SpringBootTest`注释，我们可以用它来创建一个要在测试**中使用的应用程序上下文。另外，**我们必须使用 Spring Boot 主类中的`@ImportResource`来读取 XML bean**。这个注释允许我们导入一个或多个包含 bean 定义的资源。

首先，让我们用 [`@ImportResource`](/web/20220525130539/https://www.baeldung.com/spring-boot-xml-beans#the-importresource-annotation) 在主类中标注:

```java
@SpringBootApplication
@ImportResource({"classpath*:application-context.xml"})
```

现在让我们创建一个从应用程序上下文中获取`EmployeeService` bean 的测试用例:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = XmlBeanApplication.class)
public class EmployeeServiceAppContextIntegrationTest {

    @Autowired
    private EmployeeService service;

    @Test
    public void whenContextLoads_thenServiceISNotNull() {
        assertThat(service).isNotNull();
    }

}
```

`@ImportResource`注释加载位于`resource`目录中的 XML beans。此外，`@SpringBootTest`注释在测试类中加载整个应用程序的 beans。因此，我们能够访问测试类中的`EmployeeService` bean。

### 3.2.使用`@ContextConfiguration`和`resources`进行测试

**通过将我们的测试配置文件放在`src/test/resources`目录中，我们可以用 bean**的不同配置来创建我们的测试上下文。

在这种情况下，我们**使用`@ContextConfiguration`注释从`src/test/resources`目录**加载测试上下文。

首先，让我们从`EmployeeService`接口创建另一个 bean:

```java
public class EmployeeServiceTestImpl implements EmployeeService {

    @Override
    public Employee getEmployee() {
        return new Employee("Baeldung-Test", "Admin");
    }
} 
```

然后我们将在`src/test/resources`目录中创建`test-context.xml`文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="employeeServiceTestImpl" class="process.service.EmployeeServiceTestImpl" />
</beans> 
```

最后，我们将创建测试用例:

```java
@SpringBootTest
@ContextConfiguration(locations = "/test-context.xml")
public class EmployeeServiceTestContextIntegrationTest {

    @Autowired
    @Qualifier("employeeServiceTestImpl")
    private EmployeeService serviceTest;

    @Test
    public void whenTestContextLoads_thenServiceTestISNotNull() {
        assertThat(serviceTest).isNotNull();
    }

}
```

这里，我们使用`@ContextConfiguration`注释从`test-context.xml`加载了`employeeServiceTestImpl`。

### 3.3.使用`@ContextConfiguration`和`WEB-INF`进行测试

**我们还可以从`WEB-INF `目录**中导入测试类中的应用程序上下文。为此，我们可以使用应用程序的`file` URL 来寻址应用程序上下文:

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "file:src/main/webapp/WEB-INF/application-context.xml")
```

## 4.结论

在本文中，我们学习了如何在 Spring Boot 应用程序的测试类中使用基于 XML 的配置文件。和往常一样，本文中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220525130539/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing-2)