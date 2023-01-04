# Spring Boot 认证审核支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-authentication-audit>

## 1。概述

在这篇短文中，我们将结合 Spring Security 探索 Spring Boot 执行器模块以及对发布认证和授权事件的支持。

## 2。Maven 依赖关系

首先，我们需要将`spring-boot-starter-actuator`添加到我们的`pom.xml:`中

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency> 
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220626113445/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-actuator%22) 资源库中获得。

## 3。监听认证和授权事件

要记录 Spring Boot 应用程序中的所有身份验证和授权尝试，我们可以用一个监听器方法定义一个 bean:

```java
@Component
public class LoginAttemptsLogger {

    @EventListener
    public void auditEventHappened(
      AuditApplicationEvent auditApplicationEvent) {

        AuditEvent auditEvent = auditApplicationEvent.getAuditEvent();
        System.out.println("Principal " + auditEvent.getPrincipal() 
          + " - " + auditEvent.getType());

        WebAuthenticationDetails details = 
          (WebAuthenticationDetails) auditEvent.getData().get("details");
        System.out.println("Remote IP address: " 
          + details.getRemoteAddress());
        System.out.println("  Session Id: " + details.getSessionId());
    }
} 
```

注意，我们只是输出一些在`AuditApplicationEvent`中可用的东西，以显示哪些信息是可用的。在实际应用程序中，您可能希望将这些信息存储在存储库或缓存中，以便进一步处理。

注意，任何春豆都可以；新的 Spring 事件支持的基础非常简单:

*   用`@EventListener`注释该方法
*   添加`AuditApplicationEvent`作为该方法的唯一参数

运行该应用程序的输出将如下所示:

```java
Principal anonymousUser - AUTHORIZATION_FAILURE
  Remote IP address: 0:0:0:0:0:0:0:1
  Session Id: null
Principal user - AUTHENTICATION_FAILURE
  Remote IP address: 0:0:0:0:0:0:0:1
  Session Id: BD41692232875A5A65C5E35E63D784F6
Principal user - AUTHENTICATION_SUCCESS
  Remote IP address: 0:0:0:0:0:0:0:1
  Session Id: BD41692232875A5A65C5E35E63D784F6 
```

在这个例子中，收听者已经接收到三个`AuditApplicationEvent`:

1.  在未登录的情况下，请求访问受限页面
2.  登录时使用了错误的密码
3.  第二次使用了正确的密码

## 4。认证审计监听器

如果 Spring Boot 的`AuthorizationAuditListener`暴露的信息不够，你可以**创建你自己的 bean 来暴露更多的信息。**

让我们看一个例子，其中我们还公开了授权失败时访问的请求 URL:

```java
@Component
public class ExposeAttemptedPathAuthorizationAuditListener 
  extends AbstractAuthorizationAuditListener {

    public static final String AUTHORIZATION_FAILURE 
      = "AUTHORIZATION_FAILURE";

    @Override
    public void onApplicationEvent(AbstractAuthorizationEvent event) {
        if (event instanceof AuthorizationFailureEvent) {
            onAuthorizationFailureEvent((AuthorizationFailureEvent) event);
        }
    }

    private void onAuthorizationFailureEvent(
      AuthorizationFailureEvent event) {
        Map<String, Object> data = new HashMap<>();
        data.put(
          "type", event.getAccessDeniedException().getClass().getName());
        data.put("message", event.getAccessDeniedException().getMessage());
        data.put(
          "requestUrl", ((FilterInvocation)event.getSource()).getRequestUrl() );

        if (event.getAuthentication().getDetails() != null) {
            data.put("details", 
              event.getAuthentication().getDetails());
        }
        publish(new AuditEvent(event.getAuthentication().getName(), 
          AUTHORIZATION_FAILURE, data));
    }
} 
```

我们现在可以在我们的侦听器中记录请求 URL:

```java
@Component
public class LoginAttemptsLogger {

    @EventListener
    public void auditEventHappened(
      AuditApplicationEvent auditApplicationEvent) {
        AuditEvent auditEvent = auditApplicationEvent.getAuditEvent();

        System.out.println("Principal " + auditEvent.getPrincipal() 
          + " - " + auditEvent.getType());

        WebAuthenticationDetails details
          = (WebAuthenticationDetails) auditEvent.getData().get("details");

        System.out.println("  Remote IP address: " 
          + details.getRemoteAddress());
        System.out.println("  Session Id: " + details.getSessionId());
        System.out.println("  Request URL: " 
          + auditEvent.getData().get("requestUrl"));
    }
} 
```

因此，输出现在包含请求的 URL:

```java
Principal anonymousUser - AUTHORIZATION_FAILURE
  Remote IP address: 0:0:0:0:0:0:0:1
  Session Id: null
  Request URL: /hello 
```

注意，在这个例子中我们从抽象的`AbstractAuthorizationAuditListener`扩展而来，所以我们可以在我们的实现中使用那个基类的`publish`方法。

如果您想测试它，请检查源代码并运行:

```java
mvn clean spring-boot:run 
```

此后，您可以将浏览器指向`http://localhost:8080/`。

## 5。存储审计事件

默认情况下，Spring Boot 将审计事件存储在一个`AuditEventRepository`中。如果您没有用自己的实现创建 bean，那么将为您连接一个`InMemoryAuditEventRepository`。

`InMemoryAuditEventRepository`是一种循环缓冲区，在内存中存储最近的 4000 个审计事件。然后可以通过管理端点`http://localhost:8080/auditevents`访问这些事件。

这将返回审计事件的 JSON 表示:

```java
{
  "events": [
    {
      "timestamp": "2017-03-09T19:21:59+0000",
      "principal": "anonymousUser",
      "type": "AUTHORIZATION_FAILURE",
      "data": {
        "requestUrl": "/auditevents",
        "details": {
          "remoteAddress": "0:0:0:0:0:0:0:1",
          "sessionId": null
        },
        "type": "org.springframework.security.access.AccessDeniedException",
        "message": "Access is denied"
      }
    },
    {
      "timestamp": "2017-03-09T19:22:00+0000",
      "principal": "anonymousUser",
      "type": "AUTHORIZATION_FAILURE",
      "data": {
        "requestUrl": "/favicon.ico",
        "details": {
          "remoteAddress": "0:0:0:0:0:0:0:1",
          "sessionId": "18FA15865F80760521BBB736D3036901"
        },
        "type": "org.springframework.security.access.AccessDeniedException",
        "message": "Access is denied"
      }
    },
    {
      "timestamp": "2017-03-09T19:22:03+0000",
      "principal": "user",
      "type": "AUTHENTICATION_SUCCESS",
      "data": {
        "details": {
          "remoteAddress": "0:0:0:0:0:0:0:1",
          "sessionId": "18FA15865F80760521BBB736D3036901"
        }
      }
    }
  ]
} 
```

## 6。结论

有了 Spring Boot 中的执行器支持，记录用户的身份验证和授权尝试变得很简单。读者还可以参考[生产就绪审计](https://web.archive.org/web/20220626113445/https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-auditing.html)了解更多信息。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626113445/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core)