# Spring 中的上下文路径与 Servlet 路径

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-context-vs-servlet-path>

## 1.介绍

在 Spring 应用程序中扮演着重要的角色，并为应用程序提供了一个单一的入口点。而上下文路径定义了最终用户将访问应用程序的 URL。

在本教程中，我们将了解上下文路径和 servlet 路径之间的区别。

## 2.上下文路径

简单地说，上下文路径是访问 web 应用程序时使用的名称。它是应用程序的根。默认情况下，Spring Boot 在根上下文路径(“/”)上提供内容。

因此，任何具有默认配置的引导应用程序都可以通过以下方式进行访问:

```
http://localhost:8080/
```

然而，在某些情况下，我们可能希望改变应用程序的上下文。[配置上下文路径](/web/20220625073830/https://www.baeldung.com/spring-boot-context-path)有多种方式，`application.properties`是其中一种。该文件位于`src/main/resources`文件夹下。

让我们使用`application.properties` 文件来配置它:

```
server.servlet.context-path=/demo
```

因此，应用程序主页将是:

```
http://localhost:8080/demo
```

当我们将这个应用程序部署到外部服务器时，这种修改有助于我们避免可访问性问题。

## 3.Servlet 路径

**servlet path 代表主 [`DispatcherServlet`](/web/20220625073830/https://www.baeldung.com/spring-dispatcherservlet)** 的路径。`DispatcherServlet`是一个实际的 [`Servlet`](/web/20220625073830/https://www.baeldung.com/intro-to-servlets) ，它继承自`HttpSerlvet`基类。**默认值类似于上下文路径，即(“/”):**

```
spring.mvc.servlet.path=/
```

在 Boot 的早期版本中，该属性位于`ServerProperties`类中，称为`server.servlet-path=/`。

从 2.1.x 开始，这个属性被移动到`WebMvcProperties`类，并被重命名为`spring.mvc.servlet.path=/`。

让我们修改 servlet 路径:

```
spring.mvc.servlet.path=/baeldung
```

**因为一个 servlet 属于一个 servlet 上下文，改变上下文路径也会影响 servlet 路径**。因此，修改后，应用程序 servlet 路径将变成`http://localhost:8080/demo/baeldung.`

换句话说，如果一个样式表被用作`http://localhost:8080/demo/style.css,` 现在将被用作 `http://localhost:8080/demo/baeldung/style.css.`

通常情况下，我们不会自己配置 DispatcherServlet。但是，如果我们真的需要这样做，我们必须提供我们自定义的路径`DispatcherServlet`。

## 4.结论

在这篇简短的文章中，我们研究了上下文路径和 servlet 路径的语义。我们还看到了这些术语代表什么，以及它们如何在 Spring 应用程序中协同工作。