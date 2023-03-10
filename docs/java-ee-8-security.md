# 雅加达 EE 8 安全 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ee-8-security>

## 1.概观

Jakarta EE 8 安全 API 是新的标准，是处理 Java 容器中安全问题的一种可移植方式。

在本文中，**我们将看看 API 的三个核心特性:**

1.  **HTTP 认证机制**
2.  **身份存储**
3.  **安全上下文**

我们将首先了解如何配置所提供的实现，然后了解如何实现一个定制的实现。

## 2.Maven 依赖性

要设置 Jakarta EE 8 安全 API，我们需要一个服务器提供的实现或者一个显式的实现。

### 2.1.使用服务器实现

Jakarta EE 8 兼容服务器已经提供了 Jakarta EE 8 安全 API 的实现，因此我们只需要[Jakarta EE Web Profile API](https://web.archive.org/web/20220710163703/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax%22%20AND%20a%3A%22javaee-web-api%22)Maven 工件:

```java
<dependencies>
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-web-api</artifactId>
        <version>8.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

### 2.2.使用显式实现

首先，我们为 Jakarta EE 8 [安全 API](https://web.archive.org/web/20220710163703/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22javax.security.enterprise-api%22) 指定 Maven 工件:

```java
<dependencies>
    <dependency>
        <groupId>javax.security.enterprise</groupId>
        <artifactId>javax.security.enterprise-api</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

然后，我们将添加一个实现，例如，[Soteria](https://web.archive.org/web/20220710163703/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22javax.security.enterprise%22)–参考实现:

```java
<dependencies>
    <dependency>
        <groupId>org.glassfish.soteria</groupId>
        <artifactId>javax.security.enterprise</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

## 3.HTTP 认证机制

在 Jakarta EE 8 之前，我们已经通过`web.xml`文件声明性地配置了认证机制。

在这个版本中，Jakarta EE 8 安全 API 设计了新的`HttpAuthenticationMechanism`接口作为替代。因此，web 应用程序现在可以通过提供该接口的实现来配置身份验证机制。

幸运的是，容器已经为 Servlet 规范定义的三种身份验证方法提供了实现:基本 HTTP 身份验证、基于表单的身份验证和定制的基于表单的身份验证。

它还提供了一个注释来触发每个实现:

1.  `@BasicAuthenticationMechanismDefinition`
2.  `@FormAuthenticationMechanismDefinition`
3.  `@CustomFormAuthenrticationMechanismDefinition`

### 3.1.基本 HTTP 身份验证

**如上所述，web 应用只需使用`@BasicAuthenticationMechanismDefinition annotation on a CDI bean` :** 就可以配置基本的 HTTP 认证

```java
@BasicAuthenticationMechanismDefinition(
  realmName = "userRealm")
@ApplicationScoped
public class AppConfig{}
```

此时，Servlet 容器搜索并实例化提供的`HttpAuthenticationMechanism`接口的实现。

在收到未授权的请求时，容器通过`WWW-Authenticate`响应头询问客户端是否提供合适的认证信息。

```java
WWW-Authenticate: Basic realm="userRealm"
```

然后，客户机通过`Authorization`请求头发送用户名和密码，用冒号“:”分隔，并用 Base64 编码:

```java
//user=baeldung, password=baeldung
Authorization: Basic YmFlbGR1bmc6YmFlbGR1bmc= 
```

请注意，提供凭证的对话框来自浏览器，而不是服务器。

### 3.2.基于表单的 HTTP 身份验证

**`@FormAuthenticationMechanismDefinition`注释触发了 Servlet 规范定义的基于表单的认证**。

然后，我们可以选择指定登录和错误页面，或者使用默认的合理页面`/login`和`/login-error`:

```java
@FormAuthenticationMechanismDefinition(
  loginToContinue = @LoginToContinue(
    loginPage = "/login.html",
    errorPage = "/login-error.html"))
@ApplicationScoped
public class AppConfig{}
```

作为调用`loginPage,`的结果，服务器应该将表单发送给客户机:

```java
<form action="j_security_check" method="post">
    <input name="j_username" type="text"/>
    <input name="j_password" type="password"/>
    <input type="submit">
</form>
```

然后，客户端应该将表单发送到由容器提供的预定义的后台身份验证过程。

### 3.3.基于自定义表单的 HTTP 身份验证

web 应用程序可以通过使用注释`@CustomFormAuthenticationMechanismDefinition:`来触发定制的基于表单的认证实现

```java
@CustomFormAuthenticationMechanismDefinition(
  loginToContinue = @LoginToContinue(loginPage = "/login.xhtml"))
@ApplicationScoped
public class AppConfig {
}
```

但是与默认的基于表单的身份验证不同，我们配置了一个定制的登录页面并调用了`SecurityContext.authenticate()`方法作为后台身份验证过程。

让我们看一下支持`LoginBean`，它包含登录逻辑:

```java
@Named
@RequestScoped
public class LoginBean {

    @Inject
    private SecurityContext securityContext;

    @NotNull private String username;

    @NotNull private String password;

    public void login() {
        Credential credential = new UsernamePasswordCredential(
          username, new Password(password));
        AuthenticationStatus status = securityContext
          .authenticate(
            getHttpRequestFromFacesContext(),
            getHttpResponseFromFacesContext(),
            withParams().credential(credential));
        // ...
    }

    // ...
}
```

作为调用定制`login.xhtml`页面的结果，客户端将收到的表单提交给`LoginBean'`的 `login()`方法:

```java
//...
<input type="submit" value="Login" jsf:action="#{loginBean.login}"/>
```

### 3.4.自定义身份验证机制

`HttpAuthenticationMechanism`接口定义了三种方法。**最重要的是我们必须提供实现的`validateRequest()`** 。

其他两个方法的默认行为，`secureResponse()`和`cleanSubject()**,**`在大多数情况下就足够了。

让我们来看一个示例实现:

```java
@ApplicationScoped
public class CustomAuthentication 
  implements HttpAuthenticationMechanism {

    @Override
    public AuthenticationStatus validateRequest(
      HttpServletRequest request,
      HttpServletResponse response, 
      HttpMessageContext httpMsgContext) 
      throws AuthenticationException {

        String username = request.getParameter("username");
        String password = response.getParameter("password");
        // mocking UserDetail, but in real life, we can obtain it from a database
        UserDetail userDetail = findByUserNameAndPassword(username, password);
        if (userDetail != null) {
            return httpMsgContext.notifyContainerAboutLogin(
              new CustomPrincipal(userDetail),
              new HashSet<>(userDetail.getRoles()));
        }
        return httpMsgContext.responseUnauthorized();
    }
    //...
}
```

在这里，实现提供了验证过程的业务逻辑，但在实践中，建议通过`IdentityStoreHandler` b `y`调用`validate`委托给`IdentityStore`。

我们还用`@ApplicationScoped`注释对实现进行了注释，因为我们需要使它支持 CDI。

在凭证的有效验证和用户角色的最终检索之后，**实现应该通知容器，然后**:

```java
HttpMessageContext.notifyContainerAboutLogin(Principal principal, Set groups)
```

### 3.5.实施 Servlet 安全性

**web 应用程序可以通过在 Servlet 实现上使用@ `ServletSecurity`注释来实施安全约束**:

```java
@WebServlet("/secured")
@ServletSecurity(
  value = @HttpConstraint(rolesAllowed = {"admin_role"}),
  httpMethodConstraints = {
    @HttpMethodConstraint(
      value = "GET", 
      rolesAllowed = {"user_role"}),
    @HttpMethodConstraint(     
      value = "POST", 
      rolesAllowed = {"admin_role"})
  })
public class SecuredServlet extends HttpServlet {
}
```

这个注释有两个属性——`httpMethodConstraints`和 `value`；`httpMethodConstraints`用于指定一个或多个约束，每个约束代表一个允许角色列表对 HTTP 方法的访问控制。

然后，对于每个`url-pattern`和 HTTP 方法，容器将检查连接的用户是否具有访问资源的合适角色。

## 4.身份存储

这个特性是由**的`IdentityStore`接口抽象出来的，它用于验证凭证并最终检索组成员资格。**换句话说，它可以提供认证和/或授权功能`.`

`IdentityStore`旨在并鼓励由`HttpAuthenticationMecanism`通过一个名为`IdentityStoreHandler`的接口使用。Servlet 容器提供了`IdentityStoreHandler`的默认实现。

应用程序可以提供其对`IdentityStore`的实现，或者使用容器为数据库和 LDAP 提供的两个内置实现之一。

### 4.1.内置身份存储

Jakarta EE 兼容服务器应该为两个身份存储提供实现:数据库和 LDAP。

**通过将配置数据传递给`@DataBaseIdentityStoreDefinition`注释:**来初始化数据库`IdentityStore`的实现

```java
@DatabaseIdentityStoreDefinition(
  dataSourceLookup = "java:comp/env/jdbc/securityDS",
  callerQuery = "select password from users where username = ?",
  groupsQuery = "select GROUPNAME from groups where username = ?",
  priority=30)
@ApplicationScoped
public class AppConfig {
}
```

作为配置数据，**我们需要一个到外部数据库**的 JNDI 数据源，两个用于检查呼叫者和他的组的 JDBC 语句，以及最后一个用于多个商店的优先级参数。

具有高优先级的`IdentityStore`由`IdentityStoreHandler.`稍后处理

与数据库一样， **LDAP IdentityStore 实现通过传递配置数据由`@LdapIdentityStoreDefinition`** 初始化:

```java
@LdapIdentityStoreDefinition(
  url = "ldap://localhost:10389",
  callerBaseDn = "ou=caller,dc=baeldung,dc=com",
  groupSearchBase = "ou=group,dc=baeldung,dc=com",
  groupSearchFilter = "(&(member=%s)(objectClass=groupOfNames))")
@ApplicationScoped
public class AppConfig {
}
```

这里我们需要外部 LDAP 服务器的 URL，如何在 LDAP 目录中搜索调用者，以及如何检索他的组。

### 4.2.实现自定义`IdentityStore`

**`IdentityStore`接口定义了四种默认方法:**

```java
default CredentialValidationResult validate(
  Credential credential)
default Set<String> getCallerGroups(
  CredentialValidationResult validationResult)
default int priority()
default Set<ValidationType> validationTypes()
```

`priority() `方法返回一个迭代顺序的值。这个实现由`IdentityStoreHandler.`处理，具有较低优先级的`IdentityStore`首先被处理。

默认情况下，`IdentityStore`处理凭证验证 `(ValidationType.VALIDATE)`和组检索(`ValidationType.PROVIDE_GROUPS`)。我们可以覆盖这种行为，使它只提供一种功能。

因此，我们可以将`IdentityStore`配置为仅用于凭证验证:

```java
@Override
public Set<ValidationType> validationTypes() {
    return EnumSet.of(ValidationType.VALIDATE);
}
```

在这种情况下，我们应该为`validate()`方法提供一个实现:

```java
@ApplicationScoped
public class InMemoryIdentityStore implements IdentityStore {
    // init from a file or harcoded
    private Map<String, UserDetails> users = new HashMap<>();

    @Override
    public int priority() {
        return 70;
    }

    @Override
    public Set<ValidationType> validationTypes() {
        return EnumSet.of(ValidationType.VALIDATE);
    }

    public CredentialValidationResult validate( 
      UsernamePasswordCredential credential) {

        UserDetails user = users.get(credential.getCaller());
        if (credential.compareTo(user.getLogin(), user.getPassword())) {
            return new CredentialValidationResult(user.getLogin());
        }
        return INVALID_RESULT;
    }
}
```

或者我们可以选择配置`IdentityStore`,使其仅用于组检索:

```java
@Override
public Set<ValidationType> validationTypes() {
    return EnumSet.of(ValidationType.PROVIDE_GROUPS);
}
```

然后我们应该为`getCallerGroups()`方法提供一个实现:

```java
@ApplicationScoped
public class InMemoryIdentityStore implements IdentityStore {
    // init from a file or harcoded
    private Map<String, UserDetails> users = new HashMap<>();

    @Override
    public int priority() {
        return 90;
    }

    @Override
    public Set<ValidationType> validationTypes() {
        return EnumSet.of(ValidationType.PROVIDE_GROUPS);
    }

    @Override
    public Set<String> getCallerGroups(CredentialValidationResult validationResult) {
        UserDetails user = users.get(
          validationResult.getCallerPrincipal().getName());
        return new HashSet<>(user.getRoles());
    }
}
```

因为`IdentityStoreHandler`期望实现是一个 CDI bean，所以我们用`ApplicationScoped`注释来修饰它。

## 5.安全上下文 API

Jakarta EE 8 安全 API 通过`SecurityContext`接口为**提供了一个编程安全性的访问点。当容器实施的声明性安全模型不够时，这是一种替代方法。**

`SecurityContext`接口的默认实现应该在运行时作为 CDI bean 提供，因此我们需要注入它:

```java
@Inject
SecurityContext securityContext;
```

此时，我们可以对用户进行身份验证，检索已通过身份验证的用户，检查其角色成员资格，并通过五种可用的方法授予或拒绝对 web 资源的访问。

### 5.1.检索呼叫者数据

在 Jakarta EE 的早期版本中，我们会检索`Principal`或者检查每个容器中不同的角色成员。

当我们在 servlet 容器中使用`HttpServletRequest`的`getUserPrincipal()`和` isUserInRole()`方法时，类似的`EJBContext`的`getCallerPrincipal() `和`isCallerInRole() methods `方法也在 EJB 容器中使用。

**新的 Jakarta EE 8 安全 API 已经通过标准化了这个*，通过`SecurityContext`接口***提供了一个类似的方法

```java
Principal getCallerPrincipal();
boolean isCallerInRole(String role);
<T extends Principal> Set<T> getPrincipalsByType(Class<T> type);
```

`getCallerPrincipal()`方法返回经过身份验证的调用者的容器特定表示，而`getPrincipalsByType()`方法检索给定类型的所有主体。

在特定于应用程序的调用者不同于容器调用者的情况下，这可能是有用的。

### 5.2.Web 资源访问测试

首先，我们需要配置一个受保护的资源:

```java
@WebServlet("/protectedServlet")
@ServletSecurity(@HttpConstraint(rolesAllowed = "USER_ROLE"))
public class ProtectedServlet extends HttpServlet {
    //...
}
```

然后，为了检查对这个受保护资源的访问，我们应该调用`hasAccessToWebResource() method:`

```java
securityContext.hasAccessToWebResource("/protectedServlet", "GET");
```

在这种情况下，如果用户处于角色`USER_ROLE.`中，该方法返回 true

### 5.3.以编程方式验证调用方

应用程序可以通过调用`authenticate()`以编程方式触发认证过程:

```java
AuthenticationStatus authenticate(
  HttpServletRequest request, 
  HttpServletResponse response,
  AuthenticationParameters parameters);
```

然后通知容器，容器将依次调用为应用程序配置的身份验证机制。`AuthenticationParameters`参数为`HttpAuthenticationMechanism:`提供凭证

```java
withParams().credential(credential)
```

`AuthenticationStatus`的`SUCCESS`和`SEND_FAILURE`值设计成功和失败的认证，而`SEND_CONTINUE`表示认证过程的进行中状态。

## 6.运行示例

为了突出这些例子，我们使用了最新开发的支持 Jakarta EE 8 的 Open Liberty 服务器。这是通过 [liberty-maven-plugin](https://web.archive.org/web/20220710163703/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22liberty-maven-plugin%22) 下载和安装的，它还可以部署应用程序和启动服务器。

要运行示例，只需访问相应的模块并调用以下命令:

```java
mvn clean package liberty:run
```

因此，Maven 将下载服务器，构建、部署和运行应用程序。

## 7.结论

在本文中，我们介绍了新的 Jakarta EE 8 安全 API 的主要特性的配置和实现。

首先，我们从展示如何配置默认的内置身份验证机制以及如何实现一个定制的机制开始。稍后，我们看到了如何配置内置身份存储以及如何实现自定义身份存储。最后，我们看到了如何调用`SecurityContext.`的方法

和往常一样，本文的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20220710163703/https://github.com/eugenp/tutorials/tree/master/java-ee-8-security-api)