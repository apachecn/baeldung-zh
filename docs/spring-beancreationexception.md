# Spring BeanCreationException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-beancreationexception>

## 1。概述

在本教程中，我们将讨论 **Spring `org.springframework.beans.factory.BeanCreationException.`** ，这是一个非常常见的异常，当`BeanFactory`创建 bean 定义的 bean 并遇到问题时抛出。本文将探讨该异常最常见的原因以及解决方案。

## 延伸阅读:

## [Spring 控制反转和依赖注入简介](/web/20220529013244/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)

A quick introduction to the concepts of Inversion of Control and Dependency Injection, followed by a simple demonstration using the Spring Framework[Read more](/web/20220529013244/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) →

## [Spring 中的 BeanNameAware 和 BeanFactoryAware 接口](/web/20220529013244/https://www.baeldung.com/spring-bean-name-factory-aware)

Have a look at working with the BeanNameAware and BeanFactoryAware interfaces in Spring.[Read more](/web/20220529013244/https://www.baeldung.com/spring-bean-name-factory-aware) →

## [Spring 5 功能 Bean 注册](/web/20220529013244/https://www.baeldung.com/spring-5-functional-beans)

See how to register beans using the functional approach in Spring 5.[Read more](/web/20220529013244/https://www.baeldung.com/spring-5-functional-beans) →

## 2。`org.springframework.beans.factory.NoSuchBeanDefinitionException`起因:

到目前为止，`BeanCreationException`最常见的原因是 Spring 试图**注入一个在上下文中不存在的**bean。

例如，`BeanA`正在尝试注入`BeanB`:

```java
@Component
public class BeanA {

    @Autowired
    private BeanB dependency;
    ...
}
```

如果在上下文中没有找到`BeanB`，那么将抛出以下异常(创建 Bean 时出错):

```java
Error creating bean with name 'beanA': Injection of autowired dependencies failed; 
nested exception is org.springframework.beans.factory.BeanCreationException: 
Could not autowire field: private com.baeldung.web.BeanB cpm.baeldung.web.BeanA.dependency; 
nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [com.baeldung.web.BeanB] found for dependency: 
expected at least 1 bean which qualifies as autowire candidate for this dependency. 
Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

为了诊断这种类型的问题，我们将首先确保 bean 被声明为:

*   或者在 XML 配置文件中使用`<bean />`元素
*   或者通过`@Bean`注释在 Java `@Configuration`类中实现
*   或者用`@Component`、`@Repository`、`@Service`、`@Controller,`进行注释，并且类路径扫描对于该包是活动的

我们还将检查 Spring 是否真的获取了配置文件或类，并将它们加载到主上下文中。

## 延伸阅读:

## [Spring 控制反转和依赖注入简介](/web/20220529013244/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)

A quick introduction to the concepts of Inversion of Control and Dependency Injection, followed by a simple demonstration using the Spring Framework[Read more](/web/20220529013244/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) →

## [Spring 中的 BeanNameAware 和 BeanFactoryAware 接口](/web/20220529013244/https://www.baeldung.com/spring-bean-name-factory-aware)

Have a look at working with the BeanNameAware and BeanFactoryAware interfaces in Spring.[Read more](/web/20220529013244/https://www.baeldung.com/spring-bean-name-factory-aware) →

## [Spring 5 功能 Bean 注册](/web/20220529013244/https://www.baeldung.com/spring-5-functional-beans)

See how to register beans using the functional approach in Spring 5.[Read more](/web/20220529013244/https://www.baeldung.com/spring-5-functional-beans) →

## 3。`org.springframework.beans.factory.NoUniqueBeanDefinitionException`起因:

bean 创建异常的另一个类似原因是 Spring 试图通过类型注入 bean，即通过它的接口，并在上下文中找到两个或更多实现接口的 bean。

例如，`BeanB1`和`BeanB2`都实现了相同的接口:

```java
@Component
public class BeanB1 implements IBeanB { ... }
@Component
public class BeanB2 implements IBeanB { ... }

@Component
public class BeanA {

    @Autowired
    private IBeanB dependency;
    ...
}
```

这将导致 Spring bean 工厂抛出以下异常:

```java
Error creating bean with name 'beanA': Injection of autowired dependencies failed; 
nested exception is org.springframework.beans.factory.BeanCreationException: 
Could not autowire field: private com.baeldung.web.IBeanB com.baeldung.web.BeanA.b; 
nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type [com.baeldung.web.IBeanB] is defined: 
expected single matching bean but found 2: beanB1,beanB2
```

## 4。`org.springframework.beans.BeanInstantiationException`起因:

### 4.1。自定义异常

接下来是一个在创建过程中抛出异常的 **bean。**一个容易理解问题的简化示例是在 bean 的构造函数中抛出一个异常:

```java
@Component
public class BeanA {

    public BeanA() {
        super();
        throw new NullPointerException();
    }
    ...
}
```

正如所料，这将导致 Spring 快速失败，并出现以下异常:

```java
Error creating bean with name 'beanA' defined in file [...BeanA.class]: 
Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: 
Could not instantiate bean class [com.baeldung.web.BeanA]: 
Constructor threw exception; 
nested exception is java.lang.NullPointerException
```

### 4.2。`java.lang.InstantiationException`

`BeanInstantiationException`的另一个可能出现的情况是在 XML 中将抽象类定义为 bean 这必须在 XML 中，因为在 Java `@Configuration`文件中没有办法做到这一点，并且类路径扫描将忽略抽象类:

```java
@Component
public abstract class BeanA implements IBeanA { ... }
```

下面是 bean 的 XML 定义:

```java
<bean id="beanA" class="com.baeldung.web.BeanA" />
```

此设置将导致类似的异常:

```java
org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'beanA' defined in class path resource [beansInXml.xml]: 
Instantiation of bean failed; 
nested exception is org.springframework.beans.BeanInstantiationException: 
Could not instantiate bean class [com.baeldung.web.BeanA]: 
Is it an abstract class?; 
nested exception is java.lang.InstantiationException
```

### 4.3。`java.lang.NoSuchMethodException`

如果 bean 没有默认的构造函数，而 Spring 试图通过查找该构造函数来实例化它，这将导致运行时异常:

```java
@Component
public class BeanA implements IBeanA {

    public BeanA(final String name) {
        super();
        System.out.println(name);
    }
}
```

当类路径扫描机制选择这个 bean 时，失败将是:

```java
Error creating bean with name 'beanA' defined in file [...BeanA.class]: Instantiation of bean failed; 
nested exception is org.springframework.beans.BeanInstantiationException: 
Could not instantiate bean class [com.baeldung.web.BeanA]: 
No default constructor found; 
nested exception is java.lang.NoSuchMethodException: com.baeldung.web.BeanA.<init>()
```

当类路径上的 Spring 依赖项没有**相同版本时，可能会出现类似的异常，但更难诊断。**这种版本不兼容可能会因为 API 的变化而导致`NoSuchMethodException`。解决这个问题的方法是确保所有的 Spring 库在项目中有完全相同的版本。

## 5。`org.springframework.beans.NotWritablePropertyException`起因:

还有一种可能性是定义一个 bean，`BeanA,`并引用另一个 bean，`BeanB,`，而在`BeanA`中没有相应的 setter 方法:

```java
@Component
public class BeanA {
    private IBeanB dependency;
    ...
}
@Component
public class BeanB implements IBeanB { ... }
```

爱马仕

```java
<bean id="beanA" class="com.baeldung.web.BeanA">
    <property name="beanB" ref="beanB" />
</bean>
```

同样，这个**只能出现在 XML 配置**中，因为当使用 Java `@Configuration`时，编译器将使这个问题不可能重现。

当然，为了解决这个问题，我们需要为`IBeanB`添加 setter:

```java
@Component
public class BeanA {
    private IBeanB dependency;

    public void setDependency(final IBeanB dependency) {
        this.dependency = dependency;
    }
}
```

## 6。`org.springframework.beans.factory.CannotLoadBeanClassException`起因:

**Spring 在无法加载已定义 bean** 的类时抛出这个异常。如果 Spring XML 配置包含一个没有相应类的 bean，就会出现这种情况。例如，如果类`BeanZ`不存在，下面的定义将导致一个异常:

```java
<bean id="beanZ" class="com.baeldung.web.BeanZ" />
```

本例中`ClassNotFoundException`和完整异常的根本原因是:

```java
nested exception is org.springframework.beans.factory.BeanCreationException: 
...
nested exception is org.springframework.beans.factory.CannotLoadBeanClassException: 
Cannot find class [com.baeldung.web.BeanZ] for bean with name 'beanZ' 
defined in class path resource [beansInXml.xml]; 
nested exception is java.lang.ClassNotFoundException: com.baeldung.web.BeanZ
```

## 7。`BeanCreationException` 【T2 的孩子】

### 7.1。`org.springframework.beans.factory.BeanCurrentlyInCreationException`

`BeanCreationException`的一个子类是`BeanCurrentlyInCreationException.`，这通常发生在使用构造函数注入时，例如，在循环依赖的情况下:

```java
@Component
public class BeanA implements IBeanA {
    private IBeanB beanB;

    @Autowired
    public BeanA(final IBeanB beanB) {
        super();
        this.beanB = beanB;
    }
}
@Component
public class BeanB implements IBeanB {
    final IBeanA beanA;

    @Autowired
    public BeanB(final IBeanA beanA) {
        super();
        this.beanA = beanA;
    }
}
```

Spring 无法解决这种连接场景，最终结果将是:

```java
org.springframework.beans.factory.BeanCurrentlyInCreationException: 
Error creating bean with name 'beanA': 
Requested bean is currently in creation: Is there an unresolvable circular reference?
```

完整的异常非常详细:

```java
org.springframework.beans.factory.UnsatisfiedDependencyException: 
Error creating bean with name 'beanA' defined in file [...BeanA.class]: 
Unsatisfied dependency expressed through constructor argument with index 0 
of type [com.baeldung.web.IBeanB]: : 
Error creating bean with name 'beanB' defined in file [...BeanB.class]: 
Unsatisfied dependency expressed through constructor argument with index 0 
of type [com.baeldung.web.IBeanA]: : 
Error creating bean with name 'beanA': Requested bean is currently in creation: 
Is there an unresolvable circular reference?; 
nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: 
Error creating bean with name 'beanA': 
Requested bean is currently in creation: 
Is there an unresolvable circular reference?; 
nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: 
Error creating bean with name 'beanB' defined in file [...BeanB.class]: 
Unsatisfied dependency expressed through constructor argument with index 0 
of type [com.baeldung.web.IBeanA]: : 
Error creating bean with name 'beanA': 
Requested bean is currently in creation: 
Is there an unresolvable circular reference?; 
nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: 
Error creating bean with name 'beanA': 
Requested bean is currently in creation: Is there an unresolvable circular reference?
```

### 7.2。`org.springframework.beans.factory.BeanIsAbstractException`

当 bean 工厂试图检索和实例化被声明为抽象的 Bean 时，可能会发生此实例化异常:

```java
public abstract class BeanA implements IBeanA {
   ...
}
```

我们在 XML 配置中将其声明为:

```java
<bean id="beanA" abstract="true" class="com.baeldung.web.BeanA" />
```

如果我们试图通过名称从 Spring 上下文中检索`BeanA`，就像实例化另一个 bean 一样:

```java
@Configuration
public class Config {
    @Autowired
    BeanFactory beanFactory;

    @Bean
    public BeanB beanB() {
        beanFactory.getBean("beanA");
        return new BeanB();
    }
}
```

这将导致以下异常:

```java
org.springframework.beans.factory.BeanIsAbstractException: 
Error creating bean with name 'beanA': Bean definition is abstract
```

和完整的异常堆栈跟踪:

```java
org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'beanB' defined in class path resource 
[org/baeldung/spring/config/WebConfig.class]: Instantiation of bean failed; 
nested exception is org.springframework.beans.factory.BeanDefinitionStoreException: 
Factory method 
[public com.baeldung.web.BeanB com.baeldung.spring.config.WebConfig.beanB()] threw exception; 
nested exception is org.springframework.beans.factory.BeanIsAbstractException: 
Error creating bean with name 'beanA': Bean definition is abstract
```

## 8。结论

在本文中，我们学习了如何处理可能导致春天到来的各种原因和问题，以及如何解决所有这些问题。

所有异常示例的实现都可以在 github 项目中找到。这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。