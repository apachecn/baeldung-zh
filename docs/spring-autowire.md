# Spring @Autowired 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-autowire>

## 1.概观

从 Spring 2.5 开始，框架引入了注释驱动的`Dependency Injection`。这个特性的主要注解是`@Autowired` `.` **，它允许 Spring 解析协作 bean 并将其注入我们的 bean。**

## 延伸阅读:

## [弹簧组件扫描](/web/20220701005438/https://www.baeldung.com/spring-component-scanning)

Learn about the mechanism behind Spring component scanning, and how you can tweak it to your own needs[Read more](/web/20220701005438/https://www.baeldung.com/spring-component-scanning) →

## [Spring 控制反转和依赖注入简介](/web/20220701005438/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)

A quick introduction to the concepts of Inversion of Control and Dependency Injection, followed by a simple demonstration using the Spring Framework[Read more](/web/20220701005438/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) →

在本教程中，我们将首先看看如何启用自动连接以及自动连接 beans 的各种*方式。之后，我们将讨论使用`@Qualifier`注释、的**解决 bean 冲突，以及潜在的异常场景。***

 *## 2.启用`@Autowired`注释

Spring 框架支持自动依赖注入。换句话说，**通过在一个 Spring 配置文件中声明所有的 bean 依赖，Spring container 可以自动连接协作 bean 之间的关系**。这叫做 **`Spring bean autowiring`** 。

为了在我们的应用程序中使用基于 Java 的配置，让我们启用注释驱动注入来加载我们的 Spring 配置:

```java
@Configuration
@ComponentScan("com.baeldung.autowire.sample")
public class AppConfig {}
```

或者， [`<context:annotation-config>`注释](/web/20220701005438/https://www.baeldung.com/spring-contextannotation-contextcomponentscan#:~:text=The%20%3Ccontext%3Aannotation%2Dconfig,annotation%2Dconfig%3E%20can%20resolve.)主要用于激活 Spring XML 文件中的依赖注入注释。

再者， **Spring Boot 引入了 [`@SpringBootApplication`](/web/20220701005438/https://www.baeldung.com/spring-boot-annotations#spring-boot-application) 注解**。这个单独的注释相当于使用了`@Configuration`、`@EnableAutoConfiguration`和`@ComponentScan`。

让我们在应用程序的主类中使用这个注释:

```java
@SpringBootApplication
class VehicleFactoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(VehicleFactoryApplication.class, args);
    }
}
```

因此，当我们运行这个 Spring Boot 应用程序时，**会自动扫描当前包及其子包**中的组件。因此，它将在 Spring 的应用程序上下文中注册它们，并允许我们使用`@Autowired`注入 beans。

## 3.使用`@Autowired`

启用注释注入后，**我们可以在属性、设置器和构造器上使用自动连接**。

### 3.1。`@Autowired`论属性

让我们看看如何使用`@Autowired`注释一个属性。这消除了对 getters 和 setters 的需要。

首先，让我们定义一个`fooFormatter` bean:

```java
@Component("fooFormatter")
public class FooFormatter {
    public String format() {
        return "foo";
    }
}
```

然后，我们将使用字段定义上的`@Autowired`将这个 bean 注入到`FooService` bean 中:

```java
@Component
public class FooService {  
    @Autowired
    private FooFormatter fooFormatter;
}
```

因此，当`FooService`被创建时，Spring 注入`fooFormatter`。

### 3.2。`@Autowired`论二传手

现在让我们尝试在 setter 方法上添加 `@Autowired`注释。

在下面的例子中，创建`FooService`时，setter 方法被调用，并带有`FooFormatter` 的实例:

```java
public class FooService {
    private FooFormatter fooFormatter;
    @Autowired
    public void setFooFormatter(FooFormatter fooFormatter) {
        this.fooFormatter = fooFormatter;
    }
} 
```

### 3.3。`@Autowired`论构造者

最后，让我们在构造函数上使用`@Autowired`。

我们将看到一个`FooFormatter`的实例被 Spring 作为参数注入到`FooService`构造函数中:

```java
public class FooService {
    private FooFormatter fooFormatter;
    @Autowired
    public FooService(FooFormatter fooFormatter) {
        this.fooFormatter = fooFormatter;
    }
}
```

## 4.`@Autowired`与可选依赖项

当构建 bean 时，`@Autowired`依赖项应该是可用的。否则，**如果 Spring 不能解析一个 bean 进行连接，它将抛出一个异常**。

因此，它会阻止 Spring 容器成功启动，但以下形式除外:

```java
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [com.autowire.sample.FooDAO] found for dependency: 
expected at least 1 bean which qualifies as autowire candidate for this dependency. 
Dependency annotations: 
{@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

要解决这个问题，我们需要声明所需类型的 bean:

```java
public class FooService {
    @Autowired(required = false)
    private FooDAO dataAccessor; 
}
```

## 5.自动布线歧义消除

默认情况下，Spring 按类型解析`@Autowired`条目。**如果容器中有多个相同类型的 bean 可用，框架将抛出致命异常**。

为了解决这个冲突，我们需要明确地告诉 Spring 我们想要注入哪个 bean。

### 5.1。由`@Qualifier` 自动布线

例如，让我们看看如何使用 [`@Qualifier`](/web/20220701005438/https://www.baeldung.com/spring-qualifier-annotation) 注释来指示所需的 bean。

首先，我们将定义两个类型为`Formatter`的 beans:

```java
@Component("fooFormatter")
public class FooFormatter implements Formatter {
    public String format() {
        return "foo";
    }
}
```

```java
@Component("barFormatter")
public class BarFormatter implements Formatter {
    public String format() {
        return "bar";
    }
}
```

现在让我们尝试将一个`Formatter` bean 注入到`FooService`类中:

```java
public class FooService {
    @Autowired
    private Formatter formatter;
}
```

在我们的例子中，Spring 容器有两个具体的`Formatter`实现。因此， **Spring 在构造`FooService`:时会抛出一个`NoUniqueBeanDefinitionException` 异常**

```java
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type [com.autowire.sample.Formatter] is defined: 
expected single matching bean but found 2: barFormatter,fooFormatter 
```

**我们可以通过使用`@Qualifier`注释:**缩小实现来避免这种情况

```java
public class FooService {
    @Autowired
    @Qualifier("fooFormatter")
    private Formatter formatter;
}
```

当有多个相同类型的 beans 时，**使用`@Qualifier`来避免歧义是个好主意。**

请注意，`@Qualifier` 注释的值与我们的`FooFormatter`实现的`@Component` 注释中声明的名称相匹配。

### 5.2。通过自定义限定符自动布线

Spring 还允许我们**创建自己的自定义`@Qualifier`注释**。为此，我们应该为`@Qualifier` 注释提供定义:

```java
@Qualifier
@Target({
  ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface FormatterType {  
    String value();
}
```

然后我们可以在各种实现中使用`FormatterType`来指定一个自定义值:

```java
@FormatterType("Foo")
@Component
public class FooFormatter implements Formatter {
    public String format() {
        return "foo";
    }
}
```

```java
@FormatterType("Bar")
@Component
public class BarFormatter implements Formatter {
    public String format() {
        return "bar";
    }
}
```

最后，我们的自定义限定符注释已经可以用于自动连接了:

```java
@Component
public class FooService {  
    @Autowired
    @FormatterType("Foo")
    private Formatter formatter;
} 
```

在 **`@Target`元注释中指定的值限制了在哪里应用限定符**，在我们的例子中是字段、方法、类型和参数。

### 5.3。按名称自动布线

Spring 使用 bean 的名称作为默认的限定符值。它将检查容器并寻找一个名称与属性完全相同的 bean 来自动连接它。

因此，在我们的例子中，Spring 将`fooFormatter` 属性名与`FooFormatter`实现相匹配。因此，它在构造`FooService`时注入了那个特定的实现:

```java
public class FooService {
 @Autowired 
private Formatter fooFormatter; 
}
```

## 6.结论

在本文中，我们讨论了自动布线以及使用它的不同方法。我们还研究了解决两个常见的自动连接异常的方法，这两个异常是由缺少 bean 或不明确的 bean 注入引起的。

本文的源代码可以在 GitHub 项目的[上找到。](https://web.archive.org/web/20220701005438/https://github.com/eugenp/tutorials/tree/master/spring-core-2)*