# 春季关闭回调

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-shutdown-callbacks>

## 1。概述

在本教程中，我们将学习使用 Spring 关闭回调的不同方法**。**

使用关闭回调的主要优点是，它让我们能够控制一个优雅的应用程序退出。

## 2。关机回调方法

Spring 支持组件级和上下文级的关闭回调。我们可以使用以下方法创建这些回调:

*   `@PreDestroy`
*   `DisposableBean`界面
*   灭豆法
*   全球`ServletContextListener`

让我们看看所有这些方法的例子。

### 2.1。使用`@PreDestroy`

让我们创建一个使用 *@PreDestroy* 的 bean:

```
@Component
public class Bean1 {

    @PreDestroy
    public void destroy() {
        System.out.println(
          "Callback triggered - @PreDestroy.");
    }
}
```

在 bean 初始化期间，Spring 将注册所有用`@PreDestroy`标注的 bean 方法，并在应用程序关闭时调用它们。

### 2.2。使用`DisposableBean`界面

我们的第二个 bean 将实现`DisposableBean`接口来注册关闭回调:

```
@Component
public class Bean2 implements DisposableBean {

    @Override
    public void destroy() throws Exception {
        System.out.println(
          "Callback triggered - DisposableBean.");
    }
}
```

### 2.3。声明一个 Bean 销毁方法

对于这种方法，首先我们将创建一个带有自定义销毁方法的类:

```
public class Bean3 {

    public void destroy() {
        System.out.println(
          "Callback triggered - bean destroy method.");
    }
}
```

然后，我们创建初始化 bean 的配置类，并将它的`destroy()`方法标记为我们的关闭回调:

```
@Configuration
public class ShutdownHookConfiguration {

    @Bean(destroyMethod = "destroy")
    public Bean3 initializeBean3() {
        return new Bean3();
    }
}
```

注册销毁方法的 XML 方式是:

```
<bean class="com.baeldung.shutdownhooks.config.Bean3"
  destroy-method="destroy">
```

### 2.4。`ServletContextListener`使用全局

与其他三种在 bean 级别注册回调的方法不同，**`ServletContextListener`在上下文级别**注册回调。

为此，让我们创建一个自定义上下文监听器:

```
public class ExampleServletContextListener
  implements ServletContextListener {

    @Override
    public void contextDestroyed(ServletContextEvent event) {
        System.out.println(
          "Callback triggered - ContextListener.");
    }

    @Override
    public void contextInitialized(ServletContextEvent event) {
        // Triggers when context initializes
    }
}
```

我们需要将其注册到配置类中的`ServletListenerRegistrationBean`:

```
@Bean
ServletListenerRegistrationBean<ServletContextListener> servletListener() {
    ServletListenerRegistrationBean<ServletContextListener> srb
      = new ServletListenerRegistrationBean<>();
    srb.setListener(new ExampleServletContextListener());
    return srb;
}
```

## 3。结论

我们已经了解了 Spring 提供的注册关闭回调的不同方式，包括 bean 级和上下文级。

这些可以用来优雅地关闭应用程序并有效地释放已用资源。

和往常一样，本文中提到的所有例子都可以在 Github 上找到[。](https://web.archive.org/web/20220524112612/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-deployment)