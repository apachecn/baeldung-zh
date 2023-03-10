# 使用 HttpURLConnection 发出 JSON POST 请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpurlconnection-post>

## 1.概观

在本教程中，我们将演示如何使用 [`HttpURLConnection`](/web/20221117183257/https://www.baeldung.com/java-http-request) 生成一个 [JSON](/web/20221117183257/https://www.baeldung.com/category/json/) POST 请求。

## 延伸阅读:

## [用 Java 做一个简单的 HTTP 请求](/web/20221117183257/https://www.baeldung.com/java-http-request)

A quick and practical guide to performing basic HTTP requests using Java's built-in HttpUrlConnection.[Read more](/web/20221117183257/https://www.baeldung.com/java-http-request) →

## [使用 HttpUrlConnection 认证](/web/20221117183257/https://www.baeldung.com/java-http-url-connection)

Learn how to authenticate HTTP requests using HttpUrlConnection.[Read more](/web/20221117183257/https://www.baeldung.com/java-http-url-connection) →

## [通过核心 Java 中的代理服务器连接](/web/20221117183257/https://www.baeldung.com/java-connect-via-proxy-server)

Learn how to connect to proxy servers in Java using system properties or the more flexible Proxy class.[Read more](/web/20221117183257/https://www.baeldung.com/java-connect-via-proxy-server) →

## 2.用`HttpURLConnection`构建一个 JSON POST 请求

### 2.1.创建一个`URL`对象

让我们创建一个带有目标 URI 字符串的`URL`对象，它通过 HTTP POST 方法接受 JSON 数据:

```java
URL url = new URL ("https://reqres.in/api/users");
```

### 2.2.打开连接

从上面的`URL` 对象，我们可以调用`openConnection`方法来获得`HttpURLConnection`对象。

我们不能直接实例化`HttpURLConnection`，因为它是一个抽象类:

```java
HttpURLConnection con = (HttpURLConnection)url.openConnection();
```

### 2.3。设置请求方法

要发送 POST 请求，我们必须将请求方法属性设置为 POST:

```java
con.setRequestMethod("POST");
```

### 2.4.设置请求内容类型头参数

**将`“content-type”`请求头设置为`“application/json”`** 以 JSON 形式发送请求内容。必须设置该参数，以 JSON 格式发送请求正文。

否则，服务器将返回 HTTP 状态代码“400-错误请求”:

```java
con.setRequestProperty("Content-Type", "application/json");
```

### 2.5.设置响应格式类型

**将`“Accept”`请求头设置为`“application/json”` ，以期望的格式读取响应:**

```java
con.setRequestProperty("Accept", "application/json");
```

### 2.6.确保连接将用于发送内容

要发送请求内容，让我们将`URLConnection`对象的`doOutput`属性启用给`true`。

否则，我们将无法将内容写入连接输出流:

```java
con.setDoOutput(true);
```

### 2.7.创建请求正文

创建自定义 JSON 字符串后:

```java
String jsonInputString = "{"name": "Upendra", "job": "Programmer"}";
```

我们需要这样写:

```java
try(OutputStream os = con.getOutputStream()) {
    byte[] input = jsonInputString.getBytes("utf-8");
    os.write(input, 0, input.length);			
}
```

### 2.8.从输入流中读取响应

获取输入流以读取响应内容。记住使用 try-with-resources 来自动关闭响应流。

通读整个响应内容，并打印最终的响应字符串:

```java
try(BufferedReader br = new BufferedReader(
  new InputStreamReader(con.getInputStream(), "utf-8"))) {
    StringBuilder response = new StringBuilder();
    String responseLine = null;
    while ((responseLine = br.readLine()) != null) {
        response.append(responseLine.trim());
    }
    System.out.println(response.toString());
}
```

如果响应是 JSON 格式，使用任何第三方 JSON 解析器如 [`Jackson`](/web/20221117183257/https://www.baeldung.com/jackson) 库、 [`Gson`](/web/20221117183257/https://www.baeldung.com/gson-string-to-jsonobject) 或 [`org.json`](/web/20221117183257/https://www.baeldung.com/java-org-json) 解析响应。

## 3.结论

在本文中，我们学习了如何使用`HttpURLConnection`通过 JSON 内容体发出 POST 请求。

和往常一样，相关的代码片段可以在 GitHub 上找到