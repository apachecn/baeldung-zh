# Spring @Primary 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-primary>

## 1。概述

在这个快速教程中，我们将讨论 Spring 的`@Primary`注释，它是在框架的 3.0 版本中引入的。

简单来说，**当有多个同类型的 bean 时，我们使用`@Primary`赋予一个 bean 更高的优先级。**

让我们详细描述一下这个问题。

## 2。为什么需要`@Primary`？

在某些情况下，**我们需要注册多个相同类型的 bean**。

在这个例子中，我们有`Employee`类型的`JohnEmployee()`和`TonyEmployee()`bean:

```java
@Configuration
public class Config {

    @Bean
    public Employee JohnEmployee() {
        return new Employee("John");
    }

    @Bean
    public Employee TonyEmployee() {
        return new Employee("Tony");
    }
}
```

如果我们试图运行应用程序，Spring 抛出`NoUniqueBeanDefinitionException`。

为了访问相同类型的 beans，我们通常使用`@Qualifier(“beanName”)`注释。

我们将它与`@Autowired`一起应用于注射点。在我们的例子中，我们在配置阶段选择 beans，所以这里不能应用`@Qualifier` 。我们可以通过[链接](/web/20220827110142/https://www.baeldung.com/spring-autowire)了解更多关于`@Qualifier`的注释。

为了解决这个问题，Spring 提供了`@Primary`注释。

## 3。将`@Primary` 与`@Bean`和一起使用

让我们来看看配置类:

```java
@Configuration
public class Config {

    @Bean
    public Employee JohnEmployee() {
        return new Employee("John");
    }

    @Bean
    @Primary
    public Employee TonyEmployee() {
        return new Employee("Tony");
    }
}
```

**我们用`@Primary`标记`TonyEmployee()`豆。春天会优先注射`TonyEmployee()`豆而不是`JohnEmployee()`。**

现在，让我们启动应用程序上下文并从中获取`Employee` bean:

```java
AnnotationConfigApplicationContext context
  = new AnnotationConfigApplicationContext(Config.class);

Employee employee = context.getBean(Employee.class);
System.out.println(employee);
```

运行应用程序后:

```java
Employee{name='Tony'}
```

**从输出中，我们可以看到`TonyEmployee() `实例在自动连接**时有一个优先权。

## 4。将`@Primary`与`@Component`和一起使用

我们可以直接在 beans 上使用@Primary。让我们来看看下面的场景:

```java
public interface Manager {
    String getManagerName();
}
```

我们有一个`Manager`接口和两个子类 beans，`DepartmentManager`:

```java
@Component
public class DepartmentManager implements Manager {
    @Override
    public String getManagerName() {
        return "Department manager";
    }
}
```

还有`GeneralManager` 比恩:

```java
@Component
@Primary
public class GeneralManager implements Manager {
    @Override
    public String getManagerName() {
        return "General manager";
    }
}
```

它们都覆盖了接口`Manager`的`getManagerName()`。另外，请注意，我们用`@Primary`标记了`GeneralManager ` bean。

这次， **`@Primary`只有在我们启用组件扫描**时才有意义:

```java
@Configuration
@ComponentScan(basePackages="org.baeldung.primary")
public class Config {
}
```

让我们创建一个服务来使用依赖注入，同时找到正确的 bean:

```java
@Service
public class ManagerService {

    @Autowired
    private Manager manager;

    public Manager getManager() {
        return manager;
    }
}
```

这里，bean`DepartmentManager `和`GeneralManager`都符合自动连接的条件。

**由于我们用`@Primary`标记了`GeneralManager ` bean，它将被选中进行依赖注入**:

```java
ManagerService service = context.getBean(ManagerService.class);
Manager manager = service.getManager();
System.out.println(manager.getManagerName());
```

输出是"`General manager”.`

## 5。结论

在本文中，我们了解了 Spring 的`@Primary`注释。通过代码示例，我们展示了`@Primary.`的需求和用例

和往常一样，这篇文章的完整代码可以在 GitHub 项目的[中找到。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/spring-core-2)