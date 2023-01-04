# @ComponentScan 和@EnableAutoConfiguration 在 Spring Boot 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-componentscan-vs-enableautoconfiguration>

## 1.介绍

在这个快速教程中，我们将了解 Spring 框架中的 [`@ComponentScan`](/web/20220827110142/https://www.baeldung.com/spring-component-scanning) 和`@EnableAutoConfiguration` 注释之间的区别。

## 2.弹簧注释

注释使得在 Spring 中配置依赖注入变得更加容易。**不使用 XML 配置文件，我们可以在类和方法上使用 [Spring Bean](/web/20220827110142/https://www.baeldung.com/spring-bean-annotations) 注释来定义 Bean**。之后，Spring IoC 容器配置和管理 beans。

下面是我们将在本文中讨论的注释的概述:

*   `@ComponentScan` 扫描带注释的弹簧组件
*   `@EnableAutoConfiguration` 用于启用自动配置

现在让我们来看看这两个注释之间的区别。

## 3.它们有什么不同

**这些注释之间的主要区别是`@ComponentScan`扫描 Spring 组件，而`@EnableAutoConfiguration`用于自动配置出现在 [Spring Boot](/web/20220827110142/https://www.baeldung.com/spring-boot) 应用**的类路径中的 beans。

现在，让我们更详细地看一下它们。

### 3.1.`@ComponentScan`

在开发应用程序时，我们需要告诉 Spring 框架寻找 Spring 管理的组件。 **`@ComponentScan` 使 Spring 能够扫描配置、控制器、服务和我们定义的其他组件**。

特别是，`@ComponentScan` 注释与`@Configuration` 注释一起使用，指定 Spring 扫描组件的包:

```
@Configuration
@ComponentScan
public class EmployeeApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(EmployeeApplication.class, args);
        // ...
    }
}
```

**或者，Spring 也可以从指定的包开始扫描，我们可以使用`basePackageClasses()` 或`basePackages()` `.`** **来定义。如果没有指定包，那么它会将声明了`@ComponentScan`** 注释的类的包视为起始包 **:**

```
package com.baeldung.annotations.componentscanautoconfigure;

// ...

@Configuration
@ComponentScan(basePackages = {"com.baeldung.annotations.componentscanautoconfigure.healthcare",
  "com.baeldung.annotations.componentscanautoconfigure.employee"},
  basePackageClasses = Teacher.class)
public class EmployeeApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(EmployeeApplication.class, args);
        // ...
    }
}
```

在这个例子中，Spring 将扫描`healthcare`和`employee `包以及`Teacher` 类中的组件。

Spring 在指定的包及其所有子包中搜索用`@Configuration` `.` **注释的类。另外`,` `Configuration`类可以包含`@Bean` 注释，这些注释将方法注册为 Spring 应用程序上下文**中的 beans。此后@ `ComponentScan `注释可以自动检测这样的 beans:

```
@Configuration
public class Hospital {
    @Bean
    public Doctor getDoctor() {
        return new Doctor();
    }
}
```

**此外,@ `ComponentScan `注释还可以扫描、检测和注册用`@Component, @Controller, @Service`和`@Repository`** 注释的类的 beans。

例如，我们可以创建一个`Employee` 类作为可以被@ `ComponentScan `注释扫描的组件:

```
@Component("employee")
public class Employee {
    // ...
}
```

### 3.2.`@EnableAutoConfiguration`

**`@EnableAutoConfiguration` 注释使 Spring Boot 能够自动配置应用程序上下文**。**因此，它根据类路径中包含的 jar 文件和我们定义的 bean 自动创建和注册 bean。**

例如，当我们在类路径中定义 [`spring-boot-starter-web`](/web/20220827110142/https://www.baeldung.com/spring-boot-starters) 依赖时，Spring boot 会自动配置 [Tomcat](/web/20220827110142/https://www.baeldung.com/tomcat) 和 [Spring MVC](/web/20220827110142/https://www.baeldung.com/spring-mvc-tutorial) 。但是，在我们定义自己的配置时，这种自动配置的优先级较低。

**声明`@EnableAutoConfiguration`注释的类的包被认为是默认的**。因此，我们应该总是在根包中应用`@EnableAutoConfiguration`注释，以便可以检查每个子包和类:

```
@Configuration
@EnableAutoConfiguration
public class EmployeeApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(EmployeeApplication.class, args);
        // ...
    }
}
```

此外，`@EnableAutoConfiguration`注释提供了两个参数来手动排除任何参数:

我们可以使用`exclude`来禁用我们不想自动配置的类列表:

```
@Configuration
@EnableAutoConfiguration(exclude={JdbcTemplateAutoConfiguration.class})
public class EmployeeApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(EmployeeApplication.class, args);
        // ...
    }
}
```

我们可以使用`excludeName` 来定义我们想要从自动配置中排除的类名的完全限定列表:

```
@Configuration
@EnableAutoConfiguration(excludeName = {"org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration"})
public class EmployeeApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(EmployeeApplication.class, args);
        // ...
    }
}
```

从 Spring Boot 1.2.0 开始，我们可以使用 **`@SpringBootApplication `注释，它是三个注释`@Configuration, @EnableAutoConfiguration,` 和`@ComponentScan` 及其默认属性**的组合:

```
@SpringBootApplication
public class EmployeeApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(EmployeeApplication.class, args);
        // ...
    }
}
```

## 4.结论

在这篇文章中，我们了解了 Spring Boot 的`@ComponentScan` 和`@EnableAutoConfiguration`之间的区别。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations)