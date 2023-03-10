# web.xml vs 带有 Spring 的初始化器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-xml-vs-java-config>

## 1。概述

在本文中，我们将介绍在最近版本的`Spring Framework:`中配置`DispatcherServlet` 的三种不同方法

1.  我们将从一个`XML`配置和一个`web.xml`文件开始
2.  然后，我们将 Servlet 声明从`web.xml`文件迁移到 Java config，但是我们将把任何其他配置留在`XML`中
3.  最后，在重构的第三步，也是最后一步，我们将有一个 100% Java 配置的项目

## 2。`DispatcherServlet`

`Spring MVC`的核心概念之一是`DispatcherServlet`。 [Spring 文档](https://web.archive.org/web/20220625174414/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)将其定义为:

> HTTP 请求处理器/控制器的中央调度程序，例如 web UI 控制器或基于 HTTP 的远程服务导出程序。分派给已注册的处理程序来处理 web 请求，提供方便的映射和异常处理功能。

基本上，`DispatcherServlet`是每个`Spring MVC`应用程序的入口点。它的目的是拦截`HTTP`请求，并将它们分派给知道如何处理它的正确组件。

## 3。配置有`w` `eb.xml`

如果你处理遗留的`Spring`项目，找到`XML`配置是很常见的，直到`Spring` 3.1，配置`DispatcherServlet`的唯一方法是使用`WEB-INF/web.xml`文件。在这种情况下，需要两步。

让我们来看一个配置示例——第一步是 Servlet 声明:

```java
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/dispatcher-config.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

通过这个`XML`块，我们声明了一个 servlet，它:

1.  被命名为`dispatcher`
2.  是`org.springframework.web.servlet.DispatcherServlet`的一个实例
3.  将用名为`contextConfigLocation`的参数初始化，该参数包含配置`XML`的路径

`load-on-startup`是一个整数值，指定多个 servlets 的加载顺序。因此，如果您需要声明多个 servlet，您可以定义它们的初始化顺序。用较小整数标记的 servlet 在用较大整数标记的 servlet 之前加载。

现在我们的 servlet 已经配置好了。第二步是声明一个`servlet-mapping`:

```java
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

通过 servlet 映射，我们将它的名字绑定到一个`URL` `pattern`，指定它将处理什么`HTTP`请求。

## 4。混合动力配置

随着`Servlet APIs`3.0 版本的采用，`web.xml`文件变得可选，我们现在可以使用 Java 来配置`DispatcherServlet`。

我们可以注册一个实现了`WebApplicationInitializer`的 servlet。这相当于上面的`XML`配置:

```java
public class MyWebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext context = new XmlWebApplicationContext();
        context.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic dispatcher = container
          .addServlet("dispatcher", new DispatcherServlet(context));

        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");
    }
}
```

在本例中，我们是:

1.  实现`WebApplicationInitializer`接口
2.  通过覆盖`onStartup`方法，我们创建了一个新的`XmlWebApplicationContext`,它配置了与`XML`示例中传递给 servlet 的`contextConfigLocation`相同的文件
3.  然后我们用刚刚实例化的新上下文创建一个`DispatcherServlet`的实例
4.  最后，我们用映射`URL` `pattern`注册 servlet

所以我们使用`Java`来声明 servlet 并将其绑定到`URL mapping`，但是我们将配置保存在一个单独的`XML`文件中:`dispatcher-config.xml`。

## 5。100% `Java`配置

使用这种方法，我们的 servlet 是用 Java `,`声明的，但是我们仍然需要一个`XML`文件来配置它。有了`WebApplicationInitializer` ，你可以实现 100% `Java` 的配置。

让我们看看如何重构前面的例子。

我们需要做的第一件事是为 servlet 创建应用程序上下文。

这一次我们将使用基于注释的上下文，这样我们可以使用`Java`和注释进行配置，并且不再需要像`dispatcher-config.xml`这样的`XML`文件:

```java
AnnotationConfigWebApplicationContext context
  = new AnnotationConfigWebApplicationContext();
```

然后，可以通过注册配置类来配置这种类型的上下文:

```java
context.register(AppConfig.class);
```

或者设置将被扫描配置类的整个包:

```java
context.setConfigLocation("com.example.app.config");
```

既然我们的应用程序上下文已经创建，我们可以向`ServletContext`添加一个监听器来加载上下文:

```java
container.addListener(new ContextLoaderListener(context));
```

下一步是创建和注册我们的 dispatcher servlet:

```java
ServletRegistration.Dynamic dispatcher = container
  .addServlet("dispatcher", new DispatcherServlet(context));

dispatcher.setLoadOnStartup(1);
dispatcher.addMapping("/");
```

现在我们的`WebApplicationInitializer`应该是这样的:

```java
public class MyWebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
        AnnotationConfigWebApplicationContext context
          = new AnnotationConfigWebApplicationContext();
        context.setConfigLocation("com.example.app.config");

        container.addListener(new ContextLoaderListener(context));

        ServletRegistration.Dynamic dispatcher = container
          .addServlet("dispatcher", new DispatcherServlet(context));

        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");
    }
}
```

`Java`注释配置提供了许多优势。通常这会导致更短、更简洁的配置，注释为声明提供了更多的上下文，因为它与它们配置的代码位于同一位置。

但是这并不总是一个更好的甚至是可能的方法。例如，一些开发人员可能喜欢将他们的代码和配置分开，或者您可能需要使用您无法修改的第三方代码。

## 6。结论

在本文中，我们介绍了在`Spring 3.2+`中配置`DispatcherServlet`的不同方法，您可以根据自己的喜好决定使用哪一种。`Spring` 无论你选择什么，都会顺从你的决定。

你可以在 Github [这里](https://web.archive.org/web/20220625174414/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java)和[这里](https://web.archive.org/web/20220625174414/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-xml)找到这篇文章的源代码。