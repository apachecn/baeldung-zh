# Spring 中的循环依赖

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/circular-dependencies-in-spring>

## 1.什么是循环依赖？

当 bean A 依赖于另一个 bean B，并且 bean B 也依赖于 bean A 时，就会出现循环依赖:

豆 A →豆 B →豆 A

当然，我们可以隐含更多的 beans:

豆 A →豆 B →豆 C →豆 D →豆 E →豆 A

## 延伸阅读:

## [弹簧依赖注入](/web/20221216225650/https://www.baeldung.com/spring-dependency-injection)

Learn about Dependency Injection using the Spring framework.[Read more](/web/20221216225650/https://www.baeldung.com/spring-dependency-injection) →

## [Guice vs Spring——依赖注入](/web/20221216225650/https://www.baeldung.com/guice-spring-dependency-injection)

Learn about the similarities and differences between Guice and Spring for dependency injection[Read more](/web/20221216225650/https://www.baeldung.com/guice-spring-dependency-injection) →

## [将 Spring Beans 注入非托管对象](/web/20221216225650/https://www.baeldung.com/spring-inject-bean-into-unmanaged-objects)

Learn how to inject a Spring bean into an ordinary object.[Read more](/web/20221216225650/https://www.baeldung.com/spring-inject-bean-into-unmanaged-objects) →

## 2。春天发生了什么

当 Spring 上下文加载所有的 bean 时，它试图按照它们完全工作所需的顺序创建 bean。

假设我们没有循环依赖。取而代之的是这样的:

豆 A →豆 B →豆 C

Spring 会创建 bean C，然后创建 bean B(并将 bean C 注入其中)，然后创建 bean A(将 bean B 注入其中)。

但是对于循环依赖，Spring 不能决定应该先创建哪个 beans，因为它们相互依赖。在这些情况下，Spring 将在加载上下文时引发一个`BeanCurrentlyInCreationException`。

在 Spring 中使用**构造函数注入时可能会发生这种情况。**如果我们使用其他类型的注入，我们应该不会有这个问题，因为依赖项将在需要时注入，而不是在上下文加载时注入。

## 3。一个简单的例子

让我们定义两个相互依赖的 beans(通过构造函数注入):

```java
@Component
public class CircularDependencyA {

    private CircularDependencyB circB;

    @Autowired
    public CircularDependencyA(CircularDependencyB circB) {
        this.circB = circB;
    }
}
```

```java
@Component
public class CircularDependencyB {

    private CircularDependencyA circA;

    @Autowired
    public CircularDependencyB(CircularDependencyA circA) {
        this.circA = circA;
    }
}
```

现在我们可以为测试编写一个配置类(让我们称之为`TestConfig`)来指定扫描组件的基础包。

让我们假设我们的 beans 是在包"`com.baeldung.circulardependency`"中定义的:

```java
@Configuration
@ComponentScan(basePackages = { "com.baeldung.circulardependency" })
public class TestConfig {
}
```

最后，我们可以编写一个 JUnit 测试来检查循环依赖。

测试可以为空，因为循环依赖将在上下文加载期间被检测到:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { TestConfig.class })
public class CircularDependencyIntegrationTest {

    @Test
    public void givenCircularDependency_whenConstructorInjection_thenItFails() {
        // Empty test; we just want the context to load
    }
}
```

如果我们尝试运行这个测试，我们将得到这个异常:

```java
BeanCurrentlyInCreationException: Error creating bean with name 'circularDependencyA':
Requested bean is currently in creation: Is there an unresolvable circular reference?
```

## 4.变通办法

我们现在将展示一些处理这个问题的最流行的方法。

### 4.1。重新设计

当我们有一个循环依赖时，很可能我们有一个设计问题，并且责任没有很好的分离。我们应该尝试适当地重新设计组件，以便它们的层次结构设计得很好，并且不需要循环依赖。

然而，有许多可能的原因导致我们无法进行重新设计，例如遗留代码、已经测试过的代码以及无法修改的代码、没有足够的时间或资源来完成重新设计等等。如果我们不能重新设计组件，我们可以尝试一些变通办法。

### 4.2。使用`@Lazy`

打破这种循环的一个简单方法是告诉 Spring 延迟初始化其中一个 beans。因此，它不是完全初始化 bean，而是创建一个代理将它注入到另一个 bean 中。只有在第一次需要时，注入的 bean 才会被完全创建。

为了在我们的代码中尝试这一点，我们可以更改`CircularDependencyA`:

```java
@Component
public class CircularDependencyA {

    private CircularDependencyB circB;

    @Autowired
    public CircularDependencyA(@Lazy CircularDependencyB circB) {
        this.circB = circB;
    }
}
```

如果我们现在运行测试，我们会发现这次不会发生错误。

### 4.3。使用 Setter/Field Injection

最流行的解决方法之一，也是 [Spring 文档所建议的](https://web.archive.org/web/20221216225650/https://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html)，是使用 setter 注入。

简而言之，我们可以通过改变 beans 的连接方式来解决这个问题——使用 setter 注入(或字段注入)而不是构造函数注入。通过这种方式，Spring 创建了 beans，但是依赖项直到需要时才被注入。

因此，让我们更改我们的类，使用 setter 注入，并添加另一个字段(`message`)到`CircularDependencyB`，这样我们就可以进行适当的单元测试:

```java
@Component
public class CircularDependencyA {

    private CircularDependencyB circB;

    @Autowired
    public void setCircB(CircularDependencyB circB) {
        this.circB = circB;
    }

    public CircularDependencyB getCircB() {
        return circB;
    }
}
```

```java
@Component
public class CircularDependencyB {

    private CircularDependencyA circA;

    private String message = "Hi!";

    @Autowired
    public void setCircA(CircularDependencyA circA) {
        this.circA = circA;
    }

    public String getMessage() {
        return message;
    }
}
```

现在，我们必须对我们的单元测试进行一些更改:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { TestConfig.class })
public class CircularDependencyIntegrationTest {

    @Autowired
    ApplicationContext context;

    @Bean
    public CircularDependencyA getCircularDependencyA() {
        return new CircularDependencyA();
    }

    @Bean
    public CircularDependencyB getCircularDependencyB() {
        return new CircularDependencyB();
    }

    @Test
    public void givenCircularDependency_whenSetterInjection_thenItWorks() {
        CircularDependencyA circA = context.getBean(CircularDependencyA.class);

        Assert.assertEquals("Hi!", circA.getCircB().getMessage());
    }
}
```

让我们仔细看看这些注释。

`@Bean`告诉 Spring framework 必须使用这些方法来检索要注入的 beans 的实现。

使用`@Test`注释，测试将从上下文中获取`CircularDependencyA` bean，并断言其`CircularDependencyB`已被正确注入，检查其`message`属性的值。

### 4.4。使用`@PostConstruct`

打破这种循环的另一种方法是在一个 beans 上使用`@Autowired`注入一个依赖项，然后使用一个用`@PostConstruct`注释的方法来设置另一个依赖项。

我们的 beans 可能有这样的代码:

```java
@Component
public class CircularDependencyA {

    @Autowired
    private CircularDependencyB circB;

    @PostConstruct
    public void init() {
        circB.setCircA(this);
    }

    public CircularDependencyB getCircB() {
        return circB;
    }
}
```

```java
@Component
public class CircularDependencyB {

    private CircularDependencyA circA;

    private String message = "Hi!";

    public void setCircA(CircularDependencyA circA) {
        this.circA = circA;
    }

    public String getMessage() {
        return message;
    }
}
```

我们可以运行与之前相同的测试，因此我们检查循环依赖异常是否仍未被抛出，以及依赖项是否被正确注入。

### 4.5。执行`ApplicationContextAware`和`InitializingBean`和

如果其中一个 bean 实现了`ApplicationContextAware`，那么这个 bean 就可以访问 Spring context，并且可以从那里提取另一个 bean。

通过实现`InitializingBean`，我们指出这个 bean 必须在它的所有属性都设置好之后做一些动作。在这种情况下，我们希望手动设置我们的依赖关系。

以下是我们的 beans 的代码:

```java
@Component
public class CircularDependencyA implements ApplicationContextAware, InitializingBean {

    private CircularDependencyB circB;

    private ApplicationContext context;

    public CircularDependencyB getCircB() {
        return circB;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        circB = context.getBean(CircularDependencyB.class);
    }

    @Override
    public void setApplicationContext(final ApplicationContext ctx) throws BeansException {
        context = ctx;
    }
}
```

```java
@Component
public class CircularDependencyB {

    private CircularDependencyA circA;

    private String message = "Hi!";

    @Autowired
    public void setCircA(CircularDependencyA circA) {
        this.circA = circA;
    }

    public String getMessage() {
        return message;
    }
}
```

同样，我们可以运行前面的测试，并看到没有抛出异常，测试按预期运行。

## 5。结论

在 Spring 中，有很多方法可以处理循环依赖。

我们应该首先考虑重新设计我们的 beans，这样就不需要循环依赖。这是因为循环依赖通常是可以改进的设计的征兆。

但是如果我们的项目中绝对需要循环依赖，我们可以遵循这里建议的一些变通方法。

首选方法是使用 setter 进样。但是还有其他的选择，通常基于停止 Spring 管理 beans 的初始化和注入，以及我们自己使用不同的策略来完成这些。

本文中的例子可以在 [GitHub 项目](https://web.archive.org/web/20221216225650/https://github.com/eugenp/tutorials/tree/master/spring-di-2)中找到。