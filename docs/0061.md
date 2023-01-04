# 弹簧组件扫描

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-component-scanning>

## 1。概述

在本教程中，我们将讨论 Spring 中的组件扫描。在使用 Spring 时，我们可以注释我们的类，以便将它们变成 Spring beans。此外，**我们可以告诉 Spring 在哪里搜索这些带注释的类，**因为在这个特定的运行中，不是所有的类都必须成为 beans。

当然，有一些默认的组件扫描，但我们也可以自定义搜索的包。

首先，我们来看看默认设置。

## 延伸阅读:

## [春豆注解](/web/20221022100854/https://www.baeldung.com/spring-bean-annotations)

Learn how and when to use the standard Spring bean annotations - @Component, @Repository, @Service and @Controller.[Read more](/web/20221022100854/https://www.baeldung.com/spring-bean-annotations) →

## [Spring @ components can–过滤器类型](/web/20221022100854/https://www.baeldung.com/spring-componentscan-filter-type)

Explore different types of filter options available with the @ComponentScan annotation.[Read more](/web/20221022100854/https://www.baeldung.com/spring-componentscan-filter-type) →

## [使用 Spring Boot 创建自定义自动配置](/web/20221022100854/https://www.baeldung.com/spring-boot-custom-auto-configuration)

A quick, practical guide to creating a custom auto-configuration in Spring Boot.[Read more](/web/20221022100854/https://www.baeldung.com/spring-boot-custom-auto-configuration) →

## 2。`@ComponentScan`没有争论

### 2.1。在 Spring 应用程序中使用`@ComponentScan`

对于 Spring，**我们使用`@ComponentScan`注释和`@Configuration`注释来指定我们希望被扫描的包**。 `@ComponentScan`无参数告诉 Spring 扫描当前包及其所有的子包。

假设我们在`com.baeldung.componentscan.springapp`包中有以下`@Configuration`:

```
@Configuration
@ComponentScan
public class SpringComponentScanApp {
    private static ApplicationContext applicationContext;

    @Bean
    public ExampleBean exampleBean() {
        return new ExampleBean();
    }

    public static void main(String[] args) {
        applicationContext = 
          new AnnotationConfigApplicationContext(SpringComponentScanApp.class);

        for (String beanName : applicationContext.getBeanDefinitionNames()) {
            System.out.println(beanName);
        }
    }
}
```

此外，`com.baeldung.componentscan.springapp.animals`封装中还有`Cat`和`Dog`组件:

```
package com.baeldung.componentscan.springapp.animals;
// ...
@Component
public class Cat {}
```

```
package com.baeldung.componentscan.springapp.animals;
// ...
@Component
public class Dog {}
```

最后，我们在`com.baeldung.componentscan.springapp.flowers`包中有`Rose`组件:

```
package com.baeldung.componentscan.springapp.flowers;
// ...
@Component
public class Rose {}
```

`main()`方法的输出将包含`com.baeldung.componentscan.springapp`包及其子包的所有 beans:

```
springComponentScanApp
cat
dog
rose
exampleBean
```

请注意，主应用程序类也是一个 bean，因为它用作为`@Component`的`@Configuration,`进行了注释。

我们还应该注意，主应用程序类和配置类不一定是相同的。如果它们不同，那么我们将主应用程序类放在哪里并不重要。**只有配置类的位置有关系，因为默认情况下组件扫描从其包开始**。

最后，注意在我们的例子中，`@ComponentScan`相当于:

```
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp")
```

`basePackages`参数是要扫描的包或包的数组。

### 2.2。在 Spring Boot 应用程序中使用`@ComponentScan`

Spring Boot 的诀窍在于很多事情都是隐性发生的。我们使用`@SpringBootApplication`注释，但是它是三个注释的组合:

```
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

让我们在`com.baeldung.componentscan.springbootapp`包中创建一个类似的结构。这一次主要应用将是:

```
package com.baeldung.componentscan.springbootapp;
// ...
@SpringBootApplication
public class SpringBootComponentScanApp {
    private static ApplicationContext applicationContext;

    @Bean
    public ExampleBean exampleBean() {
        return new ExampleBean();
    }

    public static void main(String[] args) {
        applicationContext = SpringApplication.run(SpringBootComponentScanApp.class, args);
        checkBeansPresence(
          "cat", "dog", "rose", "exampleBean", "springBootComponentScanApp");

    }

    private static void checkBeansPresence(String... beans) {
        for (String beanName : beans) {
            System.out.println("Is " + beanName + " in ApplicationContext: " + 
              applicationContext.containsBean(beanName));
        }
    }
}
```

所有其他的包和类保持不变，我们只是将它们复制到附近的`com.baeldung.componentscan.springbootapp`包中。

Spring Boot 扫描包裹类似于我们的前一个例子。让我们检查输出:

```
Is cat in ApplicationContext: true
Is dog in ApplicationContext: true
Is rose in ApplicationContext: true
Is exampleBean in ApplicationContext: true
Is springBootComponentScanApp in ApplicationContext: true
```

在第二个例子中，我们只是检查 bean 是否存在(而不是打印出所有的 bean)，原因是输出会太大。

这是因为隐式的`@EnableAutoConfiguration`注释，它使得 Spring Boot 自动创建许多 beans，依赖于`pom.xml`文件中的依赖项。

## 3。`@ComponentScan`与争论

现在让我们自定义扫描路径。例如，假设我们想要排除`Rose` bean。

### 3.1.`@ComponentScan`针对特定包装

我们可以用几种不同的方法来做这件事。首先，我们可以更改基础包:

```
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals")
@Configuration
public class SpringComponentScanApp {
   // ...
}
```

现在输出将是:

```
springComponentScanApp
cat
dog
exampleBean
```

让我们看看这背后是什么:

*   创建`springComponentScanApp`是因为它是作为参数传递给`AnnotationConfigApplicationContext`的配置
*   `exampleBean`是在配置中配置的 bean
*   `cat`和`dog`在指定的`com.baeldung.componentscan.springapp.animals`包中

上面列出的所有定制也适用于 Spring Boot。我们可以将`@ComponentScan`和`@SpringBootApplication`一起使用，结果是一样的:

```
@SpringBootApplication
@ComponentScan(basePackages = "com.baeldung.componentscan.springbootapp.animals")
```

### 3.2.`@ComponentScan`带多个包装

Spring 提供了一种便捷的方式来指定多个包名。为此，我们需要使用一个字符串数组。

数组中的每个字符串表示一个包名:

```
@ComponentScan(basePackages = {"com.baeldung.componentscan.springapp.animals", "com.baeldung.componentscan.springapp.flowers"})
```

或者，从 spring 4.1.1 开始，**我们可以使用逗号、分号或空格来分隔包列表**:

```
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals;com.baeldung.componentscan.springapp.flowers")
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals,com.baeldung.componentscan.springapp.flowers")
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals com.baeldung.componentscan.springapp.flowers")
```

### 3.3.`@ComponentScan`有除外条款

另一种方法是使用过滤器，指定要排除的类的模式:

```
@ComponentScan(excludeFilters = 
  @ComponentScan.Filter(type=FilterType.REGEX,
    pattern="com\\.baeldung\\.componentscan\\.springapp\\.flowers\\..*"))
```

我们还可以选择不同的过滤器类型，因为注释支持几个灵活的选项来过滤扫描的类:

```
@ComponentScan(excludeFilters = 
  @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = Rose.class))
```

## 4。默认包

我们应该避免将`@Configuration`类[放在默认的包](https://web.archive.org/web/20221022100854/https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-structuring-your-code.html)中(也就是根本不指定包)。如果我们这样做，Spring 会扫描一个类路径中所有 jar 中的所有类，这会导致错误，应用程序可能无法启动。

## 5。结论

在本文中，我们了解了默认情况下 Spring 扫描哪些包，以及如何定制这些路径。

和往常一样，完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221022100854/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-di)