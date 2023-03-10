# Spring Boot 注解

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-annotations>

[This article is part of a series:](javascript:void(0);)[• Spring Core Annotations](/web/20220625083101/https://www.baeldung.com/spring-core-annotations)
[• Spring Web Annotations](/web/20220625083101/https://www.baeldung.com/spring-mvc-annotations)
• Spring Boot Annotations (current article)[• Spring Scheduling Annotations](/web/20220625083101/https://www.baeldung.com/spring-scheduling-annotations)
[• Spring Data Annotations](/web/20220625083101/https://www.baeldung.com/spring-data-annotations)
[• Spring Bean Annotations](/web/20220625083101/https://www.baeldung.com/spring-bean-annotations)

## 1.概观

Spring Boot 通过其自动配置特性使 Spring 的配置变得更加容易。

在这个快速教程中，我们将探索来自`org.springframework.boot.autoconfigure`和`org.springframework.boot.autoconfigure.condition`包的注释。

## 2.`@SpringBootApplication`

我们使用这个注释来标记 Spring Boot 应用程序的主类:

```java
@SpringBootApplication
class VehicleFactoryApplication {

    public static void main(String[] args) {
        SpringApplication.run(VehicleFactoryApplication.class, args);
    }
}
```

`@SpringBootApplication`用默认属性封装**、`@EnableAutoConfiguration`和`@ComponentScan`、**标注。

## 3.`@EnableAutoConfiguration`

`@EnableAutoConfiguration`顾名思义，启用自动配置。这意味着 **Spring Boot 在其类路径中寻找自动配置 bean**，并自动应用它们。

注意，我们必须将这个注释与`@Configuration`一起使用:

```java
@Configuration
@EnableAutoConfiguration
class VehicleFactoryConfig {}
```

## 4.自动配置条件

通常，当我们编写我们的**定制自动配置**时，我们希望 Spring**有条件地使用它们**。我们可以通过本节中的注释来实现这一点。

我们可以将本节中的注释放在`@Configuration`类或`@Bean`方法上。

在接下来的部分中，我们将只介绍每个条件背后的基本概念。欲了解更多信息，请访问本文[。](/web/20220625083101/https://www.baeldung.com/spring-boot-custom-auto-configuration)

### 4.1.`@ConditionalOnClass`和`@ConditionalOnMissingClass`

使用这些条件，如果注释的**参数中的类存在/不存在**，Spring 将只使用标记的自动配置 bean:

```java
@Configuration
@ConditionalOnClass(DataSource.class)
class MySQLAutoconfiguration {
    //...
}
```

### 4.2.`@ConditionalOnBean`和`@ConditionalOnMissingBean`

当我们想要基于特定 bean 的**存在或不存在来定义条件时，我们可以使用这些注释:**

```java
@Bean
@ConditionalOnBean(name = "dataSource")
LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    // ...
}
```

### 4.3.`@ConditionalOnProperty`

有了这个注释，我们可以对属性的**值设置条件:**

```java
@Bean
@ConditionalOnProperty(
    name = "usemysql", 
    havingValue = "local"
)
DataSource dataSource() {
    // ...
}
```

### 4.4.`@ConditionalOnResource`

我们可以让 Spring 只在特定的**资源存在时使用定义**:

```java
@ConditionalOnResource(resources = "classpath:mysql.properties")
Properties additionalProperties() {
    // ...
}
```

### 4.5.`@ConditionalOnWebApplication`和`@ConditionalOnNotWebApplication`

有了这些注释，我们可以根据当前的**应用程序是否是 web 应用程序**来创建条件:

```java
@ConditionalOnWebApplication
HealthCheckController healthCheckController() {
    // ...
}
```

### 4.6.`@ConditionalExpression`

我们可以在更复杂的情况下使用这种注释。当 **SpEL 表达式被评估为 true** 时，Spring 将使用标记的定义:

```java
@Bean
@ConditionalOnExpression("${usemysql} && ${mysqlserver == 'local'}")
DataSource dataSource() {
    // ...
}
```

### 4.7.`@Conditional`

对于更复杂的条件，我们可以创建一个评估定制条件的类。我们告诉 Spring 使用这个带有`@Conditional`的自定义条件:

```java
@Conditional(HibernateCondition.class)
Properties additionalProperties() {
    //...
}
```

## 5.结论

在本文中，我们看到了如何微调自动配置过程并为定制自动配置 beans 提供条件的概述。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625083101/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations)

Next **»**[Spring Scheduling Annotations](/web/20220625083101/https://www.baeldung.com/spring-scheduling-annotations)**«** Previous[Spring Web Annotations](/web/20220625083101/https://www.baeldung.com/spring-mvc-annotations)