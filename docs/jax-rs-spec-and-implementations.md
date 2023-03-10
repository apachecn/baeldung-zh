# JAX-RS 只是一个 API！

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jax-rs-spec-and-implementations>

## 1。概述

REST 范式已经存在好几年了，并且仍然受到很多关注。

一个 RESTful API 可以用多种方式在 Java 中实现:你可以使用 Spring、JAX RS，或者如果你足够优秀和勇敢，你可以只编写自己的简单 servlets。您所需要的是公开 HTTP 方法的能力——剩下的就是如何组织它们以及如何在调用 API 时指导客户端。

正如你可以从标题中看出，这篇文章将涵盖 JAX 遥感。但是“只是一个 API”是什么意思呢？这意味着这里的重点是澄清 JAX 遥感及其实现之间的混淆，并提供一个正确的 JAX 遥感网络应用程序的例子。

## 2。Java EE 中的包含

JAX-RS 只不过是一个规范，是 Java EE 提供的一组接口和注释。然后，当然，我们有实现；一些更著名的是 RESTEasy 和 T2 球衣。

此外，如果您决定构建一个符合 JEE 标准的应用服务器，Oracle 的人会告诉您，除了其他事情之外，您的服务器应该为部署的应用程序提供 JAX-RS 实现。所以才叫 Java 企业版**平台**。

规范和实现的另一个好例子是 [JPA](https://web.archive.org/web/20220930134946/https://docs.oracle.com/javaee/6/tutorial/doc/bnbpz.html) 和 [Hibernate。](https://web.archive.org/web/20220930134946/http://hibernate.org/)

### 2.1。轻量级战争

那么这一切对我们这些开发者有什么帮助呢？帮助在于我们的可部署性可以而且应该非常瘦，让应用服务器提供所需的库。这也适用于开发 RESTful API:最终的工件不应该包含任何关于所使用的 JAX-RS 实现的信息。

当然，我们可以提供实现(这里的是 RESTeasy 的教程)。但是我们不能再称我们的应用程序为“Java EE app”了。如果明天有人来说“T0”，我们也许能够做到，但这不会是一件容易的工作。

如果我们提供自己的实现，我们必须确保服务器知道排除自己的实现——这通常是通过在可部署的内部拥有一个专有的 XML 文件来实现的。不用说，这样一个文件应该包含各种各样的标签和说明，除了三年前离开公司的开发人员之外，没有人对这些内容一无所知。

### 2.2。始终了解您的服务器

到目前为止，我们说我们应该利用我们提供的平台。

在决定使用哪种服务器之前，我们应该看看它提供了什么样的 JAX-RS 实现(名称、供应商、版本和已知缺陷)，至少对于生产环境是这样的。例如，Glassfish 与 Jersey 一起提供，而 Wildfly 或 Jboss 与 RESTEasy 一起提供。

当然，这意味着要花一点时间在研究上，但是应该只做一次，在项目开始的时候或者当它迁移到另一个服务器上的时候。

### 3。一个例子

如果你想开始玩 JAX-RS，最短的路径是:在`pom.xml`中有一个具有以下依赖关系的 Maven webapp 项目:

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency> 
```

我们使用 JavaEE 7，因为已经有很多应用服务器实现了它。这个 API jar 包含您需要使用的注释，位于包`javax.ws.rs`中。为什么范围是“规定”的？因为这个 jar 也不需要在最终版本中——我们在编译时需要它，它在运行时由服务器提供。

添加完依赖项后，我们首先必须编写入口类:一个空类，它扩展了`javax.ws.rs.core.Application` ，并用`javax.ws.rs.ApplicationPath:` 进行了注释

```java
@ApplicationPath("/api")
public class RestApplication extends Application {
} 
```

我们将入口路径定义为`/api.` ，无论我们为资源声明什么其他路径，它们都将带有前缀`/api`。

接下来，我们来看一个资源:

```java
@Path("/notifications")
public class NotificationsResource {
    @GET
    @Path("/ping")
    public Response ping() {
        return Response.ok().entity("Service online").build();
    }

    @GET
    @Path("/get/{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getNotification(@PathParam("id") int id) {
        return Response.ok()
          .entity(new Notification(id, "john", "test notification"))
          .build();
    }

    @POST
    @Path("/post/")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response postNotification(Notification notification) {
        return Response.status(201).entity(notification).build();
    }
}
```

我们有一个简单的 ping 端点来调用和检查我们的应用程序是否正在运行，一个 GET 和一个 POST 来通知(这只是一个带有属性加上 getters 和 setters 的 POJO)。

将这个 war 部署在任何实现 JEE7 的应用服务器上，下面的命令将会起作用:

```java
curl http://localhost:8080/simple-jaxrs-ex/api/notifications/ping/

curl http://localhost:8080/simple-jaxrs-ex/api/notifications/get/1

curl -X POST -d '{"id":23,"text":"lorem ipsum","username":"johana"}' 
  http://localhost:8080/simple-jaxrs-ex/api/notifications/post/ 
  --header "Content-Type:application/json"
```

其中 simple-jaxrs-ex 是 webapp 的上下文根。

这已经在 Glassfish 4.1.0 和 Wildfly 9.0.1.Final 中测试过了。请注意，最后两个命令不能在 Glassfish 4.1.1 中使用，因为[这个](https://web.archive.org/web/20220930134946/https://stackoverflow.com/questions/33319659/moxy-exceptions-in-javaee-jersey-2-0-project)错误。在这个 Glassfish 版本中，关于 JSON 的序列化，这显然是一个已知的问题(如果您必须使用这个服务器版本，您必须自己管理 JSON 封送)

## 4。结论

在这篇文章的最后，请记住 JAX-RS 是一个强大的 API，你需要的大部分(如果不是全部)东西已经被你的 web 服务器实现了。没有必要把你的可部署库变成一堆难以管理的库。

这篇文章给出了一个简单的例子，事情可能会变得更加复杂。例如，您可能想要编写自己的封送拆收器。如果需要的话，可以找一些教程来解决 JAX RS 的问题，而不是 Jersey、Resteasy 或其他具体的实现。你的问题很有可能用一两个注解就能解决。