# 春季日程安排注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-scheduling-annotations>

[This article is part of a series:](javascript:void(0);)[• Spring Core Annotations](/web/20220525012031/https://www.baeldung.com/spring-core-annotations)
[• Spring Web Annotations](/web/20220525012031/https://www.baeldung.com/spring-mvc-annotations)
[• Spring Boot Annotations](/web/20220525012031/https://www.baeldung.com/spring-boot-annotations)
• Spring Scheduling Annotations (current article)[• Spring Data Annotations](/web/20220525012031/https://www.baeldung.com/spring-data-annotations)
[• Spring Bean Annotations](/web/20220525012031/https://www.baeldung.com/spring-bean-annotations)

## 1.概观

当单线程执行不够时，我们可以使用`org.springframework.scheduling.annotation`包中的注释。

在这个快速教程中，我们将探索 Spring Scheduling 注释。

## 2.`@EnableAsync`

有了这个注释，我们可以在 Spring 中启用异步功能。

我们必须和`@Configuration`一起使用:

```java
@Configuration
@EnableAsync
class VehicleFactoryConfig {}
```

现在，我们已经启用了异步调用，我们可以使用`@Async`来定义支持它的方法。

## 3.`@EnableScheduling`

有了这个注释，我们可以在应用程序中启用调度。

我们还必须将它与`@Configuration`结合使用:

```java
@Configuration
@EnableScheduling
class VehicleFactoryConfig {}
```

因此，我们现在可以用`@Scheduled`定期运行方法。

## 4.`@Async`

我们可以定义我们希望**在不同的线程**上执行的方法，从而异步运行它们。

为了实现这一点，我们可以用`@Async`来注释该方法:

```java
@Async
void repairCar() {
    // ...
}
```

如果我们将这个注释应用于一个类，那么所有的方法都将被异步调用。

注意，我们需要使用`@EnableAsync`或 XML 配置来启用这个注释的异步调用。

更多关于`@Async`的信息可以在[这篇文章](/web/20220525012031/https://www.baeldung.com/spring-async)中找到。

## 5.`@Scheduled`

如果我们需要一个方法来使**周期性地执行**，我们可以使用这个注释:

```java
@Scheduled(fixedRate = 10000)
void checkVehicle() {
    // ...
}
```

我们可以用它在固定的时间间隔内执行一个方法，或者我们可以用类似 T2 的表达式对它进行微调。

`@Scheduled`利用 Java 8 的重复注释特性，这意味着我们可以用它多次标记一个方法:

```java
@Scheduled(fixedRate = 10000)
@Scheduled(cron = "0 * * * * MON-FRI")
void checkVehicle() {
    // ...
}
```

注意，用`@Scheduled`标注的方法应该有一个`void`返回类型。

此外，我们必须为这个注释启用调度，例如使用`@EnableScheduling`或 XML 配置。

关于日程安排的更多信息，请阅读这篇文章。

## 6.`@Schedules`

我们可以使用这个注释来指定多个`@Scheduled`规则:

```java
@Schedules({ 
  @Scheduled(fixedRate = 10000), 
  @Scheduled(cron = "0 * * * * MON-FRI")
})
void checkVehicle() {
    // ...
}
```

注意，从 Java 8 开始，我们可以通过上述的重复注释特性实现同样的功能。

## 7.结论

在本文中，我们看到了最常见的 Spring scheduling 注释的概述。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220525012031/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations)

Next **»**[Spring Data Annotations](/web/20220525012031/https://www.baeldung.com/spring-data-annotations)**«** Previous[Spring Boot Annotations](/web/20220525012031/https://www.baeldung.com/spring-boot-annotations)