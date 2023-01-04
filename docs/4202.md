# 用 OkHttp 发布请求的快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/okhttp-post>

## 1.介绍

我们在 OkHttp 的[指南中介绍了](/web/20220617075806/https://www.baeldung.com/guide-to-okhttp) [OkHttp](https://web.archive.org/web/20220617075806/https://square.github.io/okhttp/) 客户端的基础知识。

在这个简短的教程中，我们将专门研究 3.x 版客户端的不同类型的 POST 请求。

## 2.基本职位

我们可以使用`FormBody.Builder`构建一个基本的`RequestBody`来发送两个参数——*用户名*和`password` ——以及一个 POST 请求:

```
@Test
public void whenSendPostRequest_thenCorrect() 
  throws IOException {
    RequestBody formBody = new FormBody.Builder()
      .add("username", "test")
      .add("password", "test")
      .build();

    Request request = new Request.Builder()
      .url(BASE_URL + "/users")
      .post(formBody)
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 3.授权发布

如果我们想要认证请求，我们可以使用`Credentials.basic`构建器将凭证添加到消息头中。

在这个简单的例子中，我们还将发送一个`String`作为请求的主体:

```
@Test
public void whenSendPostRequestWithAuthorization_thenCorrect() 
  throws IOException {
    String postBody = "test post";

    Request request = new Request.Builder()
      .url(URL_SECURED_BY_BASIC_AUTHENTICATION)
      .addHeader("Authorization", Credentials.basic("username", "password"))
      .post(RequestBody.create(
        MediaType.parse("text/x-markdown), postBody))
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 4.用 JSON 发布

为了在请求体中发送 JSON，我们必须设置它的媒体类型`application/json`。我们可以使用`RequestBody.create`构建器来实现:

```
@Test
public void whenPostJson_thenCorrect() throws IOException {
    String json = "{\"id\":1,\"name\":\"John\"}";

    RequestBody body = RequestBody.create(
      MediaType.parse("application/json"), json);

    Request request = new Request.Builder()
      .url(BASE_URL + "/users/detail")
      .post(body)
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 5.多方发布请求

我们要看的最后一个例子是 POST multipart 请求。我们需要将我们的`RequestBody`构建成一个`MultipartBody` 来发布文件、用户名和密码:

```
@Test
public void whenSendMultipartRequest_thenCorrect() 
  throws IOException {	
    RequestBody requestBody = new MultipartBody.Builder()
      .setType(MultipartBody.FORM)
      .addFormDataPart("username", "test")
      .addFormDataPart("password", "test")
      .addFormDataPart("file", "file.txt",
        RequestBody.create(MediaType.parse("application/octet-stream"), 
          new File("src/test/resources/test.txt")))
      .build();

    Request request = new Request.Builder()
      .url(BASE_URL + "/users/multipart")
      .post(requestBody)
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
} 
```

## 6.使用非默认字符编码发布

OkHttp 的默认字符编码是 UTF-8:

```
@Test
public void whenPostJsonWithoutCharset_thenCharsetIsUtf8() throws IOException {
    final String json = "{\"id\":1,\"name\":\"John\"}";

    final RequestBody body = RequestBody.create(
        MediaType.parse("application/json"), json);

    String charset = body.contentType().charset().displayName();

    assertThat(charset, equalTo("UTF-8"));
}
```

如果我们想使用不同的字符编码，我们可以将它作为第二个参数传递给`MediaType.parse()`:

```
@Test
public void whenPostJsonWithUtf16Charset_thenCharsetIsUtf16() throws IOException {
    final String json = "{\"id\":1,\"name\":\"John\"}";

    final RequestBody body = RequestBody.create(
        MediaType.parse("application/json; charset=utf-16"), json);

    String charset = body.contentType().charset().displayName();

    assertThat(charset, equalTo("UTF-16"));
}
```

## 7.结论

在这篇短文中，我们看到了几个使用`OkHttp`客户端的 POST 请求的例子。

像往常一样，代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20220617075806/https://github.com/eugenp/tutorials/tree/master/libraries-http)