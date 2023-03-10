# RESTEasy 客户端 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/resteasy-client-tutorial>

## 1。简介

在[之前的文章](/web/20220815045247/https://www.baeldung.com/resteasy-tutorial)中，我们重点介绍了 **JAX-RS 2.0** 的 **RESTEasy** 服务器端实现。

JAX-RS 2.0 引入了一个新的客户端 API，这样你就可以向你的远程 RESTful web 服务发出 HTTP 请求。Jersey、Apache CXF、Restlet 和 RESTEasy 只是最流行的实现的子集。

在本文中，我们将探索如何通过使用 **RESTEasy API** 发送请求来使用 **REST API** 。

## 2。项目设置

在您的 **`pom.xml`** 中添加以下依赖项:

```java
<properties>
    <resteasy.version>4.7.2.Final</resteasy.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-client</artifactId>
        <version>${resteasy.version}</version>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
    </dependency>
    ...
</dependencies>
```

## 3。客户端代码

客户端实现非常小，由 3 个主要类组成:

`Client`接口是`WebTarget`实例的构建器。

`WebTarget`代表一个独特的 URL 或 URL 模板，您可以从中构建更多的子资源 WebTargets 或调用请求。

创建客户端有两种方法:

*   标准方式是使用`org.jboss.resteasy.client.ClientRequest`
*   **RESTeasy 代理框架**:通过使用`ResteasyClientBuilder`类

这里我们将重点关注 RESTEasy 代理框架。

客户端框架不使用 JAX-RS 注释将传入请求映射到 RESTful Web 服务方法，而是构建一个 HTTP 请求，用于在远程 RESTFul Web 服务上调用。

因此，让我们开始编写一个 Java 接口，并在方法和接口上使用 JAX-RS 注释。

### 3.1。`ServicesClient`界面

```java
@Path("/movies")
public interface ServicesInterface {

    @GET
    @Path("/getinfo")
    @Produces({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    Movie movieByImdbId(@QueryParam("imdbId") String imdbId);

    @POST
    @Path("/addmovie")
    @Consumes({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    Response addMovie(Movie movie);

    @PUT
    @Path("/updatemovie")
    @Consumes({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    Response updateMovie(Movie movie);

    @DELETE
    @Path("/deletemovie")
    Response deleteMovie(@QueryParam("imdbId") String imdbId);
} 
```

### 3.2。电影课

```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "movie", propOrder = { "imdbId", "title" })
public class Movie {

    protected String imdbId;
    protected String title;

    // getters and setters
}
```

### 3.3。请求创建

我们现在将生成一个代理客户端，我们可以使用它来消费 API:

```java
String transformerImdbId = "tt0418279";
Movie transformerMovie = new Movie("tt0418279", "Transformer 2");
UriBuilder FULL_PATH = UriBuilder.fromPath("http://127.0.0.1:8082/resteasy/rest");

ResteasyClient client = (ResteasyClient)ClientBuilder.newClient();
ResteasyWebTarget target = client.target(FULL_PATH);
ServicesInterface proxy = target.proxy(ServicesInterface.class);

// POST
Response moviesResponse = proxy.addMovie(transformerMovie);
System.out.println("HTTP code: " + moviesResponse.getStatus());
moviesResponse.close();

// GET
Movie movies = proxy.movieByImdbId(transformerImdbId);

// PUT
transformerMovie.setTitle("Transformer 4");
moviesResponse = proxy.updateMovie(transformerMovie);
moviesResponse.close();

// DELETE
moviesResponse = proxy.deleteMovie(batmanMovie.getImdbId());
moviesResponse.close(); 
```

注意，RESTEasy 客户端 API 是基于 Apache `HttpClient`的。

还要注意，在每个操作之后，我们需要在执行新操作之前关闭响应。这是必要的，因为默认情况下，客户端只有一个可用的 HTTP 连接。

最后，注意我们是如何直接使用 dto 的——我们不处理与`JSON` 或 `XML`之间的编组/解组逻辑；这发生在使用 `JAXB` 或 `Jackson` 的幕后，因为`Movie`类被正确地注释了`.`

### 3.4。用连接池创建请求

前一个例子中的一个注意事项是，我们只有一个可用的连接。例如，如果我们尝试:

```java
Response batmanResponse = proxy.addMovie(batmanMovie);
Response transformerResponse = proxy.addMovie(transformerMovie); 
```

没有 invoke`close()`on`batmanResponse`–执行第二行时会抛出异常:

```java
java.lang.IllegalStateException:
Invalid use of BasicClientConnManager: connection still allocated.
Make sure to release the connection before allocating another one. 
```

同样——这只是因为 RESTEasy 使用的默认`HttpClient`是`org.apache.http.impl.conn.SingleClientConnManager`——这当然只使一个连接可用。

现在——为了解决这个限制——必须以不同的方式创建 `RestEasyClient`实例(使用连接池):

```java
PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
CloseableHttpClient httpClient = HttpClients.custom().setConnectionManager(cm).build();
cm.setMaxTotal(200); // Increase max total connection to 200
cm.setDefaultMaxPerRoute(20); // Increase default max connection per route to 20
ApacheHttpClient43Engine engine = new ApacheHttpClient43Engine(httpClient);

ResteasyClient client = ((ResteasyClientBuilder) ClientBuilder.newBuilder()).httpEngine(engine).build();
ResteasyWebTarget target = client.target(FULL_PATH);
ServicesInterface proxy = target.proxy(ServicesInterface.class);
```

现在我们可以**从一个合适的连接池**中受益，并且可以让多个请求通过我们的客户端运行，而不必每次都释放连接。

## 4。结论

在这个快速教程中，我们介绍了 RESTEasy 代理框架，并用它构建了一个超级简单的客户端 API。

该框架为我们提供了更多配置客户端的帮助方法，可以定义为 JAX-RS 服务器端规范的镜像对立面。

本文中使用的例子是 GitHub 上的一个[示例项目。](https://web.archive.org/web/20220815045247/https://github.com/eugenp/tutorials/tree/master/resteasy)