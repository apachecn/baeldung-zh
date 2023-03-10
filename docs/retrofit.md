# 改装简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/retrofit>

## 1。概述

[翻新](https://web.archive.org/web/20221210093438/https://square.github.io/retrofit/)是一款适用于 Android 和 Java 的类型安全 HTTP 客户端——由 Square ( [Dagger](https://web.archive.org/web/20221210093438/https://square.github.io/dagger/) 、 [Okhttp](https://web.archive.org/web/20221210093438/https://square.github.io/okhttp/) )开发。

在这篇文章中，我们将解释如何使用翻新，重点是它最有趣的功能。更值得注意的是，我们将讨论同步和异步 API，如何将它与身份验证、日志记录和一些良好的建模实践结合使用。

## 2。设置示例

我们将从添加改造库和 Gson 转换器开始:

```java
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>retrofit</artifactId>
    <version>2.3.0</version>
</dependency>  
<dependency>  
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>converter-gson</artifactId>
    <version>2.3.0</version>
</dependency>
```

对于最新版本，看看 Maven 中央存储库上的[改型](https://web.archive.org/web/20221210093438/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.retrofit2%22%20AND%20a%3A%22retrofit%22)和[转换器-gson](https://web.archive.org/web/20221210093438/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.retrofit2%22%20AND%20a%3A%22converter-gson%22) 。

## 3。API 建模

改型模型将端点作为 Java 接口，使得它们非常容易理解和使用。

我们将对来自 GitHub 的用户 API 建模；它有一个以 JSON 格式返回的`GET`端点:

```java
{
  login: "mojombo",
  id: 1,
  url: "https://api.github.com/users/mojombo",
  ...
}
```

翻新的工作原理是在一个基本 URL 上建模，并让接口从 REST 端点返回实体。

为了简单起见，我们将通过对我们的`User`类建模来获取 JSON 的一小部分，该类将在我们收到值时获取这些值:

```java
public class User {
    private String login;
    private long id;
    private String url;
    // ...

    // standard getters an setters

}
```

我们可以看到，对于这个例子，我们只取了属性的一个子集。**翻新不会抱怨缺少属性——因为它只映射我们需要的东西**,即使我们添加了 JSON 中没有的属性，它也不会抱怨。

现在我们可以转到接口建模，并解释一些改进的注释:

```java
public interface UserService {

    @GET("/users")
    public Call<List<User>> getUsers(
      @Query("per_page") int per_page, 
      @Query("page") int page);

    @GET("/users/{username}")
    public Call<User> getUser(@Path("username") String username);
}
```

注释提供的元数据足以让工具生成可工作的实现。

`@GET` 注释告诉客户端使用哪个 HTTP 方法以及在哪个资源上使用，例如，通过提供一个基本 URL“https://API . github . com”，它将把请求发送到“https://api.github.com/users”。

我们的相对 URL **上的前导“/”告诉 Retrofit 它是主机上的绝对路径。**

另一件要注意的事情是，我们使用完全可选的`@Query`参数，如果我们不需要它们，可以将它们作为 null 传递，如果它们没有值，工具会忽略这些参数。

最后但并非最不重要的是，`@Path`让我们指定一个路径参数，它将代替我们在路径中使用的标记。

## 4。同步/异步 API

为了构造一个 HTTP 请求调用，我们需要首先构建我们的改进对象:

```java
OkHttpClient.Builder httpClient = new OkHttpClient.Builder();
Retrofit retrofit = new Retrofit.Builder()
  .baseUrl("https://api.github.com/")
  .addConverterFactory(GsonConverterFactory.create())
  .client(httpClient.build())
  .build();
```

改型为构造我们需要的对象提供了一个方便的生成器。**它需要将用于每个服务调用的基本 URL 和一个转换器工厂**——它负责解析我们发送的数据以及我们得到的响应。

在这个例子中，我们将使用`GsonConverterFactory`，它将把我们的 JSON 数据映射到我们之前定义的`User`类。

需要注意的是，不同的工厂服务于不同的目的，所以请记住，我们也可以为 XML、proto-buffer 使用工厂，甚至为自定义协议创建一个工厂。对于已经实现的工厂列表，我们可以在这里看一下[。](https://web.archive.org/web/20221210093438/https://github.com/square/retrofit/tree/master/retrofit-converters)

最后一个依赖项是`OKHttpClient`——这是一个用于 Android 和 Java 应用程序的 HTTP & HTTP/2 客户端。这将负责连接到服务器以及发送和检索信息。我们还可以为每个调用添加头和拦截器，这将在我们的身份验证部分看到。

既然我们已经有了改造对象，我们就可以构造我们的服务调用了，让我们来看看如何以同步的方式来做这件事:

```java
UserService service = retrofit.create(UserService.class);
Call<User> callSync = service.getUser("eugenp");

try {
    Response<User> response = callSync.execute();
    User user = response.body();
} catch (Exception ex) { ... }
```

在这里，我们可以看到 retrieval 如何根据我们之前的注释，通过注入发出请求所必需的代码来处理我们的服务接口的构造。

之后，我们得到一个`Call<User>`对象，用于执行对 GitHub API 的请求。**这里最重要的方法是`execute`** `,`，用于同步执行一个调用，在传输数据时会阻塞当前线程。

在调用成功执行后，由于我们的`GsonConverterFactory`，我们可以检索响应的主体——已经在用户对象上。

进行同步调用非常容易，但通常，我们使用非阻塞异步请求:

```java
UserService service = retrofit.create(UserService.class);
Call<User> callAsync = service.getUser("eugenp");

callAsync.enqueue(new Callback<User>() {
    @Override
    public void onResponse(Call<User> call, Response<User> response) {
        User user = response.body();
    }

    @Override
    public void onFailure(Call<User> call, Throwable throwable) {
        System.out.println(throwable);
    }
});
```

现在我们不用 execute 方法，而是使用`enqueue`方法——该方法将一个`Callback<User>`接口作为参数来处理请求的成功或失败。请注意，这将在一个单独的线程中执行。

呼叫成功结束后，我们可以像以前一样检索尸体。

## 5。制作一个可重用的`ServiceGenerator`类

既然我们已经看到了如何构造我们的改造对象以及如何使用 API，我们可以看到我们不想一遍又一遍地编写构建器。

我们想要的是一个可重用的类，它允许我们创建一次这个对象，并在应用程序的整个生命周期中重用它:

```java
public class GitHubServiceGenerator {

    private static final String BASE_URL = "https://api.github.com/";

    private static Retrofit.Builder builder
      = new Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create());

    private static Retrofit retrofit = builder.build();

    private static OkHttpClient.Builder httpClient
      = new OkHttpClient.Builder();

    public static <S> S createService(Class<S> serviceClass) {
        return retrofit.create(serviceClass);
    }
}
```

创建改造对象的所有逻辑现在都转移到这个`GitHubServiceGenerator`类，这使它成为一个可持续的客户端类，它阻止代码重复。

下面是如何使用它的一个简单示例:

```java
UserService service 
  = GitHubServiceGenerator.createService(UserService.class);
```

现在，如果我们，例如，要创建一个`RepositoryService,`，我们可以重用这个类并简化创建。

**在下一节**中，我们将对其进行扩展，并添加认证功能。

## 6。认证

大多数 API 都有一些身份验证来保护对它的访问。

考虑到我们之前的生成器类，我们将添加一个创建服务方法，该方法采用一个带有`Authorization`头的 JWT 令牌:

```java
public static <S> S createService(Class<S> serviceClass, final String token ) {
   if ( token != null ) {
       httpClient.interceptors().clear();
       httpClient.addInterceptor( chain -> {
           Request original = chain.request();
           Request request = original.newBuilder()
             .header("Authorization", token)
             .build();
           return chain.proceed(request);
       });
       builder.client(httpClient.build());
       retrofit = builder.build();
   }
   return retrofit.create(serviceClass);
}
```

为了给我们的请求添加一个头，我们需要使用`OkHttp`的拦截器功能；我们通过使用先前定义的构建器和重建改造对象来实现这一点。

注意，这是一个简单的身份验证示例，但是通过使用拦截器，我们可以使用任何身份验证，比如 OAuth、用户/密码等。

## 7。记录日志

在这一节中，我们将进一步扩展我们的`GitHubServiceGenerator`的日志功能，这对于每个项目中的调试都非常重要。

我们将使用我们以前关于拦截器的知识，但是我们需要一个额外的依赖项，即 OkHttp 中的`HttpLoggingInterceptor`，让我们将它添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>logging-interceptor</artifactId>
    <version>3.9.0</version>
</dependency>
```

现在让我们扩展我们的`GitHubServiceGenerator`类:

```java
public class GitHubServiceGenerator {

    private static final String BASE_URL = "https://api.github.com/";

    private static Retrofit.Builder builder
      = new Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create());

    private static Retrofit retrofit = builder.build();

    private static OkHttpClient.Builder httpClient
      = new OkHttpClient.Builder();

    private static HttpLoggingInterceptor logging
      = new HttpLoggingInterceptor()
        .setLevel(HttpLoggingInterceptor.Level.BASIC);

    public static <S> S createService(Class<S> serviceClass) {
        if (!httpClient.interceptors().contains(logging)) {
            httpClient.addInterceptor(logging);
            builder.client(httpClient.build());
            retrofit = builder.build();
        }
        return retrofit.create(serviceClass);
    }

    public static <S> S createService(Class<S> serviceClass, final String token) {
        if (token != null) {
            httpClient.interceptors().clear();
            httpClient.addInterceptor( chain -> {
                Request original = chain.request();
                Request.Builder builder1 = original.newBuilder()
                  .header("Authorization", token);
                Request request = builder1.build();
                return chain.proceed(request);
            });
            builder.client(httpClient.build());
            retrofit = builder.build();
        }
        return retrofit.create(serviceClass);
    }
}
```

这是我们类的最终形式，我们可以看到我们是如何添加`HttpLoggingInterceptor`的，我们将它设置为基本日志记录，它将记录发出请求所用的时间、端点、每个请求的状态等。

看一下我们如何检查拦截器是否存在是很重要的，这样我们就不会不小心添加了两次。

## 8。结论

在这本内容丰富的指南中，我们通过关注它的 Sync/Async API、建模、身份验证和日志记录的一些最佳实践，了解了这个优秀的改进库。

该库可以以非常复杂和有用的方式使用；关于 RxJava 的高级用例，请查看本教程[。](/web/20221210093438/https://www.baeldung.com/retrofit-rxjava)

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221210093438/https://github.com/eugenp/tutorials/tree/master/libraries-http)