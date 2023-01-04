# Spring @Enable 注释快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-enable-annotations>

## 1。概述

Spring 附带了一组 **`@Enable`注释，使得开发人员更容易配置 Spring 应用程序**。这些注释与`@Configuration`注释一起使用**。**

在本文中，我们将会看到其中一些注释:

*   `@EnableWebMvc`
*   `@EnableCaching`
*   `@EnableScheduling`
*   `@EnableAsync`
*   `@EnableWebSocket`
*   `@EnableJpaRepositories`
*   `@EnableTransactionManagement`
*   `@EnableJpaAuditing`

## 2。`@EnableWebMvc`

`@EnableWebMvc`注释用于**在应用**中启用 Spring MVC，并通过从`WebMvcConfigurationSupport`导入 Spring MVC 配置来工作。

具有类似功能的 XML 等价物是`<mvc:annotation-driven/>.`

配置可以由实现`WebMvcConfigurer`的`@Configuration`类定制:

```
@Configuration
@EnableWebMvc
public class SpringMvcConfig implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(
      List<HttpMessageConverter<?>> converters) {

        converters.add(new MyHttpMessageConverter());
    }

    // ...
}
```

## 3。`@EnableCaching`

`@EnableCaching`注释**支持应用程序中注释驱动的缓存管理**功能，而**允许我们在应用程序中使用`@Cacheable`和`@CacheEvict`注释**。

具有类似功能的 XML 等价物是`<cache:*>`名称空间:

```
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(
          Arrays.asList(new ConcurrentMapCache("default")));
        return cacheManager;
    }
}
```

此注释还具有以下选项:

*   `**mode**` —指示应该如何应用高速缓存建议
*   `**order**` —表示在特定连接点应用执行高速缓存指导时的顺序
*   `**proxyTargetClass**` —指示是否要创建基于子类的(CGLIB)代理，而不是基于标准 Java 接口的代理

这个配置也可以由实现`CachingConfigurerSupport`类的`@Configuration`类定制:

```
@Configuration
@EnableCaching
public class CacheConfig extends CachingConfigurerSupport {

    @Bean
    @Override
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(
          Arrays.asList(new ConcurrentMapCache("default")));
        return cacheManager;
    }

    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return new MyKeyGenerator();
    }
}
```

关于使用 Spring 缓存的更多信息，你可以参考这篇文章。

## 4。`@EnableScheduling`

`@EnableScheduling`注释**支持预定任务功能，并允许我们在应用程序中使用`@Scheduled`注释**。具有类似功能的 XML 等价物是使用`scheduler`属性的`<task:*>`名称空间。

这个配置也可以由实现`SchedulingConfigurer`类的`@Configuration`类定制:

```
@Configuration
@EnableScheduling
public class SchedulingConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(
      ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(taskExecutor());
    }

    @Bean(destroyMethod = "shutdown")
    public Executor taskExecutor() {
        return Executors.newScheduledThreadPool(100);
    }
}
```

关于使用 Spring scheduling 的更多信息，你可以参考这篇[文章](/web/20220626194339/https://www.baeldung.com/spring-scheduled-tasks)。

## 5。`@EnableAsync`

在我们的应用程序中，`@EnableAsync`注释**支持异步处理。具有类似功能的 XML 等价物是使用`executor`属性的`<task:*>`名称空间。**

```
@Configuration
@EnableAync
public class AsyncConfig { ... }
```

关于使用 Spring async 的更多内容，可以参考这篇[文章](/web/20220626194339/https://www.baeldung.com/spring-async)。

## 6。`@EnableWebSocket`

`@EnableWebSocket`注释用于**配置 web socket 请求**的处理。定制可以通过实现`WebSocketConfigurer`类来完成:

```
@Configuration
@EnableWebSocket
public class MyConfiguration implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(echoWebSocketHandler(), "/echo").withSockJS();
    }

    @Bean
    public WebSocketHandler echoWebSocketHandler() {
        return new EchoWebSocketHandler();
    }
}
```

关于使用 Spring Websockets 的更多信息，你可以参考这篇[文章](/web/20220626194339/https://www.baeldung.com/websockets-spring)。

## 7。`@EnableJpaRepositories`

`@EnableJpaRepositories`注释**通过扫描带注释的配置类包中的存储库来启用 Spring Data JPA 存储库**。

```
@Configuration
@EnableJpaRepositories
public class JpaConfig { ... }
```

此注释的一些可用选项包括:

*   `**value**`—`basePackages()`属性的别名
*   `**basePackages**` —扫描带注释组件的基础包
*   `**enableDefaultTransactions**` —配置是否为 Spring 数据 JPA 存储库启用默认事务
*   `**entityManagerFactoryRef**` —配置要使用的`EntityManagerFactory` bean 定义的名称

## 8。`@EnableTransactionManagement`

`@EnableTransactionManagement`注释**启用了 Spring 的注释驱动的事务管理能力**。XML 的对等物是`<tx:*>`名称空间。

```
@Configuration
@EnableTransactionManagement
public class JpaConfig { ... }
```

关于使用 Spring 事务管理的更多信息，您可以参考本文。

## 9。`@EnableJpaAuditing`

`@EnableJpaAuditing`注释**支持对 JPA 实体**进行审计。

```
@Configuration
@EnableJpaAuditing
public class JpaConfig {

    @Bean
    public AuditorAware<AuditableUser> auditorProvider() {
        return new AuditorAwareImpl();
    }
}
```

关于使用 Spring Web Sockets 的更多信息，你可以参考这篇文章。

## 10。结论

在这篇简短的文章中，我们看了一些`@Enable` Spring 注释，以及如何使用它们来帮助我们配置 Spring 应用程序。