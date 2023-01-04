# Spring Security 中的@CurrentSecurityContext 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-currentsecuritycontext>

## 1.概观

Spring Security 为我们处理认证凭证的接收和解析。

在这个简短的教程中，我们将看看如何在我们的处理程序代码中从请求中获取`SecurityContext`信息。

## 2.`@CurrentSecurityContext`注解

我们可以使用一些样板代码来读取安全上下文:

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
```

不过，**现在有了一个`@CurrentSecurityContext`注解来帮助我们**。

此外，使用注释使代码更具声明性，并使`authentication`对象可注入。通过`@CurrentSecurityContext`，我们还可以访问当前用户的`Principal`实现。

在下面的例子中，我们将研究几种获取安全上下文数据的方法，比如`Authentication`和`Principal`的名称。我们还将看到如何测试我们的代码。

## 3.Maven 依赖性

如果我们有一个 Spring Boot 的最新版本，那么我们只需要包含对`[spring-boot-starter-security](https://web.archive.org/web/20221125130800/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-security):`的依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

否则，我们可以将 [spring-security-core](https://web.archive.org/web/20221125130800/https://search.maven.org/search?q=g:org.springframework.security%20a:spring-security-core) 升级到 5.2.1 的最低版本。

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>5.2.1.RELEASE</version>
</dependency>
```

## 4.用`@CurrentSecurityContext`实现

我们可以用 SpEL (Spring Expression Language)用`@CurrentSecurityContext`到**注入`Authentication`对象或者`Principal`T6。SpEL 与类型查找一起工作。默认情况下，类型检查不是强制的，但是我们可以通过`@CurrentSecurityContext`注释的`errorOnInvalidType`参数来启用它。**

### 4.1.获取`Authentication`对象

让我们读取`Authentication`对象，这样我们就可以返回它的详细信息:

```java
@GetMapping("/authentication")
public Object getAuthentication(@CurrentSecurityContext(expression = "authentication") 
  Authentication authentication) {
    return authentication.getDetails();
} 
```

注意，SpEL 表达式指的是`authentication` 对象本身。

让我们来测试一下:

```java
@Test
public void givenOAuth2Context_whenAccessingAuthentication_ThenRespondTokenDetails() {
    ClientCredentialsResourceDetails resourceDetails = 
      getClientCredentialsResourceDetails("baeldung", singletonList("read"));
    OAuth2RestTemplate restTemplate = getOAuth2RestTemplate(resourceDetails);

    String authentication = executeGetRequest(restTemplate, "/authentication");

    Pattern pattern = Pattern.compile("\\{\"remoteAddress\":\".*"
      + "\",\"sessionId\":null,\"tokenValue\":\".*"
      + "\",\"tokenType\":\"Bearer\",\"decodedDetails\":null}");
    assertTrue("authentication", pattern.matcher(authentication).matches());
}
```

我们应该注意到，在这个例子中，我们得到了连接的所有细节。由于我们的测试代码无法预测`remoteAddress`或`tokenValue`，我们使用正则表达式来检查结果 JSON。

### 4.2.获取`Principal`

如果我们只想从我们的认证数据中得到`Principal`，我们可以改变 SpEL 表达式和注入的对象:

```java
@GetMapping("/principal")
public String getPrincipal(@CurrentSecurityContext(expression = "authentication.principal") 
  Principal principal) { 
    return principal.getName(); 
}
```

在这种情况下，我们使用`getName`方法只返回`Principal`名称。

让我们来测试一下:

```java
@Test
public void givenOAuth2Context_whenAccessingPrincipal_ThenRespondBaeldung() {
    ClientCredentialsResourceDetails resourceDetails = 
       getClientCredentialsResourceDetails("baeldung", singletonList("read"));
    OAuth2RestTemplate restTemplate = getOAuth2RestTemplate(resourceDetails);

    String principal = executeGetRequest(restTemplate, "/principal");

    assertEquals("baeldung", principal);
}
```

这里我们看到名字`baeldung`，它被添加到客户端凭证中，从注入到处理程序中的`Principal `对象中找到并返回。

## 5.结论

在本文中，我们看到了如何在当前安全上下文中访问属性，并将它们注入到处理程序方法的参数中。

我们已经通过利用 SpEL 和`@CurrentSecurityContext`注释做到了这一点。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221125130800/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-security)