# Spring Boot 嵌入式 Tomcat 日志

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-embedded-tomcat-logs>

## 1.介绍

Spring Boot 附带了一个嵌入式 Tomcat 服务器，非常方便。但是，我们默认看不到 Tomcat 的日志。

在本教程中，我们将学习如何通过一个玩具应用程序**配置 Spring Boot 来显示 Tomcat 的内部和访问日志**。

## 2.示例应用程序

首先，我们来创建一个 REST API。我们将定义一个`GreetingsController`来问候用户:

```java
@GetMapping("/greetings/{username}")
public String getGreetings(@PathVariable("username") String userName) {
    return "Hello " + userName + ", Good day...!!!";
}
```

## 3.Tomcat 日志类型

嵌入式 Tomcat 存储两种类型的日志:

*   访问日志
*   内部服务器日志

`access logs`保存应用程序处理的所有请求的记录。这些日志可以用来**跟踪页面点击率、** **用户会话活动**等事情。相比之下，`internal server logs`将帮助我们解决运行应用程序中的任何问题。

## 4.访问日志

默认情况下，不会启用访问日志。

不过，我们可以通过向`application.properties`添加一个属性来轻松启用它们:

```java
server.tomcat.accesslog.enabled=true
```

类似地，我们可以使用 VM 参数来启用访问日志:

```java
java -jar -Dserver.tomcat.basedir=tomcat -Dserver.tomcat.accesslog.enabled=true app.jar
```

**这些日志文件将创建在一个临时目录中。**例如，在 Windows 上，访问日志的目录看起来类似于`AppData\Local\Temp\tomcat.2142886552084850151.40123\logs`

### 4.1.格式

因此，启用该属性后，我们将在运行的应用程序中看到如下内容:

```java
0:0:0:0:0:0:0:1 - - [13/May/2019:23:14:51 +0530] "GET /greetings/Harry HTTP/1.1" 200 27
0:0:0:0:0:0:0:1 - - [13/May/2019:23:17:23 +0530] "GET /greetings/Harry HTTP/1.1" 200 27
```

这些是访问日志，格式如下:

```java
%h %l %u %t \"%r\" %>s %b
```

我们可以理解为:

`%h`–发送请求的客户端 IP，`0:0:0:0:0:0:0:1`在这种情况下是
`%l`–用户的身份
`%u`–由 HTTP 认证
`%t`确定的用户名–收到请求的时间
`%r`–来自客户端的请求行，`GET /greetings/Harry HTTP/1.1`在这种情况下是
`%>s`–从服务器发送到客户端的状态代码，如`200 `在这种情况下是
`%b`–对客户端的响应大小，或者是`27 `

因为这个请求没有经过验证的用户，`%l and %u` 打印了破折号。

事实上，如果**缺少任何信息，Tomcat 将为那个槽**打印一个破折号。

### 4.2.自定义访问日志

我们可以通过在`application.properties. `中添加一些属性来覆盖默认的 Spring Boot 配置

首先，要更改默认日志文件名:

```java
server.tomcat.accesslog.suffix=.log
server.tomcat.accesslog.prefix=access_log
server.tomcat.accesslog.file-date-format=.yyyy-MM-dd
```

此外，我们可以更改日志文件的位置:

```java
server.tomcat.basedir=tomcat
server.tomcat.accesslog.directory=logs
```

最后，我们可以覆盖日志在日志文件中的写入方式:

```java
server.tomcat.accesslog.pattern=common
```

在 Spring Boot 也有更多的[可配置属性](https://web.archive.org/web/20221023230809/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)。

## 5.内部日志

Tomcat 服务器的内部日志对于解决任何服务器端的问题都非常有帮助。

要查看这些日志，我们必须在`application.properties`中添加以下日志记录配置:

```java
logging.level.org.apache.tomcat=DEBUG
logging.level.org.apache.catalina=DEBUG
```

然后我们会看到这样的情况:

```java
2019-05-17 15:41:07.261 DEBUG 31160 --- [0124-Acceptor-0] o.apache.tomcat.util.threads.LimitLatch  : Counting up[http-nio-40124-Acceptor-0] latch=1
2019-05-17 15:41:07.262 DEBUG 31160 --- [0124-Acceptor-0] o.apache.tomcat.util.threads.LimitLatch  : Counting up[http-nio-40124-Acceptor-0] latch=2
2019-05-17 15:41:07.278 DEBUG 31160 --- [io-40124-exec-1] org.apache.tomcat.util.modeler.Registry  : Managed= Tomcat:type=RequestProcessor,worker="http-nio-40124",name=HttpRequest1
...
2019-05-17 15:41:07.279 DEBUG 31160 --- [io-40124-exec-1] m.m.MbeansDescriptorsIntrospectionSource : Introspected attribute virtualHost public java.lang.String org.apache.coyote.RequestInfo.getVirtualHost() null
...
2019-05-17 15:41:07.280 DEBUG 31160 --- [io-40124-exec-1] o.a.tomcat.util.modeler.BaseModelMBean   : preRegister [[email protected]](/web/20221023230809/https://www.baeldung.com/cdn-cgi/l/email-protection) Tomcat:type=RequestProcessor,worker="http-nio-40124",name=HttpRequest1
2019-05-17 15:41:07.292 DEBUG 31160 --- [io-40124-exec-1] org.apache.tomcat.util.http.Parameters   : Set query string encoding to UTF-8
2019-05-17 15:41:07.294 DEBUG 31160 --- [io-40124-exec-1] o.a.t.util.http.Rfc6265CookieProcessor   : Cookies: Parsing b[]: jenkins-timestamper-offset=-19800000
2019-05-17 15:41:07.296 DEBUG 31160 --- [io-40124-exec-1] o.a.c.authenticator.AuthenticatorBase    : Security checking request GET /greetings/Harry
2019-05-17 15:41:07.296 DEBUG 31160 --- [io-40124-exec-1] org.apache.catalina.realm.RealmBase      :   No applicable constraints defined
```

## 6.结论

在这篇简短的文章中，我们了解了 Tomcat 的内部日志和访问日志之间的区别。然后，我们看到了如何启用和定制它们。

请务必在 GitHub 上查看样本[。](https://web.archive.org/web/20221023230809/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-runtime)