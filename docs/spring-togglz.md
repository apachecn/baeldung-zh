# Spring Boot 和多哥方面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-togglz>

## 1。概述

在本教程中，我们将看看如何在 Spring Boot 应用程序中使用`Togglz`库。

## 2。Togglz

[`Togglz`库](https://web.archive.org/web/20221208143832/https://www.togglz.org/)提供了`Feature Toggles`设计模式的实现。这种模式指的是拥有一种机制，允许在应用程序运行时基于切换来确定某个特性是否被启用。

在运行时禁用某个功能在各种情况下都很有用，例如处理尚未完成的新功能，希望只允许部分用户访问某个功能，或者运行 A/B 测试。

在接下来的小节中，我们将创建一个方面，它拦截带有提供特性名称的注释的方法，并根据特性是否被启用来确定是否继续执行这些方法。

## 3。Maven 依赖关系

除了 Spring Boot 依赖项，`Togglz`库还提供了一个 Spring Boot 启动 jar:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>

<dependency>
    <groupId>org.togglz</groupId>
    <artifactId>togglz-spring-boot-starter</artifactId>
    <version>2.4.1</version>
<dependency>
    <groupId>org.togglz</groupId>
    <artifactId>togglz-spring-security</artifactId>
    <version>2.4.1</version>
</dependency>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-test</artifactId> 
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.194</version>
</dependency>
```

最新版本的[tog glz-spring-boot-starter](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Ctogglz%20spring%20boot%20starter)、 [togglz-spring-security](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Ctogglz%20spring%20security) 、 [spring-boot-starter-web](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-web%22) 、[spring-boot-starter-data-JPA](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-jpa%22)、 [spring-boot-starter-test](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-test%22) 、 [h2](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22) 可从 Maven Central 下载。

## 4。Togglz 配置

`togglz-spring-boot-starter`库包含创建必要 beans(如`FeatureManager`)的自动配置。我们唯一需要提供的是`featureProvider`豆。

首先，让我们创建一个枚举来实现`Feature`接口，并包含一个特性名称列表:

```java
public enum MyFeatures implements Feature {

    @Label("Employee Management Feature")
    EMPLOYEE_MANAGEMENT_FEATURE;

    public boolean isActive() {
        return FeatureContext.getFeatureManager().isActive(this);
    }
}
```

枚举还定义了一个名为`isActive()`的方法，用于验证某个特性是否被启用。

然后我们可以在 Spring Boot 配置类中定义一个类型为`EnumBasedFeatureProvider`的 bean:

```java
@Configuration
public class ToggleConfiguration {

    @Bean
    public FeatureProvider featureProvider() {
        return new EnumBasedFeatureProvider(MyFeatures.class);
    }
}
```

## 5。创建方面

接下来，我们将创建一个方面，它截取一个定制的`AssociatedFeature`注释，并检查注释参数中提供的特性，以确定它是否是活动的:

```java
@Aspect
@Component
public class FeaturesAspect {

    private static final Logger LOG = Logger.getLogger(FeaturesAspect.class);

    @Around(
      "@within(featureAssociation) || @annotation(featureAssociation)"
    )
    public Object checkAspect(ProceedingJoinPoint joinPoint, 
      FeatureAssociation featureAssociation) throws Throwable {

        if (featureAssociation.value().isActive()) {
            return joinPoint.proceed();
        } else {
            LOG.info(
              "Feature " + featureAssociation.value().name() + " is not enabled!");
            return null;
        }
    }
}
```

让我们定义一个名为`FeatureAssociation`的定制注释，它将有一个`MyFeatures`枚举类型的`value()`参数:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.METHOD, ElementType.TYPE })
public @interface FeatureAssociation {
    MyFeatures value();
}
```

如果特征是活动的，方面将继续执行方法；否则，它将记录一条消息，而不运行方法代码。

## 6。功能激活

`Togglz`中的功能可以是活动的，也可以是不活动的。这个行为由一个`enabled`标志和一个可选的激活策略控制。

要将`enabled`标志设置为 true，我们可以在枚举值定义上使用`@EnabledByDefault`注释。

`Togglz`库还提供了多种激活策略，可以根据某种条件来决定某个特性是否启用。

在我们的示例中，让我们将`SystemPropertyActivationStrategy` 用于 EMPLOYEE_MANAGEMENT_FEATURE，它根据系统属性的值来评估特性的状态。可以使用`@ActivationParameter` 注释指定所需的属性名和值:

```java
public enum MyFeatures implements Feature {

    @Label("Employee Management Feature") 
    @EnabledByDefault 
    @DefaultActivationStrategy(id = SystemPropertyActivationStrategy.ID, 
      parameters = { 
      @ActivationParameter(
        name = SystemPropertyActivationStrategy.PARAM_PROPERTY_NAME,
        value = "employee.feature"),
      @ActivationParameter(
        name = SystemPropertyActivationStrategy.PARAM_PROPERTY_VALUE,
        value = "true") }) 
    EMPLOYEE_MANAGEMENT_FEATURE;
    //...
}
```

我们已经将我们的特性设置为只有当`employee.feature`属性的值为`true`时才启用。

`Togglz`库提供的其他类型的激活策略有:

*   `UsernameActivationStrategy` –允许该功能对指定的用户列表有效
*   UserRoleActivationStrategy–当前用户的角色用于确定功能的状态
*   `ReleaseDateActivationStrategy`–在特定日期和时间自动激活功能
*   `GradualActivationStrategy`–为特定比例的用户启用一项功能
*   `ScriptEngineActivationStrategy`–允许使用由 JVM 的`ScriptEngine`支持的语言编写的自定义脚本来确定某个特性是否处于活动状态
*   `ServerIpActivationStrategy`–根据服务器的 IP 地址启用一项功能

## 7。测试方面

### 7.1。应用示例

为了查看我们的方面的运行情况，让我们创建一个简单的例子，它包含一个管理组织雇员的特性。

随着这个特性的开发，我们可以添加用值为 EMPLOYEE_MANAGEMENT_FEATURE 的`@AssociatedFeature`注释注释的方法和类。这确保了它们仅在功能处于活动状态时才可访问。

首先，让我们基于 Spring 数据定义一个`Employee`实体类和存储库:

```java
@Entity
public class Employee {

    @Id
    private long id;
    private double salary;

    // standard constructor, getters, setters
}
```

```java
public interface EmployeeRepository
  extends CrudRepository<Employee, Long>{ }
```

接下来，让我们添加一个带有增加员工工资的方法的`EmployeeService`。我们将用参数`EMPLOYEE_MANAGEMENT_FEATURE`将`@AssociatedFeature`注释添加到方法中:

```java
@Service
public class SalaryService {

    @Autowired
    EmployeeRepository employeeRepository;

    @FeatureAssociation(value = MyFeatures.EMPLOYEE_MANAGEMENT_FEATURE)
    public void increaseSalary(long id) {
        Employee employee = employeeRepository.findById(id).orElse(null);
        employee.setSalary(employee.getSalary() + 
          employee.getSalary() * 0.1);
        employeeRepository.save(employee);
    }
} 
```

该方法将从一个`/increaseSalary`端点调用，我们将调用该端点进行测试:

```java
@Controller
public class SalaryController {

    @Autowired
    SalaryService salaryService;

    @PostMapping("/increaseSalary")
    @ResponseBody
    public void increaseSalary(@RequestParam long id) {
        salaryService.increaseSalary(id);
    }
}
```

### 7.2。JUnit 测试

首先，让我们添加一个测试，在将`employee.feature`属性设置为`false`之后，我们调用我们的 POST 映射。在这种情况下，该功能不应处于活动状态，员工的工资值也不应改变:

```java
@Test
public void givenFeaturePropertyFalse_whenIncreaseSalary_thenNoIncrease() 
  throws Exception {
    Employee emp = new Employee(1, 2000);
    employeeRepository.save(emp);

    System.setProperty("employee.feature", "false");

    mockMvc.perform(post("/increaseSalary")
      .param("id", emp.getId() + ""))
      .andExpect(status().is(200));

    emp = employeeRepository.findOne(1L);
    assertEquals("salary incorrect", 2000, emp.getSalary(), 0.5);
}
```

接下来，让我们添加一个测试，在将属性设置为`true`后执行调用。在这种情况下，工资的价值应该增加:

```java
@Test
public void givenFeaturePropertyTrue_whenIncreaseSalary_thenIncrease() 
  throws Exception {
    Employee emp = new Employee(1, 2000);
    employeeRepository.save(emp);
    System.setProperty("employee.feature", "true");

    mockMvc.perform(post("/increaseSalary")
      .param("id", emp.getId() + ""))
      .andExpect(status().is(200));

    emp = employeeRepository.findById(1L).orElse(null);
    assertEquals("salary incorrect", 2200, emp.getSalary(), 0.5);
}
```

## 8。结论

在本教程中，我们已经展示了如何通过使用一个方面将`Togglz`库与 Spring Boot 集成。

这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries)