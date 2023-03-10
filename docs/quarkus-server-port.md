# 在 quartus 应用程序上配置服务器端口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/quarkus-server-port>

## 1.概观

在这个快速教程中，我们将学习如何在 Quarkus 应用程序上配置服务器端口。

## 2.配置端口

默认情况下，类似于许多其他 Java 服务器应用程序，Quarkus 监听端口 8080。为了改变默认的服务器端口，**我们可以使用`quarkus.http.port `属性。**

Quarkus 从[各种来源](https://web.archive.org/web/20220523152627/https://quarkus.io/guides/config-reference#configuration-sources)读取其配置属性。因此，我们也可以从不同的来源更改`quarkus.http.port `属性。例如，我们可以让 Quarkus 监听端口 9000，方法是将这个添加到我们的`application.properties`:

```java
quarkus.http.port=9000
```

现在，如果我们向`localhost:9000, `发送请求，服务器将返回某种 HTTP 响应:

```java
>> curl localhost:9000/hello
Hello RESTEasy
```

也可以通过 Java 系统属性和`-D`参数来配置端口:

```java
>> mvn compile quarkus:dev -Dquarkus.http.port=9000
// omitted
Listening on: http://localhost:9000
```

如上所示，系统属性覆盖了这里的默认端口。除此之外，我们还可以使用环境变量:

```java
>> QUARKUS_HTTP_PORT=9000 
>> mvn compile quarkus:dev
```

在这里，我们将`quarkus.http.port `中的所有字符转换成大写，并用下划线代替点，形成[环境变量名](https://web.archive.org/web/20220523152627/https://github.com/eclipse/microprofile-config/blob/master/spec/src/main/asciidoc/configsources.asciidoc#default-configsources)。

## 3.结论

在这个简短的教程中，我们看到了在 Quarkus 中配置应用程序端口的几种方法。