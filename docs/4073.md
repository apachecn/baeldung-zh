# Spring 核心注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-core-annotations>

[This article is part of a series:](javascript:void(0);)• Spring Core Annotations (current article)[• Spring Web Annotations](/web/20220930092237/https://www.baeldung.com/spring-mvc-annotations)
[• Spring Boot Annotations](/web/20220930092237/https://www.baeldung.com/spring-boot-annotations)
[• Spring Scheduling Annotations](/web/20220930092237/https://www.baeldung.com/spring-scheduling-annotations)
[• Spring Data Annotations](/web/20220930092237/https://www.baeldung.com/spring-data-annotations)
[• Spring Bean Annotations](/web/20220930092237/https://www.baeldung.com/spring-bean-annotations)

## 1.概观

我们可以使用`org.springframework.beans.factory.annotation `和`org.springframework.context.annotation`包中的注释来利用 Spring DI 引擎的功能。

我们通常称这些为“Spring 核心注释”,我们将在本教程中回顾它们。

## 2.与 DI 相关的注释

### 2.1.`@Autowired`

我们可以使用`@Autowired`到**来标记 Spring 将要解决的依赖，并注入**。我们可以将该注释与构造函数、设置器或字段注入一起使用。

构造函数注入:

```
class Car {
    Engine engine;

    @Autowired
    Car(Engine engine) {
        this.engine = engine;
    }
}
```

Setter 注入:

```
class Car {
    Engine engine;

    @Autowired
    void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

现场注射:

```
class Car {
    @Autowired
    Engine engine;
}
```

`@Autowired`有一个名为`required`的`boolean`参数，默认值为`true`。当没有找到合适的 bean 来连接时，它会调整 Spring 的行为。当`true`出现时，抛出一个异常，否则，不连接任何东西。

注意，如果我们使用构造函数注入，所有的构造函数参数都是强制的。

从 4.3 版开始，我们不需要用`@Autowired`显式地注释构造函数，除非我们声明了至少两个构造函数。

更多细节请访问我们关于 [`@Autowired`](/web/20220930092237/https://www.baeldung.com/spring-autowire) 和[构造函数注入](/web/20220930092237/https://www.baeldung.com/constructor-injection-in-spring)的文章。

### 2.2.`@Bean`

`@Bean`标记实例化 Spring bean 的工厂方法:

```
@Bean
Engine engine() {
    return new Engine();
}
```

当需要返回类型的新实例时，Spring 调用这些方法。

生成的 bean 与工厂方法同名。如果我们想用不同的名字命名它，我们可以用这个注释的参数`name`或`value`来命名(参数`value`是参数`name`的别名):

```
@Bean("engine")
Engine getEngine() {
    return new Engine();
}
```

注意，所有用`@Bean`标注的方法都必须在`@Configuration`类中。

### 2.3.`@Qualifier`

我们使用 `@Qualifier`和`@Autowired`到**来提供我们想要在不明确的情况下使用的 bean id 或 bean 名称**。

例如，以下两个 beans 实现了相同的接口:

```
class Bike implements Vehicle {}

class Car implements Vehicle {}
```

如果 Spring 需要注入一个`Vehicle` bean，它最终会有多个匹配的定义。在这种情况下，我们可以使用`@Qualifier`注释显式地提供 bean 的名称。

使用构造函数注入:

```
@Autowired
Biker(@Qualifier("bike") Vehicle vehicle) {
    this.vehicle = vehicle;
}
```

使用 setter 进样:

```
@Autowired
void setVehicle(@Qualifier("bike") Vehicle vehicle) {
    this.vehicle = vehicle;
}
```

或者:

```
@Autowired
@Qualifier("bike")
void setVehicle(Vehicle vehicle) {
    this.vehicle = vehicle;
}
```

使用现场注射:

```
@Autowired
@Qualifier("bike")
Vehicle vehicle;
```

更详细的描述，请阅读[这篇文章](/web/20220930092237/https://www.baeldung.com/spring-autowire)。

### 2.4.`@Required`

`@Required`在 setter 方法上标记我们希望通过 XML 填充的依赖关系:

```
@Required
void setColor(String color) {
    this.color = color;
}
```

```
<bean class="com.baeldung.annotations.Bike">
    <property name="color" value="green" />
</bean>
```

否则会抛出`BeanInitializationException`。

### 2.5.`@Value`

我们可以使用 [*@Value*](/web/20220930092237/https://www.baeldung.com/spring-value-annotation) 为 beans 注入属性值。它与构造器、设置器和字段注入兼容。

构造函数注入:

```
Engine(@Value("8") int cylinderCount) {
    this.cylinderCount = cylinderCount;
}
```

Setter 注入:

```
@Autowired
void setCylinderCount(@Value("8") int cylinderCount) {
    this.cylinderCount = cylinderCount;
}
```

或者:

```
@Value("8")
void setCylinderCount(int cylinderCount) {
    this.cylinderCount = cylinderCount;
}
```

现场注射:

```
@Value("8")
int cylinderCount;
```

当然，注入静态值是没有用的。因此，我们可以使用`@Value`中的**占位符字符串**来连接外部源中定义的值**，例如在`.properties`或`.yaml`文件中。**

让我们假设下面的`.properties`文件:

```
engine.fuelType=petrol
```

我们可以用以下公式注入`engine.fuelType`的值:

```
@Value("${engine.fuelType}")
String fuelType;
```

我们甚至可以在 SpEL 中使用`@Value`。更高级的例子可以在我们关于`@Value` 的[文章中找到。](/web/20220930092237/https://www.baeldung.com/spring-value-annotation)

### 2.6.`@DependsOn`

我们可以使用这个注释让 Spring **在被注释的 beanss】之前初始化其他 bean。通常，这种行为是自动的，基于 beans 之间的显式依赖关系。**

只有当依赖关系是隐式的时，我们才需要这个注释**，例如，JDBC 驱动加载或静态变量初始化。**

我们可以在依赖类上使用`@DependsOn`来指定依赖 beans 的名称。注释的`value`参数需要一个包含依赖 bean 名称的数组:

```
@DependsOn("engine")
class Car implements Vehicle {}
```

或者，如果我们用`@Bean`注释定义一个 bean，那么工厂方法应该用`@DependsOn`注释:

```
@Bean
@DependsOn("fuel")
Engine engine() {
    return new Engine();
}
```

### 2.7.`@Lazy`

当我们想要懒惰地初始化我们的 bean 时，我们使用 [`@Lazy`](/web/20220930092237/https://www.baeldung.com/spring-lazy-annotation) 。默认情况下，Spring 会在应用程序上下文启动/引导时急切地创建所有单例 beans。

然而，有些情况下**我们需要在请求时创建一个 bean，而不是在应用程序启动时**。

这个注释根据我们放置它的确切位置而有不同的行为。我们可以把它放在:

*   一个带注释的 bean 工厂方法，用于延迟方法调用(因此创建了 bean)
*   @ `Configuration`类和所有包含的`@Bean`方法都将受到影响
*   一个不是`@Configuration`类的`@Component`类，这个 bean 将被延迟初始化
*   一个`@Autowired`构造器、设置器或字段，用来惰性地加载依赖项本身(通过代理)

这个注释有一个名为`value`的参数，默认值为`true`。覆盖默认行为很有用。

例如，当全局设置为惰性时，将 beans 标记为急切加载，或者在标记有`@Lazy`的`@Configuration`类中，将特定的`@Bean`方法配置为急切加载:

```
@Configuration
@Lazy
class VehicleFactoryConfig {

    @Bean
    @Lazy(false)
    Engine engine() {
        return new Engine();
    }
}
```

如需进一步阅读，请访问[本文](/web/20220930092237/https://www.baeldung.com/spring-lazy-annotation)。

### 2.8.`@Lookup`

用`@Lookup`注释的方法告诉 Spring 在我们调用它时返回该方法返回类型的一个实例。

关于注释[的详细信息可以在本文](/web/20220930092237/https://www.baeldung.com/spring-lookup)中找到。

### 2.9.`@Primary`

有时我们需要定义多个相同类型的 beans。在这些情况下，注入将会失败，因为 Spring 不知道我们需要哪个 bean。

我们已经看到了处理这个场景的一个选项:用`@Qualifier`标记所有的连接点，并指定所需 bean 的名称。

然而，大多数时候我们需要一个特定的 bean，很少需要其他的。我们可以用`@Primary`来简化这种情况:如果**我们用`@Primary`** 标记最常用的 bean，它将被选在不合格的注入点上；

```
@Component
@Primary
class Car implements Vehicle {}

@Component
class Bike implements Vehicle {}

@Component
class Driver {
    @Autowired
    Vehicle vehicle;
}

@Component
class Biker {
    @Autowired
    @Qualifier("bike")
    Vehicle vehicle;
}
```

在前面的示例中，`Car`是主车辆。因此，在`Driver`类中，Spring 注入了一个`Car` bean。当然，在`Biker` bean 中，字段`vehicle`的值将是一个`Bike`对象，因为它是限定的。

### 2.10.`@Scope`

我们使用`@Scope`来定义一个`@Component`类的[作用域](/web/20220930092237/https://www.baeldung.com/spring-bean-scopes)或者一个`@Bean`定义`.` 它可以是`singleton, prototype, request, session, globalSession`或者一些自定义作用域。

例如:

```
@Component
@Scope("prototype")
class Engine {}
```

## 3.上下文配置注释

我们可以用本节中描述的注释来配置应用程序上下文。

### 3.1.`@Profile`

如果我们想让 Spring**只在一个特定的概要文件激活**时使用一个`@Component`类或者一个`@Bean`方法，我们可以用`@Profile`来标记它。我们可以用注释的`value`参数来配置概要文件的名称:

```
@Component
@Profile("sportDay")
class Bike implements Vehicle {}
```

你可以在本文的[中阅读更多关于个人资料的信息。](/web/20220930092237/https://www.baeldung.com/spring-profiles)

### 3.2.`@Import`

我们可以使用**特定的`@Configuration`类，而不需要组件扫描**这个注释。我们可以为这些类提供`@Import`的`value`参数:

```
@Import(VehiclePartSupplier.class)
class VehicleFactoryConfig {}
```

### 3.3.`@ImportResource`

我们可以用这个注释**导入 XML 配置**。我们可以用参数`locations`或者它的别名`value`来指定 XML 文件的位置:

```
@Configuration
@ImportResource("classpath:/annotations.xml")
class VehicleFactoryConfig {}
```

### 3.4.`@PropertySource`

有了这个注释，我们可以**为应用程序设置**定义属性文件:

```
@Configuration
@PropertySource("classpath:/annotations.properties")
class VehicleFactoryConfig {}
```

`@PropertySource`利用 Java 8 的重复注释特性，这意味着我们可以用它多次标记一个类:

```
@Configuration
@PropertySource("classpath:/annotations.properties")
@PropertySource("classpath:/vehicle-factory.properties")
class VehicleFactoryConfig {}
```

### 3.5.`@PropertySources`

我们可以使用这个注释来指定多个`@PropertySource` 配置:

```
@Configuration
@PropertySources({ 
    @PropertySource("classpath:/annotations.properties"),
    @PropertySource("classpath:/vehicle-factory.properties")
})
class VehicleFactoryConfig {}
```

注意，从 Java 8 开始，我们可以通过上述的重复注释特性实现同样的功能。

## 4.结论

在本文中，我们看到了最常见的 Spring 核心注释的概述。我们看到了如何配置 bean 连接和应用程序上下文，以及如何为组件扫描标记类。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220930092237/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations)

Next **»**[Spring Web Annotations](/web/20220930092237/https://www.baeldung.com/spring-mvc-annotations)