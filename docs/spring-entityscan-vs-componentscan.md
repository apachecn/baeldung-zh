# Spring @EntityScan 与@ComponentScan

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-entityscan-vs-componentscan>

## 1.介绍

当编写我们的 Spring 应用程序时，我们可能需要指定包含我们的实体类的特定包列表。类似地，在某些时候，我们只需要初始化一个特定的 Spring beans 列表。这就是我们可以利用 [`@EntityScan`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/domain/EntityScan.html) 或者 [`@ComponentScan`](/web/20220827110142/https://www.baeldung.com/spring-component-scanning) 注解的地方。

为了澄清我们在这里使用的术语，组件是带有[`@Controller`](/web/20220827110142/https://www.baeldung.com/spring-controller-vs-restcontroller) [`@Service` `@Repository``@Component`](/web/20220827110142/https://www.baeldung.com/spring-component-repository-service)[`@Bean,`等的类。注释](/web/20220827110142/https://www.baeldung.com/spring-bean-annotations)。实体是用 [`@Entity`](/web/20220827110142/https://www.baeldung.com/jpa-entities) 标注的类。

在这个简短的教程中，我们将讨论`@EntityScan`和`@ComponentScan`在 Spring 中的用法，解释它们的用途，然后指出它们的区别。

## 2.`@EntityScan`注解

当编写我们的 Spring 应用程序时，我们通常会有实体类——那些用`@Entity`注释的类。我们可以考虑两种放置实体类的方法:

*   在应用程序主包或其子包下
*   使用完全不同的根包

在第一个场景中，我们可以使用`@EnableAutoConfiguration`让 Spring 自动配置应用程序上下文。

在第二个场景中，我们将为应用程序提供在哪里可以找到这些包的信息。为此，我们将使用`@EntityScan.`

当实体类没有放在主应用程序包或其子包中时，使用注释。在这种情况下，我们将在`@EntityScan`注释中的主配置类中声明包或包列表。这将告诉 Spring 在哪里可以找到我们的应用程序中使用的实体:

```
@Configuration
@EntityScan("com.baeldung.demopackage")
public class EntityScanDemo {
    // ...
}
```

**我们应该知道，使用`@EntityScan`将禁用实体的 Spring Boot 自动配置扫描。**

## 3.`@ComponentScan A`注释

类似于`@EntityScan`和实体，如果我们希望 Spring 只使用一组特定的 bean 类，我们将使用`@ComponentScan`注释。**它将指向我们希望 Spring 初始化的 bean 类的具体位置**。

该注释可以带参数使用，也可以不带参数使用。如果没有参数，Spring 将扫描当前包及其子包，而当参数化时，它将告诉 Spring 确切地在哪里搜索包。

关于参数，我们可以提供一个要扫描的包的列表(使用`basePackages` 参数)，或者我们可以指定它们所属的包也将被扫描的特定类(使用`basePackageClasses`参数)。

让我们看一个@ComponentScan 注释用法的例子:

```
@Configuration
@ComponentScan(
  basePackages = {"com.baeldung.demopackage"}, 
  basePackageClasses = DemoBean.class)
public class ComponentScanExample {
    // ...
}
```

## 4.`@EntityScan`对`@ComponentScan`

最后，我们可以说这两种注释的目的完全不同。

它们的相似之处在于，它们都有助于我们的 Spring 应用程序配置。应该指定我们要扫描哪些包中的实体类。另一方面，`@ComponentScan`是在指定应该扫描哪些包中的 Spring beans 时的一个选择。

## 5.结论

在这个简短的教程中，我们讨论了`@EntityScan`和`@ComponentScan`注释的用法，并指出了它们的区别。