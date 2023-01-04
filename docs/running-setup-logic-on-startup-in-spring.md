# Spring 启动时运行逻辑指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/running-setup-logic-on-startup-in-spring>

## 1。概述

在本教程中，我们将关注如何在 Spring 应用程序启动时运行逻辑。

## 延伸阅读:

## [配置 Spring Boot 网络应用](/web/20220707143743/https://www.baeldung.com/spring-boot-application-configuration)

Some of the more useful configs for a Spring Boot application.[Read more](/web/20220707143743/https://www.baeldung.com/spring-boot-application-configuration) →

## [Spring Boot:配置一个主类](/web/20220707143743/https://www.baeldung.com/spring-boot-main-class)

Learn how to configure your Spring Boot application's main class in Maven and Gradle.[Read more](/web/20220707143743/https://www.baeldung.com/spring-boot-main-class) →

## 2。启动时运行逻辑

在 Spring 应用程序启动期间/之后运行逻辑是一个常见的场景。但这也是导致许多问题的原因之一。

为了从反向控制中获益，我们需要放弃对应用程序流向容器的部分控制。这就是为什么实例化、启动时设置逻辑等。需要特别注意。

在实例化任何对象后，我们不能简单地将我们的逻辑包含在 beans 的构造函数或调用方法中，因为在这些过程中我们不受控制。

让我们看一个现实生活中的例子:

```
@Component
public class InvalidInitExampleBean {

    @Autowired
    private Environment env;

    public InvalidInitExampleBean() {
        env.getActiveProfiles();
    }
}
```

这里我们试图访问构造函数中的一个`autowired`字段。当调用构造函数时，Spring bean 还没有完全初始化。这是一个问题，因为**调用尚未初始化的字段会导致`NullPointerException` s.**

让我们看看 Spring 为我们提供的几种处理这种情况的方法。

### 2.1。`@PostConstruct`注解

我们可以使用 Javax 的 `@PostConstruct`注释来注释应该在 bean 初始化后立即运行**一次的方法。**请记住，即使没有注入任何东西，Spring 也会运行带注释的方法。

下面是`@PostConstruct`的行动:

```
@Component
public class PostConstructExampleBean {

    private static final Logger LOG 
      = Logger.getLogger(PostConstructExampleBean.class);

    @Autowired
    private Environment environment;

    @PostConstruct
    public void init() {
        LOG.info(Arrays.asList(environment.getDefaultProfiles()));
    }
}
```

我们可以看到，`Environment`实例被安全地注入，然后在`@PostConstruct`带注释的方法中被调用，而没有抛出`NullPointerException`。

### 2.2。`InitializingBean` 界面

`InitializingBean`方法以类似的方式工作。我们需要实现`InitializingBean`接口和`afterPropertiesSet()`方法，而不是注释一个方法。

这里我们使用`InitializingBean`接口实现了前面的例子:

```
@Component
public class InitializingBeanExampleBean implements InitializingBean {

    private static final Logger LOG 
      = Logger.getLogger(InitializingBeanExampleBean.class);

    @Autowired
    private Environment environment;

    @Override
    public void afterPropertiesSet() throws Exception {
        LOG.info(Arrays.asList(environment.getDefaultProfiles()));
    }
}
```

### 2.3。一个`ApplicationListener`

在 Spring 上下文初始化之后，我们可以使用这种方法来运行逻辑。因此，我们并不关注任何特定的豆子。相反，我们在等待它们全部初始化。

为此，我们需要创建一个实现`ApplicationListener<ContextRefreshedEvent>`接口的 bean:

```
@Component
public class StartupApplicationListenerExample implements 
  ApplicationListener<ContextRefreshedEvent> {

    private static final Logger LOG 
      = Logger.getLogger(StartupApplicationListenerExample.class);

    public static int counter;

    @Override public void onApplicationEvent(ContextRefreshedEvent event) {
        LOG.info("Increment counter");
        counter++;
    }
} 
```

我们可以通过使用新引入的`@EventListener`注释得到相同的结果:

```
@Component
public class EventListenerExampleBean {

    private static final Logger LOG 
      = Logger.getLogger(EventListenerExampleBean.class);

    public static int counter;

    @EventListener
    public void onApplicationEvent(ContextRefreshedEvent event) {
        LOG.info("Increment counter");
        counter++;
    }
}
```

我们想确保根据我们的需求选择一个合适的活动。在这个例子中，我们选择了`ContextRefreshedEvent`。

### 2.4。`@Bean` `initMethod`属性

我们可以使用` initMethod`属性在 bean 初始化后运行一个方法。

这是一颗豆子的样子:

```
public class InitMethodExampleBean {

    private static final Logger LOG = Logger.getLogger(InitMethodExampleBean.class);

    @Autowired
    private Environment environment;

    public void init() {
        LOG.info(Arrays.asList(environment.getDefaultProfiles()));
    }
}
```

请注意，我们没有实现任何特殊的接口或使用任何特殊的注释。

然后我们可以使用`@Bean`注释来定义 bean:

```
@Bean(initMethod="init")
public InitMethodExampleBean initMethodExampleBean() {
    return new InitMethodExampleBean();
}
```

XML 配置中的 bean 定义是这样的:

```
<bean id="initMethodExampleBean"
  class="com.baeldung.startup.InitMethodExampleBean"
  init-method="init">
</bean>
```

### 2.5。构造函数注入

如果我们使用构造函数注入来注入字段，我们可以简单地在构造函数中包含我们的逻辑:

```
@Component 
public class LogicInConstructorExampleBean {

    private static final Logger LOG 
      = Logger.getLogger(LogicInConstructorExampleBean.class);

    private final Environment environment;

    @Autowired
    public LogicInConstructorExampleBean(Environment environment) {
        this.environment = environment;
        LOG.info(Arrays.asList(environment.getDefaultProfiles()));
    }
}
```

### 2.6。Spring Boot `CommandLineRunner`

Spring Boot 提供了一个带有回调`run()`方法的`CommandLineRunner`接口。这可以在 Spring 应用程序上下文实例化之后，在应用程序启动时调用。

让我们看一个例子:

```
@Component
public class CommandLineAppStartupRunner implements CommandLineRunner {
    private static final Logger LOG =
      LoggerFactory.getLogger(CommandLineAppStartupRunner.class);

    public static int counter;

    @Override
    public void run(String...args) throws Exception {
        LOG.info("Increment counter");
        counter++;
    }
}
```

**注意**:正如[文档](https://web.archive.org/web/20220707143743/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html)中提到的，多个`CommandLineRunner`bean 可以在同一个应用程序上下文中定义，并且可以使用`@Ordered`接口或`@Order`注释进行排序。

### 2.7。Spring Boot `ApplicationRunner`

与`CommandLineRunner`类似，Spring Boot 也提供了一个带有`run()`方法的`ApplicationRunner` 接口，在应用程序启动时被调用。然而，我们没有将原始的`String`参数传递给回调方法，而是有了一个 [`ApplicationArguments`](https://web.archive.org/web/20220707143743/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationArguments.html) 类的实例。

`ApplicationArguments` 接口有获取参数值的方法，这些参数值是选项和普通参数值。前缀为–的参数是选项参数。

让我们看一个例子:

```
@Component
public class AppStartupRunner implements ApplicationRunner {
    private static final Logger LOG =
      LoggerFactory.getLogger(AppStartupRunner.class);

    public static int counter;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        LOG.info("Application started with option names : {}", 
          args.getOptionNames());
        LOG.info("Increment counter");
        counter++;
    }
}
```

## 3。组合机构

为了完全控制我们的 beans，我们可以将上述机制结合在一起。

执行顺序如下:

1.  构造器
2.  `@PostConstruct`注释方法
3.  初始化 Bean 的`afterPropertiesSet()`方法
4.  在 XML 中被指定为`init-method`的初始化方法

让我们创建一个结合了所有机制的 Spring bean:

```
@Component
@Scope(value = "prototype")
public class AllStrategiesExampleBean implements InitializingBean {

    private static final Logger LOG 
      = Logger.getLogger(AllStrategiesExampleBean.class);

    public AllStrategiesExampleBean() {
        LOG.info("Constructor");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        LOG.info("InitializingBean");
    }

    @PostConstruct
    public void postConstruct() {
        LOG.info("PostConstruct");
    }

    public void init() {
        LOG.info("init-method");
    }
}
```

如果我们尝试实例化这个 bean，我们可以看到与上面指定的顺序相匹配的日志:

```
[main] INFO o.b.startup.AllStrategiesExampleBean - Constructor
[main] INFO o.b.startup.AllStrategiesExampleBean - PostConstruct
[main] INFO o.b.startup.AllStrategiesExampleBean - InitializingBean
[main] INFO o.b.startup.AllStrategiesExampleBean - init-method
```

## 4。结论

在本文中，我们展示了在 Spring 应用程序启动时运行逻辑的多种方式。

代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143743/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data)