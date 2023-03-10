# Spring Boot 项目的推荐包装结构

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-package-structure>

## 1.概观

当建立一个新的 Spring Boot 项目时，在如何组织我们的课程上有高度的灵活性。

尽管如此，我们还是需要记住一些建议。

## 2.没有默认包

鉴于像`@ComponentScan`、`@EntityScan, @ConfigurationPropertiesScan` 和`@SpringBootApplication`这样的 Spring Boot 注释使用包来定义扫描位置，建议我们避免使用默认的包——也就是说，**我们应该总是在我们的类**中声明这个包。

## 3.主类

`@SpringBootApplication`注释触发对当前包及其子包的组件扫描。因此，一个可靠的方法是让项目的主类驻留在基础包中。

这是可配置的，我们仍然可以通过手动指定基础包来定位它。然而，在大多数情况下，这种选择当然更简单。

甚至，一个基于 JPA 的项目需要在主类上有一些额外的注释:

```java
@SpringBootApplication(scanBasePackages = "example.baeldung.com")
@EnableJpaRepositories("example.baeldung.com")
@EntityScan("example.baeldung.com")
```

另外，请注意可能需要额外的配置。

## 4.设计

包装结构的设计独立于 Spring Boot。所以应该是我们项目的要求强加的。

**一个流行的策略是基于特性的包**，它增强了模块性，并支持子包内部的包私有可见性。

让我们以[宠物诊所](https://web.archive.org/web/20220908085902/https://github.com/spring-projects/spring-petclinic)项目为例。这个项目是由 Spring 开发人员构建的，目的是阐明他们对一个普通的 Spring Boot 项目应该如何构建的观点。

它是以按特性分组的方式组织的。因此，我们有主包、`org.springframework.samples.petclinic`和 5 个子包:

*   `org.springframework.samples.petclinic.**model**`
*   `org.springframework.samples.petclinic.**owner**`
*   `org.springframework.samples.petclinic.**system**`
*   `org.springframework.samples.petclinic.**vet**`
*   `org.springframework.samples.petclinic.**visit**`

它们中的每一个都代表了应用程序的一个领域或一个特性，**将高度耦合的类组合在一起，并实现高内聚**。

## 5.结论

在这篇小文章中，我们看了一些在构建 Spring Boot 项目时需要记住的建议——并了解了如何设计包结构。