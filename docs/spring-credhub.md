# Spring CredHub 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-credhub>

## 1.概观

在本教程中，我们将实现 [Spring CredHub](https://web.archive.org/web/20221207163043/https://spring.io/projects/spring-credhub) ，CredHub 的 Spring 抽象，用访问控制规则存储机密，这些规则将凭证资源映射到用户和操作。请注意，在运行代码之前，我们需要确保我们的应用程序运行在安装了 CredHub 的 [Cloud Foundry](/web/20221207163043/https://www.baeldung.com/cloud-foundry-uaa) 平台上。

## 2.Maven 依赖性

首先，我们需要安装 [`spring-credhub-starter`](https://web.archive.org/web/20221207163043/https://search.maven.org/search?q=a:spring-credhub-starter) 依赖项:

```java
<dependency>
    <groupId>org.springframework.credhub</groupId>
    <artifactId>spring-credhub-starter</artifactId>
    <version>2.2.0</version>
</dependency>
```

## 3.为什么凭据管理很重要？

**凭证管理是在凭证的整个生命周期中安全、集中地处理凭证的过程，主要包括生成、创建、轮换和撤销**。尽管每家公司的应用程序和信息技术环境各不相同，但有一点是一致的:证书，每个应用程序都需要它来访问其他应用程序、数据库或工具。

凭证管理对于数据安全至关重要，因为它允许用户和应用程序访问敏感信息。因此，无论是在运输途中还是在休息时，保持它们的安全都非常重要。最好的策略之一是用 API 调用 CredHub 之类的专用凭证管理工具以编程方式检索凭证，从而取代硬编码凭证及其手动管理。

## 4.CredHub APIs

### 4.1.证明

**认证 CredHub APIs 有两种方式: [UAA(OAuth2)](https://web.archive.org/web/20221207163043/https://github.com/cloudfoundry/uaa) 和[相互 TLS](https://web.archive.org/web/20221207163043/https://github.com/cloudfoundry-incubator/credhub/blob/master/docs/mutual-tls.md) 。**

默认情况下，CredHub 提供集成的相互 TLS 身份验证。要使用此方案，请在应用程序属性中指定 CredHub 服务器的 URL:

```java
spring:
  credhub:
    url: <CredHub URL>
```

认证 API 的另一种方法是通过 UAA，它需要客户端凭证授权令牌来获得访问令牌:

```java
spring:
  credhub:
    url: <CredHub URL>
    oauth2:
      registration-id: <credhub-client>
  security:
    oauth2:
      client:
        registration:
          credhub-client:
            provider: uaa
            client-id: <OAuth2 client ID>
            client-secret: <OAuth2 client secret>
            authorization-grant-type: <client_credentials>
        provider:
          uaa:
            token-uri: <UAA token server endpoint>
```

然后，可以将访问令牌传递给授权头。

### 4.2.凭据 API

**通过`CredHubCredentialOperations`接口，我们可以调用 CredHub APIs 来创建、更新、检索和删除凭证。【CredHub 支持的凭证类型有:**

*   `value –`单个配置的字符串
*   `json –`用于静态配置的 JSON 对象
*   `user –` 3 个字符串–用户名、密码和密码哈希
*   `password –`密码字符串和其他字符串凭证
*   `certificate –`包含根 CA、证书和私钥的对象
*   包含公钥和私钥的对象
*   一个包含 SSH 格式的公钥和私钥的对象

### 4.2.权限 API

权限是在编写凭证时提供的，用于控制用户可以访问、更新或检索的内容。Spring **CredHub 提供了`CredHubPermissionV2Operations `接口来创建、更新、检索和删除权限。**用户可以对凭证进行的操作有:`read`、`write`和`delete.`

## 5.CredHub 集成

我们现在将实现一个返回订单细节的 Spring Boot 应用程序，并将演示几个例子来解释凭证管理的完整生命周期。

### 5.1.`CredHubOperations `界面

**`CredHubOperations`接口位于`org.springframework.credhub.core`包中。它是 Spring 的 CredHub 的核心类，支持丰富的特性集来与 CredHub 交互。**它提供对接口的访问，这些接口对整个 CredHub APIs 和域对象与 CredHub 数据之间的映射进行建模:

```java
public class CredentialService {
    private final CredHubCredentialOperations credentialOperations;
    private final CredHubPermissionV2Operations permissionOperations;

    public CredentialService(CredHubOperations credHubOperations) {
        this.credentialOperations = credHubOperations.credentials();
        this.permissionOperations = credHubOperations.permissionsV2();
    }
}
```

### 5.2.凭据创建

让我们首先创建一个类型为`password,` 的新凭证，它是使用`PasswordCredentialRequest`构建的:

```java
SimpleCredentialName credentialName = new SimpleCredentialName(credential.getName());
PasswordCredential passwordCredential = new PasswordCredential((String) value.get("password"));
PasswordCredentialRequest request = PasswordCredentialRequest.builder()
  .name(credentialName)
  .value(passwordCredential)
  .build();
credentialOperations.write(request);
```

类似地，`CredentialRequest` 的其他实现可以用来构建不同的凭证类型，比如`ValueCredentialRequest`用于`value`凭证，`RsaCredentialRequest`用于`rsa`凭证，等等:

```java
ValueCredential valueCredential = new ValueCredential((String) value.get("value"));
request = ValueCredentialRequest.builder()
  .name(credentialName)
  .value(valueCredential)
  .build();
```

```java
RsaCredential rsaCredential = new RsaCredential((String) value.get("public_key"), (String) value.get("private_key"));
request = RsaCredentialRequest.builder()
  .name(credentialName)
  .value(rsaCredential)
  .build();
```

### 5.3.凭证生成

Spring CredHub 还提供了一个选项将生成 凭证到 避免了管理它们的工作量  放在应用 端。本、 中的 转、 增强了 数据 的安全性。let ' s see我们如何实现这个功能:

```java
SimpleCredentialName credentialName = new SimpleCredentialName("api_key");
PasswordParameters parameters = PasswordParameters.builder()
  .length(24)
  .excludeUpper(false)
  .excludeLower(false)
  .includeSpecial(true)
  .excludeNumber(false)
  .build();

CredentialDetails<PasswordCredential> generatedCred = credentialOperations.generate(PasswordParametersRequest.builder()
  .name(credentialName)
  .parameters(parameters)
  .build());

String password = generatedCred.getValue().getPassword();
```

### 5.4.凭据轮换和撤销

凭据管理的另一个重要阶段是轮换凭据。下面的代码演示了我们如何使用`password`凭证类型来实现这一点:

```java
CredentialDetails<PasswordCredential> newPassword = credentialOperations.regenerate(credentialName, PasswordCredential.class); 
```

CredHub 还让`certificate`凭证类型同时拥有多个活动版本，这样它们可以在不停机的情况下轮换。最终 和至关重要 相国书 管理 是

```java
credentialOperations.deleteByName(credentialName);
```

### 5.5.凭证检索

现在，我们已经了解了完整的凭据管理生命周期。我们将看到如何检索订单 API 认证的最新版本的`password`凭证:

```java
public ResponseEntity<Collection<Order>> getAllOrders() {
    try {
        String apiKey = credentialService.getPassword("api_key");
        return new ResponseEntity<>(getOrderList(apiKey), HttpStatus.OK);
    } catch (Exception e) {
        return new ResponseEntity<>(HttpStatus.FORBIDDEN);
    }
}

public String getPassword(String name) {
    SimpleCredentialName credentialName = new SimpleCredentialName(name);
    return credentialOperations.getByName(credentialName, PasswordCredential.class)
      .getValue()
      .getPassword();
}
```

### 5.6.控制对凭据的访问

**权限可以附加到凭证上以限制访问。**例如，将访问凭证的权限授予单个用户，并使该用户能够对凭证执行所有操作。可以附加另一个权限，允许所有用户只对一个凭证执行`READ`操作。

让我们看一个向凭证添加允许用户`u101`执行`READ`和`WRITE`操作的例子:

```java
Permission permission = Permission.builder()
  .app(UUID.randomUUID().toString())
  .operations(Operation.READ, Operation.WRITE)
  .user("u101")
  .build();
permissionOperations.addPermissions(name, permission);
```

除了`READ`和`WRITE,` 之外，Spring CredHub 还支持其他几个操作，允许用户:

*   `READ`–通过 id 和姓名获取凭证
*   `WRITE` –按名称设置、生成和重新生成凭证
*   `DELETE`–按名称删除凭证
*   `READ_ACL`–通过凭证名称获取 ACL(访问控制列表)
*   `WRITE_ACL`–添加和删除凭证 ACL 的条目

## 6.结论

在本教程中，我们展示了如何使用 Spring CredHub 库将 CredHub 与 Spring Boot 集成。我们集中了订单应用程序的凭证管理，涵盖了两个关键方面:凭证生命周期和权限。

本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221207163043/https://github.com/eugenp/tutorials/tree/master/spring-credhub)