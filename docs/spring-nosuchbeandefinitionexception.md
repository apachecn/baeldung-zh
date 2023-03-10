# spring nosuchbeandefinnotionexception

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-nosuchbeandefinitionexception>

## 1。概述

在本教程中，我们将讨论**弹簧`org.springframework.beans.factory.NoSuchBeanDefinitionException`。**

这是一个常见的异常，当`BeanFactory`试图解析**一个在 Spring 上下文中没有定义**的 bean 时抛出。

我们将说明这个问题的可能原因和可用的解决方案。

当然，异常总是在我们最意想不到的时候发生，所以看看 Spring 中的[异常和解决方案的完整列表。](/web/20220628130639/https://www.baeldung.com/spring-exceptions)

## 延伸阅读:

## [春季异常教程](/web/20220628130639/https://www.baeldung.com/spring-exceptions)

Some of the most common exceptions in Spring with examples - why they occur and how to solve them quickly.[Read more](/web/20220628130639/https://www.baeldung.com/spring-exceptions) →

## [Spring bean creation exception](/web/20220628130639/https://www.baeldung.com/spring-beancreationexception)

A quick and practical guide to dealing with different causes of Spring BeanCreationException[Read more](/web/20220628130639/https://www.baeldung.com/spring-beancreationexception) →

## 2。原因:没有为依赖关系找到符合条件的[…]类型的 Bean

该异常最常见的原因是试图注入一个未定义的 bean。

例如，`BeanB`正在连接一个协作者，`BeanA`:

```java
@Component
public class BeanA {

    @Autowired
    private BeanB dependency;
    //...
}
```

现在，如果依赖性`BeanB`没有在 Spring 上下文中定义，引导过程将失败，并出现**没有这样的 bean 定义异常**:

```java
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [com.baeldung.packageB.BeanB]
  found for dependency: 
expected at least 1 bean which qualifies as
  autowire candidate for this dependency. 
Dependency annotations: 
  {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

原因用 Spring 明确表示:`expected at least 1 bean which qualifies as autowire candidate for this dependency`。

一个原因`BeanB`可能在上下文中不存在——如果 bean 被**类路径扫描**自动拾取，并且如果`BeanB`被正确地注释为 bean ( `@Component`、`@Repository`、`@Service`、`@Controller`等)。)—它可能被定义在**一个不被 Spring** 扫描的包中:

```java
package com.baeldung.packageB;
@Component
public class BeanB { ...}
```

类路径扫描可以配置如下:

```java
@Configuration
@ComponentScan("com.baeldung.packageA")
public class ContextWithJavaConfig {
    ...
}
```

如果 beans 没有被自动扫描，而是手动定义了**，那么`BeanB`在当前的 Spring 上下文中根本没有定义。**

 **## 3。原因:[…]中的字段[…]需要一个类型为[…]的 Bean，但找不到该 Bean

在上述场景的 Spring Boot 应用程序中，我们得到了不同的消息。

让我们举一个同样的例子，其中`BeanB`被连接到`BeanA`中，但是它没有被定义:

```java
@Component
public class BeanA {

    @Autowired
    private BeanB dependency;
    //...
}
```

如果我们尝试运行这个简单的应用程序，它会尝试加载`BeanA`:

```java
@SpringBootApplication
public class NoSuchBeanDefinitionDemoApp {

    public static void main(String[] args) {
        SpringApplication.run(NoSuchBeanDefinitionDemoApp.class, args);
    }
}
```

应用程序将无法启动，并显示以下错误消息:

```java
***************************
APPLICATION FAILED TO START
***************************

Description:

Field dependency in com.baeldung.springbootmvc.nosuchbeandefinitionexception.BeanA required a bean of type 'com.baeldung.springbootmvc.nosuchbeandefinitionexception.BeanB' that could not be found.

Action:

Consider defining a bean of type 'com.baeldung.springbootmvc.nosuchbeandefinitionexception.BeanB' in your configuration.
```

这里的`com.baeldung.springbootmvc.nosuchbeandefinitionexception`是`BeanA`、`BeanB`和`NoSuchBeanDefinitionDemoApp`的套餐。

这个例子的片段可以在 GitHub 项目中找到。

## 4。原因:没有定义类型为[…]的合格 Bean

该异常的另一个原因是上下文中存在两个 bean 定义，而不是一个。

假设一个接口`IBeanB`由两个 bean`BeanB1`和`BeanB2`实现:

```java
@Component
public class BeanB1 implements IBeanB {
    //
}
@Component
public class BeanB2 implements IBeanB {
    //
}
```

现在，如果自动连接这个接口，Spring 将不知道注入两个实现中的哪一个:

```java
@Component
public class BeanA {

    @Autowired
    private IBeanB dependency;
    ...
}
```

同样，这将导致`BeanFactory`抛出一个`NoSuchBeanDefinitionException`:

```java
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type 
  [com.baeldung.packageB.IBeanB] is defined: 
expected single matching bean but found 2: beanB1,beanB2
```

同样，Spring 明确指出了接线故障的原因:`expected single matching bean but found 2`。

然而，请注意，在这种情况下，抛出的确切异常不是`NoSuchBeanDefinitionException`，而是一个子类:**`NoUniqueBeanDefinitionException`。**这个新的异常是在 Spring 3.2.1 中引入的[,正是因为这个原因——区分没有找到 bean 定义的原因和在上下文中找到几个定义的原因。](https://web.archive.org/web/20220628130639/https://jira.springsource.org/browse/SPR-10194 " distinguish "none found" from "several found" in NoSuchBeanDefinitionException")

在此更改之前，这是上面的异常:

```java
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [com.baeldung.packageB.IBeanB] is defined: 
expected single matching bean but found 2: beanB1,beanB2
```

这个问题的一个**解决方案是使用`@Qualifier`注释**来准确指定我们想要连接的 bean 的名称:

```java
@Component
public class BeanA {

    @Autowired
    @Qualifier("beanB2")
    private IBeanB dependency;
    ...
}
```

现在 Spring 有足够的信息来决定注入哪个 bean—`BeanB1`还是`BeanB2`(`BeanB2`的默认名称是`beanB2`)。

## 5。原因:没有定义名为[…]的 Bean】

当一个未定义的 bean 被 Spring 上下文中的名称请求时，也会抛出一个`NoSuchBeanDefinitionException`:

```java
@Component
public class BeanA implements InitializingBean {

    @Autowired
    private ApplicationContext context;

    @Override
    public void afterPropertiesSet() {
        context.getBean("someBeanName");
    }
}
```

在这种情况下，没有“someBeanName”的 bean 定义，导致以下异常:

```java
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No bean named 'someBeanName' is defined
```

再次，Spring 清晰而简洁地指出了失败的原因:`No bean named X is defined`。

## 6。原因:代理的 bean

当使用 JDK 动态代理机制代理上下文中的 bean 时，**代理将不会扩展目标 bean** (但是它将实现相同的接口)。

因此，如果 bean 是通过接口注入的，它将被正确连接。但是，如果 bean 是由实际的类注入的，Spring 将不会找到与该类匹配的 bean 定义，因为代理实际上并没有扩展该类。

bean 可能被代理的一个非常常见的原因是 **Spring 事务支持**，即被标注了`@Transactional`的 bean。

例如，如果`ServiceA`注入`ServiceB`，并且两个服务都是事务性的，那么通过类定义注入的**将不起作用:**

```java
@Service
@Transactional
public class ServiceA implements IServiceA{

    @Autowired
    private ServiceB serviceB;
    ...
}

@Service
@Transactional
public class ServiceB implements IServiceB{
    ...
}
```

同样的两个服务，这次由接口正确地**注入，将会正常:**

```java
@Service
@Transactional
public class ServiceA implements IServiceA{

    @Autowired
    private IServiceB serviceB;
    ...
}

@Service
@Transactional
public class ServiceB implements IServiceB{
    ...
}
```

## 7。结论

本文讨论了常见`NoSuchBeanDefinitionException`的可能原因的示例，重点是如何在实践中处理这些异常。

所有这些异常实例的实现都可以在 GitHub 项目中找到。这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。

最后，Spring 中的 [**异常和解决方案**](/web/20220628130639/https://www.baeldung.com/spring-exceptions) 的完整列表可能是一个很好的书签资源。**