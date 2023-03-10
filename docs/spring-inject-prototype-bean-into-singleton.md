# 在 Spring 中将原型 Beans 注入到单例实例中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-inject-prototype-bean-into-singleton>

## 1。概述

在这篇简短的文章中，我们将展示**将原型 beans 注入单例实例**的不同方法。我们将讨论用例以及每个场景的优缺点。

默认情况下，Spring beans 是单例的。当我们试图连接不同作用域的 beans 时，问题就出现了。例如，一个原型 bean 到一个单例。**这就是所谓的** **范围内的 bean 注入问题**。

为了学习更多关于 bean 作用域的知识，[这篇文章是一个开始](/web/20220611184746/https://www.baeldung.com/spring-bean-scopes)的好地方。

## 2。 **原型注豆问题**

为了描述这个问题，让我们配置以下 beans:

```java
@Configuration
public class AppConfig {

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public PrototypeBean prototypeBean() {
        return new PrototypeBean();
    }

    @Bean
    public SingletonBean singletonBean() {
        return new SingletonBean();
    }
}
```

注意，第一个 bean 有一个原型范围，另一个是单例的。

现在，让我们将原型范围的 bean 注入到 singleton 中——然后通过`getPrototypeBean()`方法公开 if:

```java
public class SingletonBean {

    // ..

    @Autowired
    private PrototypeBean prototypeBean;

    public SingletonBean() {
        logger.info("Singleton instance created");
    }

    public PrototypeBean getPrototypeBean() {
        logger.info(String.valueOf(LocalTime.now()));
        return prototypeBean;
    }
}
```

然后，让我们加载`ApplicationContext`并两次获取单例 bean:

```java
public static void main(String[] args) throws InterruptedException {
    AnnotationConfigApplicationContext context 
      = new AnnotationConfigApplicationContext(AppConfig.class);

    SingletonBean firstSingleton = context.getBean(SingletonBean.class);
    PrototypeBean firstPrototype = firstSingleton.getPrototypeBean();

    // get singleton bean instance one more time
    SingletonBean secondSingleton = context.getBean(SingletonBean.class);
    PrototypeBean secondPrototype = secondSingleton.getPrototypeBean();

    isTrue(firstPrototype.equals(secondPrototype), "The same instance should be returned");
}
```

以下是控制台的输出:

```java
Singleton Bean created
Prototype Bean created
11:06:57.894
// should create another prototype bean instance here
11:06:58.895
```

**两个 beans 都只初始化了一次，**在应用程序上下文启动时。

## 3。注射`ApplicationContext`

我们也可以将`ApplicationContext`直接注入到 bean 中。

**要实现这一点，要么使用`@Autowire`注释，要么实现`ApplicationContextAware`接口:**

```java
public class SingletonAppContextBean implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public PrototypeBean getPrototypeBean() {
        return applicationContext.getBean(PrototypeBean.class);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) 
      throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

每次调用`getPrototypeBean()`方法，都会从`ApplicationContext`返回一个新的`PrototypeBean`实例。

然而，这种方法有严重的缺点。它违背了控制反转的原则，因为我们直接从容器中请求依赖关系。

此外，我们从`SingletonAppcontextBean`类中的`applicationContext`获取原型 bean。**这意味着** **将代码耦合到 Spring 框架**。

## 4。方法注入

另一个解决问题的方法是用 **`@Lookup`标注**的方法注入:

```java
@Component
public class SingletonLookupBean {

    @Lookup
    public PrototypeBean getPrototypeBean() {
        return null;
    }
}
```

Spring 将覆盖用`@Lookup.`注释的`getPrototypeBean()`方法，然后将 bean 注册到应用程序上下文中。每当我们请求`getPrototypeBean()`方法时，它都会返回一个新的`PrototypeBean`实例。

**它将使用 CGLIB 生成字节码**,负责从应用程序上下文中获取`PrototypeBean`。

## 5。`javax.inject` API

在这篇 [Spring wiring](/web/20220611184746/https://www.baeldung.com/spring-annotations-resource-inject-autowire) 文章中描述了设置以及所需的依赖关系。

这是单例 bean:

```java
public class SingletonProviderBean {

    @Autowired
    private Provider<PrototypeBean> myPrototypeBeanProvider;

    public PrototypeBean getPrototypeInstance() {
        return myPrototypeBeanProvider.get();
    }
}
```

我们使用`Provider` `interface`来注入原型 bean。对于每个`getPrototypeInstance()`方法调用，`myPrototypeBeanProvider.` g `et()`方法返回一个新的`PrototypeBean`实例。

## 6。作用域代理

默认情况下，Spring 保存一个对真实对象的引用来执行注入。这里，我们创建一个代理对象来连接真实对象和依赖对象。

每次调用代理对象上的方法时，代理都会自己决定是创建真实对象的新实例还是重用现有实例。

为此，我们修改了`Appconfig`类来添加一个新的`@Scope`注释:

```java
@Scope(
  value = ConfigurableBeanFactory.SCOPE_PROTOTYPE, 
  proxyMode = ScopedProxyMode.TARGET_CLASS)
```

默认情况下，Spring 使用 CGLIB 库直接对对象进行子类化。为了避免使用 CGLIB，我们可以用`ScopedProxyMode.`接口配置代理模式，使用 JDK 动态代理。

## 7。`ObjectFactory`界面

Spring 提供了[object factory<T>接口来按需生成给定类型的对象:](https://web.archive.org/web/20220611184746/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/ObjectFactoryCreatingFactoryBean.html)

```java
public class SingletonObjectFactoryBean {

    @Autowired
    private ObjectFactory<PrototypeBean> prototypeBeanObjectFactory;

    public PrototypeBean getPrototypeInstance() {
        return prototypeBeanObjectFactory.getObject();
    }
}
```

我们来看看`getPrototypeInstance()`法；`getObject()`为每个请求返回一个全新的`PrototypeBean`实例。在这里，我们对原型的初始化有更多的控制。

同样，`ObjectFactory`是框架的一部分；这意味着避免为了使用该选项而进行额外的设置。

## 8。使用`java.util.Function` 在运行时创建一个 Bean

另一个选择是在运行时创建原型 bean 实例，这也允许我们向实例添加参数。

为了查看这方面的示例，让我们向我们的`PrototypeBean`类添加一个名称字段:

```java
public class PrototypeBean {
    private String name;

    public PrototypeBean(String name) {
        this.name = name;
        logger.info("Prototype instance " + name + " created");
    }

    //...   
}
```

接下来，我们将通过使用`java.util.Function`接口将 bean 工厂注入到我们的 singleton bean 中:

```java
public class SingletonFunctionBean {

    @Autowired
    private Function<String, PrototypeBean> beanFactory;

    public PrototypeBean getPrototypeInstance(String name) {
        PrototypeBean bean = beanFactory.apply(name);
        return bean;
    }

}
```

最后，我们必须在配置中定义工厂 bean、原型 bean 和独立 bean:

```java
@Configuration
public class AppConfig {
    @Bean
    public Function<String, PrototypeBean> beanFactory() {
        return name -> prototypeBeanWithParam(name);
    } 

    @Bean
    @Scope(value = "prototype")
    public PrototypeBean prototypeBeanWithParam(String name) {
       return new PrototypeBean(name);
    }

    @Bean
    public SingletonFunctionBean singletonFunctionBean() {
        return new SingletonFunctionBean();
    }
    //...
}
```

## 9。测试

现在让我们编写一个简单的 JUnit 测试来测试使用`ObjectFactory`接口的情况:

```java
@Test
public void givenPrototypeInjection_WhenObjectFactory_ThenNewInstanceReturn() {

    AbstractApplicationContext context
     = new AnnotationConfigApplicationContext(AppConfig.class);

    SingletonObjectFactoryBean firstContext
     = context.getBean(SingletonObjectFactoryBean.class);
    SingletonObjectFactoryBean secondContext
     = context.getBean(SingletonObjectFactoryBean.class);

    PrototypeBean firstInstance = firstContext.getPrototypeInstance();
    PrototypeBean secondInstance = secondContext.getPrototypeInstance();

    assertTrue("New instance expected", firstInstance != secondInstance);
}
```

在成功启动测试后，我们可以看到每次调用`getPrototypeInstance()`方法时，都会创建一个新的原型 bean 实例。

## 10。结论

在这个简短的教程中，我们学习了几种将原型 bean 注入单体实例的方法。

一如既往，本教程的完整代码可以在 GitHub 项目中找到。