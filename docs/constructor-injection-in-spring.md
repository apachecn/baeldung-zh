# Spring 中的构造函数依赖注入

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/constructor-injection-in-spring>

## 1。简介

可以说，现代软件设计最重要的开发原则之一是`Dependency Injection (DI),` ，它很自然地衍生出另一个至关重要的原则:`Modularity.`

这篇快速教程将探索 Spring 中一种称为`Constructor-Based Dependency Injection, `的特定类型的 DI 技术，简单地说，这意味着我们在实例化时将所需的组件传递给一个类。

首先，我们需要在我们的`pom.xml`中导入`spring-context`依赖项:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

然后我们需要建立一个`Configuration` 文件。根据偏好，该文件可以是 POJO 或 XML 文件。

## 延伸阅读:

## [Spring 控制反转和依赖注入简介](/web/20220930092237/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)

A quick introduction to the concepts of Inversion of Control and Dependency Injection, followed by a simple demonstration using the Spring Framework[Read more](/web/20220930092237/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) →

## [Top Spring 框架面试问题](/web/20220930092237/https://www.baeldung.com/spring-interview-questions)

A quick discussion of common questions about the Spring Framework that might come up during a job interview.[Read more](/web/20220930092237/https://www.baeldung.com/spring-interview-questions) →

## [春季布线:@Autowired、@Resource 和@Inject](/web/20220930092237/https://www.baeldung.com/spring-annotations-resource-inject-autowire)

This article will compare and contrast the use of annotations related to dependency injection, namely the @Resource, @Inject, and @Autowired annotations.[Read more](/web/20220930092237/https://www.baeldung.com/spring-annotations-resource-inject-autowire) →

## 2。基于注释的配置

Java 配置文件看起来类似于 Java 对象，但有一些附加注释:

```java
@Configuration
@ComponentScan("com.baeldung.constructordi")
public class Config {

    @Bean
    public Engine engine() {
        return new Engine("v8", 5);
    }

    @Bean
    public Transmission transmission() {
        return new Transmission("sliding");
    }
} 
```

这里我们使用注释来通知 Spring runtime 这个类提供了 bean 定义(`@Bean`注释)，并且包`com.baeldung.spring` 需要执行额外 bean 的上下文扫描。接下来，我们定义一个`Car`类:

```java
@Component
public class Car {

    @Autowired
    public Car(Engine engine, Transmission transmission) {
        this.engine = engine;
        this.transmission = transmission;
    }
}
```

Spring 在进行包扫描时会遇到我们的`Car` 类，并且会通过调用`@Autowired` 带注释的构造函数来初始化它的实例。

通过调用`Config` 类的`@Bean` 注释方法，我们将获得`Engine and Transmission`的实例。最后，我们需要使用我们的 POJO 配置引导一个`ApplicationContext` :

```java
ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
Car car = context.getBean(Car.class);
```

## 3。隐式构造函数注入

从 Spring 4.3 开始，只有一个构造函数的类可以省略`@Autowired` 注释。这是一个很好的小方便和样板文件删除。

最重要的是，从 4.3 开始，我们可以在`@Configuration` 注释类中利用基于构造函数的注入。此外，如果这样的类只有一个构造函数，我们也可以省略`@Autowired` 注释。

## 4。基于 XML 的配置

用基于构造函数的依赖注入来配置 Spring runtime 的另一种方法是使用 XML 配置文件:

```java
<bean id="toyota" class="com.baeldung.constructordi.domain.Car">
    <constructor-arg index="0" ref="engine"/>
    <constructor-arg index="1" ref="transmission"/>
</bean>

<bean id="engine" class="com.baeldung.constructordi.domain.Engine">
    <constructor-arg index="0" value="v4"/>
    <constructor-arg index="1" value="2"/>
</bean>

<bean id="transmission" class="com.baeldung.constructordi.domain.Transmission">
    <constructor-arg value="sliding"/>
</bean>
```

注意，`constructor-arg`可以接受一个文字值或者对另一个 bean 的引用，并且可以提供一个可选的显式`index`和`type`。我们可以使用`Type`和`index`属性来解决歧义(例如，如果一个构造函数接受多个相同类型的参数)。

> `name` 属性也可以用于 xml 到 java 变量的匹配，但是你的代码`must`要在调试标志打开的情况下编译。

在这种情况下，我们需要使用`ClassPathXmlApplicationContext`引导我们的 Spring 应用程序上下文:

```java
ApplicationContext context = new ClassPathXmlApplicationContext("baeldung.xml");
Car car = context.getBean(Car.class);
```

## 5.利弊

与字段注入相比，构造函数注入有几个优点。

第一个好处是可测试性。假设我们要对一个使用字段注入的 Spring bean 进行单元测试:

```java
public class UserService {

    @Autowired 
    private UserRepository userRepository;
}
```

在构建`UserService `实例的过程中，我们不能初始化`userRepository `状态。实现这一点的唯一方法是通过[反射 API](/web/20220930092237/https://www.baeldung.com/java-reflection) ，它完全打破了封装。此外，与简单的构造函数调用相比，生成的代码不太安全。

另外，**使用** **字段注入，我们不能强制执行类级不变量，** `s` o 有可能有一个`UserService `实例没有正确初始化`userRepository`。因此，我们可能会到处经历随机的`NullPointerException`。此外，通过构造函数注入，构建不可变的组件变得更加容易。

此外，从 OOP 的角度来看，使用构造函数创建对象实例更加自然。

另一方面，构造函数注入的主要缺点是冗长，尤其是当 bean 有少量依赖项时。有时这可能是塞翁失马焉知非福，因为我们可能会更努力地保持依赖的数量最少。

## 6。结论

这篇简短的文章展示了使用 Spring 框架使用`Constructor-Based Dependency Injection` 的两种不同方式的基础。

这篇文章的**完整实现**可以在 GitHub 的[中找到。](https://web.archive.org/web/20220930092237/https://github.com/eugenp/tutorials/tree/master/spring-di-2)