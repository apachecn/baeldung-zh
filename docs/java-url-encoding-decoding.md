# Java URL 编码/解码指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-url-encoding-decoding>

## 1。概述

简而言之， [URL 编码](https://web.archive.org/web/20220807192129/https://en.wikipedia.org/wiki/Percent-encoding)将 URL 中的特殊字符翻译成符合规范的表示形式，并且可以被正确理解和解释。

在本教程中，我们将关注**如何编码/解码 URL 或表单数据**,使其符合规范并在网络上正确传输。

## 2。分析网址

让我们先来看一个基本的 [URI](https://web.archive.org/web/20220807192129/https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) 语法:

```java
scheme:[//[user:[[email protected]](/web/20220807192129/https://www.baeldung.com/cdn-cgi/l/email-protection)]host[:port]][/]path[?query][#fragment]
```

对 URI 进行编码的第一步是检查它的各个部分，然后只对相关部分进行编码。

现在让我们看一个 URI 的例子:

```java
String testUrl = 
  "http://www.baeldung.com?key1=value+1&key2;=value%40%21%242&key3;=value%253";
```

分析 URI 的一种方法是将字符串表示加载到一个`java.net.URI`类中:

```java
@Test
public void givenURL_whenAnalyze_thenCorrect() throws Exception {
    URI uri = new URI(testUrl);

    assertThat(uri.getScheme(), is("http"));
    assertThat(uri.getHost(), is("www.baeldung.com"));
    assertThat(uri.getRawQuery(),
      .is("key1=value+1&key2;=value%40%21%242&key3;=value%253"));
}
```

`URI`类解析字符串表示 URL，并通过一个简单的 API，例如`getXXX`，公开它的各个部分。

## 3。对 URL 进行编码

当编码 URI 时，一个常见的陷阱是编码完整的 URI。通常，我们只需要对 URI 的查询部分进行编码。

让我们使用`URLEncoder`类的`encode(data, encodingScheme)`方法对数据进行编码:

```java
private String encodeValue(String value) {
    return URLEncoder.encode(value, StandardCharsets.UTF_8.toString());
}

@Test
public void givenRequestParam_whenUTF8Scheme_thenEncode() throws Exception {
    Map<String, String> requestParams = new HashMap<>();
    requestParams.put("key1", "value 1");
    requestParams.put("key2", "[[email protected]](/web/20220807192129/https://www.baeldung.com/cdn-cgi/l/email-protection)!$2");
    requestParams.put("key3", "value%3");

    String encodedURL = requestParams.keySet().stream()
      .map(key -> key + "=" + encodeValue(requestParams.get(key)))
      .collect(joining("&", "http://www.baeldung.com?", ""));

    assertThat(testUrl, is(encodedURL)); 
```

`encode`方法接受两个参数:

1.  `data`–要翻译的字符串
2.  `encodingScheme`–字符编码的名称

这个`encode`方法将字符串转换成 [`**application/x-www-form-urlencoded**`](https://web.archive.org/web/20220807192129/https://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.1) 格式。

编码方案会将特殊字符转换为八位的两位十六进制表示，以“`%xy`”的形式表示。当我们处理路径参数或添加动态参数时，我们会对数据进行编码，然后发送到服务器。

**注意:**`**World Wide Web Consortium**`建议声明我们应该使用`UTF-8`。不这样做可能会引入不兼容性。(参考:`https://docs.oracle.com/javase/7/docs/api/java/net/URLEncoder.html`)

## 4。解码网址

现在让我们使用`URLDecoder`的解码方法来解码前面的 URL:

```java
private String decode(String value) {
    return URLDecoder.decode(value, StandardCharsets.UTF_8.toString());
}

@Test
public void givenRequestParam_whenUTF8Scheme_thenDecodeRequestParams() {
    URI uri = new URI(testUrl);

    String scheme = uri.getScheme();
    String host = uri.getHost();
    String query = uri.getRawQuery();

    String decodedQuery = Arrays.stream(query.split("&"))
      .map(param -> param.split("=")[0] + "=" + decode(param.split("=")[1]))
      .collect(Collectors.joining("&"));

    assertEquals(
      "http://www.baeldung.com?key1=value [[email protected]](/web/20220807192129/https://www.baeldung.com/cdn-cgi/l/email-protection)!$2&key3;=value%3",
      scheme + "://" + host + "?" + decodedQuery);
}
```

这里有两点需要记住:

*   解码前分析 URL
*   使用相同的编码方案进行编码和解码

如果我们要解码然后分析，URL 部分可能无法正确解析。如果我们使用另一种编码方案来解码数据，就会产生垃圾数据。

## 5.对路径段进行编码

我们不能使用`URLEncoder`对`URL`的路径段进行编码。路径组件是指代表目录路径的分层结构，或者它用于定位由“/”分隔的资源。

路径段中的保留字符不同于查询参数值中的保留字符。例如，“+”号是路径段中的有效字符，因此不应进行编码。

为了对路径段进行编码，我们改用 Spring 框架的`UriUtils` 类。

`UriUtils` 类提供了`encodePath`和`encodePathSegment` 方法，分别对路径和路径段进行编码；

```java
private String encodePath(String path) {
    try {
        path = UriUtils.encodePath(path, "UTF-8");
    } catch (UnsupportedEncodingException e) {
        LOGGER.error("Error encoding parameter {}", e.getMessage(), e);
    }
    return path;
}
```

```java
@Test
public void givenPathSegment_thenEncodeDecode() 
  throws UnsupportedEncodingException {
    String pathSegment = "/Path 1/Path+2";
    String encodedPathSegment = encodePath(pathSegment);
    String decodedPathSegment = UriUtils.decode(encodedPathSegment, "UTF-8");

    assertEquals("/Path%201/Path+2", encodedPathSegment);
    assertEquals("/Path 1/Path+2", decodedPathSegment);
}
```

在上面的代码片段中，我们可以看到，当我们使用`encodePathSegment`方法时，它返回的是编码后的值，而+并没有被编码，因为它是 path 组件中的值字符。

让我们为测试 URL 添加一个路径变量:

```java
String testUrl
  = "/path+1?key1=value+1&key2;=value%40%21%242&key3;=value%253";
```

为了组装和断言正确编码的 URL，我们将更改第 2 节中的测试:

```java
String path = "path+1";
String encodedURL = requestParams.keySet().stream()
  .map(k -> k + "=" + encodeValue(requestParams.get(k)))
  .collect(joining("&", "/" + encodePath(path) + "?", ""));
assertThat(testUrl, CoreMatchers.is(encodedURL)); 
```

## 6。结论

在本文中，我们看到了如何编码和解码数据，以便能够正确地传输和解释数据。

虽然本文主要关注编码/解码 URI 查询参数值，但是这种方法也适用于 HTML 表单参数。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220807192129/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking)