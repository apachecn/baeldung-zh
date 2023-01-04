# 在 Groovy 中使用 Web 服务的快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-web-services>

## 1.概观

如今，我们看到了一系列通过应用程序在 web 上公开数据的方法。

通常，应用程序使用一个 [SOAP](/web/20220630132241/https://www.baeldung.com/spring-boot-soap-web-service) 或 [REST](/web/20220630132241/https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration) web 服务来公开他们的 API。然而，也可以考虑像 RSS 和 Atom 这样的流协议。

在这个快速教程中，我们将探索在 [Groovy](/web/20220630132241/https://www.baeldung.com/groovy-language) 中为这些协议中的每一个使用 web 服务的一些简便方法。

## 2.执行 HTTP 请求

首先，让我们使用 [`URL`](/web/20220630132241/https://www.baeldung.com/java-url) 类执行一个简单的 HTTP GET 请求。我们将在探索过程中消耗[邮递员回声](https://web.archive.org/web/20220630132241/https://docs.postman-echo.com/?version=latest)API。

首先，我们将调用`URL`类的`openConnection`方法，然后设置`requestMethod`来获取:

```
def postmanGet = new URL('https://postman-echo.com/get')
def getConnection = postmanGet.openConnection()
getConnection.requestMethod = 'GET'
assert getConnection.responseCode == 200
```

类似地，我们可以通过将`requestMethod` 设置为 POST 来发出 POST 请求:

```
def postmanPost = new URL('https://postman-echo.com/post')
def postConnection = postmanPost.openConnection()
postConnection.requestMethod = 'POST'
assert postConnection.responseCode == 200
```

同样，我们可以使用 [`outputStream.withWriter`](/web/20220630132241/https://www.baeldung.com/groovy-io#writing) 将参数传递给 POST 请求:

```
def form = "param1=This is request parameter."
postConnection.doOutput = true
def text
postConnection.with {
    outputStream.withWriter { outputStreamWriter ->
        outputStreamWriter << form
    }
    text = content.text
}
assert postConnection.responseCode == 200
```

在这里，Groovy 的`with`闭包看起来非常方便，并且使代码更加整洁。

让我们使用`JsonSlurper`将`String`响应解析成 JSON:

```
JsonSlurper jsonSlurper = new JsonSlurper()
assert jsonSlurper.parseText(text)?.json.param1 == "This is request parameter."
```

## 3.RSS 和 Atom 提要

RSS 和 [Atom](https://web.archive.org/web/20220630132241/https://en.wikipedia.org/wiki/Atom_(Web_standard)) feed 是在网络上展示新闻、博客和技术论坛等内容的常见方式。

另外，这两个提要都是 XML 格式的。因此，我们可以使用 Groovy 的 [`XMLParser`](/web/20220630132241/https://www.baeldung.com/groovy-xml#xml-parser) 类来解析内容。

让我们来读一些来自谷歌新闻的头条新闻:

```
def rssFeed = new XmlParser()
    .parse("https://news.google.com/rss?hl=en-US&gl;=US&ceid;=US:en")
def stories = []
(0..4).each {
    def item = rssFeed.channel.item.get(it)
    stories << item.title.text()
}
assert stories.size() == 5
```

类似地，我们可以读取 Atom 提要。然而，由于两种协议的规范存在差异，我们将在 Atom 提要中以不同的方式访问内容:

```
def atomFeed = new XmlParser()
    .parse("https://news.google.com/atom?hl=en-US&gl;=US&ceid;=US:en")
def stories = []
(0..4).each {
    def entry = atomFeed.entry.get(it)
    stories << entry.title.text()
}
assert stories.size() == 5
```

另外，我们知道 Groovy 支持所有的 Java 库。因此，我们肯定可以使用 [Rome API](/web/20220630132241/https://www.baeldung.com/rome-rss) 来读取 RSS 提要。

## 4.SOAP 请求和响应

SOAP 是应用程序用来在 web 上公开其服务的最流行的 web 服务协议之一。

我们将使用 [groovy-wslite](https://web.archive.org/web/20220630132241/https://github.com/jwagenleitner/groovy-wslite) 库来消费 SOAP APIs。让我们将它最新的[依赖](https://web.archive.org/web/20220630132241/https://mvnrepository.com/artifact/com.github.groovy-wslite/groovy-wslite)添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>com.github.groovy-wslite</groupId>
    <artifactId>groovy-wslite</artifactId>
    <version>1.1.3</version>
</dependency>
```

或者，我们可以使用 Gradle 添加最新的依赖项:

```
compile group: 'com.github.groovy-wslite', name: 'groovy-wslite', version: '1.1.3'
```

或者如果我们想写一个 Groovy 脚本。我们可以用`@Grab`直接添加:

```
@Grab(group='com.github.groovy-wslite', module='groovy-wslite', version='1.1.3')
```

**groovy-wslite 库提供了`SOAPClient`类来与 SOAP API**通信。同时，它有一个`SOAPMessageBuilder` 类来创建请求消息。

让我们使用`SOAPClient`来消费一个[号码转换 SOAP 服务](https://web.archive.org/web/20220630132241/http://www.dataaccess.com/webservicesserver/numberconversion.wso):

```
def url = "http://www.dataaccess.com/webservicesserver/numberconversion.wso"
def soapClient = new SOAPClient(url)
def message = new SOAPMessageBuilder().build({
    body {
        NumberToWords(xmlns: "http://www.dataaccess.com/webservicesserver/") {
            ubiNum(123)
        }
    }
})
def response = soapClient.send(message.toString());
def words = response.NumberToWordsResponse
assert words == "one hundred and twenty three "
```

## 5.REST 请求和响应

REST 是用于创建 web 服务的另一种流行的架构风格。此外，API 是基于 HTTP 方法公开的，如 GET、POST、PUT 和 DELETE。

### 5.1.得到

我们将使用已经讨论过的 groovy-wslite 库来使用 REST APIs。**它提供了`RESTClient`类，用于无障碍通信**。

让我们向已经讨论过的 Postman API 发出一个 GET 请求:

```
RESTClient client = new RESTClient("https://postman-echo.com")
def path = "/get"
def response
try {
    response = client.get(path: path)
    assert response.statusCode = 200
    assert response.json?.headers?.host == "postman-echo.com"
} catch (RESTClientException e) {
    assert e?.response?.statusCode != 200
}
```

### 5.2.邮政

现在，让我们向 Postman API 发出一个 POST 请求。同时，我们将把表单参数作为 JSON 传递:

```
client.defaultAcceptHeader = ContentType.JSON
def path = "/post"
def params = ["foo":1,"bar":2]
def response = client.post(path: path) {
    type ContentType.JSON
    json params
}
assert response.json?.data == params 
```

这里，我们将 JSON 设置为默认的 accept 头。

## 6.Web 服务的身份验证

随着越来越多的 web 服务和应用程序相互通信，建议使用安全的 web 服务。

因此，**HTTPS 与基本 Auth 和 OAuth 等认证机制的结合非常重要**。

因此，应用程序在使用 web 服务 API 时必须对自己进行身份验证。

### 6.1.基本认证

我们可以使用已经讨论过的`RESTClient `类。让我们使用带有凭证的`HTTPBasicAuthorization` 类来执行基本认证:

```
def path = "/basic-auth"
client.authorization = new HTTPBasicAuthorization("postman", "password")
response = client.get(path: path)
assert response.statusCode == 200
assert response.json?.authenticated == true
```

或者，我们可以直接在`headers`参数中传递凭证(Base64 编码):

```
def response = client
.get(path: path, headers: ["Authorization": "Basic cG9zdG1hbjpwYXNzd29yZA=="])
```

### 6.2.OAuth 1.0

类似地，我们可以发出 OAuth 1.0 请求，传递像消费者密钥和消费者秘密这样的认证参数。

然而，由于我们不像对其他机制那样内置对 OAuth 1.0 的支持，我们必须自己完成这项工作:

```
def path = "/oauth1"
def params = [oauth_consumer_key: "RKCGzna7bv9YD57c", 
    oauth_signature_method: "HMAC-SHA1", 
    oauth_timestamp:1567089944, oauth_nonce: "URT7v4", oauth_version: 1.0, 
    oauth_signature: 'RGgR/ktDmclkM0ISWaFzebtlO0A=']
def response = new RESTClient("https://postman-echo.com")
    .get(path: path, query: params)
assert response.statusCode == 200
assert response.statusMessage == "OK"
assert response.json.status == "pass"
```

## 7.结论

在本教程中，我们探索了一些在 Groovy 中使用 web 服务的简便方法。

与此同时，我们看到了一种读取 RSS 或 Atom 提要的简单方法。

像往常一样，代码实现可以在 GitHub 的[上获得。](https://web.archive.org/web/20220630132241/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2)