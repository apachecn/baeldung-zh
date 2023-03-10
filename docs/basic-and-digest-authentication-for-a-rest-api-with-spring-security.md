# 具有 Spring 安全性的 REST 服务的基本和摘要认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/basic-and-digest-authentication-for-a-rest-api-with-spring-security>

**目录**

*   [**1。**概述](#overview)
*   [**2。**基本认证配置](#basic)
*   [**2.1。**满足无状态约束——摆脱会话](#basic)
*   [**3。**摘要认证配置](#digest)
*   [**4。**在同一个 RESTful 服务中支持两种认证协议](#both)
*   [**4.1。**匿名请求](#both)
*   [**4.2。**用认证凭证请求](#both)
*   [**5。**测试两种场景](#testing)
*   [**6。**结论](#conclusion)

## 1。概述

本文讨论如何在 REST API 的同一个 URI 结构上设置基本认证和摘要认证。在上一篇文章中，我们讨论了保护 REST 服务的另一种方法——[基于表单的认证](/web/20220706111623/https://www.baeldung.com/securing-a-restful-web-service-with-spring-security "Spring REST Service Security"),因此基本和摘要认证是自然的选择，也是更 RESTful 的选择。

## 2。基本认证的配置

基于表单的认证对于 RESTful 服务不理想的主要原因是 Spring Security 将**利用会话**——这当然是服务器上的状态，所以**REST**中的无状态约束实际上被忽略了。

我们将从设置基本身份验证开始——首先，我们从主要的`<http>`安全元素中删除旧的自定义入口点和过滤器:

```java
<http create-session="stateless">
   <intercept-url pattern="/api/admin/**" access="ROLE_ADMIN" />

   <http-basic />
</http>
```

请注意如何通过一个配置行—`<http-basic />`—处理`BasicAuthenticationFilter`和`BasicAuthenticationEntryPoint`的创建和连接，来添加对基本身份验证的支持。

### 2.1。满足无状态约束——消除会话

RESTful 架构风格的主要限制之一是客户端-服务器通信完全是无状态的，正如 T2 的原始论文 T3 所说:

> 5.1.3 无国籍状态
> 
> 接下来，我们为客户机-服务器交互添加一个约束:通信本质上必须是无状态的，就像 3.4.3 节中的客户机-无状态-服务器(CSS)风格一样(图 5-3)，这样，从客户机到服务器的每个请求必须包含理解请求所需的所有信息，并且不能利用服务器上任何存储的上下文。**会话状态因此完全保存在客户端**上。

服务器上的**会话**的概念在 Spring Security 中有很长的历史，直到现在完全移除它都很困难，尤其是当配置是通过使用名称空间完成的时候。

然而，Spring Security [用一个用于会话创建的**新`stateless`选项**增加了](https://web.archive.org/web/20220706111623/https://jira.springsource.org/browse/SEC-1424 "Add new option create-session="stateless"")名称空间配置，这有效地保证了 Spring 不会创建或使用任何会话。这个新选项的作用是从安全过滤器链中删除所有与会话相关的过滤器，确保为每个请求执行身份验证。

## 3。摘要认证的配置

从前面的配置开始，设置摘要式身份验证所需的过滤器和入口点将被定义为 beans。然后，**摘要入口点**将覆盖`<http-basic>`在幕后创建的入口点。最后，定制的**摘要过滤器**将被引入到安全过滤器链中，使用安全名称空间的`after`语义将其直接放置在基本认证过滤器之后。

```java
<http create-session="stateless" entry-point-ref="digestEntryPoint">
   <intercept-url pattern="/api/admin/**" access="ROLE_ADMIN" />

   <http-basic />
   <custom-filter ref="digestFilter" after="BASIC_AUTH_FILTER" />
</http>

<beans:bean id="digestFilter" class=
 "org.springframework.security.web.authentication.www.DigestAuthenticationFilter">
   <beans:property name="userDetailsService" ref="userService" />
   <beans:property name="authenticationEntryPoint" ref="digestEntryPoint" />
</beans:bean>

<beans:bean id="digestEntryPoint" class=
 "org.springframework.security.web.authentication.www.DigestAuthenticationEntryPoint">
   <beans:property name="realmName" value="Contacts Realm via Digest Authentication"/>
   <beans:property name="key" value="acegi" />
</beans:bean>

<authentication-manager>
   <authentication-provider>
      <user-service id="userService">
         <user name="eparaschiv" password="eparaschiv" authorities="ROLE_ADMIN" />
         <user name="user" password="user" authorities="ROLE_USER" />
      </user-service>
   </authentication-provider>
</authentication-manager>
```

不幸的是，在安全名称空间中没有[支持](https://web.archive.org/web/20220706111623/https://jira.springsource.org/browse/SEC-1860 "Add http-digest similar to http-basic to the security namespace")来自动配置摘要式认证，就像使用`<http-basic>`配置基本认证一样。因此，必须定义必要的 beans 并手动连接到安全配置中。

## 4。在同一个 Restful 服务中支持两种认证协议

在 Spring Security 中，仅基本或摘要式身份验证就可以轻松实现；它在相同的 URI 映射上为相同的 RESTful web 服务支持这两者，这给服务的配置和测试带来了新的复杂性。

### 4.1。匿名请求

使用安全链中的基本过滤器和摘要过滤器，Spring Security 处理**匿名请求**——不包含认证凭证(`Authorization` HTTP 头)的请求的方式是——两个认证过滤器将发现**没有凭证**，并将继续执行过滤器链。然后，看到请求没有被认证，一个`AccessDeniedException`被抛出并在`ExceptionTranslationFilter`中被捕获，这开始了摘要入口点，提示客户端输入凭证。

基本过滤器和摘要过滤器的职责都非常有限，如果它们无法识别请求中的身份验证凭据类型，它们将继续执行安全过滤器链。正因为如此，Spring Security 可以灵活地配置为在同一个 URI 上支持多个身份验证协议。

当请求包含正确的身份验证凭据(基本或摘要)时，将正确使用该协议。但是，对于匿名请求，只会提示客户端输入摘要式身份验证凭据。这是因为摘要入口点被配置为 Spring 安全链的主要和单一入口点；因此，**摘要认证可以被认为是默认的**。

### 4.2。具有认证凭证的请求

具有用于基本认证的凭证的**请求将由以前缀`“Basic”`开始的`Authorization`报头来标识。当处理这样的请求时，凭证将在基本身份验证过滤器中被解码，并且请求将被授权。类似地，带有摘要认证凭证的请求将使用前缀`“Digest”`作为其`Authorization`头。**

## 5。测试两种场景

测试将使用 REST 服务，方法是在使用 basic 或 digest 进行身份验证后创建一个新资源:

```java
@Test
public void givenAuthenticatedByBasicAuth_whenAResourceIsCreated_then201IsReceived(){
   // Given
   // When
   Response response = given()
    .auth().preemptive().basic( ADMIN_USERNAME, ADMIN_PASSWORD )
    .contentType( HttpConstants.MIME_JSON ).body( new Foo( randomAlphabetic( 6 ) ) )
    .post( paths.getFooURL() );

   // Then
   assertThat( response.getStatusCode(), is( 201 ) );
}
@Test
public void givenAuthenticatedByDigestAuth_whenAResourceIsCreated_then201IsReceived(){
   // Given
   // When
   Response response = given()
    .auth().digest( ADMIN_USERNAME, ADMIN_PASSWORD )
    .contentType( HttpConstants.MIME_JSON ).body( new Foo( randomAlphabetic( 6 ) ) )
    .post( paths.getFooURL() );

   // Then
   assertThat( response.getStatusCode(), is( 201 ) );
}
```

请注意，使用基本身份验证的测试会先发制人地向请求**添加凭证**，而不管服务器是否已经质询了身份验证。这是为了确保服务器不需要向客户端询问凭据，因为如果需要，询问的将是摘要凭据，因为这是默认设置。

## 6。结论

本文介绍了 RESTful 服务的基本身份验证和摘要式身份验证的配置和实现，主要使用了 Spring 安全名称空间支持以及框架中的一些新特性。