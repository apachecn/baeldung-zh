# 改进 2–动态 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/retrofit-dynamic-url>

## 1.概观

在这个简短的教程中，我们将学习如何在[翻新 2](/web/20220627082636/https://www.baeldung.com/retrofit)T3 中**创建一个动态 URL。**

## 2.`@Url`注释

有些情况下，我们需要在运行时在应用程序中使用动态 URL。[改进库](https://web.archive.org/web/20220627082636/https://search.maven.org/search?q=g:%20com.squareup.retrofit2%20AND%20a:%20retrofit)的版本 2 引入了`@Url`注释，允许我们**传递端点**的完整 URL:

```java
@GET
Call<ResponseBody> reposList(@Url String url);
```

这个注释基于 [OkHttp 的](https://web.archive.org/web/20220627082636/https://search.maven.org/search?q=g:com.squareup.okhttp3%20AND%20a:okhttp)库中的 [HttpUrl](https://web.archive.org/web/20220627082636/https://square.github.io/okhttp/4.x/okhttp/okhttp3/-http-url/) 类，Url 地址就像页面上的链接一样使用`<a href=` "" `>`来解析。当使用`@Url`参数时，我们不需要在`@GET`注释中指定地址。

@ `Url`参数替换了服务实现中的`baseUrl`:

```java
Retrofit retrofit = new Retrofit.Builder()
  .baseUrl("https://api.github.com/")
  .addConverterFactory(GsonConverterFactory.create()).build();
```

重要的是，如果我们要使用`@Url`注释，必须将其设置为服务方法中的第一个参数。

## 3.路径参数

如果我们知道我们的基本 URL 的某个部分将是常量，但我们不知道它的扩展名或将要使用的参数的数量，我们可以使用 **`@Path`注释和`encoded`** 标志:

```java
@GET("{fullUrl}")
Call<List<Contributor>> contributorsList(@Path(value = "fullUrl", encoded = true) String fullUrl);
```

这样所有的“/”都不会被替换成`%2F`，就好像我们没有使用过编码的参数一样。但是，所有字符“？”在传递的地址中仍然会被%3F 替换`.`

## 4.摘要

改进的库允许我们在应用程序运行时通过仅使用`@Url`注释来容易地提供动态 URL。

像往常一样，所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220627082636/https://github.com/eugenp/tutorials/tree/master/libraries-http-2)