# Java 中的 VRaptor 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vraptor>

## 1.**概述**

在本文中，我们将看看 [VRaptor](https://web.archive.org/web/20220526044748/http://www.vraptor.org/en/) ，这是一个简单明了的 Java MVC web 框架，它利用了 Java 上下文和依赖注入技术，易于掌握。

就像 Spring-**一样，它非常依赖注释，并且与 Hibernate** 配合得非常好。

它还附带了一些有用的插件——比如用于内部化和单元测试。

因此，让我们探索一下 VRaptor 的不同组件，并创建一个示例项目。

## 2。Maven 依赖性和设置

一个快速启动并运行的方法是从[官方库](https://web.archive.org/web/20220526044748/https://bintray.com/caelum/VRaptor4/br.com.caelum.vraptor/)下载`vraptor-blank-project-distribution`。

空白项目只是一个框架，可以充实成为成熟的 web 应用程序。

下载并解压完项目后，让我们将目录重命名为`vraptor`(或任何其他名称)。

该目录应包含:

*   `src/`
*   `pom.xml`
*   和`README.md`

该项目是基于 Maven 的，并带有`tomcat7` Maven 插件，它提供了运行应用程序的 servlet 容器。

它还有一个默认的`IndexController`，只有一个方法—`index()`。

默认情况下，通过该方法渲染的视图位于`webapp/WEB-INF/jsp/index/index.jsp` 中——这遵循了`WEB-INF/jsp/` `controller_name/method_name.`的惯例

为了启动服务器，我们将从项目的根目录执行命令`mvn tomcat7` : `run`。

如果成功，如果我们访问`http://localhost:8080,`，浏览器将显示`It works!! VRaptor!`。

**如果我们面对`java.lang.LinkageError: loader constraint violation”,` 那么，我们必须修改`pom.xml`中的下列依赖关系**:

```java
<dependency>
    <groupId>org.jboss.weld.servlet</groupId>
    <artifactId>weld-servlet-core</artifactId>
    <version>2.1.2.Final</version>
    <exclusions>
        <exclusion>
	    <groupId>org.jboss.spec.javax.el</groupId>
	    <artifactId>jboss-el-api_3.0_spec</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.jboss.weld</groupId>
    <artifactId>weld-core-impl</artifactId>
    <version>2.1.2.Final</version>
    <exclusions>
       <exclusion>
          <groupId>org.jboss.spec.javax.el</groupId>
  	  <artifactId>jboss-el-api_3.0_spec</artifactId>
       </exclusion>
    </exclusions>
</dependency>
```

罪魁祸首是包含在`weld-servlet-core`和`compile`范围内的`el-api`；这导致了依赖性冲突。

沿着这条线将需要以下依赖项，所以让我们将它们包含在`pom.xml`中:

```java
<dependency>
    <groupId>br.com.caelum.vraptor</groupId>
    <artifactId>vraptor-freemarker</artifactId>
    <version>4.1.0-RC3</version>
</dependency>
```

```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.8-dmr</version>
</dependency>

<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.27-incubating</version>
</dependency>
```

最新版本的 [`vraptor-freemarker`、](https://web.archive.org/web/20220526044748/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22br.com.caelum.vraptor%22%20AND%20a%3A%22vraptor-freemarker%22) [`mysql-connector-java`](https://web.archive.org/web/20220526044748/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22mysql%22%20AND%20a%3A%22mysql-connector-java%22) 、 [`freemarker`](https://web.archive.org/web/20220526044748/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.freemarker%22%20AND%20a%3A%22freemarker%22) 神器可以在 Maven Central 找到。

现在我们可以开始了，让我们建立一个简单的博客站点。

## 3。休眠支持

VRaptor 提供了各种与数据库交互的插件，其中一个是与 Hibernate 4 协同工作的`vraptor-hibernate`。

**该插件通过 CDI 使 Hibernate 的`SessionFactory` bean 在运行时可用。**

插件就绪后，我们需要一个标准的 Hibernate 配置文件——在资源库中可以找到一个例子。

VRaptor 使用一种称为生产者的技术来使对象可用于 DI 管理。更多关于这个[的细节在这里](https://web.archive.org/web/20220526044748/http://www.vraptor.org/en/docs/components/)。

## 4。 **在 VRaptor 中定义 Web 路由**

在 VRaptor 中，路由定义驻留在控制器中，这些控制器只是简单的`@Controller`-带注释的 Java 对象——就像 Spring 中一样。

`@Path`注释用于将请求路径映射到特定的控制器，而`@Get, @Post, @Put, @Delete` 和`@Patch` 注释用于指定 HTTP 请求类型。

路由映射配置看起来类似于 JAX-RS 的方式，但没有正式实施该标准。

此外，在定义路径时，可以在花括号中指定路径变量:

```java
@Get("/posts/{id}")
```

然后可以在控制器方法中访问`id`的值:

```java
@Get("/posts/{id}")
public void view(int id) {
    // ...
}
```

当一个表单被提交到一个特定的路径时，VRaptor 可以自动用提交的表单数据填充一个对象。

让我们在文章的下一部分看看这一点。

## 5。视图和模板引擎

默认情况下，可以使用 JSP 实现视图。然而，也可以使用其他模板引擎——在本文中，我们将使用 Freemarker。

让我们从在默认视图目录(src/main/resources/templates)中创建`index.ftl and saving` 开始:

```java
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>VRaptor Blank Project</title>
</head>
<body>
It works!! ${variable}
</body>
</html>
```

现在，我们可以使用带有`FreemarkerView` 类的已定义视图进行视图渲染:

```java
@Path("/")
public void index() {
    result.include("variable", "VRaptor!");
    result.use(FreemarkerView.class).withTemplate("index");
}
```

`Result`对象持有模型状态——它有重定向到另一个页面、URL 或控制器方法的方法；可以使用 CDI 将其注入控制器。

在我们的例子中， `variable`被 Freemarker 解析。因此，`index.ftl`中的`${variable}`占位符被替换为“VRaptor！”。

这里记录了更高级的用法[。](https://web.archive.org/web/20220526044748/http://www.vraptor.org/en/docs/view-and-ajax/)

## 6。表单提交处理示例

让我们看看如何处理带有验证的表单提交:

```java
@Post("/post/add")
public void add(Post post) {
    post.setAuthor(userInfo.getUser());
    validator.validate(post);
    if(validator.hasErrors()) {
        result.include("errors", validator.getErrors());
    }
    validator.onErrorRedirectTo(this).addForm();

    Object id = postDao.add(post);

    if(Objects.nonNull(id)) {
       result.include("status", "Post Added Successfully");
         result.redirectTo(IndexController.class).index();
    } else {
        result.include(
          "error", "There was an error creating the post. Try Again");
        result.redirectTo(this).addForm();
    }
}
```

在使用`postDao.add()`将对象持久化到数据库之前，`Post`对象首先使用 [Java bean 验证](/web/20220526044748/https://www.baeldung.com/javax-validation)进行验证。

`Post`对象的字段由提交的表单数据的值自动填充——对应于视图文件中的表单输入字段。

**注意，输入字段的名称必须以小写的对象名为前缀。**

例如，负责添加新文章的视图有输入字段:`post.title`和`post.post`，它们对应于`Post`中的字段`title` 和`post` 。`java` 分别为:

```java
<input type="text" class="form-control" placeholder="Title" 
  id="title" name="post.title" required />

<textarea rows="10" class="form-control" placeholder="Post" 
  id="post" name="post.post" required></textarea>
```

完整的`add.ftl`文件可以在源代码中找到。

如果表单提交中有错误，错误消息会被包含进来，用户会被重定向到同一个`add()`方法:

```java
if(validator.hasErrors()) {
    result.include("errors", validator.getErrors());
}
validator.onErrorRedirectTo(this).addForm();
```

## 7 .**。结论**

总之，我们已经浏览了 VRaptor，并了解了基本的 MVC 功能是如何实现的。

文档包含了更多关于框架和可用插件的细节。

完整的源代码，包括一个示例`database.sql`，可以在 Github 的[上获得。](https://web.archive.org/web/20220526044748/https://github.com/eugenp/tutorials/tree/master/vraptor)