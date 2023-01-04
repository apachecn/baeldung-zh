# Spring @Lazy 注释快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-lazy-annotation>

## 1。概述

默认情况下，Spring 会在应用程序上下文启动/引导时急切地创建所有单例 beans。这背后的原因很简单:立即避免和检测所有可能的错误，而不是在运行时。

然而，有时我们需要创建一个 bean，不是在应用程序上下文启动时，而是在我们请求时。

**在这个快速教程中，我们将讨论 Spring 的`@Lazy`注释。**

## 2.惰性初始化

从 Spring 版本开始，`@Lazy`注释就存在了。有几种方法可以告诉 IoC 容器延迟初始化 bean。

### 2.1。`@Configuration`阶级

**当我们将`@Lazy`注释放在`@Configuration`类上时，表示所有带有`@Bean`注释的方法都应该被惰性加载**。

这相当于基于 XML 的配置的`default-lazy-init=`true`“`属性。

让我们看看这里:

```
@Lazy
@Configuration
@ComponentScan(basePackages = "com.baeldung.lazy")
public class AppConfig {

    @Bean
    public Region getRegion(){
        return new Region();
    }

    @Bean
    public Country getCountry(){
        return new Country();
    }
}
```

现在让我们测试功能:

```
@Test
public void givenLazyAnnotation_whenConfigClass_thenLazyAll() {

    AnnotationConfigApplicationContext ctx
     = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class);
    ctx.refresh();
    ctx.getBean(Region.class);
    ctx.getBean(Country.class);
}
```

正如我们所看到的，所有的 beans 都是在我们第一次请求它们时创建的:

```
Bean factory for ...AnnotationConfigApplicationContext: 
...DefaultListableBeanFactory: [...];
// application context started
Region bean initialized
Country bean initialized
```

为了仅将此应用于特定的 bean，让我们从一个类中移除`@Lazy`。

然后我们将它添加到所需 bean 的配置中:

```
@Bean
@Lazy(true)
public Region getRegion(){
    return new Region();
}
```

### 2.2。带@自动连线

在继续之前，查看这些指南中的`[@Autowired](/web/20220701012836/https://www.baeldung.com/spring-autowire)`和`[@Component](/web/20220701012836/https://www.baeldung.com/spring-bean-annotations)`注释。

这里，**为了初始化一个懒 bean，我们从另一个引用它。**

我们希望延迟加载的 bean:

```
@Lazy
@Component
public class City {
    public City() {
        System.out.println("City bean initialized");
    }
}
```

它的参考:

```
public class Region {

    @Lazy
    @Autowired
    private City city;

    public Region() {
        System.out.println("Region bean initialized");
    }

    public City getCityInstance() {
        return city;
    }
}
```

**注意，`@Lazy`在两个地方都是强制的。**

用`City`类上的`@Component`注释，同时用`@Autowired:`引用它

```
@Test
public void givenLazyAnnotation_whenAutowire_thenLazyBean() {
    // load up ctx appication context
    Region region = ctx.getBean(Region.class);
    region.getCityInstance();
}
```

这里，**`City`bean 仅在我们调用`getCityInstance()`方法时被初始化。**

## 3。结论

在这个快速教程中，我们学习了 Spring 的`@Lazy`注释的基础知识。我们研究了几种配置和使用它的方法。

像往常一样，该指南的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220701012836/https://github.com/eugenp/tutorials/tree/master/spring-core-5)