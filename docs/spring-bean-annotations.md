# Spring Bean 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-bean-annotations>

[This article is part of a series:](javascript:void(0);)[• Spring Core Annotations](/web/20221001115718/https://www.baeldung.com/spring-core-annotations)
[• Spring Web Annotations](/web/20221001115718/https://www.baeldung.com/spring-mvc-annotations)
[• Spring Boot Annotations](/web/20221001115718/https://www.baeldung.com/spring-boot-annotations)
[• Spring Scheduling Annotations](/web/20221001115718/https://www.baeldung.com/spring-scheduling-annotations)
[• Spring Data Annotations](/web/20221001115718/https://www.baeldung.com/spring-data-annotations)
• Spring Bean Annotations (current article)

## 1。概述

在本教程中，我们将讨论最常见的用于定义不同类型 bean 的 Spring bean 注释。

有几种方法可以在 Spring 容器中配置 beans。首先，我们可以使用 XML 配置来声明它们。我们还可以在配置类中使用`@Bean`注释来声明 beans。

最后，我们可以用`org.springframework.stereotype`包中的一个注释来标记这个类，剩下的交给组件扫描。

## 2。组件扫描

如果启用了组件扫描，Spring 可以自动扫描包中的 beans。

`@ComponentScan`配置哪些**包扫描带有注释配置**的类。我们可以用`basePackages`或`value`参数之一来直接指定基础包名(`value`是`basePackages`的别名):

```java
@Configuration
@ComponentScan(basePackages = "com.baeldung.annotations")
class VehicleFactoryConfig {}
```

同样，我们可以用`basePackageClasses`参数指向基础包中的类:

```java
@Configuration
@ComponentScan(basePackageClasses = VehicleFactoryConfig.class)
class VehicleFactoryConfig {}
```

两个参数都是数组，所以我们可以为每个参数提供多个包。

如果没有指定参数，扫描将从存在`@ComponentScan`注释类的同一个包中进行。

`@ComponentScan`利用 Java 8 的重复注释特性，这意味着我们可以用它多次标记一个类:

```java
@Configuration
@ComponentScan(basePackages = "com.baeldung.annotations")
@ComponentScan(basePackageClasses = VehicleFactoryConfig.class)
class VehicleFactoryConfig {}
```

或者，我们可以使用`@ComponentScans`来指定多个`@ComponentScan`配置:

```java
@Configuration
@ComponentScans({ 
  @ComponentScan(basePackages = "com.baeldung.annotations"), 
  @ComponentScan(basePackageClasses = VehicleFactoryConfig.class)
})
class VehicleFactoryConfig {}
```

当**使用 XML 配置**时，配置组件扫描同样简单:

```java
<context:component-scan base-package="com.baeldung" />
```

## 3。`@Component`

`@Component`是类级注释。在组件扫描期间， **Spring Framework 自动检测用`@Component:`** 标注的类

```java
@Component
class CarUtility {
    // ...
}
```

默认情况下，该类的 bean 实例与类名同名，首字母小写。此外，我们可以使用这个注释的可选参数`value`指定一个不同的名称。

由于`@Repository`、`@Service`、`@Configuration`和`@Controller`都是`@Component`的元注释，它们共享相同的 bean 命名行为。Spring 还会在组件扫描过程中自动拾取它们。

## 4。`@Repository`

DAO 或 Repository 类通常代表应用程序中的数据库访问层，应该用`@Repository:`进行注释

```java
@Repository
class VehicleRepository {
    // ...
}
```

使用这个注释的一个优点是**它启用了自动持久性异常翻译**。当使用一个持久性框架时，比如 Hibernate，在用`@Repository`注释的类中抛出的本地异常将被自动翻译成 Spring 的`DataAccessExeption`的子类。

**为了启用异常转换**，我们需要声明我们自己的`PersistenceExceptionTranslationPostProcessor` bean:

```java
@Bean
public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
    return new PersistenceExceptionTranslationPostProcessor();
}
```

注意，在大多数情况下，Spring 会自动执行上述步骤。

或者通过 XML 配置:

```java
<bean class=
  "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>
```

## 5。`@Service`

应用程序的**业务逻辑**通常驻留在服务层中，因此我们将使用`@Service`注释来表示一个类属于该层:

```java
@Service
public class VehicleService {
    // ...    
}
```

## 6。`@Controller`

*@Controller* 是一个类级别的注释，它告诉 Spring 框架这个类在 Spring MVC 中作为一个**控制器:**

```java
@Controller
public class VehicleController {
    // ...
}
```

## 7.`@Configuration`

`Configuration`类可以**包含用`@Bean`标注的 bean 定义方法**:

```java
@Configuration
class VehicleFactoryConfig {

    @Bean
    Engine engine() {
        return new Engine();
    }

}
```

## 8。原型注释和 AOP

当我们使用 Spring 原型注释时，很容易创建一个切入点，这个切入点针对所有具有特定原型的类。

例如，假设我们想从 DAO 层测量方法的执行时间。我们将利用`@Repository`原型创建以下方面(使用 AspectJ 注释):

```java
@Aspect
@Component
public class PerformanceAspect {
    @Pointcut("within(@org.springframework.stereotype.Repository *)")
    public void repositoryClassMethods() {};

    @Around("repositoryClassMethods()")
    public Object measureMethodExecutionTime(ProceedingJoinPoint joinPoint) 
      throws Throwable {
        long start = System.nanoTime();
        Object returnValue = joinPoint.proceed();
        long end = System.nanoTime();
        String methodName = joinPoint.getSignature().getName();
        System.out.println(
          "Execution of " + methodName + " took " + 
          TimeUnit.NANOSECONDS.toMillis(end - start) + " ms");
        return returnValue;
    }
}
```

在这个例子中，我们创建了一个切入点，它匹配用`@Repository`注释的类中的所有方法。然后我们使用`@Around`通知来定位那个切入点，并确定被拦截的方法调用的执行时间。

此外，使用这种方法，我们可以向每个应用层添加日志记录、性能管理、审计和其他行为。

## 9。结论

在本文中，我们研究了 Spring 原型注释，并讨论了它们各自代表的语义类型。

我们还学习了如何使用组件扫描来告诉容器在哪里可以找到带注释的类。

最后，我们了解了这些注释**如何导致一个干净的、分层的设计，**以及应用程序关注点之间的分离。它们还使配置变得更小，因为我们不再需要手动显式定义 beans。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221001115718/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations)

**«** Previous[Spring Data Annotations](/web/20221001115718/https://www.baeldung.com/spring-data-annotations)