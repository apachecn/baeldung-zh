# 球衣 MVC 支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jersey-mvc>

## 1.概观

**[Jersey](https://web.archive.org/web/20220625230515/https://jersey.github.io/) 是开发 RESTFul Web 服务的开源框架**。

除了作为 JAX-RS 参考实现之外，它还包括许多扩展，以进一步简化 web 应用程序开发。

在本教程中，**我们将创建一个使用 Jersey** 提供的模型-视图-控制器(MVC)扩展的小示例应用程序。

要学习如何用 Jersey 创建 API，请点击这里的查看这篇[文章。](/web/20220625230515/https://www.baeldung.com/jersey-rest-api-with-spring)

## 2.新泽西的 MVC

Jersey 包含一个扩展来支持模型-视图-控制器(MVC)设计模式。

首先，在 Jersey 组件的上下文中，来自 MVC 模式的控制器对应于一个资源类或方法。

同样，视图对应于绑定到资源类或方法的模板。最后，模型表示一个从资源方法(控制器)返回的 Java 对象。

**为了在我们的应用程序中使用 Jersey MVC 的功能，我们首先需要注册我们希望使用的 MVC 模块扩展**。

在我们的例子中，我们将使用流行的 Java 模板引擎 [Freemarker](https://web.archive.org/web/20220625230515/https://freemarker.apache.org/) 。这是 Jersey 开箱即用支持的渲染引擎之一，还有[小胡子](https://web.archive.org/web/20220625230515/https://github.com/spullara/mustache.java)和标准 Java 服务器页面(JSP)。

关于 MVC 如何工作的更多信息，请参考这个[教程](/web/20220625230515/https://www.baeldung.com/mvc-servlet-jsp)。

## 3.应用程序设置

在这一节中，我们将从在`pom.xml.`中配置必要的 Maven 依赖项开始

然后，我们将看看如何使用一个简单的嵌入式 Grizzly 服务器来配置和运行我们的服务器。

### 3.1.Maven 依赖性

让我们从添加球衣 MVC Freemarker 扩展开始。

我们可以从 [Maven Central](https://web.archive.org/web/20220625230515/https://search.maven.org/classic/#search%7Cga%7C1%7Cjersey-mvc-freemarker) 获得最新版本:

```java
<dependency>
    <groupId>org.glassfish.jersey.ext</groupId>
    <artifactId>jersey-mvc-freemarker</artifactId>
    <version>2.27</version>
</dependency> 
```

我们还需要 Grizzly servlet 容器。

同样，我们可以在 [Maven Central](https://web.archive.org/web/20220625230515/https://search.maven.org/classic/#search%7Cga%7C1%7Cjersey-container-grizzly2-servlet) 中找到最新版本:

```java
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-grizzly2-servlet</artifactId>
    <version>2.27</version>
</dependency>
```

### 3.2.配置服务器

为了在我们的应用程序**中使用 Jersey MVC 模板支持，我们需要注册 MVC 模块**提供的特定 JAX-RS 特性。

考虑到这一点，我们定义了一个自定义资源配置:

```java
public class ViewApplicationConfig extends ResourceConfig {    
    public ViewApplicationConfig() {
        packages("com.baeldung.jersey.server");
        property(FreemarkerMvcFeature.TEMPLATE_BASE_PATH, "templates/freemarker");
        register(FreemarkerMvcFeature.class);;
    }
}
```

在上面的示例中，我们配置了三个项目:

*   首先，我们使用`packages`方法告诉 Jersey 扫描`com.baeldung.jersey.server`包中带有`@Path. `注释的类，这将注册我们的`FruitResource`
*   接下来，我们配置基本路径以解析我们的模板。这告诉 Jersey 在`/src/main/resources/templates/freemarker `中查找 Freemarker 模板
*   最后，我们通过`FreemarkerMvcFeature `类注册处理 Freemarker 渲染的特性

### 3.3.运行应用程序

现在让我们看看如何运行我们的 web 应用程序。我们将使用 [exec-maven-plugin](https://web.archive.org/web/20220625230515/https://www.mojohaus.org/exec-maven-plugin/) 来配置我们的`pom.xml`来执行我们的嵌入式 web 服务器:

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <configuration>                
        <mainClass>com.baeldung.jersey.server.http.EmbeddedHttpServer</mainClass>
    </configuration>
</plugin>
```

现在让我们使用 Maven 编译并运行我们的应用程序:

```java
mvn clean compile exec:java
...
Jul 28, 2018 6:21:08 PM org.glassfish.grizzly.http.server.HttpServer start
INFO: [HttpServer] Started.
Application started.
Try out http://localhost:8082/fruit
Stop the application using CTRL+C 
```

转到浏览器 URL-`http://localhost:8080/fruit`。瞧，“欢迎水果索引页！”已显示。

## 4.MVC 模板

**在 Jersey，MVC API 由两个类组成，将模型绑定到视图，即`Viewable`和`@Template`。**

在本节中，我们将解释将模板链接到视图的三种不同方式:

*   使用`Viewable` 类
*   使用`@Template `注释
*   如何用 MVC 处理错误并将它们传递给特定的模板

### 4.1.在资源类中使用`Viewable`

我们先来看一下`Viewable`:

```java
@Path("/fruit")
public class FruitResource {
    @GET
    public Viewable get() {
        return new Viewable("/index.ftl", "Fruit Index Page");
    }
}
```

**在这个例子中，`FruitResource` JAX-RS 资源类是控制器。**`Viewable`实例封装了被引用的数据模型，这是一个简单的`String.`

此外，我们还包含了对相关视图模板的命名引用–`index.ftl`。

### 4.2.在资源方法上使用`@Template`

**没有必要每次我们想要将模型绑定到模板**时都使用`Viewable`。

在下一个例子中，我们将简单地用`@Template`来注释我们的资源方法:

```java
@GET
@Template(name = "/all.ftl")
@Path("/all")
@Produces(MediaType.TEXT_HTML)
public Map<String, Object> getAllFruit() {
    List<Fruit> fruits = new ArrayList<>();
    fruits.add(new Fruit("banana", "yellow"));
    fruits.add(new Fruit("apple", "red"));
    fruits.add(new Fruit("kiwi", "green"));

    Map<String, Object> model = new HashMap<>();
    model.put("items", fruits);
    return model;
}
```

在这个例子中，我们使用了`@Template`注释。这避免了通过`Viewable` 将我们的模型直接包装在模板引用中，并使我们的资源方法更具可读性。

该模型现在由我们的带注释的资源方法的返回值来表示——a`Map<String, Object>.` 它被直接传递给模板`all.ftl`,模板只显示我们的水果列表。

### 4.3.用 MVC 处理错误

现在让我们看看如何使用`@ErrorTemplate`注释来处理错误:

```java
@GET
@ErrorTemplate(name = "/error.ftl")
@Template(name = "/named.ftl")
@Path("{name}")
@Produces(MediaType.TEXT_HTML)
public String getFruitByName(@PathParam("name") String name) {
    if (!"banana".equalsIgnoreCase(name)) {
        throw new IllegalArgumentException("Fruit not found: " + name);
    }
    return name;
} 
```

**一般来说,`@ErrorTemplate`注释的目的是将模型绑定到错误视图。**此错误处理程序将负责在处理请求期间引发异常时呈现响应。

在我们简单的 Fruit API 示例中，如果在处理过程中没有出现错误，那么使用`named.ftl`模板来呈现页面。否则，如果出现异常，那么向用户显示`error.ftl`模板。

在这种情况下，模型就是抛出的异常本身。这意味着我们可以在模板中直接调用异常对象的方法。

让我们快速浏览一下`error.ftl`模板中的一个片段来强调这一点:

```java
<body>
    <h1>Error - ${model.message}!</h1>
</body>
```

在我们的最后一个例子中，我们将看看一个简单的单元测试:

```java
@Test
public void givenGetFruitByName_whenFruitUnknown_thenErrorTemplateInvoked() {
    String response = target("/fruit/orange").request()
      .get(String.class);
    assertThat(response, containsString("Error -  Fruit not found: orange!"));
} 
```

在上面的例子中，我们使用来自水果资源的响应。我们检查响应是否包含来自被抛出的`IllegalArgumentException`的消息。

## 5.结论

在本文中，我们探索了 Jersey 框架 MVC 扩展。

我们首先介绍了 MVC 在 Jersey 是如何工作的。接下来，我们看一下如何配置、运行和设置一个示例 web 应用程序。

最后，我们看了在 Jersey 和 Freemarker 中使用 MVC 模板的三种方式，以及如何处理错误。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220625230515/https://github.com/eugenp/tutorials/tree/master/jersey)