# ActiveWeb 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/activeweb>

## 1。概述

在本文中，我们将举例说明[active web](https://web.archive.org/web/20220627175429/http://javalite.io/activeweb)——来自 JavaLite 的全栈 web 框架——提供开发动态 web 应用程序或 REST-ful web 服务所需的一切。

## 2。基本概念和原理

Activeweb 利用“约定胜于配置”——这意味着它是可配置的，但具有合理的默认值，不需要额外的配置。我们只需要遵循一些预定义的约定，比如以某种预定义的格式命名类、方法和字段。

它还通过将源代码重新编译和重新加载到正在运行的容器(默认为 Jetty)中来简化开发。

对于依赖管理，它使用 Google Guice 作为 DI 框架；要了解更多关于 Guice 的信息，请点击查看我们的[指南。](/web/20220627175429/https://www.baeldung.com/guice)

## 3.Maven 设置

首先，让我们添加必要的依赖项:

```java
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>activeweb</artifactId>
    <version>1.15</version>
</dependency> 
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220627175429/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.javalite%22%20AND%20a%3A%22activeweb%22)

此外，为了测试应用程序，我们需要`activeweb-testing` 依赖关系:

```java
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>activeweb-testing</artifactId>
    <version>1.15</version>
    <scope>test</scope>
</dependency>
```

点击查看最新版本[。](https://web.archive.org/web/20220627175429/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.javalite%22%20AND%20a%3A%22activeweb-testing%22)

## 4。应用程序结构

正如我们所讨论的，应用程序结构需要遵循一定的约定；下面是一个典型的 MVC 应用程序的样子:

![active web](img/42907fccb46e9c54f45cfaf514d924b4.png)

我们可以看到，`controllers`、`service`、`config`、`models`应该位于`app` 包中各自的子包中。

视图应该位于`WEB-INF/views` 目录中，每个视图都有自己的基于控制器名称的子目录。例如，`app.controllers.ArticleController` 应该有一个包含该控制器所有视图文件的`article/` 子目录。

部署描述符或`web.xml` 通常应该包含一个`<filter>` 和相应的`<filter-mapping>.` ，因为框架是一个 servlet 过滤器，而不是一个`<servlet>` 配置，有一个过滤器配置:

```java
...
<filter>
    <filter-name>dispatcher</filter-name>
    <filter-class>org.javalite.activeweb.RequestDispatcher</filter-class>
...
</filter>
...
```

我们还需要一个`<init-param>` `root_controller` 来定义应用程序的默认控制器——类似于一个`home`控制器:

```java
...
<init-param>
    <param-name>root_controller</param-name>
    <param-value>home</param-value>
</init-param>
...
```

## 5。控制器

控制器是 ActiveWeb 应用程序的主要组件；而且，如前所述，所有控制器都应位于`app.controllers` 包内:

```java
public class ArticleController extends AppController {
    // ...
}
```

请注意，控制器正在扩展`org.javalite.activeweb.AppController.`

### 5.1。控制器 URL 映射

控制器根据约定自动映射到一个 URL。例如，`ArticleController` 将被映射到:

```java
http://host:port/contextroot/article
```

现在，这会将它们映射到控制器中的默认动作。动作只不过是控制器内部的方法。将默认方法命名为`index():`

```java
public class ArticleController extends AppController {
    // ...
    public void index() {
        render("articles");    
    }
    // ...
}
```

对于其他方法或操作，将方法名称附加到 URL:

```java
public class ArticleController extends AppController {
    // ...

    public void search() {
        render("search");
    }
}
```

网址:

```java
http://host:port/contextroot/article/search
```

我们甚至可以拥有基于 HTTP 方法的控制器动作。如果我们没有注释一个动作，默认情况下它被认为是一个 GET。

### 5.2。控制器 URL 解析

框架使用控制器名称和子包名称来生成控制器 URL。例如`app.controllers.ArticleController.java`URL:

```java
http://host:port/contextroot/article
```

如果控制器在子包中，URL 就变成:

```java
http://host:port/contextroot/baeldung/article
```

对于不止一个单词的控制器名称(例如`app.controllers.PublishedArticleController.java`，URL 将使用下划线分隔:

```java
http://host:port/contextroot/published_article
```

### 5.3。检索请求参数

在控制器内部，我们使用来自`AppController class.` 的`param()` 或`params()` 方法访问请求参数。第一个方法接受一个字符串参数——要检索的参数的名称:

```java
public void search() {

    String keyword = param("key");  
    view("search",articleService.search(keyword));

}
```

如果需要，我们可以稍后使用来获取所有参数:

```java
public void search() {

    Map<String, String[]> criterion = params();
    // ...
}
```

## 6。视图

在 ActiveWeb 术语中，视图通常被称为模板；这主要是因为它使用 Apache [FreeMarker](https://web.archive.org/web/20220627175429/https://freemarker.apache.org/) 模板引擎而不是 JSP。你可以在我们的指南中阅读更多关于 FreeMarker [的信息，点击这里](/web/20220627175429/https://www.baeldung.com/freemarker-in-spring-mvc-tutorial)。

将模板放在`WEB-INF/views` 目录中。每个控制器都应该有一个子目录，保存它需要的所有模板。

### 6.1。控制器视图映射

当一个控制器被点击时，默认动作`index()`被执行，框架将从视图目录中为该控制器选择`WEB-INF/views/article/` `index.ftl` 模板。类似地，对于任何其他动作，视图将基于动作名称来选择。

这并不总是我们想要的。有时，我们可能希望返回一些基于内部业务逻辑的视图。在这个场景中，**我们可以使用父`org.javalite.activeweb.AppController` 类中的`render()` 方法**来控制流程:

```java
public void index() {
    render("articles");    
}
```

请注意，自定义视图的位置也应该位于该控制器的同一视图目录中。如果不是这样，在模板名前面加上模板所在的目录名，并将其传递给`render()`方法:

```java
render("/common/error");
```

### 6.3。带数据的视图

为了向视图发送数据，`org.javalite.activeweb.AppController` 提供了`view()` 方法:

```java
view("articles", articleService.getArticles());
```

这需要两个参数。首先是用于访问模板中对象的对象名，其次是包含数据的对象。

我们还可以使用`assign()`方法将数据传递给视图。view()和`assign()`方法之间完全没有区别——我们可以选择其中的任何一个:

```java
assign("article", articleService.search(keyword));
```

让我们映射模板中的数据:

```java
<@content for="title">Articles</@content>
...
<#list articles as article>
    <tr>
        <td>${article.title}</td>
        <td>${article.author}</td>
        <td>${article.words}</td>
        <td>${article.date}</td>
    </tr>
</#list>
</table>
```

## 7。管理依赖关系

为了管理对象和实例，ActiveWeb 使用 Google Guice 作为依赖管理框架。

假设我们的应用程序中需要一个服务类；这将把业务逻辑从控制器中分离出来。

让我们首先创建一个服务接口:

```java
public interface ArticleService {

    List<Article> getArticles();   
    Article search(String keyword);

}
```

以及实现:

```java
public class ArticleServiceImpl implements ArticleService {

    public List<Article> getArticles() {
        return fetchArticles();
    }

    public Article search(String keyword) {
        Article ar = new Article();
        ar.set("title", "Article with "+keyword);
        ar.set("author", "baeldung");
        ar.set("words", "1250");
        ar.setDate("date", Instant.now());
        return ar;
    }
}
```

现在，让我们将这个服务绑定为一个 Guice 模块:

```java
public class ArticleServiceModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(ArticleService.class).to(ArticleServiceImpl.class)
          .asEagerSingleton();
    }
}
```

最后，在应用程序上下文中注册它，并根据需要将其注入控制器:

```java
public class AppBootstrap extends Bootstrap {

    public void init(AppContext context) {
    }

    public Injector getInjector() {
        return Guice.createInjector(new ArticleServiceModule());
    }
}
```

注意，这个配置类名必须是`AppBootstrap`，并且应该位于`app.config`包中。

最后，下面是我们将它注入控制器的方法:

```java
@Inject
private ArticleService articleService;
```

## 8。测试

ActiveWeb 应用程序的单元测试是使用 JavaLite 的 [JSpec](https://web.archive.org/web/20220627175429/http://javalite.io/jspec) 库编写的。

我们将使用 JSpec 中的`org.javalite.activeweb.ControllerSpec` 类来测试我们的控制器，我们将按照类似的约定来命名测试类:

```java
public class ArticleControllerSpec extends ControllerSpec {
    // ...
}
```

请注意，该名称类似于它正在测试的控制器，末尾有一个“Spec”。

下面是测试案例:

```java
@Test
public void whenReturnedArticlesThenCorrect() {
    request().get("index");
    a(responseContent())
      .shouldContain("<td>Introduction to Mule</td>");
}
```

注意，`request()` 方法模拟了对控制器的调用，相应的 HTTP 方法`get(),` 将动作名称作为参数。

我们还可以使用`params()`方法将参数传递给控制器:

```java
@Test
public void givenKeywordWhenFoundArticleThenCorrect() {
    request().param("key", "Java").get("search");
    a(responseContent())
      .shouldContain("<td>Article with Java</td>");
}
```

为了传递多个参数，我们也可以用这个流畅的 API 链接方法。

## 9.部署应用程序

可以将应用程序部署在任何 servlet 容器中，如 Tomcat、WildFly 或 Jetty。当然，最简单的部署和测试方法是使用 Maven Jetty 插件:

```java
...
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.4.8.v20171121</version>
    <configuration>
        <reload>manual</reload>
        <scanIntervalSeconds>10000</scanIntervalSeconds>
    </configuration>
</plugin>
...
```

插件的最新版本是[这里是](https://web.archive.org/web/20220627175429/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.eclipse.jetty%22%20AND%20a%3A%22jetty-maven-plugin%22)。

现在，我们终于可以开始了:

```java
mvn jetty:run
```

## 10。结论

在本文中，我们了解了 ActiveWeb 框架的基本概念和约定。除此之外，这个框架还拥有比我们在这里讨论的更多的特性和功能。

更多详情请参考官方[文档](https://web.archive.org/web/20220627175429/http://javalite.io/documentation#activeweb)。

和往常一样，本文中使用的示例代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220627175429/https://github.com/eugenp/tutorials/tree/master/java-lite)