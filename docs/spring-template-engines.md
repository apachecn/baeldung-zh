# Spring 的模板引擎

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-template-engines>

## 1。概述

Spring web 框架是围绕 MVC(模型-视图-控制器)模式构建的，这使得在应用程序中分离关注点变得更加容易。这允许使用不同的视图技术，从成熟的 JSP 技术到各种模板引擎。

在本文中，我们将看看可以与 Spring 一起使用的主要模板引擎、它们的配置以及使用示例。

## 2。Spring View 技术

鉴于 Spring MVC 应用程序中的关注点是完全分离的，从一种视图技术切换到另一种主要是配置的问题。

为了呈现每种视图类型，我们需要定义一个对应于每种技术的`ViewResolver` bean。这意味着我们可以像通常返回 JSP 文件一样从`@Controller`映射方法中返回视图名。

在接下来的章节中，我们将回顾更多的传统技术，如`Java Server Pages`，以及可以用于 Spring 的主要模板引擎:`Thymeleaf`、`Groovy`、`FreeMarker, Jade.`

对于其中的每一个，我们将检查标准 Spring 应用程序和使用`Spring Boot`构建的应用程序中的必要配置。

## 3。`Java Server Pages`

JSP 是 Java 应用程序中最受欢迎的视图技术之一，它受 Spring 开箱即用的支持。为了呈现 JSP 文件，一种常用的`ViewResolver` bean 类型是`InternalResourceViewResolver`:

```java
@EnableWebMvc
@Configuration
public class ApplicationConfiguration implements WebMvcConfigurer {
    @Bean
    public ViewResolver jspViewResolver() {
        InternalResourceViewResolver bean = new InternalResourceViewResolver();
        bean.setPrefix("/WEB-INF/views/");
        bean.setSuffix(".jsp");
        return bean;
    }
}
```

接下来，我们可以开始在`/WEB-INF/views`位置创建 JSP 文件:

```java
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form"%>
<html>
    <head>
        <meta http-equiv="Content-Type" 
          content="text/html; charset=ISO-8859-1">
        <title>User Registration</title>
    </head>
    <body>
        <form:form method="POST" modelAttribute="user">
            <form:label path="email">Email: </form:label>
            <form:input path="email" type="text"/>
            <form:label path="password">Password: </form:label>
            <form:input path="password" type="password" />
            <input type="submit" value="Submit" />
        </form:form>
    </body>
</html>
```

如果我们将文件添加到一个`Spring Boot`应用程序中，那么我们可以在一个`application.properties`文件中定义以下属性，而不是在`ApplicationConfiguration`类中:

```java
spring.mvc.view.prefix: /WEB-INF/views/
spring.mvc.view.suffix: .jsp
```

基于这些属性，`Spring Boot`将自动配置必要的`ViewResolver`。

## 4。`Thymeleaf`

`[Thymeleaf](https://web.archive.org/web/20220909001904/http://www.thymeleaf.org/)` 是一个 Java 模板引擎，可以处理 HTML、XML、文本、JavaScript 或 CSS 文件。与其他模板引擎不同，`Thymeleaf`允许使用模板作为原型，这意味着它们可以被视为静态文件。

### 4.1。Maven 依赖关系

为了将`Thymeleaf`与 Spring 集成，我们需要添加 [`thymeleaf`](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22thymeleaf%22%20AND%20g%3A%22org.thymeleaf%22) 和 [`thymeleaf-spring4`](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22thymeleaf-spring4%22%20AND%20g%3A%22org.thymeleaf%22) 依赖项:

```java
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

如果我们有一个 Spring 4 项目，那么我们需要添加`thymeleaf-spring4`。

### 4.2。弹簧配置

接下来，我们需要添加需要一个`SpringTemplateEngine` bean 的配置，以及一个指定视图文件的位置和类型的`TemplateResolver` bean。

`SpringResourceTemplateResolver`集成了 Spring 的资源解析机制:

```java
@Configuration
@EnableWebMvc
public class ThymeleafConfiguration {

    @Bean
    public SpringTemplateEngine templateEngine() {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(thymeleafTemplateResolver());
        return templateEngine;
    }

    @Bean
    public SpringResourceTemplateResolver thymeleafTemplateResolver() {
        SpringResourceTemplateResolver templateResolver 
          = new SpringResourceTemplateResolver();
        templateResolver.setPrefix("/WEB-INF/views/");
        templateResolver.setSuffix(".html");
        templateResolver.setTemplateMode("HTML5");
        return templateResolver;
    }
}
```

此外，我们需要一个类型为`ThymeleafViewResolver`的`ViewResolver` bean:

```java
@Bean
public ThymeleafViewResolver thymeleafViewResolver() {
    ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
    viewResolver.setTemplateEngine(templateEngine());
    return viewResolver;
}
```

### 4.3。`Thymeleaf`模板

现在我们可以在`WEB-INF/views`位置添加一个 HTML 文件:

```java
<html>
    <head>
        <meta charset="ISO-8859-1" />
        <title>User Registration</title>
    </head>
    <body>
        <form action="#" th:action="@{/register}" 
          th:object="${user}" method="post">
            Email:<input type="text" th:field="*{email}" />
            Password:<input type="password" th:field="*{password}" />
            <input type="submit" value="Submit" />
        </form>
    </body>
</html>
```

模板在语法上与 HTML 模板非常相似。

在 Spring 应用程序中使用`Thymeleaf`时，一些可用的特性是:

你可以在我们的文章[在 Spring MVC](/web/20220909001904/https://www.baeldung.com/thymeleaf-in-spring-mvc) 中阅读更多关于使用`Thymeleaf`模板的内容。

### 4.4。`Spring Boot`中的`Thymeleaf`

`Spring Boot`将通过添加 [`spring-boot-starter-thymeleaf`](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-thymeleaf%22) 依赖关系为`Thymeleaf`提供自动配置:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>2.5.6</version>
</dependency>
```

不需要显式配置。默认情况下，HTML 文件应该放在`resources/templates`位置。

## 5。`FreeMarker`

`[FreeMarker](https://web.archive.org/web/20220909001904/http://freemarker.org/)` 是由`Apache Software Foundation`构建的基于 Java 的模板引擎。它可以用来生成网页，也可以生成源代码、XML 文件、配置文件、电子邮件和其他基于文本的格式。

生成基于使用`FreeMarker Template Language`编写的模板文件。

### 5.1。Maven 依赖关系

要开始在我们的项目中使用模板，我们需要 [`freemarker`](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22freemarker%22%20AND%20g%3A%22org.freemarker%22) 依赖项:

```java
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.23</version>
</dependency>
```

对于 Spring 集成，我们还需要 [`spring-context-support`](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-context-support%22) 的依赖关系:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

### 5.2。弹簧配置

将`FreeMarker`与 Spring MVC 集成需要定义一个`FreemarkerConfigurer` bean 来指定模板文件的位置:

```java
@Configuration
@EnableWebMvc
public class FreemarkerConfiguration {

    @Bean 
    public FreeMarkerConfigurer freemarkerConfig() { 
        FreeMarkerConfigurer freeMarkerConfigurer = new FreeMarkerConfigurer(); 
        freeMarkerConfigurer.setTemplateLoaderPath("/WEB-INF/views/");
        return freeMarkerConfigurer; 
    }
}
```

接下来，我们需要定义一个合适的类型为`FreeMarkerViewResolver`的`ViewResolver` bean:

```java
@Bean 
public FreeMarkerViewResolver freemarkerViewResolver() { 
    FreeMarkerViewResolver resolver = new FreeMarkerViewResolver(); 
    resolver.setCache(true); 
    resolver.setPrefix(""); 
    resolver.setSuffix(".ftl"); 
    return resolver; 
}
```

### 5.3。`FreeMarker`模板

我们可以在`WEB-INF/views`位置使用`FreeMarker`创建一个 HTML 模板:

```java
<#import "/spring.ftl" as spring/>
<html>
    <head>
        <meta charset="ISO-8859-1" />
        <title>User Registration</title>
    </head>
    <body>
        <form action="register" method="post">
            <@spring.bind path="user" />
            Email: <@spring.formInput "user.email"/>
            Password: <@spring.formPasswordInput "user.password"/>
            <input type="submit" value="Submit" />
        </form>
    </body>
</html>
```

在上面的例子中，我们导入了一组由 Spring 定义的宏，用于处理`FreeMarker`中的表单，包括将表单输入绑定到数据模型。

此外，`FreeMarker Template Language`包含大量的标签、指令和表达式，用于处理集合、流控制结构、逻辑操作符、格式化和解析字符串、数字以及更多特性。

### 5.4。`Spring Boot`中的`FreeMarker`

在`Spring Boot`应用中，我们可以通过使用 [`spring-boot-starter-freemarker`](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-freemarker%22) 依赖关系来简化所需的配置:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
    <version>2.5.6</version>
</dependency>
```

这个启动器添加了必要的自动配置。我们需要做的就是开始将模板文件放在`resources/templates`文件夹中。

## 6。`Groovy`

Spring MVC 视图也可以使用 [Groovy 标记模板引擎](https://web.archive.org/web/20220909001904/http://groovy-lang.org/templating.html#_the_markuptemplateengine)生成。该引擎基于构建器语法，可用于生成任何文本格式。

### 6.1。Maven 依赖关系

需要将 [`groovy-templates`](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22groovy-templates%22) 依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-templates</artifactId>
    <version>2.4.12</version>
</dependency>
```

### 6.2。弹簧配置

`Markup Template Engine`与 Spring MVC 的集成需要定义一个`GroovyMarkupConfigurer` bean 和一个`GroovyMarkupViewResolver`类型的`ViewResolver`:

```java
@Configuration
@EnableWebMvc
public class GroovyConfiguration {

    @Bean
    public GroovyMarkupConfigurer groovyMarkupConfigurer() {
        GroovyMarkupConfigurer configurer = new GroovyMarkupConfigurer();
        configurer.setResourceLoaderPath("/WEB-INF/views/");
        return configurer;
    }

    @Bean
    public GroovyMarkupViewResolver thymeleafViewResolver() {
        GroovyMarkupViewResolver viewResolver = new GroovyMarkupViewResolver();
        viewResolver.setSuffix(".tpl");
        return viewResolver;
    }
}
```

### 6.3。`Groovy Markup`模板

模板是用 Groovy 语言编写的，有几个特点:

让我们为“用户注册”表单创建一个 Groovy 模板，它包括数据绑定:

```java
yieldUnescaped '<!DOCTYPE html>'                                                    
html(lang:'en') {                                                                   
    head {                                                                          
        meta('http-equiv':'"Content-Type" ' +
          'content="text/html; charset=utf-8"')      
        title('User Registration')                                                            
    }                                                                               
    body {                                                                          
        form (id:'userForm', action:'register', method:'post') {
            label (for:'email', 'Email')
            input (name:'email', type:'text', value:user.email?:'')
            label (for:'password', 'Password')
            input (name:'password', type:'password', value:user.password?:'')
            div (class:'form-actions') {
                input (type:'submit', value:'Submit')
            }                             
        }
    }                                                                               
}
```

### 6.4。`Spring Boot`中的`Groovy Template Engine`

`Spring Boot`包含对`Groovy Template Engine`的自动配置，通过包含`[spring-boot-starter-groovy-templates](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-groovy-templates%22)`依赖关系来添加:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-groovy-templates</artifactId>
    <version>2.5.6</version>
</dependency>
```

模板的默认位置是`/resources/templates`。

## 7。`Jade4j`

[Jade4j](https://web.archive.org/web/20220909001904/https://github.com/neuland/jade4j) 是 Javascript 的 [`Pug`](https://web.archive.org/web/20220909001904/https://pugjs.org/) 模板引擎(原名`Jade`)的 Java 实现。`Jade4j`模板可以用来生成 HTML 文件。

### 7.1。Maven 依赖关系

对于 Spring 集成，我们需要 [spring-jade4j](https://web.archive.org/web/20220909001904/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-jade4j%22) 依赖关系:

```java
<dependency>
    <groupId>de.neuland-bfi</groupId>
    <artifactId>spring-jade4j</artifactId>
    <version>1.2.5</version>
</dependency>
```

### 7.2。弹簧配置

要将`Jade4j`用于 Spring，我们必须定义一个配置模板位置的`SpringTemplateLoader` bean，以及一个`JadeConfiguration` bean:

```java
@Configuration
@EnableWebMvc
public class JadeTemplateConfiguration {

    @Bean
    public SpringTemplateLoader templateLoader() {
        SpringTemplateLoader templateLoader 
          = new SpringTemplateLoader();
        templateLoader.setBasePath("/WEB-INF/views/");
        templateLoader.setSuffix(".jade");
        return templateLoader;
    }

    @Bean
    public JadeConfiguration jadeConfiguration() {
        JadeConfiguration configuration 
          = new JadeConfiguration();
        configuration.setCaching(false);
        configuration.setTemplateLoader(templateLoader());
        return configuration;
    }
}
```

接下来，我们需要普通的`ViewResolver` bean，在本例中是类型`JadeViewResolver`:

```java
@Bean
public ViewResolver viewResolver() {
    JadeViewResolver viewResolver = new JadeViewResolver();
    viewResolver.setConfiguration(jadeConfiguration());
    return viewResolver;
}
```

### 7.3。`Jade4j`模板

模板的特点是易于使用的区分空白的语法:

```java
doctype html
html
  head
    title User Registration
  body
    form(action="register" method="post" )
      label(for="email") Email:
      input(type="text" name="email")
      label(for="password") Password:
      input(type="password" name="password")
      input(type="submit" value="Submit")
```

该项目还提供了一个非常有用的[交互式文档](https://web.archive.org/web/20220909001904/https://naltatis.github.io/jade-syntax-docs/)，在这里你可以查看你编写的模板的输出。

`Spring Boot`没有提供`Jade4j`启动器，所以在`Boot`项目中，我们必须添加与上面定义的相同的弹簧配置。

## 8。其他模板引擎

除了到目前为止描述的模板引擎之外，还有很多可用的模板引擎。

让我们简单回顾一下其中的一些。

`[Velocity](https://web.archive.org/web/20220909001904/https://velocity.apache.org/)` 是一个较老的模板引擎，它非常复杂，但缺点是 Spring 从 4.3 版本起就不再使用它，并在 Spring 5.0.1 中完全删除。

`[JMustache](https://web.archive.org/web/20220909001904/https://github.com/samskivert/jmustache)`是一个模板引擎，通过使用`spring-boot-starter-mustache`依赖项，它可以很容易地集成到 Spring Boot 应用程序中。

`[Pebble](https://web.archive.org/web/20220909001904/https://pebbletemplates.io/)` 在其库中包含对 Spring 和`Spring Boot` 的支持。

也可以使用运行在`JSR-223`脚本引擎如`[Nashorn](https://web.archive.org/web/20220909001904/https://openjdk.java.net/projects/nashorn/),` 之上的其他模板库，如`[Handlebars](https://web.archive.org/web/20220909001904/http://handlebarsjs.com/)` 或`[React](https://web.archive.org/web/20220909001904/https://facebook.github.io/react/)`。

## 9。结论

在本文中，我们介绍了一些最流行的 Spring web 应用程序模板引擎。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220909001904/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-2)