# WebJars 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-webjars>

## 1。概述

本教程介绍了 WebJars 以及如何在 Java 应用程序中使用它们。

简单地说，WebJars 是打包到 JAR 归档文件中的客户端依赖项。它们适用于大多数 JVM 容器和 web 框架。

下面是几个流行的 webjar:`Twitter Bootstrap`、`jQuery`、`Angular JS`、`Chart.js`等；完整的名单可以在[官方网站](https://web.archive.org/web/20220812053352/http://www.webjars.org/)上找到。

## 2。为什么要使用 WebJars？

这个问题有一个非常简单的答案——因为它很容易。

手动添加和管理客户端依赖关系通常会导致难以维护代码库。

此外，大多数 Java 开发人员更喜欢使用 Maven 和 Gradle 作为构建和依赖管理工具。

WebJars 解决的主要问题是使客户端依赖关系在 Maven Central 上可用，并可用于任何标准的 Maven 项目。

以下是 WebJars 的一些有趣的优点:

1.  在基于 JVM 的 web 应用程序中，我们可以显式且轻松地管理客户端依赖性
2.  我们可以将它们与任何常用的构建工具一起使用，例如:Maven、Gradle 等
3.  WebJars 的行为类似于任何其他 Maven 依赖——这意味着我们也获得了可传递的依赖

## 3。Maven 依赖关系

让我们直接进入主题，将 Twitter Bootstrap 和 jQuery 添加到`pom.xml` :

```java
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>3.3.7-1</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.1.1</version>
</dependency> 
```

现在 Twitter Bootstrap 和 jQuery 在项目类路径中可用；我们可以简单地引用它们并在我们的应用程序中使用它们。

注意:你可以在 Maven Central 上查看最新版本的 [Twitter Bootstrap](https://web.archive.org/web/20220812053352/https://mvnrepository.com/artifact/org.webjars/bootstrap) 和 [jQuery](https://web.archive.org/web/20220812053352/https://mvnrepository.com/artifact/org.webjars/jquery) 依赖项。

## 4。简单的应用程序

定义了这两个 WebJar 依赖项之后，现在让我们建立一个简单的 Spring MVC 项目，以便能够使用客户端依赖项。

然而，在我们开始之前，理解**web jar 与 Spring** 没有任何关系是很重要的，我们在这里只使用 Spring，因为它是建立 MVC 项目的一种非常快速和简单的方法。

[这里是开始](/web/20220812053352/https://www.baeldung.com/thymeleaf-in-spring-mvc)设置 Spring MVC 和 Spring Boot 项目的好地方。

随着简单项目的建立，我们将为新的客户端依赖项定义一些映射:

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry
          .addResourceHandler("/webjars/**")
          .addResourceLocations("/webjars/");
    }
}
```

我们当然也可以通过 XML 做到这一点:

```java
<mvc:resources mapping="/webjars/**" location="/webjars/"/>
```

## 5.与版本无关的依赖项

当使用 Spring Framework 版或更高版本时，它将自动检测类路径上的`webjars-locator`库，并使用它自动解析任何 WebJars 资产的版本。

为了启用这个特性，我们将添加`webjars-locator`库作为应用程序的依赖项:

```java
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator</artifactId>
    <version>0.30</version>
</dependency>
```

在这种情况下，我们可以在不使用版本的情况下引用 WebJars 资产；请参阅下一节中的几个实例。

## 6。客户端上的 web jars

让我们在应用程序中添加一个简单的 HTML 欢迎页面(这是`index.html`):

```java
<html>
    <head>
        <title>WebJars Demo</title>
    </head>
    <body> 
    </body>
</html>
```

现在我们可以在项目中使用 Twitter Bootstrap 和 jQuery 让我们在欢迎页面中同时使用这两者，从 Bootstrap 开始:

```java
<script src="/webjars/bootstrap/3.3.7-1/js/bootstrap.min.js"></script>
```

对于版本无关的方法:

```java
<script src="/webjars/bootstrap/js/bootstrap.min.js"></script>
```

添加 jQuery:

```java
<script src="/webjars/jquery/3.1.1/jquery.min.js"></script>
```

与版本无关的方法:

```java
<script src="/webjars/jquery/jquery.min.js"></script>
```

## 7。测试

现在我们已经在 HTML 页面中添加了 Twitter Bootstrap 和 jQuery，让我们测试它们。

我们将在页面中添加一个引导程序`alert`:

```java
<div class="container"><br/>
    <div class="alert alert-success">         
        <strong>Success!</strong> It is working as we expected.
    </div>
</div> 
```

注意，这里假设了对 Twitter Bootstrap 的一些基本理解；以下是官方网站上的[入门指南](https://web.archive.org/web/20220812053352/https://getbootstrap.com/components/)。

这将显示一个如下所示的`alert` ，这意味着我们已经成功地将 Twitter 引导添加到我们的类路径中。

现在让我们使用 jQuery。我们将在此警报中添加一个关闭按钮:

```java
<a href="#" class="close" data-dismiss="alert" aria-label="close">×</a> 
```

现在我们需要为关闭按钮功能添加`jQuery`和`bootstrap.min.js` ，所以将它们添加到`index.html,` 的主体标签中，如下所示:

```java
<script src="/webjars/jquery/3.1.1/jquery.min.js"></script>
<script src="/webjars/bootstrap/3.3.7-1/js/bootstrap.min.js"></script> 
```

注意:如果您使用版本无关的方法，请确保只从路径中删除版本，否则，相对导入可能无法工作:

```java
<script src="/webjars/jquery/jquery.min.js"></script>
<script src="/webjars/bootstrap/js/bootstrap.min.js"></script>
```

这是我们最终欢迎页面的外观:

```java
<html>
    <head>
        <script src="/webjars/jquery/3.1.1/jquery.min.js"></script>
        <script src="/webjars/bootstrap/3.3.7-1/js/bootstrap.min.js"></script>
        <title>WebJars Demo</title>
        <link rel="stylesheet" 
          href="/webjars/bootstrap/3.3.7-1/css/bootstrap.min.css" />
    </head>
    <body>
        <div class="container"><br/>
            <div class="alert alert-success">
                <a href="#" class="close" data-dismiss="alert" 
                  aria-label="close">×</a>
                <strong>Success!</strong> It is working as we expected.
            </div>
        </div>
    </body>
</html>
```

这就是应用程序应该看起来的样子。(单击关闭按钮时，警告应该会消失。)

[![webjarsdemo](img/dc2db785a707cbd9b3ec9b1189a059bc.png)](/web/20220812053352/https://www.baeldung.com/wp-content/uploads/2016/10/webjarsdemo.jpg)

## 8。结论

在这篇简短的文章中，我们关注了在基于 JVM 的项目中使用 WebJars 的基础知识，这使得开发和维护变得更加容易。

我们实现了一个 Spring Boot 支持的项目，并在使用 WebJars 的项目中使用了 Twitter Bootstrap 和 jQuery。

上面使用的例子的源代码可以在 [Github 项目](https://web.archive.org/web/20220812053352/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-artifacts)中找到——这是一个 Maven 项目，因此应该很容易导入和构建。